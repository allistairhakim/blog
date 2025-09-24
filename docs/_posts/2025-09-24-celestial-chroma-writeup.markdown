---
layout: post
title:  "CSAW Quals 2025: Celestial Chroma Writeup"
date:   2025-09-24 12:30:09 -0400
categories: osint
---
![Poster of Fantastic Planet]({{ '/assets/img/fantastic-planet.jpeg' | relative_url }})
# Introduction
`Celestial Chroma` was an OSINT that looked like an art riddle at first glance. The prompt talked about primary colors taken over time, a humanoid alien race on a virtual planet, and a puppet-play style — all of which pointed at a film with a very specific visual identity. The job was to turn those hints into a film, the film into a technique, the technique into a filename, and the filename into the flag.

All they gave was the riddle description, and an image file:
![Challenge Description]({{ '/assets/img/celestial-chroma-0.png' | relative_url }})
![Barcode]({{ '/assets/img/barcode.png' | relative_url }})

# Preliminary Testing
The images showed *a lot* of cyan, magenta, yellow and black — basically CMYK — so I started by splitting the provided image into separate channels to see whether any one channel contained a barcode-like signal. I tried the following:

- channel splits (C, M, Y, K) and visual inspection  
- contrast stretching, inversion, and per-channel thresholding  
- simple LSB/LSB-like stego checks and metadata inspection

No obvious text or URL was revealed, so after discussing with my teammates and reading the riddle more carefully, we determined the stripes was likely a "movie bar code", an image where they take each frame of the movie, average out the color, and make a vertical stripe of that color. This results in a very interesting image where you can see the color pallete of the movie.

# Gathering Info
WE bounced ideas around — some thought *avatar* at first (green-ish), others pointed to *cutout animation* and *puppet play*. The riddle’s language hint (“in a language, it bears the name they bestow”) pushed me towards a stop motion alien movie. After some consultion with ChatGPT, I determined a likely candidate was *Fantastic Planet / La Planète sauvage*.

Jayden confirmed this by downloading the movie (legally, it was an old movie) and creating a script to create a movie bar code of his own, which was pretty cool.
![custom barcode]({{ '/assets/img/celestial-chroma-2.png' | relative_url }})

Once we identified the movie, we moved on to the next parts of the riddle.

# Trying stuff out
Since the flag needed to contain the name of the technique, we brainstormed multiple variants of the technique name: `papiers-decoupes`, `papier decoupe`, `cutout-animation`, `paper-cutout`, `stop-motion`.

We originally were almost sure it was `papiers-decoupes`, but later the description update clarified that the technique-name was in English. Jens suggested `stop-motion`, so I focused on trying flags with that technique along with some others.

# Breakthrough
Remembering the riddle line:

> “On the virtual abode describing where the film was made, an image file of this style, a secret we owed. In a language, it bears the name they bestow, to this art form, so intricate and slow.”

I tried many, many images from different wikis and articles about the movie. Txaber suggested we shouldn't look at websites about the movie, but rather the studio. I searched pages about the film’s production (Jiří Trnka / Krátký Film Studio) and inspected images on those pages. One studio catalogue contained an image in the style of stop-motion/puppet work with a filename that fit perfectly: `loutka.jpg`.

![Image of Website]({{ '/assets/img/celestial-chroma-1.png' | relative_url }})

That made sense on every axis:
- `Fantastic Planet` / Jiří Trnka Studio → Prague production links.
- `loutka` = puppet (Czech), which matches the puppet/stop-motion/paper-cutout hints.
- the site image was in the same visual style as the puzzle’s barcode-like artwork.

I tried `stop-motion` and `loutka.jpg` and the flag was verified! Overall, felt like a very guessy `osint`, but it was a nice break from `web` challenges and was quite fun.

# Final Flag
**Flag: csawctf{stop-motion_loutka.jpg}**