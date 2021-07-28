---
excerpt: On Tree graphs

---

{% include head.html %}
# Transfer Learning with Pytorch

Due to high computational requirements for deep learning, **transfer learning** is often used for tasks like image classification. This is especially true for fine grained classification (distinguishing subclasses of a particular group). 

I have decided to experiment with transfer learning. Here is a basic description of what I'm doing:

- I use AlexNet as a basis, taking output from its second last layer (so all but the second last layer) and attach more layers onto it.
- I will be testing on a ship dataset (I have always been into ships!)  from [kaggle](https://www.kaggle.com/arpitjain007/game-of-deep-learning-ship-datasets)
- I am running on a laptop CPU (wanna here a joke? my compute power!)
- FYI I'm using Jupyter notebook.

### Imports:
```
import torch
import torch.nn as nn
from torchvision.models import alexnet
    
from typing import Any
import cv2
import matplotlib.pyplot as plt
import os

print(os.getcwd())
if "Ship classification" not in os.getcwd():
    os.chdir('Ship classification')

model = alexnet(pretrained = True, progress = True)
print("Done")
```

Heres what AlexNet looks like in pytorch:
```
AlexNet(
  (features): Sequential(
    (0): Conv2d(3, 64, kernel_size=(11, 11), stride=(4, 4), padding=(2, 2))
    (1): ReLU(inplace=True)
    (2): MaxPool2d(kernel_size=3, stride=2, padding=0, dilation=1, ceil_mode=False)
    (3): Conv2d(64, 192, kernel_size=(5, 5), stride=(1, 1), padding=(2, 2))
    (4): ReLU(inplace=True)
    (5): MaxPool2d(kernel_size=3, stride=2, padding=0, dilation=1, ceil_mode=False)
    (6): Conv2d(192, 384, kernel_size=(3, 3), stride=(1, 1), padding=(1, 1))
    (7): ReLU(inplace=True)
    (8): Conv2d(384, 256, kernel_size=(3, 3), stride=(1, 1), padding=(1, 1))
    (9): ReLU(inplace=True)
    (10): Conv2d(256, 256, kernel_size=(3, 3), stride=(1, 1), padding=(1, 1))
    (11): ReLU(inplace=True)
    (12): MaxPool2d(kernel_size=3, stride=2, padding=0, dilation=1, ceil_mode=False)
  )
  (avgpool): AdaptiveAvgPool2d(output_size=(6, 6))
  (classifier): Sequential(
    (0): Dropout(p=0.5, inplace=False)
    (1): Linear(in_features=9216, out_features=4096, bias=True)
    (2): ReLU(inplace=True)
    (3): Dropout(p=0.5, inplace=False)
    (4): Linear(in_features=4096, out_features=4096, bias=True)
    (5): ReLU(inplace=True)
    (6): Linear(in_features=4096, out_features=1000, bias=True)
  )
)
```
### Dataset description

The dataset requires classification of ship images (both black and white and coloured) into 5 categories. There are 6250 images, which I split into 1750 for test and 4500 for train (the lack of cross validation is as I can't be stuffed tuning hyperparams). The dataset processing code, which generates the torch datasets:

```
import pandas as pd
import cv2 # for img reading
import numpy as np
from tqdm import tqdm # for progress bar
from torch.utils.data import Dataset, DataLoader
import torchvision.transforms as transforms

IMG_SIZE = 224 

class Hook():
    # as in webhook & NN hook
    def __init__(self, module, backward=False):
        if backward==False:
            self.hook = module.register_forward_hook(self.hook_fn)
        else:
            self.hook = module.register_backward_hook(self.hook_fn)
    def hook_fn(self, module, input, output):
        self.input = input
        self.output = output
    def close(self):
        self.hook.remove()
        
class CustomImageDataset(Dataset):
    
    def __init__(self, res_dir, img_dir, train_mode, regression_mode=False): # no annotations / transforms
        # 6253 images
        # split 4500 train and 1753 test
        self.dataset = []
        self.output = []
        self.regression = []
        self.SPLIT = 4500
        self.BLACK_AND_WHITE = False
        self.dat = np.array(pd.read_csv(res_dir))
        
        for i in tqdm(range(len(self.dat))):
            filename = self.dat[i][0]
            label = self.dat[i][1]
            if i >= self.SPLIT and train_mode == True:
                continue
            elif i < self.SPLIT and train_mode == False:
                continue
                
            path = os.path.join(img_dir, filename)
            #print(path)
            if self.BLACK_AND_WHITE:
                # each image must be a 3d tensor (1 by N by N) cuz thats how pytorch covnet works
                # so turn 2d into 3d
                self.dataset.append(torch.Tensor(np.array([cv2.resize(cv2.imread(path, cv2.IMREAD_GRAYSCALE),
                                          (IMG_SIZE, IMG_SIZE))])))
            else:
                # color
                # if rgb then its 3 by N by N
                self.dataset.append(torch.transpose(torch.Tensor(cv2.resize(cv2.imread(path),
                                          (IMG_SIZE, IMG_SIZE))),0,2))
            # normalise
            
            self.output.append(label-1) #zero index
            norm = transforms.Normalize(mean=[0.485, 0.456, 0.406], std=[0.229, 0.224, 0.225])
            self.dataset[-1] = norm(self.dataset[-1])
            
        if regression_mode:
            net = alexnet(pretrained=True).eval()
            with torch.no_grad():
                outHook = Hook(net.classifier[-3]) #output of linear layer, before relu
                for img in tqdm(self.dataset):
                    net(img.unsqueeze(0))
                    self.regression.append(outHook.output)

    def __len__(self):
        return len(self.dataset)
    
    def __getitem__(self, id):
        if len(self.regression) == 0:
            return self.dataset[id], self.output[id]
        else:
            return self.regression[id], self.output[id]
```

*Initialisation:*
```
TRAIN_DIR = os.path.join(os.getcwd(), 'images')
NORMAL_TRAIN = False
REG_TRAIN = False

if NORMAL_TRAIN:
    training_set = CustomImageDataset('train.csv', TRAIN_DIR, True)
    testing_set = CustomImageDataset('train.csv', TRAIN_DIR, False)

    train_loader = DataLoader(training_set, batch_size=64, shuffle=True)
    test_loader = DataLoader(testing_set, batch_size=64, shuffle=True)
if REG_TRAIN:
    regtraining_set = CustomImageDataset('train.csv', TRAIN_DIR, True, True)
    regtesting_set = CustomImageDataset('train.csv', TRAIN_DIR, False, True)
    regtrain_loader = DataLoader(regtraining_set, batch_size=64, shuffle=True)
    regtest_loader = DataLoader(regtesting_set, batch_size=64, shuffle=True)
```

**Noteworthy points:**
- The images are resized to 224 by 224, then have its pixels normalised.
- To speed things up the second last layer for each image is precomputed with the aid of pytorch hooks (which basically fire off a function everytime a module is evaluated)
- It is crucial in pytorch to set AlexNet to evaluation mode. Otherwise the dropout layers start playing up!

### Training
Below are my train and testing loops:

```
epochs = 20 # it seems 20 epochs then starts to overfit
lr = 0.00005
model2 = ShipNetStupid().to(device) #yolo
loss = nn.CrossEntropyLoss()
optim = torch.optim.AdamW(model2.parameters(), lr = lr)
device = 'cuda' if torch.cuda.is_available() else 'cpu' 
def trainloop2(dataloader, model, loss_fn, optimizer):
    
    for batch, (X,y) in enumerate(dataloader):
        X=X.to(device)
        y=y.to(device)
        pred = model(X).squeeze(1)
        #print(pred.shape)
        loss = loss_fn(pred, y)
        
        optimizer.zero_grad()
        loss.backward()
        optimizer.step()
        
        if batch % 10 == 0:
            loss, current = loss.item(), (batch+1)*len(X)
            print(f"Processed {current} examples, loss={loss}")

def testloop2(dataloader, model, loss_fn):
    size = len(dataloader.dataset)
    num_batches = len(dataloader)
    test_loss, correct = 0,0
    with torch.no_grad():
        for X,y in dataloader:
            X=X.to(device)
            y=y.to(device)
            pred = model(X).squeeze(1)
            test_loss += loss_fn(pred,y).item()
            correct += (pred.argmax(1) == y).type(torch.float).sum().item()
    print(f"Testset: Correct: {correct}/{size}. Loss={test_loss}")

print(model2) #display params
epochs = 100
for i in range(epochs):
    print(f"Epoch {i}")
    trainloop2(regtrain_loader, model2, loss, optim)
    testloop2(regtest_loader, model2, loss)
```

The execution loopwould look something like:

```
lr = 0.00005
loss = nn.CrossEntropyLoss()
optim = torch.optim.AdamW(model.parameters(), lr = lr)


print(model) #display params
for i in range(epochs):
    print(f"Epoch {i}")
    trainloop(train_loader, model, loss, optim)
    testloop(test_loader, model, loss)
```
### Baseline model:
A logistic regression is used as a classifier. It is trained for 300 iterations on AdamW optimiser with learning rate 0.0005. It obtains an accuracy of 925/1752 for an accuracy of 52.8%. Respectable, given the minimal amount of work required.
```
class ShipNetStupid(nn.Module):
    
    def __init__(self):
        # See source of alexnet: 
        # https://pytorch.org/vision/stable/_modules/torchvision/models/alexnet.html#alexnet
        super(ShipNetStupid, self).__init__()

        self.classifier = nn.Sequential(
            nn.Linear(in_features=4096, out_features=5),
            nn.Softmax()
                        )
    
    def forward(self, x):
        x = self.classifier(x)
        return x
```

### Not so baseline model
Here's the model implemented without precomputation of alexnet's second last layer. I show this because the code is very cute UwU.

```
class ShipNet(nn.Module):
    
    def __init__(self):
        # See source of alexnet: 
        # https://pytorch.org/vision/stable/_modules/torchvision/models/alexnet.html#alexnet
        super(ShipNet, self).__init__()
        heresOneIMadeEarlier = alexnet(pretrained = True).requires_grad_(requires_grad=False)
        self.features = heresOneIMadeEarlier.features
        self.avgpool = heresOneIMadeEarlier.avgpool
        old_classifier = heresOneIMadeEarlier.classifier
        self.classifier = nn.Sequential(*heresOneIMadeEarlier.classifier[:4],
                                       nn.Linear(in_features=4096, out_features=150),
                                       nn.ReLU(),
                                       nn.Linear(in_features=150, out_features=5),
                                       nn.Softmax()
                        )
    
    def forward(self, x):
        x = self.features(x)
        x = self.avgpool(x)
        x = nn.Flatten()(x)
        x = self.classifier(x)
        return x 
  ```

However, as we have done ourselves a great favour by precomputing, our neural net becomes:
```
class ShipNetSimple(nn.Module):
    
    def __init__(self):
        # See source of alexnet: 
        # https://pytorch.org/vision/stable/_modules/torchvision/models/alexnet.html#alexnet
        super(ShipNetSimple, self).__init__()

        self.classifier = nn.Sequential(
            nn.Linear(in_features=4096, out_features=512),
            nn.ReLU(),
            nn.Linear(in_features=512, out_features=5),
            nn.Softmax()
                        )
    
    def forward(self, x):
        x = self.classifier(x)
        return x
```
This 4096 -> 512 -> 5 architecture was able to get an accuracy of 803/1752 after training with the same learning rate for 60 iterations (as no further improvement was being observed). We suspect overfitting is at play so we try a bunch of things:


- Dropout layer with zero-setting probability 0.5/0.6/0.8 at various points
- Making the middle layer smaller (512-256-128)
- Changing weight decay and learning rate, including using a scheduler and running for more iterations.

Unfortunately none of these things managed to do much better than the baseline. We saw an accuracy of 930/1752 but that is it.


### Conclusion
Maybe I'm doing something wrong hmmm idk. I could try augmenting data or something but I suspect the black and white nature of a lot of images would be an issue.
Oh well...


### Edit:
Oh wait I am actually dumb lol I forgot to read images via PIL (pillow) so Im totally reading in the range [0,255] rather than [0,1] which is actually what normalisation assumes lololololol. I have now redone the experiment with proper normalisation, and the improvements where marked. Still though, logistic regression did not differ overly from a 3 layer neural network :

- Logistic Regression scored 1462/1752
- A 3 layer neural network with layer sizes 4096-128-5 scored 1472/1752
- An SVM from scikit-learn without much tuning (just the regularization) was also tried and scored 1455/1752

Thus, all 3 approaches have accuracies around 83%.

### Source code:
You can find the pdf of the jupyter-notebook [here]({% link static transferlearn_notebook.pdf %})
