---
title: Adding Link Preview to your Hexo Blog
excerpt: How to add Link Preview support to your Hexo-based blog.
date: 2023-06-03 15:09:05
tags:
---


As discussed [here](https://nickmck.net/about/), I use Hexo for this Blog. When sharing to LinkedIn or Twitter I noticed that my links were lacking the title image for the blog post. For example:

<img src="lp1.png" />

The Link Preview image is controlled via the [OG Meta Tag](https://ogp.me/). The question is how to add these tags via Hexo. 

There are two steps. 

# Front Matter
Each Hexo post contains a [Front Matter] (https://hexo.io/docs/front-matter.html) section that contains the post's metadata such as Title, Tags etc. Any data added to this section is parsed into the Page object for use during rendering. All that's needed then is to add the data we want to use in the OG Meta tags. Here Ive added a new tag 'ogimage' to hold the data.

<img src="lp3.png"/>

# Template update

The last thing to do is update the EJS template used to render the page headers. This is found under 'Themes/<Theme Name>/layout/_partial'. Simply add some code to render the meta tags if the value is provided in the post. For example:

{% codeblock lang:javascript %}
    <% if (page.ogimage){ %>
    <meta property="og:image" content="<%= page.ogimage %>"/>
    <meta property="og:description" content="<%= page.excerpt %>"/>
    <% } %>
{% endcodeblock %}

# All Done
Now when you generate your site the OG metadata is set if you provided the data in your Front Matter. It should look something like this:

<img src="lp2.png" width="300px"/>

Happy blogging...