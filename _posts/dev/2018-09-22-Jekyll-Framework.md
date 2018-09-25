---
layout: dev
categories: [dev]
---

# Jekyll Framework

> This is my first Jekyll blog post with this theme -- [cayman blog](https://github.com/lorepirri/cayman-blog). 

Before starting, if you want to preview localy your modifications as you build your blog, simply run these commands and connect via your browser at `https://localhost:4000`

```bash
script bootstrap
bundle install
script/server
```

> The first thing I did was removing that ugly banner from `./_layouts/default.html`. 
>
> To do that, simply remove all the `section` that contains the `class="page-header"` and replace it by this or w/e you want :-)

```html
{% raw %}
<div style="padding-top: 10%; text-align: center">
  <h1 class="project-name">{{ page-title }}</h1>
  <h2 class="project-tagline">{{ page-tagline }}</h2>
</div>
{% endraw %}
```
Then, I restructured this template by adding/removing these files and folders:

```
+ ./offSec.md
+ ./dev.md
+ ./_posts/{dev, home, offsec}
+ ./_layouts/{dev.html, offsec.html}
+ ./assets/imgaes
```

You'll understand why these changes were useful in a second. The 2 pages that I created in the root directory will appear in the navbar on top. Let's examine one of the 2 and pay attention to the `layout` wich point to the corresponding one in the `_layout` folder (i.e.: `dev.html`).

```md
---
layout: dev
title: Dev
tagline: Design your own desires
---

# Dev

(...)
```

> Great but not enough! :) To simplify my life, I want to keep my posts in differrent files without modifying any other pages. To do that, 

Jekyll uses the [Liquid](https://jekyllrb.com/docs/liquid/) templating language. Generally in Liquid you output content of [variables](https://jekyllrb.com/docs/variables/) using two curly braces and perform logic statements by surrounding them in a curly brace percentage sign.


```html
---
layout: default
---
{% raw %}
<div>

  {{ content }}

  <h2>Latest Posts</h2>

  <div>&nbsp;</div>
  <ul class="post-list">
    {% for post in site.categories.dev %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
    </li>
    {% endfor %}
  </ul>

</div>      
{% endraw %}
```

Now all I'll have to do is add the corresponding `category` in the `front matter` of each posts, and Jekyll will recongize it. In this example, Jekyll will automatically return every posts that are in the  `dev` category. 

> Great, so let's create one! First, make sure that you place the file in a child directory of `./_posts`, and make sur it has the `date` as prefix and `.md` as suffix. 

For example: `2018-09-22-My-Super-First-Dev-Post.md`.

```html
---
layout: dev 
categories: [dev]
---

# Welcome

**Hello world**, this is my first Jekyll blog post.

I hope you like it!
```
