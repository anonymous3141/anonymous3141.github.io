<div id="counter" markdown="0">
	<p align="center"> <img src="https://komarev.com/ghpvc/?username=anonymous3141&label=Profile%20views&color=ce9927&style=flat" alt="anonymous3141" /> </p>
</div>
Hey there, thanks for visiting my blog :) I'm Junhua, an Australian computer science and maths enthusiast who just graduated high school (as of December 2020). Thus far, my main exposure to CS and maths is through olympiads. I represented Australia in the IOI (international olympiad in informatics) and also did well in the Australia Maths Olympiad (AMO). However, I'm really excited to explore a wider range of CS and Maths tools and their applications now that there are no more olympiads to do. You will find my explorations, as well as good olympiad competitive programming ideas on this blog (and maybe some other stuff too UwU). Anyway, good to meet you (>w<) 

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

I have had the cool opportunity to lecture at the Australian Informatics School of Excellence and Team Selection Schools to share some of the tricks I devised or found out about while doing Informatics. Below are my lecture notes.

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
