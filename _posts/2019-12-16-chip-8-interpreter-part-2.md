---
layout: post
title: "Tackling a CHIP-8 Interpreter, Part 2"
summary: "Setting Up our Display Window"
featured-img: chip8libgdxsetup
---

# Not Quite Rendering Just Yet...

Well reading a byte file is painfully easy so now we're onto the next step, making a display window. Why am I doing this next? Well we will eventually want to be looping through everything in execution units. We will have different timers and all sorts of different parts working together. So I personally would like to get our basic structure of execution out of the way now. I will be using [LibGDX](https://github.com/libgdx/libgdx) since it's easy to use, is open source and works well for our needs. 

I won't really be covering how to set up LibGDX but I would suggest using IntelliJ as they play very nicely together. You should be able to follow the *Getting Started* section in the link above to set this up painlessly. For reference I have included a screenshot of my setup settings below. You'll notice that we won't need to have any of the checkboxes checked besides the desktop sub project. If you want to add any other sub projects I say go for it but that isn't something I'm targetting. 

![alt text](https://i.imgur.com/9SIgDnQ.jpg "LibGDX Settings")

Now that the project is created we can go into the desktop package to actually be able to run this. Go ahead and run this right away to ensure there aren't any errors. You should see a basic window with a red background and a 2D Sprite.

# Lifecycle of LibGDX

So this should be pretty self explanatory, I don't expect to do anything too crazy here but we've got three basic methods here. Create: This is ran on startup, basically it will be used to load our ROM. Render: This is called every time we display a frame. Dispose: This is called on application close. 

Wait a second... where's the main loop then? Since Render is only called on render is that what we are going to use to emulate each cycle? Wouldn't a variable framerate change this?

What a fantastic observation!! We will be getting around to that next time. Here's a [hint](https://github.com/AfBu/haxe-CHIP-8-emulator/wiki) though. We'll be looking to achieve a 500 Hz CPU speed.