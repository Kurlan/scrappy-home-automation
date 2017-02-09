---
layout: default

---

We are engineers that love to hack on stuff.  While there are a bunch of polished products you can buy to achieve your home automation goals, in our opinion, many are overpriced and inflexible.  This site documents our journey to automate our homes our way.  Our goal is that you can follow along and achieve home automation independence as well.
 
 <ul>
   {% for post in site.posts reversed %}
     <li>
       <a href="{{ post.url | prepend: site.baseurl }}">{{ post.title }}</a>
     </li>
   {% endfor %}
 </ul>

{% include analytics.html %}
{% include disqus.html %}
