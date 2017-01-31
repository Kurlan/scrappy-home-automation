---
# You don't need to edit this file, it's empty on purpose.
# Edit theme's home layout instead if you wanna make some changes
# See: https://jekyllrb.com/docs/themes/#overriding-theme-defaults
layout: default
---
We are engineers that love to hack on stuff.  While there are a bunch of polished products you can buy to acheive your home automation goals, in our opinion, many are overpriced and inflexible.  This site documents our journey to automate our homes *our way*.  Our goal is that you can follow along and achieve home automation independence as well.

<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url | prepend: site.baseurl }}">{{ post.title }}</a>
    </li>
  {% endfor %}
</ul>
