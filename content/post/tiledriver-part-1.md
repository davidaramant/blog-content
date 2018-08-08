---
title: "Tiledriver: Fun with Wolfenstein 3D"
draft: false
date: 2018-05-14
tags: ["Work related","Tiledriver"]
description: "Background info about Wolfenstein 3D, ECWolf, etc"
---

A few years ago at on the hackathon weekends at my company I decided to try writing a level generator for the game Wolfenstein 3D.  I had done zero research on how level generators worked going into it, but with the help of two others we managed to pull it off in a weekend!

Were these levels good?  No.  No, they were not.  We had to take a lot of shortcuts to get something working with the limited time available.  After it was over I kept thinking of how the generator could be improved, but every idea seemed like a _lot_ of work...

But wait, what's Wolfenstein 3D?

## Wolfenstein 3D

![Wolfenstein 3D title screen](/tiledriver/wolf3d-title.png)

Wolfenstein 3D, the hottest game of 1992!  Wolf 3D tells the heart-warming and completely historically accurate tale of how a single allied prisoner of war single-handedly shot a billion Nazis and eventually took down Mecha-Hitler.

![Wolfenstein 3D gameplay collage](/tiledriver/wolf3d-gameplay-collage.png)

Wolf 3D has an extremely simple level format.  Conceptually each map is a 64x64 grid with each space consiting of a wall, empty space, enemy, decoration, etc.  There is no height variation whatsoever.  

![Wolfenstein 3D example map overview](/tiledriver/wolf3d-map-overview.png)

The game seemed like an ideal starting point for exploring procedural level generation since the format is so simple.  With a game like Quake or even Doom, the level format itself has a lot of rules for what determines a valid map, whereas with Wolf 3D almost any combinations of blocks is a level (whether or not you can _play_ it is a different matter).

### Enter ECWolf

Wolfenstein 3D, as released in 1992, was a completely self-contained game.  There was no mechanism built into it for loading user-created content: you bought the game, that was it.  However, fans quickly managed to reverse engineer various formats and were soon able to hack their own levels into the game.  Once the source code for the game engine was released in 1995 it really opened the flood gate of possibilities for user-made content.

![Wolf 3D Open Source Overview](/tiledriver/wolf3d-open-source-overview.png)

The ultimate result of this is the [ECWolf](http://maniacsvault.net/ecwolf/) project.  In additon to quality of life improvements like running on modern operating systems and supporting higher resolutions, a key focus of ECWolf is making everything as data-driven as possible.  The most relevant aspect of this for my purposes is [UWMF](http://maniacsvault.net/ecwolf/wiki/Universal_Wolfenstein_Map_Format) - the Unified Wolfenstein Mapping Format.  Maps can now be defined in a flexible text-based format instead of having to deal with a quarter-century old binary format.  UWMF also removes some of the arcane limitations of the older format; for example, maps are no longer required to be exactly 64x64 in size - smaller or larger (within reason) levels can just as easily be created.

Unfortunately, Wolf 3D is not nearly as popular as its younger brother Doom, and the editing scene has been somewhat slow to embrace ECWolf.  There _are_ a few mapping tools in semi-active development for Wolf 3D, but none of them support UWMF yet.  So, I had to support this format from scratch.

## Procedural Level Generation

Procedurally generating levels is a massive topic which I won't try to completely cover here.   For a greater look at that fascinating subject I highly recommend the [Procedural Content Generation in Games](http://pcgbook.com/), a free online textbook.  After reading more about the theory, what we implemented in the first version of Tiledriver turned out to be a simplistic agent-based system.  Take a look at a map that it spit out to see if you can guess how it was made:

![Tiledriver 1 Map](/tiledriver/tiledriver-1-map.png)

If you guessed "try to recursively add rectangles on all four sides," congratulations, you guessed right!  We did some trivial theming of the level, but overall, it's only capable of making incredibly samey, boxy levels with absolutely no overall sense of gameplay progression.

### Making Caves with Cellular Automata



* Deep Learning Basics
  * Continuous vs Discrete
  * Tensors
  * Show flow of data
  * Loss
  * Overfitting
  * Generative Models
* Data Normalization
  * Remove outside areas
* Data Augmentation
  * Rotations / mirroring
* Data Format
  * One-hot encoding
  * .NET vs Python (show sides here?)
    * numpy fast, python not
* Binary Classification
  * Show model summary
  * Concept of reshaping data
  * Tiledriver vs User Maps
  * Good results!
* Variational Autoencoders
  * What is an encoder?
    * BMP vs JPG
  * What is an autoencoder?
    * Show examples here (noise reduction)
  * What is a variational autoencoder?
    * Show examples here
  * show miserable, miserable results
    * Wolf 3D maps are not continous!!
* DCGANs
  * Generator vs Discriminator
  * Show cool examples
  * mention it is horribly hard to train
  * Show miserable results
  * Doom DCGAN
* RNN / LSTM
  * Concept
  * Show cool results
  * SLOW
  * Show miserable results
  * Potential with 2D context
* Conclusion
  * Generative Models will be super cool
    * Creating music
    * Creating scripts
    * Creating... movies?
  * Future Work
    * See if the GAN can be raised to the level of mediocrity
    * 2D RNN
