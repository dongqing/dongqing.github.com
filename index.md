---
layout: page
title: 
tagline: Supporting tagline
---
{% include JB/setup %}







 {% for post in site.posts %}
 <div class="posts">
 	<div><b style="color:red">{{ post.date | date_to_string }} </b></div>
 	<br/>
 	<div> {{ post.content }} </div>
 </div>
 -------------------------------------
 {% endfor %}