---
layout: post
type: mathpost
title: How did I set up this blog
date: 2015-10-15
comments: true
tags: jekyll blog
---

After basic setup of the blog according to GitHub and Jekyll, some other features can be added as follows:

1. [comments](http://www.perfectlyrandom.org/2014/06/29/adding-disqus-to-your-jekyll-powered-github-pages/)
2. [tags](http://christianspecht.de/2014/10/25/separate-pages-per-tag-category-with-jekyll-without-plugins/)
3. [mathjax](http://haixing-hu.github.io/programming/2013/09/20/how-to-use-mathjax-in-jekyll-generated-github-pages/) (note: it seems that `markdown: redcarpet` also works)  
like this  
$$
a^2 + b^2 = c^2,
$$
this $sin(x^2)$, or this  
\begin{equation}
\mathbf{X}\_{n,p} = \mathbf{A}\_{n,k} \mathbf{B}\_{k,p}
\end{equation}
4. [code highlighting](http://tuxette.nathalievilla.org/?p=1574) (note: pygmentize.exe locates under *Python\Scripts*, also check [this incompatibility problem between pygments and MathJax](https://github.com/mathjax/mathjax-docs/wiki/MathJax-CSS-classes-etc#mathjax-vs-pygments))     
like this
   
```python
s = "Python syntax highlighting"
print s
```
