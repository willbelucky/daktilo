---
layout: post
title:  "Orthonormal & orthogonal & unit vector"
subtitle: "Definitions and Conditions for orthonormal & orthogonal & unit vector"
date:   2018-03-25 00:00:00
categories: [statistics]
---

# Orthogonal

## Definition
In mathematics, orthogonality is the generalization of the notion of perpendicularity to the linear algebra of bilinear forms. Two elements u and v of a vector space with bilinear form B are orthogonal when B(u, v) = 0. Depending on the bilinear form, the vector space may contain nonzero self-orthogonal vectors. In the case of function spaces, families of orthogonal functions are used to form a basis.
By extension, orthogonality is also used to refer to the separation of specific features of a system. The term also has specialized meanings in other fields including art and chemistry.

## Condition
A is a 3 by 3 matrix.\
$$
    A =
    \begin{matrix}
    a & b & c \\
    d & e & f \\
    g & h & i \\
    \end{matrix}
$$\
A is an orthogonal matrix if inner product values of two columns are 0.\
In other words, $$ ab + de + gh = 0 $$ and $$ bc + ef + hi = 0 $$ and $$ ca + fd + ig = 0 $$.


# Unit vector

## Definition
In mathematics, a unit vector in a normed vector space is a vector (often a spatial vector) of length 1.

## Condition
A is a unit vector if inner product values of a column are 1.\
In other words, $$ a^2 + d^2 + g^2 = 1 $$ and $$ b^2 + e^2 + h^2 = 1 $$ and $$ c^2 + f^2 + i^2 = 1 $$.


# Orthonormal

## Definition
In linear algebra, two vectors in an inner product space are orthonormal if they are orthogonal and unit vectors.

## Condition
A is an orthonormal matrix if A is an orthogonal matrix and a unit vector.