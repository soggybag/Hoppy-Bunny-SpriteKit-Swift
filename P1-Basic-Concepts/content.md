---
title: Basic concepts of a side scroller
slug: basic-concepts
---

If you have never built a side scroller before, this introduction will help you understand the basic concepts. You will be tackling this by constructing an obstacle treadmill.  The player effectively runs on the spot and must navigate the obstacles coming towards them by moving up or down.

You will learn to implement:

- A physics controlled bunny
- Dynamic obstacle object generation and removal
- Animation and actions
- World scrolling
- Implement physics based collision detection
- Your own custom button class

#Create a new project

Time to get started and make a new SpriteKit project in Xcode and get started with Hoppy Bunny!

> [action]
> Your first step is to create a new SpriteKit Xcode project, open Xcode and select `Create a new Xcode project`. Then select `iOS > Application > Game` as shown below:
>
> ![Select New game project](../Tutorial-Images/xcode_new_project.png)

##Adding artwork

> [action]
> After the project is created, you should [download our art pack for this game](https://github.com/MakeSchool-Tutorials/Hoppy-Bunny-SpriteKit-Swift/raw/master/assets.zip).
> Next you will add the art pack you just downloaded to your Xcode project by first unpacking the archive.
> Select the `Assets.xcassets` folder in Xcode project navigator, then drag the downloaded assets into Xcode as shown below:
>
> ![Dragging assets into project](../Tutorial-Images/xcode_add_artwork.png)

##Asset Scale

This would be good time to touch on the subject of asset scale. You may have noticed when you select an asset in **Assets.xcassets** you are presented with *1x 2x 3x Scale* options.  The assets you added have the `@2x` suffix so they are designed for 2x retina use.

Have a look at this handy [Device resolution guide](http://www.paintcodeapp.com/news/ultimate-guide-to-iphone-resolutions)

#Summary
Great, you've setup a basic project and added the artwork. In the next section it's time to start building our game.
