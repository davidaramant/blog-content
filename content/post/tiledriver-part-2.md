---
title: "Making Caves with Cellular Automata"
draft: false
date: 2018-05-14
tags: ["Tiledriver","Cellular Automata"]
description: "Background info about Wolfenstein 3D, ECWolf, etc"
---

As we've seen in the [introduction to Tiledriver](/post/tiledriver-part-1) there are lots of ways to procedurally create boxy man-made layouts, but what about something more organic?  How would you make a level of a cave?  

It turns out we can use cellular automata to do this fairly easily.  Remember Conway's Game of Life?

![Conway's Game of Life](/tiledriver/gameoflife.gif)
<center>*Gosper's Glider Gun in Conway's Game of Life*</center>

The concept of [cellular automata](https://en.wikipedia.org/wiki/Cellular_automaton) is pretty simple - you have a grid of cells that are either alive of dead (typically represented as black and white pixels).  There are rules defined that will cause cells to either die or spring to life in the next generation of the board depending on how many of its neighbors are living.  [Conway's Game of Life](https://en.wikipedia.org/wiki/Conway's_Game_of_Life) is merely a particular set of rules that are fairly well known.

![Cell Neighborhoods](/tiledriver/cell-neighborhood.png)
<center>*Most cellular automata use the Moore neighborhood*</center>

### Using cellular automata to make a cave

So, how can we use this to make a cave?  Well, first of all, a Wolfenstein 3D map is also conveniently a big grid of tiles.  Let's start by saying that walls are "alive" and empty space is "dead" and generate a random board:

![Generation 1 - Random Board](/tiledriver/ca-gen1.png)
<center>*Generation 1, a random board*</center>

In the above picture, the rock is black and the empty space is white.  Yes, there is a three-tile wide border of rock on the outside.  We'll come back to that, I promise.

Lets use the following rules for generating a new version of the board:

* If a tile has less than 5 neighbors that are "alive" (rock), it "dies" and becomes empty space
* If it *has* 5 or more living neighbors, it becomes "alive" (rock)

Lets see what happens after a few generations of this!

![Animation of generations](/tiledriver/ca-generations-animation.gif)
<center>*Generations 1 through 7*</center>

Now we're getting somewhere!

As you can see, further generations end up smoothing out the roughness and forming a blobs of playable space.  Remember that three-tile border?  Without it, the empty area "grabs" the edges.  This doesn't work real well for our purpose since we want and enclosed area (remember that white is playable space in the below image!).

![Generation 7](/tiledriver/ca-gen7-no-border.png)
<center>*7 generations with no starting border.  Pretty open.*</center>

### Making a Cave

OK, so we've managed to create some cave-y spaces, but what are we going to do about them not being connected?  At first I thought there would be an easy way to connect all of them, but, unfortunately, this is one of those massively hard CS problems... so we'll cheat instead.

First we'll find all the empty areas using a [connected-component labeling algorithm](https://en.wikipedia.org/wiki/Connected-component_labeling), picking a different color to easily visualize each of them:

![Colored Empty Spaces](/tiledriver/ca-gen7-playable-spaces.png)
<center>*All of the empty spaces in unique colors*</center>

Next, we'll just... delete all but the largest one:

![Largest Remaining Cave](/tiledriver/ca-gen7-only-largest-room.png)
<center>*Hey, it's a cave!*</center>

Now, there's a lot of trial and error in the above.  The percentage of live vs dead cells in the initial board, the number of generations, the border thickness, and the size of the board all play a large role.  We first used a standard Wolf 3D map size of 64x64, but quickly discovered it was too cramped to make interesting structures (as it turns out, a map tile in Wolf 3D is fairly coarse).

But after adding a random player start and using some rock textures, we have a cave!

![Boring Cave](/tiledriver/ca-boring-cave.png)
<center>*An incredibly exhilarating cave*</center>

### Making an *Exciting* Cave

OK, OK, so we have a really *boring* cave.  Lets make it more exciting...

Lets add some random stalagmites & stalactites and some treasure in the nooks and crannies (finding nooks is pretty easy, just search for tile empty tile spots surrounded by a few walls):

![Slightly More Interesting Cave](/tiledriver/ca-less-boring-cave.png)
<center>*Well, there's stuff in it now...*</center>

The main problem now is that the cave is incredibly... flat.  Since Wolf 3D has no lighting system whatsoever, everything kind of visually blurs together in a samey blob.

But hey, we're using ECWolf where it's trivial to add stuff like new textures, so lets cheat again!

First, lets create a bunch of texture variations for different light levels using [Image Magick](https://www.imagemagick.org/script/index.php):

![Wall Texture Light Variations](/tiledriver/wall-texture-variations.png)
<center>*All the wall variations*</center>

The above is all of the walls, but there's a similar set for the floor texture.  Next, we'll randomly place a scattering of light sources around the cave.

Our simplistic fake lighting system will be as follows:

* For each light source, try to shoot a ray out to each empty tile in a square surrounding the light (we'll use a square because it makes the loops easier).
* If there's a wall in between the light and the destination tile, do nothing
* If there's free line of sight, increment the light level for that tile based on how far away it is from the light

Since we have 30 different textures, we have 30 light levels.  Based on the light level we have computed for the tile space, we just pick a different set of textures to represent it being brighter.  The higher brightness levels can only be achieved if multiple light sources interact.

Turns out this looks pretty good!

![Cave with Lights](/tiledriver/ca-lights.png)
<center>*Let there be light!*</center>

In cases were the geometry blocks the light, you get some endearingly jagged shadows:

![Dramatic Shadows in Cave](/tiledriver/ca-dramatic-shadows.png)
<center>*Incredibly realistic shadows!*</center>

Of course, this lighting is 100% fake - it's just using different pre-defined textures.  The light doesn't change when you're playing the game and objects moving around (like an enemy solider) would not get brighter or darker depending on how close they are to a light.

TODO: Conclusion of some sort