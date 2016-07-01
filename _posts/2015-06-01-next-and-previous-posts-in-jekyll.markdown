---
layout: post
title:  "Add a next / previous post footer in Jekyll"
description: "Showing a preview of your next and previous posts is easy"
date:   2015-06-01 14:19:31
categories: jekyll
image: next-previous
---
A footer showing the user the next and previous posts on a blog or other series of content seems like a pretty common requirement, and it's one that many blog themes come with out of the box.

When trying to do this in Jekyll, searching the documentation took me to the page about pagination and explained how to add next/previous links to a paginator, but that wasn't what I was after.

I stumbled across what I was looking for on the [Variables page](http://jekyllrb.com/docs/variables/) where I discovered each page has the variables `page.previous` and `page.next`.

It might not be immediately obvious to newcomers that these variables are packed with quite a bit of information – they don't just house the post name.

As a result, using them to construct a previous/next footer is really simple. In your post layout template, you can just do something like:

{% highlight liquid %}             
<ul class="posts posts--previous-next">
  {{ "{% if page.previous " }}%} 
    <li>
      <h4 class="pagination-title"><a href="{{ "{{ page.previous.url " }}}}">{{ "{{ page.previous.title " }}}}</a></h4>
      <h5 class="pagination-date">{{ "{{ page.previous.date " }}}}</h5>
      <p class="pagination-description">{{ "{{ page.previous.description " }}}}</p>
    </li>
  {{ "{% endif " }}%}
  {{ "{% if page.next " }}%} 
    <li>
      <h4 class="pagination-title"><a href="{{ "{{ page.next.url " }}}}">{{ "{{ page.next.title " }}}}</a></h4>
      <h5 class="pagination-date">{{ "{{ page.next.date " }}}}</h5>
      <p class="pagination-description">{{ "{{ page.next.description " }}}}</p>
    </li>
  {{ "{% endif " }}%}
</ul>
{% endhighlight %}

### The end result
See below this post for the real, live implementation!