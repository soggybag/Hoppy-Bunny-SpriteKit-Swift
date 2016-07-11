---
title: Setting up collisions
slug: setting-up-collisions
---

Now it's time to set up collision handling so that our game finally becomes as frustrating as *Flappy Bird*. Any good game needs to be unforgivingly savage and difficult, right?

I would recommend you have a look at Apple's documentation on the subject of [Working with Collisions and Contacts.](https://developer.apple.com/library/ios/documentation/GraphicsAnimation/Conceptual/SpriteKit_PG/Physics/Physics.html#//apple_ref/doc/uid/TP40013043-CH6-SW14) It can be a little bit confusing when you first start, so don't worry if you don't understand everything all at once.

This time, to set up the collision detection we are going to use the *Scene Editor* as much as possible. In later tutorials we will expand this knowledge by implementing physics in code.

#Obstacle physics

The current obstacle has no physics bodies, let's fix that.

> [action]
> Open *Obstacle.sks*, make the following changes to both **carrots** and the invisible **goal** node.
> For all three objects, enable physics by setting the *Body Type* to `Bounding rectangle` and uncheck the subsequent 4 boxes.
>
> ![Carrot physics](../Tutorial-Images/xcode_spritekit_obstacle_physics_collision_properties.png)
>
> Next, set the *Category Mask* to `2` and the *Contact Mask* to `1` for all three.
>
> One little tweak for the invisible goal sprite: change the *Category Mask* to `8`.
>

#Physics Categories

Every physics body in a scene can be assigned up to 32 different categories, each corresponding to a bit in a 32-bit mask.
You'll notice those really big numbers in the *Collision Mask* and *Field Mask* fields: they are the integer value of `2` to the power `32` which is the decimal way of saying that every bit in the binary 32-bit mask is 1. With some spaces added for clarity, that looks like this: `11111111 11111111 11111111 11111111`. In the case of the *Collision Mask*, this means that this body will collide with **every** other body, which is the default behavior.

Think about separating your *Category Masks* into logical groups, for example:

	- 1 - Player		00000000 00000000 00000000 00000001
	- 2 - Obstacle		00000000 00000000 00000000 00000010
	- 4 - Ground		00000000 00000000 00000000 00000100
	- 8 - Goal Sensor	00000000 00000000 00000000 00001000

The *Contact Mask* allows you to define for which physics bodies you want to be informed when a collision takes place. This will be useful for when the bunny hits a carrot, because we don't want the bunny to bounce off the carrot. Instead we want the bunny to die at this point and fall from the sky. By setting the carrot *Contact Mask* to `1` (1 is going to be the bunny's category mask) the SpriteKit physics engine will inform us of carrot bunny collisions.

The **goal** is a slightly different case: we want to be informed when the bunny has entered into the goal zone so we can increase the player's score. However, we don't want the bunny to physically collide with the goal node - we want the bunny to pass through as if nothing is there. An object with this type of phyiscs behavior is called a *Sensor*: a type of physics body that is used only for contact detection without physically affecting the contacting body. This will be covered in more detail shortly.

> [info]
> So to summarize, we're currently worrying about three physics masks: the *Category Mask*, *Collision Mask* and *Contact Mask*. The *Category Mask* allows us to assign our physics object to a category. The *Collision Mask* allows us to decide which categories the physics object should collide with. Finally, the *Contact Mask* lets us tell the physics engine which categories we want to be informed of collisions with.

#Bunny physics

> [action]
> Open *Hero.sks* and click on the bunny.
> Set *Category Mask* to `1`, *Collision Mask* to `7` and the *Contact Mask* to `4294967295` this is `2^32`.
>
> ![Bunny physics](../Tutorial-Images/xcode_spritekit_bunny_physics_properties.png)
>

The *Collision Mask* is set to `7` as we want the bunny to only physically collide with *Categories* `1 + 2 + 4` e.g. `Player, Obstacle, Ground`.  We don't want to collide with `8` which is the goal sensor, because if it did, the bunny will never be able to pass through the first obstacle.  
The *Contact Mask* has been set to `2^32`, which is the max value: all categories are set to `1`, like this: `11111111 11111111 11111111 11111111`. This lets the physics engine know that we want to be informed if our bunny contacts any other physics body.

#Ground physics

Now it's time to set up the ground sprite physics; do you think you can tackle this yourself? Check above if you don't remember the *Category Mask* value we decided to use.  What value do you think you'll need for the *Contact Mask?*

> [solution]
> Open  *GameScene.sks* to modify both **ground** sprites.
> Set *Category Mask* to `4` and *Contact Mask* to `1`: we want to be informed if the bunny (category 1) has hit the ground.
>
> ![Ground physics](../Tutorial-Images/xcode_spritekit_ground_physics_properties.png)
>

Run your game! The bunny will now collide with the obstacles yet thankfully be able to flap through the goal node. If you're good enough to flapy through it, that is!

#Physics Contact Delegate

If the bunny collides with the ground, an obstacle or passes through the goal of an obstacle, we want to know about it.  Next we will implement the *Physics Contact Delegate* so oour code will be informed whenever one of these collision contacts takes place.

> [action]
> Open *GameScene.swift*, you need to declare that the *GameScene* class will implement the *SKPhysicsContactDelegate* protocol methods.
> To learn more about *Protocols* and *Delegates* please check out our [Swift Concepts Guide](https://www.makeschool.com/tutorials/swift-concepts-explained).
>
> We declare that a class is implementing this protocol by appending *SKPhysicsContactDelegate* after the class' super class *SKScene*, separated by a comma, as shown:
>
```
class GameScene: SKScene, SKPhysicsContactDelegate {
```
>

##Delegate Support

The *GameScene* class is now ready to implement the contact delegate protocol methods. First, we should inform the physics world which class will take responsibility for handling the contact messages. To do this, we assign *GameScene* as the collision delegate.

> [action]
> Add the following code to the `didMoveToView(..)` method:
>
```
/* Set physics contact delegate */
physicsWorld.contactDelegate = self
```

Finally, we can implement the *didBeginContact* method from the `SKPhysicsContactDelegate` protocol. This method will be called whenever a collision takes place that we want to know about e.g. The ones with a *contactMask* of `1`.

> [action]
> Add this new method to the *GameScene* class:
>
```
func didBeginContact(contact: SKPhysicsContact) {
  /* Hero touches anything, game over */
  print("TODO: Add contact code")
}
```
>

Run the game. Any time you collide with the ground, a carrot, or a goal sensor the *TODO* message will be logged to the console.

#Game over

Instead of simply printing a message in the console, it would be nice to think about a game over scenario.

A good game over indication might consist of:

- The bunny falling to ground
- The game scene shakes
- A restart game button appears

##Adding a button

Unfortunately, there is currently no easy way to add a button in SpriteKit so you will need to get creative and create your own solution. Only joking, we've kindly provided a starting point for you with a custom class called `MSButtonNode`.

> [action]
> [Download MSButtonNode](https://raw.githubusercontent.com/MakeSchool-Tutorials/Hoppy-Bunny-SpriteKit-Swift-Solution/master/HoppyBunny/MSButtonNode.swift) and drag this file into your project, making sure to select the *Copy items if needed* checkbox.

<!-- -->

> [action]
> Drag the *restart_button.png* to the game scene.
> Set the *Name* to `buttonRestart`, and set the *Z Position* to `10`: we want to ensure this user interface element sits on top of everything visually.
>
> ![Restart button properties](../Tutorial-Images/xcode_spritekit_restart_properties.png)
>
> To turn this sprite into a custom button, you need to change the class to be an instance of `MSButtonNode` instead of `SKSpriteNode`. Use the *Custom class* panel to change this by setting the *Custom Class* to `MSButtonNode`
>
> ![Restart button custom class](../Tutorial-Images/xcode_spritekit_restart_custom_class.png)
>

Can you set up a code connection for this button in `GameScene.swift`? Try it now.

> [solution]
> Open *GameScene.swift*, add a property for the button to the *GameScene* class:
>
```
/* UI Connections */
var buttonRestart: MSButtonNode!
```
>
> When we connect this node, we need to ensure the node is downcast to the `MSButtonNode` class.
> Add the following to the `didMoveToView(..)` method.
>
```
/* Set UI connections */
buttonRestart = self.childNodeWithName("buttonRestart") as! MSButtonNode
```
>

##Selection handler

The code connection is ready. If you run the game you can touch the button: it looks like it was touched yet nothing happens. First we need to add some code to be executed upon user touch.

> [action]
> Add the following code after the code connection:
>
```
/* Setup restart button selection handler */
buttonRestart.selectedHandler = {
>
  /* Grab reference to our SpriteKit view */
  let skView = self.view as SKView!
>
  /* Load Game scene */
  let scene = GameScene(fileNamed:"GameScene") as GameScene!
>
  /* Ensure correct aspect mode */
  scene.scaleMode = .AspectFill
>
  /* Restart game scene */
  skView.presentScene(scene)
>
}
```
>

This code loads in a fresh copy of the *GameScene.sks*, ensures the correct *scaleMode* is applied and then replaces the current scene with this fresh *GameScene*. This is how we restart the game. You can find similar code in `GameViewController.swift`, which is how the *GameScene* is initially loaded when the game starts.

##Hide the button

Great! We have a button, but as it's bang in the middle of the screen it might be a good idea to hide it once the game is in play.

> [action]
> Add the following code after the selection handler setup.
>
```
/* Hide restart button */
buttonRestart.state = .Hidden
```
>

We want the button to become visible when the bunny dies, so let's take look at how we implement our game over scenario. To do this properly, it would be really useful to be able to know the current state of the game. Has the game started? Is the player dead? Etc.

#Game State

State management is a great way to do this. To see state management in action, just look into the `MSButtonNode` code above. The button has a `state` property, which is used to track if the button is `Active`,`Hidden` or `Selected`.

For the *GameScene* class it would be great to know if the game state is either `Active` or `GameOver`.  

When this `GameOver` state applies we want to:

- Kill the bunny
- Stop the world scrolling
- Show the restart button
- Ignore any touch other than one on the restart button

An *Enumeration* is a great way to setup a custom state type.

> [action]
> Add the following `Enumeration` to the top of *GameScene.swift*, just before `class GameScene` is declared:
>
```
enum GameSceneState {
    case Active, GameOver
}
```
>
> To track the state we need to add a `gameState` property to the *GameScene* class. Create the state property, and set the default state to `Active`:
>
```
/* Game management */
var gameState: GameSceneState = .Active
```
>

#Bunny death

Great, we now have some rudimentary game state management in place. Time to kill the bunny!

> [action]
> Replace the `didBeginContact(...)` method as shown:
>
```
func didBeginContact(contact: SKPhysicsContact) {
  /* Ensure only called while game running */
  if gameState != .Active { return }
>
  /* Hero touches anything, game over */
>
  /* Change game state to game over */
  gameState = .GameOver
>
  /* Stop any new angular velocity being applied */
  hero.physicsBody?.allowsRotation = false
>
  /* Reset angular velocity */
  hero.physicsBody?.angularVelocity = 0
>
  /* Stop hero flapping animation */
  hero.removeAllActions()
>
  /* Show restart button */
  buttonRestart.state = .Active
}
```

Notice that at the beginning, we check the `gameState` to ensure that the code will not be called more than once when the player has died. The bunny's physics are effectively disabled by stopping `rotation`, resetting `angularVelocity` and removing the flapping sprite frame animation with the use of `removeAllActions()` method. The button is then shown and presented to the player with a simple `MSButtonNodeStateActive` state change.

Run the game! When the player dies the button should appear and you can restart play. You'll notice that we've introduced a bug at this point: when the player hits the goal node, the game ends! We'll fix that in the next page.

#Shutting down the world

It's not perfect yet... once it's game over, the bunny will still respond ever so slightly to touch and the world continues to scroll by.

To disable scrolling and touch, we can once again make use of the *gameState* property.

> [action]
> Add the following to the very top of the `update(...)` method:
>
```
/* Skip game update if game no longer active */
if gameState != .Active { return }
```
>

<!-- html comment to break boxes -->

> [action]
Can you figure out how to disable touch? Try it yourself!

<!-- html comment to break boxes -->

> [solution]
> Add the following to the top of the `touchesBegan(...)` method:
>
```
/* Disable touch if game state is not active */
if gameState != .Active { return }
```

Run the game! Now death should truly be final for our bunny.

#Death actions

It would look better if the bunny fell face first upon hitting an obstacle. A powerful way to do achieve this is using *SKActions*. We've already used actions to set up the the flappy animation frames.

> [action]
> Add the following code after we stopped the hero's actions with the `removeAllActions()` method in `didBeginContact(...)`:
>
```
/* Create our hero death action */
let heroDeath = SKAction.runBlock({
>
    /* Put our hero face down in the dirt */
    self.hero.zRotation = CGFloat(-90).degreesToRadians()
    /* Stop hero from colliding with anything else */
    self.hero.physicsBody?.collisionBitMask = 0
})
>
/* Run action */
hero.runAction(heroDeath)
```
>

The `runBlock` action allows us to define our own custom action. In this case, we manually rotate the bunny face down. We need to wrap this in an action to ensure it is executed at the correct **step** in the **rendering loop**. We could also achieve this with *overriding* the `didSimulatePhysics()` step and applying this rotation. However, that is kind of awkward to do. It's cleaner to wrap it in an *SKAction* like this.

Run the game! The bunny should now be face down upon any collision. It's all about those little bits of polish :]

#Shake it

It would be nice to add an old-school Star Trek style camera shake to emphasize the impact. This time we will create our own *GameEffects.sks* **SpriteKit Action file**, which enables you to store multiple effects that can be reused on any node.

> [action]
> Create a new *SpriteKit Action* file called `GameEffects`:
>
> ![Create New Action file](../Tutorial-Images/xcode_create_skaction.png)
>
> ![Add SpriteKit Action file GameEffects](../Tutorial-Images/xcode_create_skaction_file.png)
>
> Add your first *Action*, name it `Shake`
> ![Add New Action](../Tutorial-Images/xcode_spritekit_add_new_action.png)
>
> Now we have an empty action timeline ready for some actions. Drag across the *Move action* from the *Object Library*. Set the *Duration* to `0.2` seconds.
>
> ![Add Move Action](../Tutorial-Images/xcode_spritekit_action_add_move.png)
>
>
> Copy and paste this action two times and then modify all three actions as follows.
>
> 1. Set *Timing Function* to `Ease In`, set *Offset* to `(8,2)`
> 1. Set *Timing Function* to  `Ease Out`, set *Offset* to `(-4,-2)`
> 1. Set *Timing Function* to `Ease Out`, set *Offset* to `(4,2)`
>
> **Sadly it does not yet seem possible (As of Xcode 7.3.1) to preview this action on the scene from within the scene editor :[**

##Shake all the nodes

Time to try this out in our code. We don't need to worry about loading the file itself, as SpriteKit will automatically load any SpriteKit related resources and cache them at runtime :]

> [action]
> Open *GameScene.swift* and add the following code after the death action in `didBeginContact(...)`.
>
```
/* Load the shake action resource */
let shakeScene:SKAction = SKAction.init(named: "Shake")!
>  
/* Loop through all nodes  */
for node in self.children {
>      
    /* Apply effect each ground node */
    node.runAction(shakeScene)
}
```
> The effect can not be applied directly to the *GameScene*, so we need to loop through all the child nodes in the scene and apply it individually. Thankfully it's pretty straight-forward to do so.

Run the game! When the player loses the screen should give a short shake.

> [info]
> I encourage you to make this effect as crazy as you like. Experimentation is the best way to learn what works! Often it's happy little accidents that lead you to something awesome.

#Physics tweaking

You may have noticed the game is a little difficult, perhaps too difficult. It feels like the bunny falls too fast initially, and applying the touch impulse doesn't yet feel quite right.

> [action]
> Open *Hero.sks*, click on the bunny and navigate down to the physics properties, notice the *Initial Velocity* property.  Set this to `(0, 400)`.  This should give the player a much need reaction time cushion when the game first runs.
> ![Bunny Physics Tweaks](../Tutorial-Images/xcode_spritekit_bunny_physics_tweaks.png)
>

When the bunny is falling and the player touches the screen, the touch feels a little sluggish. This is due to the cumulative downward velocity generated by the bunny's fall. Sometimes, applying the touch impulse isn't enough to overcome the bunny's downward velocity! If we reset the vertical velocity at the point of touch, then apply the impulse, this might make it feel more responsive.

> [action]
> Open *GameScene.swift*, add the following in the `touchesBegan(...)` method after the `gameState` check:
>
```
/* Reset velocity, helps improve response against cumulative falling velocity */
hero.physicsBody?.velocity = CGVectorMake(0, 0)
```
>

Run the game! That little change has made the core mechanic feel much more satisfying :]

> [info]
> Bonus tip for making it so far: You've added a lot of code and your formatting may be getting a little, well ugly. Thankfully there is an easy way to tidy up your code with *Re-Indent*. Open *GameScene.swift* then select all your code with *Cmd+A*, then press *Ctrl+I* to Re-Indent.

#Summary

Wow, a lot of ground was covered in this chapter:

- Understanding the principles of SpriteKit physics collision and contact masking
- Implementing the `SKPhysicsContactDelegate` so we are informed of collision contacts
- Using our own custom button class
- Implementing a simple game state manager
- Running a custom *SKAction* and creating reusable *SKActions* visually
- Tweaking core mechanics, to make the gameplay feel just right

Next up: it wouldn't be a game without a scoring mechanism for the player.