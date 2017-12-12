---
layout: post
title:  "Equiparitioning"
date:   2017-12-10 15:46:30 -0500
categories: jekyll update
---
## Introduction
Recently, I've been working on this problem of optimal facility placement. Um et. al. reported an optimal facility placement which scales as a power law to the population density  \\( D \sim p^\alpha \\). For facilities that attempt to maximize profit \\(\alpha = 1\\), while systems that try to maximize facility access by minimizing average travel time, scale as \\(\alpha = 2/3\\). This optimal scaling relects emperically observed facility placement. Certainly my introduction to this topic began with Um et al.

Prior to this empirical study, Gastner and Newman published a paper looking at this same facility placement problem. They had previously published a paper demonstrating the applicability of diffusion methods to create density equalizing cartograms, and 
a

[//]: <> add things about how the scaling is derived

Of course this is an interesting result in its own respect, but our contribution is in recognizing that the class of problems, which are solved by optimizing some integral function subject to a set of constraints can be solved by 



## Generating Density Equalizing Projections: Cart
Newman and Gastner published a program for generating cartograms called cart, which given a discrete grid of density values, computes 


{% highlight c++ %}

void cart_density(double t, int s, int xsize, int ysize)
{
  int ix,iy;
  double kx,ky;
  double expkx;
}
{% endhighlight %}

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

![This]({{ "/images/2.png" | absolute_url }})
![This]({{ "/images/3.png" | absolute_url }})
