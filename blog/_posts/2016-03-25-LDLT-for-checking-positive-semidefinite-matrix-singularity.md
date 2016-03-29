---
layout: post
type: mathpost
title: LDLT for Checking Positive Semidefinite Matrix's Singularity
date: 2016-03-25
modified: 2016-03-29
comments: true
tags: math note code
---

* TOC
{:toc}

I was recently caught up with a seemingly classic matrix decomposition problem as follows.

# Solving Ax=b

To solve a system of equations
$ Ax=b $,
which is usually overdetermined but sometimes can be underdetermined,
one could usually equivalently solve the corresponding normal equation
$ Bx=c $,
where $B=A^\intercal A$ and $c=A^\intercal b$.
Now we are sure that matrix $B$ is
[positive semidefinite, i.e., P.S.D.](https://en.wikipedia.org/wiki/Positive-definite_matrix#Positive-semidefinite).
But we are not sure if $B$ is singular or not, since its singularity totally depends on the singularity of $A$.

# When A is non-singular
we know that the system can be efficiently solved by perform
[Cholyskey Decomposition](https://en.wikipedia.org/wiki/Cholesky_decomposition)
as
$B=LL^\intercal$.
In the widely-used
[Eigen library](http://eigen.tuxfamily.org/dox-devel/group__TopicLinearAlgebraDecompositions.html),
this can be done as simple as:

```cpp
Eigen::MatrixXd x = B.llt().solve(c);
```

or a more stable version as the
[LDLT decomposition](https://en.wikipedia.org/wiki/Cholesky_decomposition#LDL_decomposition)[^1]:

[^1]:
    As mentioned in [the note of Eigen library's documentation](http://eigen.tuxfamily.org/dox-devel/group__TopicLinearAlgebraDecompositions.html#note1),
there are two LDLT decomposition variants.
    And the Matlab command "ldl" uses the [Block variant](https://en.wikipedia.org/wiki/Cholesky_decomposition#Block_variant)
    which is different from Eigen's LDLT.

```cpp
Eigen::MatrixXd x = B.ldlt().solve(c);
```

# When A is singular
$B$ becomes a singular P.S.D.
Now the question is, can we perform Cholyskey Decomposition on a singular P.S.D.?
The answer is **yes**!
As explained [in this book (equation 2.78 to 2.85)](http://www.value-at-risk.net/cholesky-factorization/),
if we always set the indeterminate variable to 0, we can still get a valid decomposition result.
However as explained in section 2.7.4 of that book:

> The Cholesky algorithm is unstable for singular positive semidefinite matrices h.  
> It is also unstable for positive definite matrices h that have one or more eigenvalues close to 0.

So the previous answer should be augmented as "yes, but the decomposition is **NOT numerically stable**".
But wait a second, who cares about whether the decomposition is stable or not?
As long as it can be reliably determined that matrix $B$ is singular,
we can simply return that this system is underdetermined and thus fail to find a unique solution.
However we really want to avoid using SVD or QR decomposition to check the rank of $B$
(for speed consideration, if we need to repeatedly solve a large number of such kind of equations).

Since we are going to solve the system using LDLT if it is non-singular, now the question becomes:

# Can we use LDLT to check whether a P.S.D. matrix is (numerically) singular or not?
If we can, then we can directly perform LDLT on $B$ and then examine the singularity
(and return failure if it is indeed singular).
**My answer is yes** and my thoughts are explained as follows.
We know there always exists a LDLT decomposition for a given P.S.D. matrix $B$ as $B=LDL^\intercal$.
Considering the definition of $L$ and $D$, we know that $det(B)=det(D)=\prod d_i$,
where $D=diag(d_1, d_2, ..., d_n)$,
we can then conclude that:

1. if $B$ is **non-singular**, then **none** of the diagnal elements $d_i$ **is zero**,  
since otherwise the determinant of $B$ will become zero.
2. and similarly, if $B$ is **singular**, then there is **at least one** diagnal element $d_i \approx 0$ numerically,  
since otherwise the determinant of $B$ will become non-zero.

Thus in the Eigen library all we need to do is to check:

```cpp
bool is_B_singular = B.ldlt().vectorD().cwiseAbs().minCoeff() < 1e-8; // if min(abs(d_i))>eps
```

Or a more numerically stable check:

{: #cpp_ldlt_check_singularity}
```cpp
Eigen::VectorXd D = B.ldlt().vectorD().cwiseAbs();
bool is_B_singular = D.maxCoeff() < (D.minCoeff() * 1e8); // similar to check matrix condition number in some sense
```

# However Eigen library seems to say no
In its [documentation of LDLT](http://eigen.tuxfamily.org/dox-devel/classEigen_1_1LDLT.html),
it says that:

> Remember that Cholesky decompositions are not rank-revealing.  
> Also, do not use a Cholesky decomposition to determine whether a system of equations has a solution.

OK, I agree that Cholesky decompositions are not rank-revealing
(I cannot find a textbook stating this, but I think it relates to numerical stability of the decomposition.
So if a matrix is singular, we may not be able to simply count the number of non-zero elements as the rank of the matrix).
But why can't I use it to determine whether the previous $Bx=c$ has a unique solution or not
by simply looking at the singularity or $B$?
Or maybe the documentation only means that I cannot use the test of $rank([B,c]) == rank(B)$ as mentioned in
[this Matlab post](http://blogs.mathworks.com/loren/2009/12/11/how-to-check-for-existence-of-solution-to-matrix-equations/)
to test if the system still has solution even if $B$ is singular?
(i.e., "$B$ is singular" is **not equivalent to** "$Bx=c$ has no solution")
If the later is what the documentation means, then Eigen's domentation is really *not saying no to our approach*.
**This is the part that I'm not sure yet and need to ask Eigen's develop team for confirmation**,
althought according to my own experiments with Eigen it seems to work pretty well using [the above code](#cpp_ldlt_check_singularity).