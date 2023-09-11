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
	\hat{q}=a+bi+cj+dk; \qquad (Equation 1)
	\\
	 \
	\\ where \ (a,b,c,d \in R)\ and \ i^2=j^2=k^2=ijk=-1
$$

It looks very simple, and very similar to the definition of **complex number**, which is

$$
	w = a+bi \qquad (Equation 2)
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
along the y-axis. shown in the figure.
![fig_hrp](./pic/hrp.png)

Rotations around y, x and z axis are called head, pitch, and roll.

More formally, the Euler Transform denoted **E**, is given by this equation:

$$
    E(h,p,r) = R_z(r)R_x(p)R_y(h) \qquad (Equation 3)
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

    (Equation 4)
$$

Ok, here comes one of the most funny things in computer graphics. If we let p=π/2,
which means rotating around x axis 90 degrees and cos(p)=0, **Equation4** will become:

$$
    \begin{bmatrix}
    cos\ r\ cos\ h -sin\ r\ sin\ h & 0  & cos\ r\ sin\ h +sin\ r\  cos\ h \\
    sin\ r\ cos\ h +cos\ r\ sin\ h & 0  & sin\ r\ sin\ h -cos\ r\  cos\ h \\
    0                              & 1  & 0                               \\
    \end{bmatrix}
$$

In this case, no matter what value we give r and h, they can only make it rotate around y aixs, which means one degree of freedom has lost, this is called **gimbal lock**. Because of the numerical instability, The Euler Transforms is
not the ideal scheme of representing rotations.

## Properties of Quaternion
Let's get back to the **Equation1**, 

$$
    \begin{align*} 
    \because i\times j\times k &= -1  \\  
    \therefore (i\times j\times k)\times k &= -1\times k \\ 
    (i\times j)\times(k\times k) &= -k \\ 
    \because k^2 &= -1  \\  
    \therefore (i\times j)\times(-1) &= -k \\ 
    -(i\times j) &= -k \\ 
    i\times j &= k 
    \end{align*}
$$

because the same principle, the relation of ijk is:

$$
    ij=k\ \   ki=j \  \  jk=i  \qquad (Equation5)
$$

which can be represented in the following figure.

![ijk](./pic/ijk_pic.png)

A quaternion can also be written in following way:

$$
    \hat{q} = (r,\vec{v}) \ ; \qquad r \in R \ \ and \ \ \vec{v} = bi+cj+dk \qquad (Equation6)
$$

where r is the **real part** of the v is the **imaginary part**.

### Multiplication
Note that the multiplication of quaternions is **not commutative**.

Giving quaternion q1 and q2, the multiplication of them is:

$$
    \begin{align*}
    &{\hat{q}}_1 {\hat{q}}_2 = (a_1,\vec{v_1})(a_2,\vec{v_2}) \\
    &=(a_1+ib_1+jc_1+kd_1)(a_2+ib_2+jc_2+kd_2)  \\
    &=(a_1a_2-(a_1a_2+b_1b_2+c_1c_2+d_1d_2)) \\
    &+ i(a_1b_2+a_2b_1+c_1d_2-d_1c_2)\\
    &+ j(a_1c_2+a_2c_1+b_1d_2-d_1b_2)\\
    &+ k(a_1d_2+a_2d_1+b_1c_2-c_1b_2)\\
    \end{align*}
$$

Since 

$$
    \vec{v_1} \times \vec{v_2} =
    \begin{vmatrix}
    i & j & k \\
    b_1 & c_1 & d_1 \\
    b_2 & c_2 & d_2 \\
    \end{vmatrix}
    = 
    i(c_1d_2-d_1c_2)
    +j(b_1d_2-d_1b_2)
    +k(b_1c_2-c_1b_2)
$$

the product of two quaternions' multiplication can also be written as

$$

    {\hat{q}}_1 {\hat{q}}_2 = 
    (a_1a_2-\vec{v_1}\cdot\vec{v_2}\ , a_1\vec{v_2}+a_2\vec{v_1}+\vec{v_1} \times \vec{v_2})
    \qquad (Equation7)
$$

## Conjugate

$$
   \hat{q}^*  = (r,\vec{v})^*  
$$

## Addition

$$

$$


## Quaternion and rotation


## A Implement
