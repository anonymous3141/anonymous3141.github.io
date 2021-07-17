Hey there, thanks for visiting my blog :) I'm Junhua, an Australian computer science and maths enthusiast who just graduated high school (as of December 2020). Thus far, my main exposure to CS and maths is through olympiads. I represented Australia in the IOI (international olympiad in informatics) and also did well in the Australia Maths Olympiad (AMO). However, I'm really excited to explore a wider range of CS and Maths tools and their applications now that there are no more olympiads to do. You will find my explorations, as well as good olympiad competitive programming ideas on this blog (and maybe some other stuff too UwU). Anyway, good to meet you (>w<) 

## Posts
Below are my posts. 
<div id="html" markdown="0">

{% for post in site.posts %}

    <a href="{{ post.url | prepend: site.baseurl }}">
          <h5>{{ post.title }}</h5>
     </a>
     <p> <b>Date:</b> {{ post.date | date: "%-d %B %Y" }} <br> </p>
	<p> <b> About: </b> {{ post.excerpt }} </p>
{% endfor %}

</div>

## Non Maths and CS Stuff

I take great interest in history and have occasionally wrote some stuff on it.
- 30/1/2021 [The Edo period of Japan]({% link static/edo-japan.pdf %}) (includes insider jokes)

## Things I've made

- A website and quiz system for my old High School's math club which I used to run [here](https://github.com/Maths-Club). The website is hosted by heroku [here](https://cgsmathclub.herokuapp.com/) It uses the Django framework as well as various other APIs. I really need to revamp this blog to be like the github pages static component I made for that one!
- A technical report on applying machine learning to build a recommender system for the ORAC informatics training site. The report is found [here]({% link static/recommender-report.pdf %}), and describes the process from scraping data to benchmarking accuracy. Code is included. In retrospect I could probably have eliminated the "September 2020" cap but meh.
