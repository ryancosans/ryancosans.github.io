---
layout: post
title:  "building a game"
date:   2021-11-02 06:44:00 +0000
categories: tech
---

### TLDR

I built a game called yavalath, you can play it at [https://play.yavalath.com/](https://play.yavalath.com/). It was annoying as hell but I’ve got the hang of it and now I’m working on something else a little more exciting.

### Intro

A while back I decided to make a game, sort of. I decided to adapt a board game created by Cameron Browne's Ludi program but considering there were no good versions that could be played online and locally I had to do something about it.

### There was an Attempt

So I got to work. First I decided that to tackle this I’d first need to be able to make a load of hexagons and then arrange them in some way. Using CSS was my first choice, specifically styled components and React, use some simple JS to loop, do some maths for positioning and use CSS to make some divs look like hexagons and bob's your uncle. 

So all the hexagons were displayed nicely looking good, I added some state so we could make moves and swap colours for the two players and we could play. And then we noticed the first issue. Well turns out you can round the edge of a div but it’s  still a square. This meant when you clicked the in the corners of one of the hexagons it could accidentally click the wrong one. Well that’s what we call a game breaking bug.

### Out with CSS in with canvas

Ok so that didn’t work but luckily canvas exists so that was the next stop. Until this point I’d never worked with canvas before so this was going to be a bit of a learning curve. Turns out that curve is gradual and kinda fun. Before I knew it I was drawing stuff on a canvas and it was time to apply the same logic to the canvas. This is where I had to change my tact a little. It’s easy to forget when working with React that the DOM is essentially your model, and the only thing you have to keep track of is state (and some other things). But with canvas you need to keep a tidy model that is then reflected similar to state, but also state.

This came with all the benefits I wanted, simple to draw and define shapes for when there’s a mouse event and before I knew it I had a working game… again. This time it didn’t have that game breaking bug and instead had a drawing engine. I'm being liberal with the word engine but hell it sounds good plus it has the word apothem in it so it must be an engine.

What I’ve learned from my time using canvas is that it’s really powerful for graphical things like drawing “complex” shapes and arranging things that would otherwise be difficult but absolutely pointless in trying to use it for simple things that the regular old DOM is better at, buttons for example. A mistake I’ve seen a lot of sites make and only really understood properly since messing with canvas, and one that I nearly made as well, was to write the entire page in a canvas. In reality there’s no need and in fact makes the experience worse. My basic analogy for this is imagine a remote control for a TV, it’s hardware that controls software because the physical feedback and simplicity is something that’s relatively hard to replicate in software. It’s the same for canvas.

### Let’s Play Online

The next step was getting this to work so we could play online. That way I could play with my friends without being in the same room and we could enjoy Yavalath.

This was pretty simple: whip up a table in dynamo, since I figured all the data I would need is some sort of identifier and the board data itself, whose turn it is etc. Little bit of node and an aws SDK later data was being persisted and everything and we could play online. With one significant hiccup, we had to refresh any time either one of us made a move, which kinda sucked. Even when we were on Discord with each other we’d have to tell each other when we’d taken our turn to prompt the other person to refresh. This wouldn’t do at all. A few Socket.IO docs later and I had something working that would admit people to a “room” and could push the changes to the board to all of the people in that room. 

This is where I had to make some hard design decisions. My initial thoughts were to restrict each game to only two players and block entry or a turn being taken until the other player had taken theirs. This means I’d need some slightly different logic for games played locally and perhaps not even store the games. I decided instead to do none of this. My thoughts were anyone that is going to play this will have a person they are going to share the link with or someone next to them to play locally. Restricting the game in this way would stop being able to pick up a game after playing the first half locally or being able to revisit games that had been played. Sometimes the easiest solution is the best one.

### Oops I won

One of the big missing pieces that needed to be added for it to be a full game was a win and loss detector. Having to keep track of winning and losing yourself is easy enough with a game like Yavalath but occasionally you can miss an accidental loss and so can your opponent, meaning the game continues pointlessly. So the only thing to do was add a win condition. 

So it’s time to add another engine, this time a game engine. Detecting if you’ve won or lost in a game like Yavalath seems simple until you realise how many checks you have to perform. The strategy I started with was to start with the hexagon that was just pressed, find the hexagons that are the same colour not including the current one then try and find out if any of them are in a row by checking each one. This turned out to be really inefficient due to the amount of pointless checks I’d end up doing.

This was kinda complicated and really didn’t solve any problem so instead I decided to assign each neighbouring hexagon of the same colour a direction flag and then check in the same direction for each neighbouring hexagon. I wrote some bad code debugged a little (thank fuck for unit tests) and thought outloud to my best bud and also great dev Nick. And it worked, I had a working example of detecting wins and losses until I realised after writing a load more unit tests that this strategy would work until we got just past half way.

Due to the layout of the board the indexes that would be in one direction in the first half would be in a different direction in the second half. Enter Nick with the big brain idea. He pointed out that we wouldn’t need to add logic that flips directions but instead just add a bunch of fake hexagons to the board to ensure the indexes remain the same. This worked perfectly and solved the last problem. I thanked him by sticking his name at the bottom of the site as a contributor and I will buy him a beer at some point.

### Finishing up

I decided that there’s a lot of other things that I could achieve with this project. I could optimise the win loss, add comprehensive acceptance tests and adapt it to work properly on mobile. Sorry to everyone this only works on desktop, but I achieved what I set out to achieve, learnt a lot in the process and that’s where I wanted it to end. Now it’s on to the next game.

Note: the design is very much inspired by the hit netflix show Squid Game, which is the reason for all the attempted korean translations and colour theme. Not to mention it kinda feels like an arcade which I like. I guess there was room for CSS after all.
