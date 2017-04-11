+++
title = "The Buddhabrot: Rendering a 68 Gigapixel Fractal - Part 2"
draft = false
date = "2017-04-10T13:05:10-04:00"
tags = ["Work related","Buddhabrot"]
+++

In [part 1](/post/the-buddhabrot-part-1) we learned what the Buddhabrot is and why it's pretty slow to generate.  As a recap, my version of it uses the following values:

* Minimum iteration limit: 1,000,000
* Maximum iteration limit: 5,000,000
* Size: 68,719,476,736 pixels (262,144 x 262,144)

## Algorithmic Optimizations

> The real problem is that programmers have spent far too much time worrying about efficiency in the wrong places and at the wrong times; **premature optimization is the root of all evil** (or at least most of it) in programming.
> 
> Donald Knuth

Before we start throwing code at the problem we're going to have to put on our thinking caps and try to figure out some ways to reduce the amount of work we're doing.

Let's take a look at the Mandelbrot set again:

![Mandelbrot Set](/buddhabrot/complex_plane_mandelbrot.png)

Since we only care about points that _aren't_ in the set and our maximum iteration count is pretty high (5 million), picking a point that's inside the set is obviously very expensive.  Remember, those points will never leave the circle even under infinite iteration, so we will always loop for our maximum number before we have to throw away all that work.

Is there some way to guesstimate whether a point is in the set?

### Main Bulb Check

As it turns out, yes!  The main bulb of the set can be perfectly described by a cardoid, and it's possible to fit circles inside of all of the smaller bulbs:

![Mandelbrot Set with the bulbs excluded](/buddhabrot/mandelbrot_bulbs_excluded.png)

With this optimization, we can easily discard any starting point from those areas with a quick geometric check.

How do you find the equations for those areas?  Well, uh, you search the internet.  There's an equation that you can solve to get the information for all those circles, but I'm not sure I even know the name of the math class that I didn't take in college that would have let me solve for those.

### Where are the interesting points?

OK, so far, so good - that eliminates a bunch of points we _don't_ want.  Where are the points we _do_ want?

Lets look at the visualization of the escape times from part 1:

![Escape Time visualized in Black and White](/buddhabrot/escape_time_bw.png)

From this, we can see that the escape time increases as you move closer and closer to the border of the Mandelbrot set.  In other words, since we are looking for points that escape between 1 - 5 million iterations, we only need to search in areas that are really close to the edge.

### Border Regions

![Mandelbrot Set with edge regions highlighted](/buddhabrot/mandelbrot_edge_areas.png)

By find the edge areas we can _dramatically_ increase the chances that we pick a point that will match our criteria.  Instead of picking a starting point from the giant gray circle, we can now limit ourselves to only those highlighted in red above.  This is by far the most important optimization we can do.

The algorithm for finding the border regions isn't particularly complex:

* Divide up the area into a grid of points.
* Iterate each point to see whether it is inside or outside the set.
* The border regions are going to be the squares where some of the points are in the set and some aren't:
    * If all four corners are not in the set, we assume it's outside
    * If all four corners are in the set, we assume it's inside
    * If the corners are a mix, the region must include part of the border

Yes, there will be regions that the set dips into without covering one of the corners.  That will be true no matter what resolution of the grid you choose, however, since the Mandelbrot set has infinite detail.

### Any other optimizations?

In our first optimization, we were able to filter out _some_ of the points inside the Mandelbrot set.  However, it's still possible some of them will slip through.  What actually happens to the points that are inside the set when you iterate them?

Perhaps unsurprisingly, they will eventually get caught in an infinite loop, generating the same points over and over again.

### Cycle Detection

To catch points like that we can use a cycle detection algorithm like [Brent's algorithm](https://en.wikipedia.org/wiki/Cycle_detection#Brent.27s_algorithm).  Essentially, it periodically checks the current _z_ value against an older version to see if we're stuck in a loop.

This algorithm felt a bit weird to me when I first saw it because it's directly comparing floating-point numbers, which is probably the biggest thing you're never supposed to do with floating-point numbers.  Well, this is one of the few exceptions to the rule.  If you think about it, if the _z_ value is ever the exact same binary value as a previous _z_ it will naturally become an infinite loop since the algorithm is deterministic - the same input will always generate the same output.

When I implemented this I had no concept of how much or little it improved the speed.  Lets come back to it in a little bit to see if we can answer that...

### What optimization _aren't_ we doing?

Lets take a look at a plain rendering of the set again:

![Crisp B/W Mandelbrot](/buddhabrot/crisp_mandelbrot.png)

Can you think of an optimization we haven't talked about?  Hint: it would make things twice as fast...

If you guessed something involving _symmetry_ you are correct.  The Mandelbrot set is perfectly symmetrical across the real axis.  For the purpose of the Buddhabrot, we _could_ mirror everything we did and thereby find points twice as fast.  If your maximum iteration limits are fairly low this actually makes perfect sense; however, we're using fairly high limits because I claimed they looked cooler.  We'll revisit this when I talk about the final rendering, but this is one optimization I intentionally skipped.

## On to Code Optimizations!

![Ryan Gosling meme](/buddhabrot/gosling_meme.jpg)

OK - we've drastically reduced the amount of work we have to do, so lets try to optimize what's left as much as possible.

### A note on benchmarks...

Benchmarks are _hard_.

> There are three kinds of lies: lies, damned lies, and ~~statistics~~ benchmarks.
>
> Jon Skeet, probably

Benchmarks in .NET are even harder - you've got the just-in-time compiler, the garbage collector, etc.  Now, Jon Skeet might not have said the above, but he did work on the excellent [BenchmarkDotNet](https://github.com/dotnet/BenchmarkDotNet) project that makes some of the mechanics easier.

![BenchmarkDotNet logo](/buddhabrot/benchmarkdotnet_logo.png)

Even with tools like this, it can be pretty challenging coming up with realistic scenarios.  When I dusted off my fractal code to prepare for my talk, some of the improved benchmarks I wrote didn't remotely match what I had done just a year earlier (I think my earlier attempts were far too artificial).  I can only say that the results I'm presenting are what I saw on my machine with my code.  Your results may vary.

#### Is Brent's Algorithm worth it?

Let's actually measure the impact of using a cycle detection algorithm:

![Brent's Algorithm Benchmark](/buddhabrot/brents_algorithm_benchmark.png)

Welp.

### Break up processing

OK, back to the code.  The first concept is to break up the work into multiple phases:

![Stages of generating the Buddhabrot image](/buddhabrot/stages_of_processing.png)

Unless you're only making a teeny-tiny image, rendering a Buddhabrot is _not_ something you want to do in a single step.  By breaking it up we can also more easily focus on optimizing each phase.  The stages I used are:

* **Find border regions** - This doesn't take too long, but it's something we only have to do once.
* **Find points** - This is _only_ finding points that match our iteration limit criteria.  The output will be a bunch of complex numbers.
* **Plot trajectories** - Run through the points to find how many times each pixel location was visited.  This stage will give us something of the same dimensions as the final image, but it's just numbers at this point.
* **Render image** - Turn the raw numbers from the last phase into pretty pixels.

By breaking it up like this we can save our progress and more easily iterate on certain phases.  For example, finding the points is _by far_ the slowest stage of the process, but it's also something we can throw a lot of hardware at because there is no interaction between the data.  Also, if we make a complete image and decide that we really need some more points, well, just find some more.  The trajectory map from phase 3 can be added on to without having to re-create it.  The last stage is likely the one that will be run the most number of times, since creating an aesthetically pleasing image is pretty subjective and there's a lot of trial and error.

### Phase 1 - Finding the border regions

It's no surprise that some shortcuts get taken when you try to do a project in a weekend.  Our initial attempt at find the border regions was a bit...  _suboptimal_.  Below you can see our old edge regions to the left, along with the newer version I made:

![Old vs New Edge Areas](/buddhabrot/old_vs_new_edges.png)

If you're thinking to yourself "gee, the left side seems to include a lot more than just the edges" you would be absolutely correct.  Since we didn't bother visualizing what we had done, we didn't notice there was a bug in the algorithm that caused it to spit out regions _inside_ the set as well.  It's also far, far coarser than the new version I made (the right image is actually heavily shrunk down).

Having good border regions is key to making phase 2 faster, so it's worth some time making sure the output of this phase is good.  This is actually an area where it makes perfect sense to exploit the symmetry of the Mandelbrot set - the regions are going to be identical on both halves, so computing both sides is a waste.  I only realized this _after_ I was done with it, but if I ever come back to it...

### Phase 2 - Thread-level parallelism

Finding the interesting points is what's know as an ["embarrassingly parallel"](https://en.wikipedia.org/wiki/Embarrassingly_parallel) problem - there is no shared state what-so-ever that we have to keep track of.  In .NET we can use [Task Parallel Library](https://en.wikipedia.org/wiki/Parallel_Extensions#Task_Parallel_Library) to take advantage of this.

Honestly, I don't have much to say about this - just like Ballmer-Jobs below says, it just works.

![Baller-Jobs](/buddhabrot/ballmer-jobs.jpg)

### Phase 2 - SIMD

Another form of parallelization that you might not be as familiar with is vectorization.  For this we're going to have to dive-down to the CPU instruction level:

![SISD vs SIMD](/buddhabrot/sisd_vs_simd.gif)

Traditionally, CPU instructions are what's known as **single instruction, single data (SISD)** - a particular instructions (like an 'add') would work on one piece of data (ok, it might be two different variables) and produce a single output.  With **single instruction, _multiple_ data (SIMD)** instructions, the data and the output aren't just a single scalar value but a _vector_ of values.  The 'add' instruction can now do multiple additions at the same time.

SIMD support was added to .NET 4.6 with the new RyuJIT compiler.  To use it, you have to target x64-only and include the `System.Numerics.Vectors` NuGet package.  The CPU I ran the following benchmark on supports the [AVX-2](https://en.wikipedia.org/wiki/Advanced_Vector_Extensions#Advanced_Vector_Extensions_2) instruction set which means it had 256-bit (32 byte) vectors to work with.  Since a `float` is four bytes, we can cram 8 of them into a single vector for a theoretical speedup of 8x.

![SISD vs SIMD benchmark](/buddhabrot/sisd_vs_simd_benchmark.png)

I'm not entirely sure this benchmark is entirely accurate (I would guess the real-life speedup is somewhat less dramatic than this) but exploiting SIMD is absolutely a fantastic way speed up the point search.

A final word I have on the subject is that vectorization is _weird_.  I would consider it to be it's own way of programming: there's single-threaded code, multi-threaded code, and vectorized code as its own thing.  Converting code to be vectorized can require a different way of thinking of things that's not exactly intuitive.

### GPU?

The following is the die layout of a recent-ish Intel CPU (I believe it's the Skylake architecture):

![CPU die layout](/buddhabrot/cpu_layout.jpg)

As you can see, this particular chip has 4 cores... along with a GPU that's the same size as all of them combined!  If you know anything about GPUs you know that Intel's offerings are pretty modest performance-wise, but even so they're willing to devote that much space to it.

GPUs would be super-swell at finding points, but unfortunately the state of GPUs in .NET isn't great right now:

* [CUDAfy.NET](https://cudafy.codeplex.com/) _was_ a popular option (and it supported Intel GPUs!) but it hasn't had a release since April 2017.
* [Alea GPU](http://www.quantalea.com/) seems like a promising option - it's commercially supported and was even featured on NVidia's blog.  Unfortunately, it's NVidia-only and I don't have one of those cards.

GPUs are something I want to look more into in the future, but I haven't found the time for it yet.

## Phase 3 - Plotting the points

Once we have a bunch of points, we need to plot their trajectories to generate the density map.

When we first did this we just wrote to a bunch of `int` arrays in memory.  Why `int`?  We had no concept of how high the counts could get and .NET also has the `Interlocked` class that could atomically increment an `int`.  This approach was "good enough" for a 500 megapixel version - since an `int` is 4 bytes, that's about 2GB of data.  We actually did run into the dreaded `OutOfMemoryException` if we had too much other stuff running.  Persisting it to disk was also relatively simple.

Clearly, this approach is not remotely scalable to 68.7 gigapixels.  For this approach I changed some things:

* After (gasp!) _examining the data_, I decided that `int` was complete overkill and used `ushort` instead (unsigned 2-bytes).
* At 2 bytes per pixel location, that's still a 128GB file.  I used a [memory-mapped file](https://en.wikipedia.org/wiki/Memory-mapped_file) to hold this instead of messing around with arrays.  A memory-mapped file allows you to open an arbitrarily huge file and leave it up to the operating system to manage how much of it should be loaded into RAM (it's very similar to virtual memory).  The more RAM the merrier obviously, but this way I didn't have to manually manage anything.
* Instead of laying out the pixels in the same order as the final image, they were instead broken up into square regions.  This was for two reasons:
  * Each region was protected independently with a lock.  Multiple threads could still run into a bunch of contention issues, but if they're working on points that are on completely different parts of the main image there shouldn't be any collisions.
  * Each region is 256 x 256 pixels.  This is important for the next stage.

![Buddhabrot with Grid](/buddhabrot/buddhabrot_with_grid.png)

As example showing the grid concept.  The full size rendering used 1024 x 1024 regions.

## Stage 4 - Generating the Image

Once we have all the trajectories plotted we need some way of turning these numbers into pixels.  Each of the 256 x 256 regions from the last stage will be turned into an independent image.  Spitting out a single image becomes infeasible after a couple of hundred megapixels, so for 68.7 gigapixels we will have to rely on a deep-zoom framework (more on that later).

### Fit the data!

In our first attempts at generating the Buddhabrot we were essentially going blind - we had no idea what the data looked like, so the exact algorithm to turn a single number (the hit count for a pixel location) into a color was entirely trial-and-error.  If you go down that route, your first attempts will probably resemble the left and right versions in the below image.  Either your image will be incredibly dim or utterly blown out.  How do we get a Goldilocks image like in the middle?

![Buddhabrots with differing dynamic ranges](/buddhabrot/buddhabrot_dynamic_range.jpg)

I eventually came upon the novel concept of examining the data inside the giant trajectory plot.  By doing some simple histograms, it became clear that a vanishing small number of pixels were visited a tremendous number of times (around the origin, for example) while the vast majority of pixels had orders of magnitudes less hits.  This distribution of data is what causes extremely dim images like the above left if you try to linearly map the hit counts to colors - the handful of locations with high hits will be the only bright parts.  If you attempt to correct it too much to bring out the detail in the dim areas, the areas with greater counts will become blown out like in the right image.  To get something better, we have to reduce the highs will bringing up the lows.

### Tone Mapping

The eventual algorithm I came up with was informed by the histogram of the hit counts to try to balance the distribution of colors based on how many pixels fell into each range.  After the fact, I realized that what I had done more or less fell into the domain of [tone mapping](https://en.wikipedia.org/wiki/Tone_mapping).  Tone mapping is a process of converting high-dynamic data into a more limited range.  Typically it is used for combining multiple photographs with different exposures like the below example:

![Example of tone mapping](/buddhabrot/tone_mapping_example.png)

Using the parlance of tone mapping, what I had come up with was a _global operator_ - every single pixel in the image was independently calculated.  There is another class of tone mapping algorithms called _local operators_ that also take into account the _surrounding_ data.

I think my results are OK but there are still some areas that don't show a lot of detail.  Playing around with some existing tone mapping algorithms (especially the local operator varieties) would be an interesting experiment.

### RBG vs HSV

One recommendation I have is that anytime you're doing actual computations with colors, _do not use RGB_.

![RBG colors](/buddhabrot/rgb.png)

Representing colors with red-blue-green components is simple to understand and fast for computers, but computing gradients with them gives pretty miserable results.  Instead, something like hue-saturation-value is a much, much better alternative.

![HSV colors](/buddhabrot/hsv.jpg)

Linearly interpolating between two colors represented as HSV will give _much_ more pleasing results than doing the same with RGB.

### The Tyranny of Color Theory

Picking the colors you want is ultimately subjective, but you might run into problems with color perception.

![Color gradients](/buddhabrot/color_gradients.png)

The problem is that human eyes are not equally sensitive to the entire color spectrum.  Some colors will simply look brighter than others even though they are "mathematically" equal.  This can be pretty frustrating if you're trying to put together a palette that maximizes the use of color in order to show more detail.  I experimented a lot with different palettes for my output before landing on the fairly conservative one in the bottom right corner.

### Beware of `System.Drawing.Bitmap.SetPixel`

The standard bitmap class in .NET only has one obvious way of setting colors: `SetPixel`.  Unfortunately, `SetPixel` is _brutally_ slow.  The better way to do things is to use `LockBits`/`UnlockBits` to directly manipulate the pixel buffer inside of the image.  I'm not sure why they haven't added a nicer convenience method to allow you to set all the colors at once, but it is very much worth your while to do things the hard way:

![Bitmap benchmark](/buddhabrot/bitmap_benchmark.png)

Remember that we're generating 1,048,576 tiles (1024 x 1024) so the time savings from this change is nearly 5.6 hours!

### Generating the Zoom Levels

The tile images that we generate represent the most zoomed-in version of the Buddhabrot.  To create the other zoom levels, I used [ImageMagick](http://www.imagemagick.org/script/index.php) to "glue" tiles together.

![Zoom levels](/buddhabrot/zoom_levels.png)

For each zoom level, four tiles are combined to a single tile.  This process is repeated until eventually t

## Summary and Future Work

[Buddhabrot](https://www.sep.com/labs/buddhabrot)