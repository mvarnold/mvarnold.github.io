---
layout: post
title:  "Equiparitioning"
date:   2017-12-10 15:46:30 -0500
categories: jekyll update
---
<script type="text/javascript" async
  src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-MML-AM_CHTML">
</script>

## Introduction
Recently, I've been working on this problem of optimal facility placement. Briefly stated, if we have a finite number of resources allowing us to place \\(n\\) facilities (hospitals, schools, or coffee shops), where should we put them to get maximum benefit, or maximum access. Formulated as an integral equation, we want  

$$ f(\vec{r}_1 \dots \vec{r}_p) =\int_{A} \rho (\vec{r}) \min_{i \in \{ 1\dots p} \|\vec{r}-\vec{r}_i\| d^2\vec{r} $$

to be minimized. Here \\(\vec{r}_1\\) are the facility locations, \\(\rho (\vec{r}) \\) is the population density, and \\(\|\vec{r}-\vec{r}_i\|\\) is the distance from location r to the nearest facility.

Um et. al. reported an optimal facility placement which scales as a power law to the population density
\\( D \sim \rho ^\alpha \\)
For facilities that attempt to maximize profit \\(\alpha = 1\\)
, while systems that try to maximize facility access by minimizing average travel time, scale as 
\\(\alpha = 2/3\\).
This optimal scaling relects emperically observed facility placement. Certainly my introduction to this topic began with Um et al.


Prior to this empirical study, Gastner and Newman published a paper looking at this same facility placement problem. They had previously published a paper demonstrating the applicability of diffusion methods to create density equalizing cartograms, and now showed that these cartograms, and the diffusion method powering them, could be used to place facilities. Below is shown their plot of 5000 facilities placed optimally, with the US diffused to equalize \\(\rho^{2/3}\\) on the right, and equalizing \\(\rho\\) on the left. 

![Newman]({{ "/images/newman.png" | absolute_url }})

Of course this is an interesting result in its own respect, but our contribution is in recognizing that the class of problems that are solved by optimizing some integral function subject to a set of constraints, such as this k-medians problems, Highly Optimized Tolerance, or Elastic Net can be better understood through this connection to probability diffusion.



## Generating Density Equalizing Projections: Cart
Newman and Gastner published a program for generating cartograms called cart, which given a discrete grid of density values, computes the final position of massless points after diffusion occurs, given initial positions in the density distribution.

By length, most of the c++ code is declaration of variables and allocation of memory, but I briefly go through the relevant computational method.  

The code uses fourth-order Runge-Kutta to intergrate the diffusion equation forward in time, and uses two different step sizes to get an error bound, and to pick an adaptive step size.  

{% highlight c++ %}

/* Do all three RK steps for each point in turn */
  esqmax = drsqmax = 0.0;

  for (p=0; p < npoints; p++) {

    rx1 = pointx[p];
    ry1 = pointy[p];

    /* Do the big combined (2h) RK step */

    cart_velocity(rx1,ry1,s0,xsize,ysize,&v1x,&v1y);
    k1x = 2*h*v1x;
    k1y = 2*h*v1y;
    cart_velocity(rx1+0.5*k1x,ry1+0.5*k1y,s2,xsize,ysize,&v2x,&v2y);
    k2x = 2*h*v2x;
    k2y = 2*h*v2y;
    cart_velocity(rx1+0.5*k2x,ry1+0.5*k2y,s2,xsize,ysize,&v3x,&v3y);
    k3x = 2*h*v3x;
    k3y = 2*h*v3y;
    cart_velocity(rx1+k3x,ry1+k3y,s4,xsize,ysize,&v4x,&v4y);
    k4x = 2*h*v4x;
    k4y = 2*h*v4y;

    dx12 = (k1x+k4x+2.0*(k2x+k3x))/6.0;
    dy12 = (k1y+k4y+2.0*(k2y+k3y))/6.0;
{% endhighlight %}
Here we see the familiar integration scheme, where the new position of the points is integrated forward in time by an amount 2h multiplied by the velocity in the x and y directions. A nearly identical step integrates forward the same initial position with two smaller h step Runge Kutta iterations. 












## Generating Arbiratry Probability Densities for Cart

Adapting Cart for my own uses involved writing a script to generate my probability distributions and formatting them for cart do the heavy lifting. The probability distributions were generated and normalized then stretched to be discretized by the function generate_dist.

{% highlight python %}
def generate_dist(n,dist_type):
	if dist_type == 'exp':
		x = stats.expon.rvs(size = n)
		x *= 1/np.max(x)
		x=x*(dimx-2*pixel_gap)+pixel_gap
		y = stats.expon.rvs(size = n)
		y *=1/np.max(y)
		y = y*(dimy-2*pixel_gap)+pixel_gap
		return x,y
	## add distribution types
	elif dist_type == "gaussian":
		x = np.abs(stats.norm.rvs(size = n))
		x *= 1/np.max(x)
		x=x*(dimx-2*pixel_gap)+pixel_gap		
		y = np.abs(stats.norm.rvs(size = n))
		y *=1/np.max(y)
		y = y*(dimy-2*pixel_gap)+pixel_gap
		return x,y
	else:
		print("Error: distribution not recognized")
{% endhighlight %}

After we have a random sample drawn from some specified probability distribution, the script saves these xy points to file for plotting, and finds the discretized density using the function find_density, which just bins the data, and finds the average density.

{% highlight python %}
def find_density(x,y,dimx,dimy,pixel_gap):
	'''finds spatial density in grid'''
	nx,ny = dimx-(2*pixel_gap),dimy-(2*pixel_gap) # change to be number of pixels
	H,xedges,yedges = np.histogram2d(x/np.max(x),y/np.max(y),bins=[nx,ny])
	ave_density = np.average(H)
	return H,xedges,yedges,ave_density
{% endhighlight %}

Last we create an array to surround this binned density data with a sea of the average density, and save the array to be the input of cart. 

{% highlight python %}
def generate_array(H,xedges,yedges,dimx,dimy,pixel_gap,ave_density):
	'''make array of data in correct dimensions to feed to cart.c'''
	data = np.ones([dimx,dimy])
	data *= ave_density
	for i,x in enumerate(H):
		for j,y in enumerate(x):
			data[i+pixel_gap,j+pixel_gap] = y
	return data
{% endhighlight %}
Below is a plot of the array for a gaussian distribution, just to give an idea.
![Array]({{ "/images/array.png" | absolute_url }})

Cart takes this array, allows the density to diffuse, and then calculates the velocity feild to see to where massless particles would diffuse. 

Here is a copy of [dist_gen.py]({{ "/assets/dist_gen1.py" | absolute_url }})  for your perusal.

## Results
Shown below are the current iteration of figures for this project. The first figure shows on the left 100000 points drawn from both a 2-d exponential and gaussian distribution, color coded by their parition into 50 optimally distributed cells, where the placement of the cells minimizes average distance to the closest cell center. Below is a histogram of the density of the distribution. 

On the right are the same 100000 xy points, after being diffused by cart, still color coded by voronoi cells. Clearly the density is more uniform than before, looking at the xy points themselves, the concentration of black voronoi cell centers, or the histogram below, which is computed be taking a sampling inside the thin black rectangle.  
![50 Partions]({{ "/images/2.png" | absolute_url }})

The second figure just demonstrates that the code works for an arbitrary  number of partitions, in this case 500. You can see near the top edge of this image, that the density is still noticably non-uniform. This is likely due to the course grained binning that occurs, and that the transition from zero density in the top right corner to average density just outside the 512x512 array of interest is instantaneous. 
![500 Partions]({{ "/images/500.png" | absolute_url }})
The code used to generate these images is [Equipartitioning]({{ "/assets/Equipartitioning.ipynb" | absolute_url }}), though you should be warned its path depence hasn't been designed with other machines in mind, and I haven't added my implementation of k-medians to the scipy.cluster.vq repository yet. 








