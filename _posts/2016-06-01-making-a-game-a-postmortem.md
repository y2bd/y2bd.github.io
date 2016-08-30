---
layout: post
title: "Making a Game: a Postmortem"
categories: [graphics, school]
---

_Note: This post was originally featured on [Excessive Furniture](http://cse125.ucsd.edu/2016/cse125g1/postmortem), a blog for my final project course at UCSD._

We did it!

It’s been an extremely long ten weeks. We all spent most of our time on this project (to the point where I personally didn’t go to any of my other classes’ lectures!), and even though we didn’t end up having to pull any all-nighters (save for Dexter, but those were totally of his own free will) we obviously put a lot of effort into things. {%sidenote Jeff Jeff has considerably more lines than us because he committed his 167 code to the repo. Elton doesn’t have many lines because he forgot to commit with his Github email :( %}

![](http://cse125.ucsd.edu/2016/cse125g1/assets/images/pm/github.png)

Now that everything has been finalized, let’s talk about the ten weeks in detail.

<!--more-->

## Overall Impressions

_Q: What game did you seek out to make? What did you end up with?_

At its core, we wanted to make a cooperative puzzle game. We’re all big puzzle game fans, and furthermore we’re all big game _design_ nerds. A lot of the games that come out of CSE125 are competitive, usually in the arena shooter/brawler vein. While competitive shooters certainly do require game design (I’m not about to tell you that Quake isn’t a brilliantly designed game) there are a lot of givens and defaults just because of the genre.

A puzzle game however has almost no prior assumptions in terms of mechanics. Furthermore, a puzzle game is all about the _puzzles_, which require careful design even outside of the mechanics of the game. We knew a game like this would be difficult to make in the given timeframe, and that we’d probably end up spending a much greater portion of our time on design vs. implementation compared to the other groups. I definitely don’t regret that decision.

Unfortunately, time does catch up. A big part of our game (it’s about robots escaping from prison by the way) was supposed to be the competitive aspect. Not all of the robots were going to escape the prison ship, and we wanted puzzles that while forcing the players to cooperate would keep them on their toes, knowing that at any time any other player can just “accidentally” push the wrong button and open the trap door rather than the real one. Due to the time restraints of the class, we weren’t able to explore this idea, at least not the point that we would be satisfied by. We ended up dropping the entire competitive aspect around Week 8 and focused on making it a strictly-cooperative puzzle game.

The design goals we had during the competitive-phase of our game production kept on through the entire project though. Each puzzle was designed and sequenced in a specific way to force certain player interactions: the prison puzzle cannot be solved if _any_ player doesn’t cooperate. The corridor puzzle teaches players that even if they can split up, they shouldn’t necessarily. The keycards teach the players that the puzzles are connected, even if the individual rooms are separate. When you have four players, you have four actors that you have to control simultaneously, and I feel we did a good job of it.

_Q: What changed when it came to implementation expectations?_

We actually implemented a lot of the crazy technical aspects we strove for. From SSAO in week three to octree-based collisions (we had ramps! Ask anyone who’s ever written a game, and they will tell you ramps are impossible), tech wasn’t often a problem for us. We even explored audio much more than I had ever expected. We did however struggle more with particle effects and animations than we expected, and ended up having to drop both.

_Q: Did things go to schedule?_

We’ve joked a lot in the prior blog posts about how even though we kept feeling that we were falling behind, we would just check out schedule and find out that we were on track! We definitely underestimated ourselves in many regards when making the schedule.

The big block, one that you’d see coming if you read any of the prior posts, was collision detection. We strove from the beginning to do collision the “right” way, and that cost us dearly. Not only did implementing the octree system take much longer than we expected (we didn’t “stop” working on collision until week 8!!) but it cost us in other regards. Implementing and testing puzzles is surprisingly difficult when you can’t walk into things!

We might have underestimated ourselves, but we surely overestimated Blender. Blender is _not_ a level editor, and even with our _ActivatorRegistrator_{% sidenote ar © 2016 Jason Lo, Excessive Furniture. %} system, we ended up fighting Blender much more than we should have. There really wasn’t an alternative though—a puzzle game requires some sort of level editor, and making a level editor is essentially making a game in itself.

## Implementation

_Q: What methodologies did you use? Which ones worked well?_

We didn’t use any particular development methodologies when making the game, no Kanban boards or user experiences or anything like that.

When it came to development, we did stick to a Git Flow-esque{% sidenote gitflow [http://nvie.com/posts/a-successful-git-branching-model/](http://nvie.com/posts/a-successful-git-branching-model/) %} workflow where we had `master` and `develop` branches that housed most of the code, and had independent feature branches that would split off and merge back into `develop` as required. We rarely ran into merge errors, due to use preferring a rebase-based workflow, causing us to deal with merge conflicts up front rather than pushing them nervously to the future.

We used Dropbox for asset management, which actually turned out to be much more of a hassle than we expected?? We routinely ran into syncing issues, and at some points Dropbox would serve certain team members us older versions of files that we had uploaded, while giving newer versions to others. Very strange stuff.

_Q: What difficulties did you have in implementation? Did anything turn out easier than you expected?_

We’ve gone to lengths explaining how difficult collision was for us. We strove to do it the “right” way, with an octree-based system that would separate our space into octants, allowing us to only consider collisions and raycasting in a local space, rather than across all colliders in our world (of which there were quite a few). It turned out absolutely fabulous though (ramps!!wow) so it’s not as if it wasn’t effort well-spent.

Networking also turned out to be quite difficult, even more difficult than we had imagined going in. We decided to roll our own TCP stack rather than using a library, so we had expected difficulties, but we ran into problems much more frightening than the lag and desync we were expecting. We ran into issues with some clients refusing packets after a certain period of time (with no TCP errors thrown), we ran into issues with a single client stuttering and cause all other clients to follow suit, and we even ran into a bug we deem the “Sanic” bug, where randomly on start all of the clients will run at superspeed for 10-15 seconds. That bug we never figured out how to fix…

Sound was our saving grace. We used the popular FMOD library for sound, and it works extremely well{% sidenote fmod After you read the documentation first… %}. No qualms, it just worked. One of our team members, as well as one our friends, even went on to implement FMOD-based sound in their 167 projects by our suggestions, and were very happy with the results.

_Q: Time to brag: what parts of the implementation were you really proud of?_

You don’t have to look at our game for very long to know it looks good. We had resident graphical wiz-kid Jeffrey Johnson on our team, and that decision really paid off. Besides literally doing all of the art, Jeff did most of the graphics programming, giving us real-time point light shadows, screen space ambient occlusion, and even IES-profile simulation for complex-shaped lights. It even runs relatively smoothly{% sidenote ubisoft Take that, Ubisoft! %}.

We’re also really happy with the network _infrastructure_ (not the TCP code driving it all, mind you). All stateful objects, whether it be transforms, mesh data, or even sounds, in our game essentially send their state on changes across the network with no developer interaction. That means that you can write game code as if you were making a single player game, and you got multiplayer for free! We paid a rather high up-front cost getting that to work (Jeff also had to yell at me quite a bit before I agreed to implement it) but I’m so happy with how it turned out. Writing game code is a breeze when you don’t have to worry about sharing state over the network.

_Q: Did you use anything besides the expected C++?_

Not really, our game was written with C++ and OpenGL. We did however use GLSL a lot for shaders. The only advice we can offer for that is NVIDIA Nsight is greatly recommend for shader debugging, because you’re going to have a lot of fun debugging shaders without such a tool.

_Q: How many lines of code did you write?_

According to Cloc{% sidenote cloc [https://github.com/AlDanial/cloc](https://github.com/AlDanial/cloc) %}, we had 12376 lines of C++ and 824 lines of GLSL. This includes graphical code from the 167 project our game is based on.

_Q: How did you handle media content in your game?_

We used Blender exclusively for our 3D modeling of individual scenes as props, as well as UV unwrapping t. Actual materials were created with the awesome Substance Painter software{% sidenote marriage Jeff has a legally recognized relationship with Substance Painter in over 10 states. %}. Rather that directly attaching materials in Blender, we had a custom configuration-based system that allowed to write material data in `.ini` files, allowing us to change properties such as textures, tints, brightness, and the like on the fly without having to reopen Blender. To get things into our game, we used ASSIMP for model loading, and SOIL2{% sidenote soil2 [https://bitbucket.org/SpartanJ/soil2](https://bitbucket.org/SpartanJ/soil2) %} for material loading.

We also used Blender for stitching together the individual scenes and props, as well as giving them metadata that our ActivatorRegistrator system took advantage above.

For sound, we composed audio with Audacity, and did voice effects with the awesome Melodyne software. We included audio in our game with FMOD.

_Q: Even after all of the above, would you have rather started with a game engine?_

We briefly discussed this among those of us who have experience with the Unity game engine. We estimated that we could probably remake our game in its current state with Unity in about a week, with the only thing we’d need to learn is networking (I hear though that in recent versions Unity networking works really cleanly). Our game engine doesn’t really do many custom things, and having an existing game engine would give us way more time to spend on game design, something our game sorely depends upon.

Of course, we’d have to do things the Unity Way, and any straying from the Unity Way would cause harm. I could see Unity physics not working extremely well for a platforming game, and Unity also wouldn’t necessarily give us as much freedom in the Graphics department as raw OpenGL would (although we would get a lot more effects for free).

_Q: Did you use any major libraries? What are your opinions of them?_

We rolled our own graphics, our own physics, and our own networking. I’m perfectly happy with our graphics, and from what I’ve heard, if you’re not doing physics simulation (which we certainly aren’t, realistic physics make a terrible platformer) then using Bullet will be a struggle, even if it gives us collision for free.

Networking on the other hand is something we’d definitely use a library for. At the end of our day, our underlying networking code just works, and not much more than that. Using a networking library that hundreds of people have worked on gives us reliability and performance. Plus, if we were making an actual game, we’d of course use UDP over TCP to avoid the overheads of TCP, and a networking library might abstract away the technical differences of TCP vs UDP.

_Q: You’ve talked about group dynamics in a game, what about in game development?_

Working in a large group is often a balancing act. If everyone is off in their own worlds, not following anyone, people will get lost and the project will never get done. However, if there is one leader that no one can agree with, then mutiny would happen.

In Elton’s personal experience when working with a 60 person team in CSE 112, they started off with no clear leader or instruction and ended up in the first boat: no real tasks, no clear place to go. But when they turned towards the leaders, nobody really agreed with any of them. Only when a clear leader emerged and everybody’s skills and experience lined up right, did things start rolling.

One thing that we quickly learned is the need to trust the competency of your teammates{% sidenote rocket I wish I could say the same when playing Rocket League… %}. I ended up being the task manager of our group, and therefore had to trust that Bert could indeed get wall-sliding to work, or that Elton could indeed get lights to sync over the network. It’s especially hard to trust people you’ve only met recently, but if you don’t and instead just try to implement everything yourself, you’re just going to run out of time, and drive down morale. Trusting everyone to be competent allows you to most effectively divide up tasks which speeds development up along.

_Q: What would have you done differently if you could do CSE125 all-over again?_

First, I’d find a better to develop on the lab machines! We spent 15-30 minutes setting up every single time due to the computers being wiped. Elton had the right idea, making an external hard drive his development station.

Development-wise, we’d work a lot harder to get something playable right at the front. It wasn’t until Week 8 that we really got to play around with our ideas and see what was viable, which obviously hurts when it comes to proper game design.

We’d also really try to get hold of an artist. Jeff did a great job, of course, but even Jeff’s skills and time are limited. Having dedicated people would have allowed Jeff to do more programming, as well as getting potentially even cooler art-direction that our feeble programmer-minds could have never thought of.

_Q: What UCSD courses do you feel prepared you for CSE125?_

CSE167, obviously. It’s the only OpenGL introductory course you’ll find at UCSD, and even if you’re not knee-deep in Uniform Buffer Objects and Fresnel coefficients, you really should know what the Model, View, and Perspective matrices are, as well as how transformations work, the role of the camera, scene-graphs, all kinds of stuff. CSE167 is also one of the few classes to allow freeform final projects, which will really help you when coming up with a CSE125 project.

CSE124 is almost essential if you want to start networking with sound footing. A lot of our initial networking exploration was just repurposing old CSE124 code.

_Q: What was the most important thing you learned in the class?_

I can’t speak for my group, but the most important thing I learned in this class is that shortcuts hurt. It is way too easy to do things the “easy” way just because it’s the quickest, or not very fun to take seriously. Of course, when you just want to get something on the screen, doing things the easy way is often recommend. But if you only take shortcuts, and you don’t remedy those shortcuts as soon as you can, developing actual structure rather than just hacks upon hacks, your codebase will become very unwieldy very fast.

We put a lot more effort into a proper code than I’ve ever done in any of my prior projects{% sidenote proper Don’t tell my past employers! %}, and even with the outrageously-high upfront costs, I’d do it all again because it made our code _so_ much easier to reason with once we started reaching crunch time and brainpower started dropping. Good code pays for itself.

## Media

Here’s some media we’ve collected of the game. Enjoy!

{% image http://cse125.ucsd.edu/2016/cse125g1/assets/images/pm/s6.png "Game screenshots" 903 665 fw %}

{% image http://cse125.ucsd.edu/2016/cse125g1/assets/images/pm/s7.png "Game screenshots" 903 665 fw %}

{% image http://cse125.ucsd.edu/2016/cse125g1/assets/images/pm/s2.png "Game screenshots" 903 665 fw %}

{% image http://cse125.ucsd.edu/2016/cse125g1/assets/images/pm/s4.png "Game screenshots" 903 665 fw %}

{% image http://cse125.ucsd.edu/2016/cse125g1/assets/images/pm/s1.png "Game screenshots" 903 665 fw %}

{% image http://cse125.ucsd.edu/2016/cse125g1/assets/images/pm/s3.png "Game screenshots" 903 665 fw %}

{% image http://cse125.ucsd.edu/2016/cse125g1/assets/images/pm/s5.png "Game screenshots" 903 665 fw %}

{% image http://cse125.ucsd.edu/2016/cse125g1/assets/images/w9/armory.png "Game screenshots" 903 665 fw %}

{% image http://cse125.ucsd.edu/2016/cse125g1/assets/images/w9/maze.png "Game screenshots" 903 665 fw %}

<iframe width="100%" height="620" src="https://www.youtube.com/embed/6D8eo8RskyI" frameborder="0" allowfullscreen=""></iframe>

## Feedback

_Q: What books do you recommend for learning?_

_Real-Time Rendering_ by Akenine-Moller, Haines, and Hoffman is a book that Jeff recommends time and time again when it comes to all things graphics (Dexter even found use for it when making his collision code!). Thankfully you guys provide the second edition of the book.

Not a book, but [learnopengl.com](//learnopengl.com) is an absolutely fantastic resource for anyone getting up to speed with modern OpenGL programming. It teaches everything you need to know (and then some!) while remaining completely approachable.

_Q: What tips do you have for future students?_

*   Start early. No, really. this isn’t CSE131, you can’t do the assignments the day before they’re due.
*   Establish some sort of methods of communication right from the beginning. We used Slack.
*   Decide on some sort of source control. Make sure everyone is competent with it, and make sure to establish some sort of etiquette on how it should be used.
*   Always communicate what you’re doing. The worst feeling is when you’re halfway working on something and then find out your teammate already did it.
*   Make sure everyone is doing _something_ at all times. Even if it’s just looking through code to make sure they understand how it works, people should always be busy. If someone’s not busy, then
    *   Someone else is _too_ busy and tasks need to be split up
    *   You forgot something and are therefore not planning well
    *   Someone is blocking the project and that needs to be resolved ASAP (maybe you can help!!)

_Q: Any final feedback?_

For me (and the others I’m sure), CSE 125 is a course that I’ve been waiting to take since my freshman year at UCSD, and I’m so happy that I’ve been able to be a part of it with my group members. So definitely don’t stop doing it! I want to come by next year and watch next year’s presentation!

One of the biggest blocks for us was the work environment. From the computers being reset on log-off (we’ve lost work a couple of times due to that), to non-CSE 125 students taking space in the lab even though it’s reserved, to the darn department destroying the hallway and making all foot traffic go through us, B220 had to have been my least favorite part of CSE 125\. Obviously the latter problem was a situation specific to us, but the rest were actively bothersome. The problem though is I can’t really see how this would be resolved—it honestly seems like things were handled the best that they could.

Obviously such things can’t be forced, but it would be great if more groups in the future made games that didn’t occupy the competitive arena space. We were quite surprised when we found out that we were the first cooperative puzzler team! It would be great if more teams tried super-wacky ideas (not to put down the other teams; from the team-based naval game to capture-the-farm I really loved the other games this year). Perhaps we could get a speaker to come and talk about game design? Not having experience with/time for game design seems to be the biggest reason why wacky games might not be coming out of CSE 125.

_Q: Anything else?_

Thank you Professor Voelker and Ruiqing for the best closing of our time at UCSD that we could have ever wished for!
