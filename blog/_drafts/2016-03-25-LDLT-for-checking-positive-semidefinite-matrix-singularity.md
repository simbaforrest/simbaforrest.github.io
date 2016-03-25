---
layout: post
type: mathpost
title: LDLT for Checking Positive Semidefinite Matrix's Singularity
date: 2016-03-25
modified: 2016-03-25
tags: math note code
---

I was recently caught up with a seemingly classic matrix decomposition problem as follows.

To solve a system of equations
$ Ax=b $,
which is usually overdetermined but sometimes can be underdetermined,
one could usually equivalently solve the corresponding normal equation
$Bx= c$,
where $B=A\intercal A$ and $c=A\intercal b$.
Now we are sure that matrix $B$ is
[positive semidefinite or P.S.D.](https://en.wikipedia.org/wiki/Positive-definite_matrix#Positive-semidefinite).
But we are not sure if $B$ is singular or not, since its singularity totally depends on the singularity of $A$.
When $A$ is non-singular, we know that the system can be efficiently solved by perform
[Cholyskey Decomposition](https://en.wikipedia.org/wiki/Cholesky_decomposition)
as
$B=LL\intercal$.
In the widely-used
[Eigen library](http://eigen.tuxfamily.org/dox-devel/group__TopicLinearAlgebraDecompositions.html),
this can be done as simple as

```cpp
Eigen::MatrixXd x = B.ldlt().solve(c);
```

But when $A$ is singular, $B$ is a singular P.S.D.
