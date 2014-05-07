Getting Started
***************

This section provides an overview on what PyTrajectory is and how to use it.
For a more detailed view please have a look at the :ref:`reference`.

.. contents:: Contents
   :local:


What is PyTrajectory?
=====================

PyTrajectory is a Python library for the determination of the feed forward control 
to achieve a transition between desired states of a nonlinear control system.

Planning and designing of trajectories represents an important task in 
the control of technological processes. Here the problem is attributed 
on a multi-dimensional boundary value problem with free parameters.
In general this problem can not be solved analytically. It is therefore 
resorted to the method of collocation in order to obtain a numerical 
approximation.

PyTrajectory allows a flexible implementation of various tasks and enables an easy 
implementation. It suffices to supply a function :math:`f(x,u)` that represents the 
vectorfield of a control system and to specify the desired boundary values.


Installation
============

PyTrajectory has been developed and tested on Python 2.7

Dependencies
------------

Before you install PyTrajectory make sure you have the following 
dependencies installed on your system.

* numpy
* sympy
* scipy
* optional
   * matplotlib [visualisation]
   * ipython [debugging]

Source
------

To install PyTrajectory from the source files please download the latest release from
here (insert link!). After the download is complete open the archive and change directory
into the extracted folder. Then all you have to do is run the following command ::

   $ python setup.py install

PyPI
----

The easiest way of installing PyTrajectory would be ::

   $ pip install pytrajectory

or ::

   $ easy_install pytrajectory

provided that you have the Python modules `pip` or `setuptools` installed on your system.


Usage
=====

In order to illustrate the usage of PyTrajectory we consider the following simple example.


A pendulum mass :math:`m_p` is connected by a massless rod of length :math:`l` to a cart :math:`M_w`
on which a force :math:`F` acts to accelerate it.

.. image:: /../pic/inv_pendulum.png

A possible task would be the transfer between two angular positions of the pendulum. 
In this case, the pendulum should hang at first down (:math:`\varphi = \pi`) and is 
to be turned upwards (:math:`\varphi = 0`). At the end of the process, the car should be at 
the same position and both the pendulum and the cart should be at rest.
The (partial linearised) system is represented by the following differential equations,
where :math:`[x_1, x_2, x_3, x_4] = [x_w, \dot{x_w}, \varphi, \dot{\varphi}]` and 
:math:`u = \ddot{x}_w` is our control variable:

.. math::
   :nowrap:

   \begin{eqnarray*}
       \dot{x_1} & = & x_2 \\
       \dot{x_2} & = & u \\
       \dot{x_3} & = & x_4 \\
       \dot{x_4} & = & \frac{1}{l}(g\ sin(x_3) + u\ cos(x_3))
   \end{eqnarray*}

To solve this problem we first have to define a function that returns the vectorfield of
the system above. Therefor it is important that you use SymPy functions if necessary, which is
the case here with :math:`sin` and :math:`cos`.

So in Python this would be ::

   >>> from sympy import sin, cos
   >>>
   >>> def f(x,u):
   ...     x1, x2, x3, x4 = x  # system variables
   ...     u1, = u             # input variable
   ...     
   ...     l = 0.5     # length of the pendulum
   ...     g = 9.81    # gravitational acceleration
   ...     
   ...     # this is the vectorfield
   ...     ff = [          x2,
   ...                     u1,
   ...                     x4,
   ...         (1/l)*(g*sin(x3)+u1*cos(x3))]
   ...     
   ...     return ff
   ...
   >>> 

Wanted is now the course for :math:`u(t)`, which transforms the system with the following start 
and end states within :math:`T = 2 [s]`.

.. math::
   :nowrap:

   \begin{equation*}
      x(0) = \begin{bmatrix} 0 \\ 0 \\ \pi \\ 0 \end{bmatrix} 
      \rightarrow
      x(T) = \begin{bmatrix} 0 \\ 0 \\ 0 \\ 0 \end{bmatrix}
   \end{equation*}

so we have to specify the boundary values at the beginning ::

   >>> from numpy import pi
   >>> 
   >>> a = 0.0
   >>> xa = [0.0, 0.0, pi, 0.0]

and end ::

   >>> b = 2.0
   >>> xb = [0.0, 0.0, 0.0, 0.0]

The boundary values for the input variable are

   >>> uab = [0.0, 0.0]

because we want :math:`u(0) = u(T) = 0`.

Now we import all we need from PyTrajectory ::

   >>> from pytrajectory import Trajectory

and pass our parameters. ::

   >>> T = Trajectory(f, a, b, xa, xb, uab)

All we have to do now, to solve our problem is ::

   >>> T.startIteration()

After the 7th iteration step with 320 spline parts, this would yield a solution which satisfies 
the default tolerance for the boundary values of :math:`10^{-02}`. But PyTrajectory enables 
you to improve its performance by altering some of its method parameters.

For example if we increase the factor for raising the spline parts (default: 2) ::

   >>> T.setParam('kx', 5)

and don't take advantage of the system structure (integrator chains) ::

   >>> T.setParam('use_chains', False)

we get a solution after 3 steps with 125 spline parts.

There are more method parameters you can change to speed things up, i.e. the type of 
collocation points to use or the number of spline parts for the input variables. 
To do so, just type::

   >>> T.setParam('<param>', <value>)

Please have a look at the :ref:`reference` for more information.

.. only:: html

   .. image:: /../pic/inv_pendulum_osc.gif

.. only:: latex

  .. image:: /../pic/inv_pendulum_osc.png
