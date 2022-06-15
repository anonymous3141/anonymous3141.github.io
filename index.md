Hey there! I'm Junhua. As of June 2021 I have just completed the first year of my math degree at Trinity College Cambridge. Despite being a math student, my background as an informatics olympiad student in High School (I represented Australia in the IOI) means that I'm also really interested in computer science. You can find some technical and non-technical articles and discussions I wrote below.

## Posts
Below are my posts. Click on the title to go to the post.
<div id="html" markdown="0">

{% for post in site.posts %}

    <a href="{{ post.url | prepend: site.baseurl }}">
	    <h5>{{ post.title }}</h5> 
     </a>
	<p><i> {{ post.date | date: "%-d %B %Y" }} </i><br> </p>
	<p> {{ post.excerpt }} </p>
{% endfor %}

</div>

## Non Maths and CS Stuff

I take great interest in history and have occasionally wrote some stuff on it. 
- 30/1/2021 [The Edo period of Japan]({% link static/edo-japan.pdf %}) (includes insider jokes & probably factual errors e.g the relation between buddhism and Shinto)

## AIOC

I have had the privilege to lecture at the Australian Informatics School of Excellence and Team Selection Schools to share some of the tricks I devised or found out. Below are my lecture notes.

**December 2020:** 

[Beta Data structures: Divide and conquer algorithms]({% link static/info-lectures/DandC.pdf %})

**April 2021:**

[Selection School: Approaches to Tree problems]({% link static/info-lectures/Trees.pdf %})

**December 2021:**

[Beta Dynamic programming: Assorted nonstandard techniques]({% link static/info-lectures/DPIBeta21Dec.pdf %})

[Beta Data Structures: 3 ways to use a segment tree]({% link static/info-lectures/DSIIBeta21DecCleaned.pdf %})

## Things I've made

- A website and quiz system for my old High School's math club which I used to run [here](https://github.com/Maths-Club). The website is hosted by heroku [here](https://cgsmathclub.herokuapp.com/) It uses the Django framework as well as various other APIs. I really need to revamp this blog to be like the github pages static component I made for that one!


## Technical Reports
- A technical report on applying machine learning to build a recommender system for the ORAC informatics training site. The report is found [here]({% link static/recommender-report.pdf %}), and describes the process from scraping data to benchmarking accuracy. Code is included. In retrospect I could probably have eliminated the "September 2020" cap and tried undersampling the non-solves but meh.
- A technical report describing a NLP semantic classification system that I made for a hackathon [here]({% link static/Technical report_final.pdf %}). I think it really shows model complexity is often less important than the data quality.

## Analytics

<div id="counter" markdown="0">
<p align="center"> <img src="https://komarev.com/ghpvc/?username=anonymous3141&label=Profile%20views&color=ce9927&style=flat" alt="anonymous3141" /> </p> 
<script async src="//static.getclicky.com/101348632.js"></script>
<noscript><p><img alt="Clicky" width="1" height="1" src="//in.getclicky.com/101348632ns.gif" /></p></noscript>
	<script src="//widgets.clicky.com/tally/?site_id=101348632&sitekey=eaeb7af83e9066d35d6790f7defc3f7a&width=175&height=250&title=&hide_title=0&hide_branding=0" type="text/javascript"></script>
	<br><br>
	<a title="Real Time Web Analytics" href="http://clicky.com/101348632"><img alt="Clicky" src="//static.getclicky.com/media/links/badge.gif" border="0" /></a>
</div>

