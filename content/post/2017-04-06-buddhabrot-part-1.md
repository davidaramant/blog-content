+++
title = "The Buddhabrot: Part 1 - What's a Buddhabrot?"
draft = false
date = "2017-04-06T13:05:10-04:00"
tags = ["Work related"]
+++

At the innaugural [Indy.Code()](https://indycode.amegala.com/) conference last week I presented on how I generated a 68.7 gigapixel rendering of the Buddhabrot fractal.  For now, all you need to know that it's a really, really big crazy-looking picture.  This series is based on that talk.

# Introduction

I was introduced to the Buddhabrot in a college class I took senior year about fractals (actually, it was about *Chaotic Dynamical Systems* but I don't remember too much about that!).  The Buddhabrot stuck with me and I tinkered with it a little bit further after graduating.

A few years ago at [SEP](https://www.sep.com) we introduced [Hackathon Weekends](https://www.sep.com/labs/hackathon/).  The Buddhabrot seemed like the perfect topic for a weekend project so we (I managed to convince two others to help) set out to render a 500 megapixel version.

We succeeded!  Although impressive, I was a bit unsatisfied with all the shortcuts we had to take to accomplish anything in a weekend. I decided to revisit it in a second Hackathon and raised it to 625 megapixels.  That might seem like a modest increase, but the 500 megapixels actually used to be a very hard limit with how we were doing things.  *(As it turns out, the .NET Image class will throw an undocumented Win32 exception if you try to create an image larger than 500MP.  Geez, don't they test anything over there at Microsoft!!!)*

![History of progress](/buddhabrot/history_of_progress.png)

At this point I was hooked so I pushed it up to 10 gigapixels.  I immediately realized my mistake and corrected it by further pushing it to the largest possible version I could create without radically altering my approach - 68.7 gigapixels!

### Aside - 68.7 gigapixels?  What a weird number

Now, the number 68.7 might jump out at you because it doesn't sound very computer-sciencey.  How you refer to the size depends on whether you consider a gigapixel to be 1,000 or 1,024 pixels - if you go by 1,024, the size is actually 64 gigapixels.  However, since the only people who actually *care* about megapixels are camera manufacturers (and they cheat as much as they can get away with) I'll go for the more impressive number.

In unambiguous terms, the final rendering is 68,719,476,736 pixels (262,144 by 262,144).

## Part 1 - Background

In this part we'll start from the beginning and answer the following questions:

* What is a fractal?
* What is the Mandelbrot Set?
* What is the Buddhabrot?
* Why is it a big deal to make a 68.7 gigapixel version of it?

This first part does include a bit of math, but don't worry, we won't be getting into any scary theoretical stuff.

If you're already familiar with the Buddhabrot you can probably skip to Part 2.

## Part 2 - How did I do this?

In part two I'll explain *how* I made this thing.

* Algorithmic optimizations
* Code optimizations
* Visualization techniques


Let's begin!

# What is a Fractal?

A **fractal** is defined by *self similarity*.  Quite simply, parts of the object are similar to the whole.

A classic example of a fractal is the **Sierpinski Triangle**:
![Sierpinski Triangle](/buddhabrot/Sierpinski_triangle.png)

As you can see, each small triangle is identical to the whole.

Fractals have sometimes been described as "the mathematics of nature" because this concept often shows up in the real world.  Take the example below:


On the left, we have the comet that NASA recently landed on.  On the right, grains of dust under extreme magnification.  Even though the scales are wildely different they look pretty similar!

Fractals *do* have real-world applications, but the one we'll be looking at in this series doesn't do too much other than look interesting.

# What is the Mandelbrot Set?


# What is the Buddhabrot?

