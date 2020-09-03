
===============  
C-DTLZ
===============
The baseline test problems of C-DTLZ are from the classic DTLZ benchmark suite. There are three types of C-DTLZ.
 
Type-1 Constrained Problems
-------------------------------
The original Pareto front(PF) is kept the same, but there is an infeasible barrier that causes difficulties for an algorithm in converging toward the PF. 

C1-DTLZ1
~~~~~~~~~~

the form of C1-DTLZ1
#####################
  .. math:: 
    \mbox{min} f_1(x)&= \frac{1}{2}x_1x_2\dots x_{m-1}(1+g(x_m))\\
    \mbox{min} f_2(x)&= \frac{1}{2}x_1x_2\dots (1-x_{m-1})(1+g(x_m))\\
    \dots\\ 
    \mbox{min} f_{m-1}(x)&= \frac{1}{2}x_1(1-x_2)(1+g(x_m))\\
    \mbox{min} f_m(x)&= \frac{1}{2}(1-x_1)(1+g(x_m))

where

  .. math::
    g(x_m)=100[|x_m|+\sum_{x_i\in x_m}(x_i - 0.5)^2 - cos(20\pi(x_i -0.5))]   


:math:`x_m=(x_m, \dots, x_n)^T`. The constraints is defined as:

  .. math::
    c(x)=1-\frac{f_m(x)}{0.6}-\sum_{i=1}^{m-1}\frac{f_i(x)}{0.5}\geq 0

Code
######


.. code-block:: c
   
    void c1dtlz1 (individual_real *ind){
        int i, j, k, aux;
        double gx, re;
        double *xreal, *obj;

        obj   = ind->obj;
        xreal = ind->xreal;

        gx = 0.0;
        k  = number_variable - number_objective + 1;
        for(i = number_variable - k; i < number_variable; i++)
            gx += pow ((xreal[i] - 0.5), 2.0) - cos (20.0 * PI * (xreal[i] - 0.5));
        gx = 100.0 * (k + gx);

        for (i = 0; i < number_objective; i++)
            obj[i] = (1.0 + gx) * 0.5;

        for (i = 0; i < number_objective; i++)
        {
            for (j = 0; j < number_objective - (i + 1); j++)
                obj[i] *= xreal[j];
            if (i != 0)
            {
                aux     = number_objective - (i + 1);
                obj[i] *= 1 - xreal[aux];
            }
        }
        re = 1;
        for (i = 0; i < number_objective; i++)
        {
            if (i != number_objective - 1)
                re = re - obj[i] / 0.5;
            else
                re = re - obj[i] / 0.6;
        }
        if (re > 0) re = 0;
        ind->cv = re;
    }

objective space view
########################

.. figure:: ../_static/CDTLZ/c1dtlz1.png

C1-DTLZ3
~~~~~~~~~~

the form of C1-DTLZ3
#####################
  .. math:: 
    \mbox{min} f_1(x)&=(1+g(x_m))cos(x_1\pi/2)\dots cos(x_{m-2}\pi/2)cos(x_m\pi/2)\\
    \mbox{min} f_2(x)&=(1+g(x_m))cos(x_1\pi/2)\dots cos(x_{m-2}\pi/2)sin(x_m\pi/2)\\
    \mbox{min} f_3(x)&=(1+g(x_m))cos(x_1\pi/2)\dots sin(x_{m-2}\pi/2)\\
    \dots\\
    \mbox{min}f_m(x)&=(1+g(x_m))sin(x_1\pi/2)\\

where

  .. math::
    g(x_m)=100[|x_m|+\sum_{x_i\in x_m}(x_i - 0.5)^2 - cos(20\pi(x_i -0.5))]   

:math:`x_m=(x_m, \dots, x_n)^T`. The constraints is defined as:

  .. math::
    c(x)=(\sum_{i=1}^m f_i(x)^2-16)(\sum_{i=1}^m f_i(x)^2-r^2)\geq0
 
In general, we set r = {9,12.5,12.5,15,15} for m\in {3,5,8,10,15}.

Code
######


.. code-block:: c
   
    void c1dtlz3 (individual_real *ind){
        int i, j, k, aux;
        double gx, fsum, re1, re2, r; // r determines the outermost feasible boundary's radius
        double *xreal, *obj;

        if (number_objective == 3)
            r = 9.0;
        else if (number_objective == 5)
            r = 12.5;
        else if (number_objective == 8)
            r = 12.5;
        else if (number_objective == 10)
            r = 15.0;
        else if (number_objective == 15)
            r = 15.0;
        else
            r = 6.0;

        obj   = ind->obj;
        xreal = ind->xreal;

        gx = 0.0;
        k  = number_variable - number_objective + 1;
        for (i = number_variable - k; i < number_variable; i++)
            gx += pow ((xreal[i] - 0.5), 2.0) - cos (20.0 * PI * (xreal[i] - 0.5));
        gx = 100.0 * (k + gx);

        for (i = 0; i < number_objective; i++)
            obj[i] = 1.0 + gx;

        for (i = 0; i < number_objective; i++)
        {
            for (j = 0; j < number_objective - (i + 1); j++)
                obj[i] *= cos (PI * 0.5 * xreal[j]);
            if (i != 0)
            {
                aux     = number_objective - (i + 1);
                obj[i] *= sin (PI * 0.5 * xreal[aux]);
            }
        }

        // evaluate the constraint violation
        fsum = 0;
        for (i = 0; i < number_objective; i++)
            fsum = obj[i] * obj[i] + fsum;
        //re1 = (fsum - 16.0) * (fsum - r * r);
        //re2 = (fsum - 16.0) * (1 - fsum);

        re1 = (fsum - 16.0) * (fsum - r * r);
        re2 = (fsum - 16.0) * (4 - fsum);

        if (re1 >= 0.0)
            re1 = 0.0;
    
        if(re2 >=0.0)
            re2 = 0.0;

        ind->cv = re1;

        return;
    }

objective space view
########################

.. figure:: ../_static/CDTLZ/c1dtlz3.png

Type-2 Constrained Problems
-------------------------------
This constraint is designed to introduce infeasibility to some parts of the PF.

C2-DTLZ2
~~~~~~~~~~

the form of C2-DTLZ2
#####################
  .. math::
    \mbox{min} f_1(x)&=(1+g(x_m))cos(x_1\pi/2)\dots cos(x_{m-2}\pi/2)cos(x_m\pi/2)\\
    \mbox{min} f_2(x)&=(1+g(x_m))cos(x_1\pi/2)\dots cos(x_{m-2}\pi/2)sin(x_m\pi/2)\\
    \mbox{min} f_3(x)&=(1+g(x_m))cos(x_1\pi/2)\dots sin(x_{m-2}\pi/2)\\
    \dots\\
    \mbox{min}f_m(x)&=(1+g(x_m))sin(x_1\pi/2)\\

where

  .. math::
    g(x_m)=\sum_{x_i\in x_m}(x_i-0.5)^2

:math:`x_m=(x_m, \dots, x_n)^T`. The constraints is defined as:

  .. math::
    c(x)=max\{{\mathop{max}\limits_{i=1}^m[(f_i(x)-1)^2 + \sum_{j=1, j\neq i}^mf_j^2 - r^2],[\sum_{i=1}^m(f_i(x)- \frac{1}{\sqrt m})^2 - r^2]\}}

In general, if the objective number is equal to 2, we set r=0.1. If the objective number is equal to 3, we set r=0.05, otherwise r=0.5.

Code
######

.. code-block:: c
   
    void c2dtlz2 (individual_real *ind){
        int i, j, k, aux;
        double re, gx, fsum, sum1, sum2, sqr;
        double r;  // radius of a feasible region
        double *xreal, *obj;

        if (number_objective == 3)
            r = 0.05;
        else if (number_objective > 3)
            r = 0.5;
        else
            r=0.1;
    
        obj   = ind->obj;
        xreal = ind->xreal;
    
        gx = 0.0;
        k  = number_variable - number_objective + 1;
        for (i = number_variable - k; i < number_variable; i++)
            gx += pow ((xreal[i] - 0.5), 2.0);

        for (i = 0; i < number_objective; i++)
            obj[i] = 1.0 + gx;

        for (i = 0; i < number_objective; i++)
        {
            for (j = 0; j < number_objective - (i + 1); j++)
                obj[i] *= cos (PI * 0.5 * xreal[j]);
            if (i != 0)
            {
                aux     = number_objective - (i + 1);
                obj[i] *= sin (PI * 0.5 * xreal[aux]);
            }
        }

        fsum = 0.0;
        for (i = 0; i < number_objective; i++)
            fsum = fsum + obj[i] * obj[i];

        re   = (obj[0] - 1) * (obj[0] - 1) + fsum - obj[0] * obj[0] - r * r;
        sum2 = 0.0;
        sqr  = sqrt (number_objective);
        for (i = 0; i < number_objective; i++)
        {
            sum1 = (obj[i] - 1) * (obj[i] - 1) + fsum - obj[i] * obj[i] - r * r;
            if (sum1 < re)
                re = sum1;
            sum2 = sum2 + (obj[i] - 1.0 / sqr) * (obj[i] - 1.0 / sqr);
        }
        sum2 = sum2 - r * r;

        if (sum2 < re)
            re = sum2;
        if (re > 0)
            re = -re;
        else
            re = 0.0;
    
        ind->cv = re;

        return;
    }

objective space view
########################

.. figure:: ../_static/CDTLZ/c2dtlz2.png

Type-3 Constrained Problems
-------------------------------       
These problems involve multiple constraints and the PF is formed by portions of the added constraint surfaces.

C3-DTLZ1
~~~~~~~~~

the form of C3-DTLZ1
#####################
  .. math:: 
    \mbox{min} f_1(x)&= \frac{1}{2}x_1x_2\dots x_{m-1}(1+g(x_m))\\
    \mbox{min} f_2(x)&= \frac{1}{2}x_1x_2\dots (1-x_{m-1})(1+g(x_m))\\
    \dots\\ 
    \mbox{min} f_{m-1}(x)&= \frac{1}{2}x_1(1-x_2)(1+g(x_m))\\
    \mbox{min} f_m(x)&= \frac{1}{2}(1-x_1)(1+g(x_m))

where

  .. math::
    g(x_m)=100[|x_m|+\sum_{x_i\in x_m}(x_i - 0.5)^2 - cos(20\pi(x_i -0.5))]   


:math:`x_m=(x_m, \dots, x_n)^T`. The constraints is defined as:

  .. math::
    c(x)=\sum_{i=1,i\neq j}^m f_j(x) + \frac{f_j(x)}{0.5} - 1\geq 0, \forall j=1,\dots ,m.

Code
######

.. code-block:: c
   
    void c3dtlz1 (individual_real *ind){
        int i, j, k, aux;
        double gx, fsum, re;
        double *xreal, *obj;

        obj   = ind->obj;
        xreal = ind->xreal;

        gx = 0.0;
        k  = number_variable - number_objective + 1;
        for(i = number_variable - k; i < number_variable; i++)
            gx += pow ((xreal[i] - 0.5), 2.0) - cos (20.0 * PI * (xreal[i] - 0.5));
        gx = 100.0 * (k + gx);
    
        for (i = 0; i < number_objective; i++)
            obj[i] = (1.0 + gx) * 0.5;
    
        for (i = 0; i < number_objective; i++)
        {
            for (j = 0; j < number_objective - (i + 1); j++)
                obj[i] *= xreal[j];
            if (i != 0)
            {
                aux     = number_objective - (i + 1);
                obj[i] *= 1 - xreal[aux];
            }
        }

        fsum = 0;
        for(i = 0; i < number_objective ; i++)
            fsum = obj[i] + fsum;

        re = 0;
        for (i = 0; i < number_objective; i++)
        {
            if (fsum + obj[i] - 1 < 0)
                re = re + fsum + obj[i] - 1;
    
        }
    
        ind->cv = re;
    }
    
objective space view
########################

.. figure:: ../_static/CDTLZ/c3dtlz1.png

C3-DTLZ4
~~~~~~~~~

the form of C3-DTLZ4
#####################
  .. math:: 
    \mbox{min} f_1(x)&=(1+g(x_m))cos(x_1^\alpha\pi/2)\dots cos(x_{m-2}^\alpha\pi/2)cos(x_{m-1}^\alpha\pi/2)\\
    \mbox{min} f_2(x)&=(1+g(x_m))cos(x_1^\alpha\pi/2)\dots cos(x_{m-2}^\alpha\pi/2)sin(x_{m-1}^\alpha\pi/2)\\
    \mbox{min} f_3(x)&=(1+g(x_m))cos(x_1^\alpha\pi/2)\dots sin(x_{m-2}^\alpha\pi/2)\\
    \dots\\
    \mbox{min} f_m(x)&=(1+g(x_m))sin(x_1^\alpha\pi/2)\\

where 
  .. math:: 
    g(x_m)=\sum_{x_i\in x_m}(x_i-0.5)^2

:math:`x_m=(x_m, \dots, x_n)^T`. The constraints is defined as:

  .. math::
    c_j(x)=\frac{f_j^2}{4} + \sum_{i=1,i\neq j}^m f_i(x)^2 - 1 \geq 0, \forall j=1,\dots ,m.

Code
######

.. code-block:: c
   
    void c3dtlz4 (individual_real *ind){
        int i, j, k, aux;
        double gx, alpha, fsum, re;
        double *xreal, *obj;

        obj   = ind->obj;
        xreal = ind->xreal;

        gx    = 0.0;
        alpha = 100.0;
        k     = number_variable - number_objective + 1;
        for(i = number_variable - k; i < number_variable; i++)
            gx += pow ((xreal[i] - 0.5), 2.0);

        for (i = 0; i < number_objective; i++)
            obj[i] = 1.0 + gx;
    
        for (i = 0; i < number_objective; i++)
        {
            for (j = 0; j < number_objective - (i + 1); j++)
                obj[i] *= cos (PI * 0.5 * pow (xreal[j], alpha));
            if (i != 0)
            {
                aux     = number_objective - (i + 1);
                obj[i] *= sin (PI * 0.5 * pow (xreal[aux], alpha));
            }
        }
    
        fsum = 0;
        for(i = 0; i < number_objective ; i++)
            fsum = obj[i] * obj[i] + fsum;
        re = 0;
        for (i = 0; i < number_objective; i++)
        {
            if (fsum - obj[i]* obj[i] + obj[i]* obj[i] / 4.0 - 1.0 < 0)
            re = re + fsum - obj[i]* obj[i] + obj[i]* obj[i] / 4.0 - 1.0;
        }
        ind->cv = re;
    }

objective space view
########################

.. figure:: ../_static/CDTLZ/c3dtlz4.png    
    
