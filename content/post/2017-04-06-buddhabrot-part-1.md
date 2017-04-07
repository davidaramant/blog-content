+++
title = "The Buddhabrot: Rendering a 68 Gigapixel Fractal - Part 1"
draft = false
date = "2017-04-06T13:05:10-04:00"
tags = ["Work related","Buddhabrot"]
+++

At the inaugural [Indy.Code()](https://indycode.amegala.com/) conference last week I presented on how I generated a 68.7 gigapixel rendering of the Buddhabrot fractal.  For now, all you need to know that it's a really, really big crazy-looking picture.  This series is based on that talk.

## Introduction

I was introduced to the Buddhabrot in a college class I took senior year about fractals (actually, it was about *Chaotic Dynamical Systems* but I don't remember too much about that!).  The Buddhabrot stuck with me and I tinkered with it a little bit further after graduating.

A few years ago at [SEP](https://www.sep.com) we introduced [Hackathon Weekends](https://www.sep.com/labs/hackathon/).  The Buddhabrot seemed like the perfect topic for a weekend project so we (I managed to convince two others to help) set out to render a 500 megapixel version.  This was heavily inspired by [Johann Korndoerfer's 500 megapixel version](http://erleuchtet.org/2010/07/ridiculously-large-buddhabrot.html) he made in LISP.  I decided to use C#/.NET because that's what I'm most comfortable with.

We succeeded!  Although impressive, I was a bit unsatisfied with all the shortcuts we had to take to accomplish anything in a weekend. I decided to revisit it in a second Hackathon and raised it to 625 megapixels.  That might seem like a modest increase, but the 500 megapixels actually used to be a very hard limit with how we were doing things.  *(As it turns out, the .NET Image class will throw an undocumented Win32 exception if you try to create an image larger than 500MP.  Geez, don't they test anything over there at Microsoft!!!)*

![History of progress](/buddhabrot/history_of_progress.png)

At this point I was hooked so I pushed it up to 10 gigapixels.  I immediately realized my mistake and corrected it by further pushing it to the largest possible version I could create without radically altering my approach - 68.7 gigapixels!

#### Aside - 68.7 gigapixels?  What a weird number

Now, the number 68.7 might jump out at you because it doesn't sound very computer-sciencey.  How you refer to the size depends on whether you consider a gigapixel to be 1,000 or 1,024 pixels - if you go by 1,024, the size is actually 64 gigapixels.  However, since the only people who actually *care* about megapixels are camera manufacturers (and they cheat as much as they can get away with) I'll go for the more impressive number.

In unambiguous terms, the final rendering is 68,719,476,736 pixels (262,144 by 262,144).

### Part 1 - Background

In this part we'll start from the beginning and answer the following questions:

* What is a fractal?
* What is the Mandelbrot Set?
* What is the Buddhabrot?
* Why is it a big deal to make a 68.7 gigapixel version of it?

This first part does include a bit of math, but don't worry, we won't be getting into any scary theoretical stuff.

If you're already familiar with the Buddhabrot you can probably skip to Part 2.

### Part 2 - How did I do this?

In part two I'll explain *how* I made this thing.

* Algorithmic optimizations
* Code optimizations
* Visualization techniques


Let's begin!

## What is a Fractal?

A **fractal** is defined by *self similarity*.  Quite simply, that means that parts of the object are similar to the whole.

A classic example of a fractal that you may have seen is the **Sierpinski Triangle**:
![Sierpinski Triangle](/buddhabrot/Sierpinski_triangle.png)

As you can see, each small triangle is identical to the whole.

Fractals have sometimes been described as "the mathematics of nature" because this concept often shows up in the real world.  Take the example below:
![Comet vs Dust](/buddhabrot/comet_vs_dust.jpg)

On the left, we have the comet that the *Rosetta* spacecraft landed on in 2014.  On the right, grains of dust under extreme magnification.  Even though the scales are wildly different they look pretty similar!

Fractals *do* have real-world applications, but the one we'll be looking at in this series doesn't do too much other than look cool.

## What is the Mandelbrot Set?

### Complex Numbers

The most complicated mathematical concept we have to introduce is [**complex numbers**](https://en.wikipedia.org/wiki/Complex_number).  A complex number is written as **a + b_i_** where **a** is a real number and **b** is an imaginary number (remember that _i_ is the square root of negative one).

Just like "normal" numbers, you can add and multiply complex numbers.  These rules are very straightforward, but you can look those up yourself if you are interested. The important thing to keep in mind is that a complex number has two independent components and that addition/multiplication isn't quite the same as a normal number.

### The Complex Plane

Since a complex number has two different parts, we can draw this in two dimensions.  Mathematicians call this the **complex plane**:

![Complex Plane](/buddhabrot/complex_plane.png)

Here we're looking at a circle of radius 2 around the origin (yes, the value of 2 is special, but we're not going to go into the underlying mathematics).  The real numbers go across the X-axis and the imaginary numbers are on the Y-axis.  See the white dot?  That's -1 + 0.25_i_, as an example.

### The Magic Function

Let's introduce a function:
![z = z*z +c](/buddhabrot/function.svg)

_z_ and _c_ are both complex numbers.  We'll pick any starting point in the gray circle of radius 2 and call it _c_.  Starting with _z_ = 0, the function says that to calculate the next _z_ value, we take the previous one, square it, and add _c_ to it.  In other words, we will generate an infinite series of complex numbers.

Something interesting happens with those numbers:

* For some _c_ values, the numbers we generate will eventually escape the circle of radius 2 and go flying off to infinity, never to return.
* For others, they will never leave the circle of radius two _even under infinite iteration_.

That last part is pretty weird!  What do the the starting points look like that will never escape?

![Mandelbrot Set](/buddhabrot/complex_plane_mandelbrot.png)

We've just found the [Mandelbrot Set](https://en.wikipedia.org/wiki/Mandelbrot_set).

### Hold on, "infinite iteration?"

I used a bad word in that definition - computers don't like the concept of _infinity_ very much.  I mean, they're happy enough to do something forever, but us impatient humans aren't very happy waiting around that long.

So, like the lazy programmers we are, we'll pick a value for "infinity" and leave the theoretical perfection to the mathematicians.  As it turns out, the value we pick changes the output considerably:

![Mandelbrot Set with differing upper limits](/buddhabrot/mandelbrot_limits.png)

Here we have three renderings using values of 5, 10, and 15 as the maximum limit.  If the points are still within the circle after that many iterations, we assume they are in the Mandelbrot Set.  As you can see, raising the limit gets us a more defined image.

### For context...

To make it really concrete, the code to do the important bits we've covered so far is as follows:

```cs
public bool IsInMandelbrotSet(Complex c, int iterationLimit)
{
    var z = Complex.Zero;

    for (int i = 0; i < iterationLimit; i++)
    {
        z = z * z + c;

        // check if the point has escaped the circle of radius 2
        if (z.Magnitude * z.Magnitude > 4)
            return false;
    }

    return true;
}
```

Not too complicated.  The only thing to keep in mind is that multiplication and addition with complex numbers is slightly more complicated than it appears.  The check to see whether we've escaped the circle of radius 2 also looks scarier than it is - we're just using the [Pythagorean theorem](https://en.wikipedia.org/wiki/Pythagorean_theorem).

### OK, but how do I make it look cool?

With what we've talked about so far, we can make a very crisp, black-and-white version of the Mandelbrot Set:

![Crisp B/W Mandelbrot](/buddhabrot/crisp_mandelbrot.png)

That's cool and all, but normally when you see it there's more going on than this.  What info do we have available to us?

### Escape Time

One thing we can visualize is the **escape time** - how many iterations it took for the point to escape the circle and bail out of the loop:

![Escape Time visualized in Black and White](/buddhabrot/escape_time_bw.png)

Here I'm just visualizing odd escape times in black and even ones in white.  You can see that a fair chunk of the circle only takes one iteration before it fails the check, a tighter oval takes two, etc.  As we get closer to the border of the Mandelbrot Set, it takes longer and longer for the point to break out of the loop.

This is certainly more interesting, but it _still_ doesn't look much like how you normally see it.

### Escape Time - Fancier!

![Escape Time visualized with Color](/buddhabrot/escape_time_color.png)

This is more like it!  Here I've taken the exact same data we saw on the previous image and mapped it to a simple color ramp instead.

Although this is a very simplistic version, visualizing the escape time is one of the most common ways you see the Mandelbrot Set (or, more accurately, the points _around_ the Mandelbrot Set).  For whatever reason, it seems to be extremely popular to visualize the poor thing using a 60s tie-dye or 70s van-art aesthetic (don't ask me).  I'm sure you can already think of ways to make that image more visually impressive by zooming in more, picking different colors, smoothing out the color gradients, etc...

### Is there any other data we can visualize?

Let's go back and review what we're doing: for a given input point, we're going to generate a series of points until it escapes (or not) the circle of radius 2.  That series of points are sometimes referred to as the **trajectory** of the starting point.

Let's visualize a trajectory:

![Trajectory of a point](/buddhabrot/trajectory.png)

(Note that this is a totally made up example!  If you plot the trajectory of that real point it wouldn't look like this)

Our completely-made-up trajectory starts on the left side and under iteration the points move rightwards until one escapes the circle.  What if we used information from the trajectory itself?

### Trajectory Visualizations

Let's base the color intensity from how close any of the points in the trajectory come to the origin, and the color hue from the angle of the point from the real axis (X-axis):

![Trajectory Mandelbrot Visualization](/buddhabrot/trajectory_visualization.png)

Well, uh, let's try to ignore that what I did ended up looking like an abstract Oscar Mayer nightmare and notice that we are getting some much weirder output.  All those lines come out of things are called [Pickover stalks](https://en.wikipedia.org/wiki/Pickover_stalk), which is another rabbit hole you can explore.

The point I want to emphasis is that all these visualizations are using data that we've already seen how to generate.  The complex nature of the output comes from the magical nature of the Mandelbrot Set itself; how complicated you want to get in your visualization technique is up to you.

## OK, so what about the Buddhabrot?

With the information we've learned we can now introduce the Buddhabrot:

![Buddhabrot](/buddhabrot/buddhabrot.png)

First of all you'll notice that the Buddhabrot is always displayed rotated 90 degrees.  Mathematicians typically look at the Mandlebrot set in the traditional orientation, but we're not too concerned about math, just something that looks cool.  The Buddhabrot technique was first described by [Melinda Green](http://superliminal.com/fractals/bbrot/bbrot.htm) in 1993.  It's kinda-sorta reminiscent of Buddhist/Hindu artwork and if you squint at it _just_ right you might see a Buddha figure, hence the name.

The Buddhabrot is defined as:

> a density plot of trajectories from points that are not in the Mandelbrot set.

Lets unpack that:

* We're going to start with only the points that are _not_ in the Mandelbrot Set; i.e. they escape the circle of radius 2 after some number of iterations.
* That series of points is called the **trajectory**.
* We're going to take those trajectories and keep track of where they land.  For each point in a trajectory, we'll figure out the pixel location that it corresponds to and increment that location by 1.
* The final image is a visualization of how often each pixel location was visited by a trajectory.  In the above image, brighter pixels were visited more often, while the black images were totally skipped.

### Why do we only plot the numbers that are _not_ in the Mandelbrot set?

If you reverse it and only plot points _in_ the set, you'll wind up with the **Anti-Buddhabrot**:

![Anti-Buddhabrot](/buddhabrot/anti-buddhabrot.png)

This visualization is kinda interesting too (my rendering doesn't quite do it justice), but it's way more symmetrical and less chaotic.

## Minimum Iteration Depth

Remember how the maximum iteration limit changed the appearance of the Mandelbrot Set?  Well, for the Buddhabrot, we also care about the _minimum_ iteration limit:

![Buddhabrot with different minimum iterations](/buddhabrot/buddhabrot_minimums.jpg)

These are different renderings with a minimum escape count of 20, 100, and 1000.  Any point that escapes the circle _before_ that minimum limit is discarded.  As you can see, increasing the minimum limit makes the Buddhabrot 'crisper' and reduces some of the noise.  If you _really_ crank up the minimum limit, you'll start seeing some really cool stuff, as we'll find out later...

### Back to the code...

Sometimes it's clearer to read code instead of words, so here's a simple method to check whether a complex number should be plotted in the Buddhabrot:

```cs
public bool IsBuddhabrotPoint(Complex c, int minLimit, int maxLimit)
{
    var z = Complex.Zero;

    for (int i = 0; i < maxLimit; i++)
    {
        z = z * z + c;

        // check if the point has escaped the circle of radius 2
        if (z.Magnitude * z.Magnitude > 4)
        {
            // Filter by the minimum iteration limit too!
            // Points that escape too fast are 'noisey'
            return i >= minLimit;
        }
    }

    // Point never escaped, so we think it's in the Mandelbrot set
    return false;
}
```

### Nebulabrot

You can actually utilize the effects of changing the minimum iteration limit to make yet _another_ visualization called the **Nebulabrot**:

![Nebulabrot](/buddhabrot/nebulabrot.png)

This looks pretty 'spacey' because it's similar to how NASA generates false-color images of things like galaxies from radio waves - each color channel is a rendering of the Buddabrot with a different minimum iteration limit.

If you're wondering about those weird boxes everywhere - well, they're actually artifacts from an optimization we'll cover in Part 2.  If I find the motivation I should probably re-render this guy to get rid of those.

## Effects of the Number of Points

In addition to the maximum / minimum iteration limits, another variable that is very important for the Buddhabrot is the **number of points we plot**:

![Noisy Buddhabrot](/buddhabrot/noisy_buddhabrot.png)

Remember that we're visualizing a density map of how many times each pixel location has been visited.  If we don't plot enough points, our map will be extremely noisy like the above image.

## Part 1 Summary - The Buddhabrot is Slow!

Let's wrap up Part 1 by summarizing what we've learned so far:

* A high minimum iteration limit will result in a cooler Buddhabrot
* The Buddhabrot needs a lot of points to look cool
* Unlike the normal Mandelbrot visualization, there is not a 1:1 mapping between points and pixels.

A normal Mandelbrot visualization can be done in real-time (or close to it) and there are lots of fractal viewer programs you can download to play around with it.  One popular thing to do is to zoom in at incredible magnifications - the Mandelbrot set has infinite detail, so you will always find something down there.  This is feasible because every pixel in the output is one point.  Computing an image at the "normal" level versus something zoomed in 1000x is the same effort (minus any overhead from using an arbitrary-precision data type).

Not so for the Buddhabrot.  Because we don't know which starting points will contribute to a particular pixel location, zooming in real-time is not feasible.  You _can_ estimate which points you should iterate to get an image using technique like the Metropolis-Hastings algorithm, but that's pretty advanced.  It's actually much conceptually simpler to pre-render a gigantic version of it and zoom in after-the-fact.

My particular version has the following stats:

* Minimum iteration limit: 1,000,000
* Maximum iteration limit: 5,000,000
* Size: 68,719,476,736 pixels (262,144 x 262,144)

That's a lot of work!  If we want to get this done before the heat death of the universe we'll have to come up with some clever ways of speeding it up...  See Part 2 for those details!
