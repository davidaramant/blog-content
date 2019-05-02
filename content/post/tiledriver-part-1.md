---
title: "Introducing Tiledriver: Fun with Wolfenstein 3D"
draft: false
date: 2018-05-14
tags: ["Tiledriver"]
description: "Background info about Wolfenstein 3D, ECWolf, etc"
---

A few years ago at a [Hackathon weekend](https://www.sep.com/labs/hackathon) at my company I decided to try writing a level generator for the game Wolfenstein 3D.  I had done zero research on level generator going into the weekend, but with the help of two colleagues we managed to pull it off!

Were these levels good?  **No.** No, they were not.  We had to take a lot of shortcuts to get something working with the limited time available.  After it was over I kept thinking of how the generator could be improved, but every idea seemed like a _lot_ of work...

More important than the mediocre generator we wrote was the general framework I had built up to deal with Wolfenstein 3D - [Tiledriver](https://github.com/davidaramant/tiledriver). This series will explore some of things I've done with it and why Wolf 3D is a great playground for a lot of techniques.

But wait, what's Wolfenstein 3D?

## Wolfenstein 3D

![Wolfenstein 3D title screen](/tiledriver/wolf3d-title.png)

Wolfenstein 3D, the hottest game of 1992!  Wolf 3D tells the heartwarming and completely historically accurate tale of how a single allied prisoner of war single-handedly shot about a billion Nazis and eventually took down Mecha-Hitler.

![Wolfenstein 3D gameplay collage](/tiledriver/wolf3d-gameplay-collage.png)

Wolf 3D has an extremely simple level format.  Each map is a 64x64 grid with each space (known as a "tile") being either a wall, an enemy, a decoration, empty space, etc.  There is no height variation whatsoever.  

![Wolfenstein 3D example map overview](/tiledriver/wolf3d-e1l1-overview.png)
<center>*Wolf 3D's first map as seen in a level editor*</center>

Since the level format is so simple, Wolf 3D is an ideal starting point for exploring procedural level generation.  With a later game like Quake or even Doom, the level format itself has a lot of rules for what determines a valid map since those games use levels composed of geometric shapes.  With Wolf 3D, almost any combinations of blocks is a level (whether or not you can _play_ it is a different matter).

### Enter ECWolf

Wolfenstein 3D as released in 1992 was a completely self-contained game.  There was no mechanism built into it for loading user-created content: you bought the game, that was it.  However, fans quickly managed to reverse engineer various formats and were soon able to hack their own levels into the game.  Once the [source code for the game engine](https://github.com/id-Software/wolf3d) was released in 1995 it really opened a flood gate of possibilities for user-made content.

![Wolf 3D Open Source Overview](/tiledriver/wolf3d-open-source-overview.png)
<center>*ECWolf's relation to Wolfenstein 3D*</center>

The ultimate result of this is the [ECWolf](http://maniacsvault.net/ecwolf/) project.  In addition to quality-of-life improvements like running on modern operating systems and supporting higher resolutions, a key focus of ECWolf is making everything as data-driven as possible.  The most relevant aspect of this for my purposes is the [Universal Wolfenstein Map Format](http://maniacsvault.net/ecwolf/wiki/Universal_Wolfenstein_Map_Format), or *UWMF* for short.  Maps can now be defined in a flexible text format instead of having to deal with a quarter-century old binary format.  UWMF also removes some of the arcane limitations of the older format.  For example, maps are no longer required to be exactly 64x64 in size - smaller or larger (within reason) levels can just as easily be created.

Unfortunately, Wolf 3D is not nearly as popular as its younger brother Doom, and the editing scene has been somewhat slow to embrace ECWolf.  There _are_ a few mapping tools in semi-active development for Wolf 3D, but none of them support UWMF yet.  So, I had to support this format from scratch.

## Procedural Level Generation

Procedurally generating levels is a massive topic which I won't try to completely cover here.   For a greater look at that fascinating subject I highly recommend the free online textbook *[Procedural Content Generation in Games](http://pcgbook.com/)*.  After reading more about the theory, what we implemented in the first version of Tiledriver turned out to be a simplistic agent-based system.  Take a look at a map that it spit out to see if you can guess how it was made:

![Tiledriver 1 Map](/tiledriver/tiledriver-1-map.png)
<center>*A map created from the original weekend Tiledriver effort*</center>

If you guessed "try to recursively add rectangles on all four sides," congratulations, you guessed right!  We did some trivial theming of the level, but overall, it's only capable of making incredibly same-y, boxy levels with absolutely no overall sense of gameplay progression.