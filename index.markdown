---
title: ls /etc | more
layout: skel
---
<div id="doc" class="home_page">
<div id="bd">

  <ul id="navbar">
    <li><a href="/">/</a></li>
    <li><a href="/articles.html">/articles</a></li>
    <li><a href="/code.html">/code</a></li>
  </ul><br style="clear:both">


  <div id="logo_container">
    <img src="/img/logo_home.png" />
  </div>

  <div style="margin-left:80px"> <!-- div-outer -->

    <ul class="posts">
      <li><span class="dirlist">$ ls /etc | more</span></li>
      {% for post in site.posts limit:5 %}
      <li><span class="dirlist">-rwxr-x-r-x root root</span><span class="dirlist count">{{ post.content | number_of_words }}&nbsp;</span><span class="dirlist"> {{ post.date | date_to_string }} <a href="{{ post.url }}">{{ post.title }}</a></span></li>
      {% endfor %}
      <li><span class="dirlist"><a href="/articles.html">----more----</a></span></li>
    </ul>

  </div>	<!-- /div-outer -->

<div id="ft"></div>
</div>
