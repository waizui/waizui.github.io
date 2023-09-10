<head>
    <script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>
    <script type="text/x-mathjax-config">
        MathJax.Hub.Config({
            tex2jax: {
            skipTags: ['script', 'noscript', 'style', 'textarea', 'pre'],
            inlineMath: [['$','$']]
            }
        });
    </script>
</head>

# A glimpse of quaternion 
Under Constructing...


## Definition of Quaternion
First, the definition of quaternion is:

$$
	q=a+bi+cj+dk; \ \ \ \ \ \ \ \ \ \ (Equation 1)
	\\
	 \
	\\ where \ (a,b,c,d \in R)\ and \ i^2=j^2=k^2=ijk=-1
$$

It looks very simple, and very similar to the definition of **complex number**, which is

$$
	w = a+bi \ \ \ \ \ \ \ \ \ \ (Equation 2)
	\\
	 \
	\\ where \ (a,b, \in R)\ and \ i^2=-1
$$

In fact, there are some links between quaternions and complex numbers, and if you are interested
in the history of quaternions and the inventor of it, check this [wiki](https://en.wikipedia.org/wiki/William_Rowan_Hamilton).

Quaternion is useful and compact in terms of representing three-dimensional rotation. So it's widely
used in various places. 

In this article, only small fractions of quaternions will be discussed; Since the whole picture of quaternion
is huge and lot of mathematical knowledge are required to be able to understand it.

## Euler Transform
Let's take a look at Euler Transform at first. To represent rotations, Euler Transform 
is an intuitive choice. It's the multiplication of three matrices, each matrix is the rotation around it's
respective axis. 

### The head-pitch-roll transform

A commonly used view direction lies along the negative z-axis with the head oriented 
along the y-axis. shown in the picture.
![fig_hrp](./pic/hrp.png)

Rotations around y, x and z axis are called head, pitch, and roll.

More formally, the Euler Transform denoted **E**, is given by this equation:

$$
    E(h,p,r) = R_z(r)R_x(p)R_y(h) \ \ \ \ \ \ \ \ \ \ (Equation 3)
$$

where the h, p, r, are the Euler angles, representing in which order and how much the head, pitch, and
roll should rotate around their respective axes. 

### Why not using Euler Transform
Despite of being intuitive and simple, Euler Transform has a fetal defect.

Since the last column and row of a pure rotation 4x4 matrix are zero, we can use
a 3x3 matrix as the equivalent matrix, thus, **Equation3** yields:

$$
    E(h,p,r) = R_z(r)R_x(p)R_y(h) \\ 
    =
    \begin{bmatrix}
    cos\ r & -sin\ r & 0 \\
    sin\ r  & cos\ r  & 0 \\
    0          & 0          & 1 \\
    \end{bmatrix}

    \begin{bmatrix}
    1 & 0          & 0 \\
    0 & cos\ p  & -sin\ p \\
    0 & sin\ p  & cos\ p \\
    \end{bmatrix}


    \begin{bmatrix}
    cos\ h   & 0  & sin\ h  \\
    0           & 1  & 0          \\
    -sin\ h  & 0  & cos\ h  \\
    \end{bmatrix}
    \\
    = 
    \begin{bmatrix}
    cos\ r\ cos\ h -sin\ r\ sin\ p\ sin\ h & -sin\ r\ cos\ p  & cos\ r\ sin\ h +sin\ r\ sin\ p\ cos\ h \\
    sin\ r\ cos\ h +cos\ r\ sin\ p\ sin\ h &  cos\ r\ cos\ p  & sin\ r\ sin\ h -cos\ r\ sin\ p\ cos\ h \\
    -cos\ p\ sin\ h                        &  sin\ p          & cos\ p\ cos\ h                         \\
    \end{bmatrix}
    \\

    \ \ \ \ \ \ \ \ \ \ (Equation 4)
$$

Ok, here comes one of the most funny things in computer graphics. If we let p=π/2,
which means rotate around x axis 90 degrees and cos(p)=0, **Equation4** will become:

$$
    \begin{bmatrix}
    cos\ r\ cos\ h -sin\ r\ sin\ h & 0  & cos\ r\ sin\ h +sin\ r\  cos\ h \\
    sin\ r\ cos\ h +cos\ r\ sin\ h & 0  & sin\ r\ sin\ h -cos\ r\  cos\ h \\
    0                              & 1  & 0                               \\
    \end{bmatrix}
$$

In this case, no matter what value we give r and h, they can be interchangeable, which
means there are more than one combination of angles can yield same product, this is
called **gimbal lock**. Because of the numerical instability, The Euler Transforms is
not the ideal scheme of representing rotations.

## Properties of Quaternion




## Calculation of Quaternion


## A Implement