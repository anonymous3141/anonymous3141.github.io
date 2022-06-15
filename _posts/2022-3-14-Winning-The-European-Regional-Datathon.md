---
excerpt: How me and a few friends did a datathon and won it

---

{% include head.html %}
# Winning Correlation One & Citadel's 2022 Spring European Datathon

This is a brief account to how my team won the Citadel 2022 Spring European Datathon.

### End Product

The goal of the dataset was to write an open-ended report on a supplied data set analysing its characteristics. Teams of 4 compete to make the best report they can in a week. You can see our report [here](https://anonymous3141.github.io/static/Team_6_Report.pdf)
### Team Formation

A few weeks ago, I saw this event via a Cambridge notice bulletin, an Datathon jointly hosted by Correlation One and Citadel (a big quant fund). Having done some data science in the past year but never too comfortable with it, I decided to participate to help alleviate my fear of pandas dataframes. 

With some effort, I had scratched together a team of fellow Cambridge first year friends to do the contest with me. There was tunan, my Australian compatriot and a computer science student, Sherman, fellow trinity maths student and data science veteran who had won the last iteration of the datathon, and Wilson, an maths student from Emmanuel College. We passed the entrance test to the datathon despite failing some questions that first years have no hope of doing, and thus successfully formed our team. 

As first years, we didn't expect to be able to outshine the 15 other teams, containing 2nd and 3rd (maybe even Masters) students from the top universities in the UK, so we only aimed to do our best and have some fun.

### Competition Schedule
The task was announced on Sunday the 6th March, concerning the Supply Chain of an international Company in the form of a list of orders and customers. The data was released the morning of the 7th, and our team of 4 would have a full week to write an open-ended report (there were some suggested prompts but we could do anything we thought was interesting) on the data, and submit before 10pm the following Sunday (13th), and the results would be announced on Pi Day the 14th.


### How we went about it

Everyday for the week, the team would meet at Trinity to work for some hours on the project. Initially it was just about brainstorming avenues of exploration, and using exploratory data analysis (plotting lots of stuff, eyeballing series) to see if our questions were promising to explore. 

The data exhibited surprising uniformity and it was initially hard to find any interesting behaviour within the dataset. However, as the week went on, nontrivial trends and relations emerged.

By the middle of the week we had to start choosing directions to dig into, and to start constructing models as well as running hypothesis tests to give the necessary depth and rigour to our research. 

By Saturday, we had to start wrapping up our analysis and start writing the report, as well as collating and arranging data for presentation. The report was extremely long to write up, and took the entirety of Sunday to write and to convert into latex format. When we submitted with 20 minutes to spare (about 9:40pm, our group had collaboratively worked since the morning to write and format the report),  our report spanned 9 pages and 5000 words.

### The Teamwork

We were extremely proud of the product we made when we submitted. This was partially due to the division of labour being especially effective, as every person was able to demonstrate their technical strengths (for instance, Tunan's speed-latexing skills and Sherman's knowledge of statistics).  

I focused on exploratory data analysis and the machine learning model fitting, was able to learn a lot from the others, with my abilities in Latex, Pandas and Matplotlib now much improved.

### A technical Comment: Scrutinise your model!

One of the points that was noted to have distinguished our report was our careful analysis of models. When we first started running our machine learning algorithms, our baseline Naive Bayes Model got an 95% test accuracy. While usually this would be cause to whip out the champagne, we fortunately were more suspicious and asked ourselves 'just how does Naive Bayes do this well?'. After all, we were expecting accuracies in the 70s, tops, and didn't believe there was nearly enough signal in the data, let alone signals that can be picked apart by so simple a model to get 95%.

Turns out, after some detailed study with permutation importance and other metrics, that our model was 'memorising' the time column of the data, and due to the random assignment of data to train and test sets (as opposed to time period seperation) was resulting in the absurd test accuracy. A good analogy would be predicting the gender of a child when his identical twin is in the training set. You can also read section VIII of our report to learn more about this.

Tunan has written an excellent account of the datathon too, especially concerning the examination of our models, which you can read [here](https://boyandhisteahouse.com/posts/datathon/) (well worth it!). 

### Announcing the winners and conclusion

The winners were announced just after 7pm on Monday the 14th. While we had not expected to win, we were very hopeful that we'd get at least a podium place. 

Thus we were very surprised, but nevertheless very pleased to hear that we had in fact won, against the aformentioned heavy competition, but above all was just happy to have learnt so much from it; my data science skills have improved markedly.

We are grateful for Citadel and Correlation One's superb organisation of the datathon; it was a valuable learning opportunity to our team, and winning was just a pleasant addition to a great experience.
