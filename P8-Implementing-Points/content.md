---
title: Implementing points
slug: implementing-points
---

There is one thing missing, the player's score! It's not much fun dodging obstacles without any reward.

#Adding the score label

> [action]
> Drag a *Label* object into the *GameScene.sks*, set the *Name* to `scoreLabel`. Set the font to something you like and will also be easy for the player to read.
> You could try `Helvetica Neue, Bold, 72`. Set the *Z Position* to `10` as you want to ensure it's always in the foreground.
>
> ![Add label](../Tutorial-Images/xcode_spritekit_add_score_label.png)
>
> Position it somewhere clear and easy to read, near top of the screen is good.
>

<!-- -->

> [info]
> If you are having difficulty moving the *scoreLabel* or just want finer control over node placement in general: you can use the *Arrow Keys* to move any node a pixel at a time or hold *Shift + Arrow Keys* to move in bigger steps.
>

<!-- html comment to break boxes -->

> [action]
> Can you add a code connection in *GameScene.swift* for `scoreLabel`? Try it!
**Tip: The node type will be SKLabel**

<!-- html comment to break boxes -->

> [solution]
> Open *GameScene.swift* and add the following property:
>
```
var scoreLabel: SKLabelNode!
```
>
> Add the following code after the `buttonRestart` connection:
>
```
scoreLabel = self.childNodeWithName("scoreLabel") as! SKLabelNode
```
>

Great the connection has been made! If you did it yourself, virtual high five.

##Tracking score

The *scoreLabel* will display game score. However, as it stands there is no way to track the score. Let's add a score counter.

> [action]
> Open *GameScene.swift* and add the following property to the *GameScene* class:
>
```
var points = 0
```
>

When the game starts, we want to ensure the label is reset to `0`. Let's make that happen.

> [action]
> Add the following code to the bottom of the `didMoveToView(...)` method:
>
```
/* Reset Score label */
scoreLabel.text = String(points)
```
>
> `String(...)` is great for converting a number quickly into a string representation. We will be reusing this snippet when it comes to updating the score.

The next part will take a little bit of extra work. As it stands, if the bunny collides with anything, including the goal node, it will trigger the death sequence. So we need some way of knowing if the collision in question was a collision between the **bunny** and the **goal**. One way would be to compare the *categoryMask*. Remember this list?

- 1 - Player
- 2 - Obstacle
- 4 - Ground
- 8 - Goal Sensor

However in this case, we will be using the *name* of the node, not the *categoryMask*.

> [action]
> Open *GameScene.swift* and add this code to the start of the `didBeginContact(...)` method, after `gameState` check.
>
```
/* Get references to bodies involved in collision */
let contactA:SKPhysicsBody = contact.bodyA
let contactB:SKPhysicsBody = contact.bodyB
>
/* Get references to the physics body parent nodes */
let nodeA = contactA.node!
let nodeB = contactB.node!
>
/* Did our hero pass through the 'goal'? */
if nodeA.name == "goal" || nodeB.name == "goal" {
>    
  /* Increment points */
  points += 1
>  
  /* Update score label */
  scoreLabel.text = String(points)
>  
  /* We can return now */
  return
}
```

When a collision takes place between two physics bodies, the information is stored in a `SKPhysicsContact` object. We can use this to find out more information about the collision, so first we grab a reference to the `SKPhysicsBodies`. However, we may have a custom class with our own properties, and we might want to access those, so we go up a level and get a reference to the parent node this physics body belongs to.

Now that we have references to `SKSpriteNode`s, we can check each node to see if it has the *Name* `goal`. We can then update the player's *points* and use that to update the **score label**. Straight after that, we *return* from the `didBeginContact(...)` method to stop execution of all following code, because otherwise the player will suffer a needless death.

Run the game! With a bit of skill you should be able to pass through the goal and get a point. You can always make the goal area bigger for testing :]

#Summary

Congratulations on finishing *Hoppy Bunny*, give the person next to you a high five!

In this chapter you learned to:

- Add a *SKLabel*
- Manage the player score
- Identify specific physics collisions

In the next chapter we will cover the steps to test your game on device.
