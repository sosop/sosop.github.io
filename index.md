---
layout: page
title: 一个程序员的平凡之路
tagline: Supporting tagline
---
{% include JB/setup %}

Blog记录生活的色彩、技术的感悟。  
  
> 我终于相信，每一条走上来的路，都有它不得不那样跋涉的理由。每一条要走下去的路，都有它不得不那样选择的方向。——席慕容  
   
> ![一条自己选择的路](assets/pic/road.jpg "坚持")
 
> 我可以接受失败，但绝对不能接受未曾奋斗过的自己。  

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>
