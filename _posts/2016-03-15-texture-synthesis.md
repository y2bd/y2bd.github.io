---
layout: post
title: "Texture Synthesis"
categories: [graphics, school]
---

*Note: This post was originally featured on [Graphical Blues](https://medium.com/@graphicalblues), a blog for my Advanced Graphics course*

For my final project, I worked on implementing image quilting — generating
larger versions of smaller images that approximate the textures of the original
image. Below are some of the original textures that’ll we be trying to expand.

{% image /assets/posts/texture/original.png "Original textures" 931 632 fw %}

The general algorithm works best with images that already have the general
texture “appearance”. As we’ll see later, it doesn’t work very well on
non-textural images.

<!--more-->

### Random Patches

{% image /assets/posts/texture/random.png "Random transfers" 926 309 fw %}

This first batch of attempts is done by randomly selecting blocks from the
original image and then assorting them into a grid pattern. It’s understandably
simple to implement, and while not *horrible* it’s certainly not the best we can
do.

### Overlaps

{% image /assets/posts/texture/overlap.png "Overlapping transfers" 773 775 fw %}

Again, we choose chunks from the original image and then lay them on top of each
other to form the final generation. However, we don’t choose randomly. Instead,
when deciding on adjacent chunks we sample blocks that have some degree of
overlap, where overlap is defined as adjacent edges (left+right or top+bottom)
having similar pixels when overlaid upon each other by some user-specified
amount.

We sample all possible chunks from the original image and then find the chunks
that when overlaid have the smallest difference from the texture we’ve built up
so far. In my particular implementation I calculate the difference of two image
regions (the *error*) through the difference in luminosity.

I also added some randomness to my selection algorithm — I found that often my
algorithm would find pairs of blocks that overlapped well on both edges and
looped them, which caused unpleasant repetition within the texture. Randomly
discarding possible candidates helped alleviate this.

This ends up looking much better than the random selection. However, it is near
impossible to always find good overlaps, so edge seams are still visible to an
extent.

### Minimum Cut

{% image /assets/posts/texture/minimum.png "Minimum cut" 927 775 fw %}

The core algorithm here is extremely similar to the above. The only difference
is how we handle the overlapping regions.

In the prior section, we overlaid images left to right, top to bottom, giving
chunks priority in that scan order (the image to the right of another would
occupy the entire overlap space, causing the rectangular seam).

In this case, we instead look to find the best possible edge (a ragged edge)
that separates adjacent chunks. We do this by calculating the squared difference
between overlaid images (again, using luminosity), and create an image
representation from that. Then we walk along the image, doing our best to stay
on pixels with minimum error (that in our image representation will be
represented as darker pixels). We then split along that bound.

{% image https://cdn-images-1.medium.com/max/800/1*WMhHRwTTJp8xeeq4QDROOw.png "Purple cut lines" 800 800 fw %}

The purple lines in the image above demonstrate the cut points.

Something to note is that cell size and overlap size (as said above), need to be
manually tuned on a per-texture basis. Here’s an example of trying to generate
textures given a checkerboard patter with a cell size that doesn’t align with
the checkerboard size:

{% image https://cdn-images-1.medium.com/max/800/1*EtX1GATgGZOrQFqmMG0NJQ.png "Cell size mismatch" 800 800 fw %}

We do get a decent texture out of it, but the differing cell size doesn’t
properly preserve the original texture of the image.

{% image https://cdn-images-1.medium.com/max/800/1*4I6qrTiJUjCIrXytnxJU_w.png "Bad sampler" 800 800 fw %}

{% image https://cdn-images-1.medium.com/max/800/1*fr3O5HcJe4uU4-MTIELSsA.png "Even more bad sampling" 800 800 fw %}

Just for fun, as we said before, the general algorithm only works on images that
already represent “textures” in some way. Trying them on portraits for example
gives (un)expected results.

### Texture Transfer

{% image /assets/posts/texture/transfer.png "Texture Transfer" 929 932 fw %}

We can repurpose the algorithms above to add the ability to transfer textures
between images.

The way we do this is to keep generation tied to a source image while referring
to a texture. When selecting blocks from the texture, besides our edge-overlap
heuristic, we also detect how similar that chunk is to the relevant chunk from
the source image.

{% image /assets/posts/texture/badtransfer.png "Bad samples" 927 331 fw %}

Note that this still requires proper selection of cell size, as well as a bias
property that decides whether we weigh texture continuity or image relevance
higher. Trying to map the leaves picture above to a portrait with too small of a
cell size and too high of an image relevance produces a matching texture, but
one that doesn’t properly represent the original texture.

### Sources

[http://www.eecs.berkeley.edu/Research/Projects/CS/vision/papers/efros-siggraph01.pdf](http://www.eecs.berkeley.edu/Research/Projects/CS/vision/papers/efros-siggraph01.pdf)

[http://web.engr.illinois.edu/~vrgsslv2/cs498dwh/proj2/](http://web.engr.illinois.edu/~vrgsslv2/cs498dwh/proj2/)
