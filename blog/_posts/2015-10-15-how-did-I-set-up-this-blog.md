---
layout: post
type: mathpost
title: How did I set up this blog
date: 2015-10-15
modified: 2016-03-24
comments: true
tags: jekyll blog
---

After basic setup of the blog according to GitHub and Jekyll, some other features can be added as follows:

1. [comments](http://www.perfectlyrandom.org/2014/06/29/adding-disqus-to-your-jekyll-powered-github-pages/)  

2. [tags](http://christianspecht.de/2014/10/25/separate-pages-per-tag-category-with-jekyll-without-plugins/)  

3. [mathjax](http://haixing-hu.github.io/programming/2013/09/20/how-to-use-mathjax-in-jekyll-generated-github-pages/) and [here](https://gist.github.com/mikelove/cbf6eb431406852ba725) (note: it seems that `markdown: redcarpet` also works)  
like this  
   $$
   a^2 + b^2 = c^2,
   $$
this $sin(x^2)$, or this numbered equation $(\ref{eq:theeq})$ with reference   
\begin{equation}\label{eq:theeq}
\mathbf{X}\_{n,p} = \mathbf{A}\_{n,k} \mathbf{B}\_{k,p}
\end{equation}  

4. [code highlighting](http://tuxette.nathalievilla.org/?p=1574) (note: pygmentize.exe locates under *Python\Scripts*, also check [this incompatibility problem between pygments and MathJax](https://github.com/mathjax/mathjax-docs/wiki/MathJax-CSS-classes-etc#mathjax-vs-pygments))  
like this
   
   ```python
   s = "Python syntax highlighting"
   print s
   ```   
   
5. <del> [redcarpet markdown configuration](http://sholsinger.com/2014/03/jekyll-github-flavored-markdown) and [here](https://george-hawkins.github.io/basic-gfm-jekyll/redcarpet-extensions.html) (note: *"superscript"* in the redcarpet extension seems to be incompatible with mathjax, so I removed that from _config.yml) </del>  
GitHub doesn't support **redcarpet** now, switching to **kramdown**, following [this](http://idratherbewriting.com/2016/02/21/bug-with-kramdown-and-rouge-with-github-pages/) and [this](http://mazhuang.org/2016/02/04/switch-to-kramdown-from-redcarpet/).

And some other notes:

1. DO NOT trust jekyll's --watch function: it seems to ignore changes in _config.yml, USE `jekyll build` each time you changed this file or you suspect something is not updated when using --watch!  

2. jekyll's --watch seems to ignore _config.yml's redcarpet extension, this maybe a bug that I should report on GitHub...  

3. [MathJax/Latex help](http://meta.math.stackexchange.com/questions/5020/mathjax-basic-tutorial-and-quick-reference)  

4. [Another nice Chinese blog explaining the setup process](http://leequangang.github.io/tech/2013/06/09/github_jekyll_mathjax.html)  