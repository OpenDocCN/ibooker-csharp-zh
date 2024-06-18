# Unity Lab #6 Scene Navigation

In the last Unity Lab, you created a scene with a floor (a plane) and a player (a sphere nested under a cylinder), and you used a NavMesh, a NavMesh Agent, and raycasting to get your player to follow your mouse clicks around the scene.

Now we’ll pick up where the last Unity Lab left off. The goal of these labs is to get you familiar with Unity’s **pathfinding and navigation system**, a sophisticated AI system that lets you create characters that can find their way around the worlds you create. In this lab, you’ll use Unity’s navigation system to make your GameObjects move themselves around a scene.

Along the way you’ll learn about useful tools: you’ll create a more complex scene and bake a NavMesh that lets an agent navigate it, you’ll create static and moving obstacles, and most importantly, you’ll **get more practice writing C# code**.

# Let’s pick up where the last Unity Lab left off

In the last Unity Lab, you created a player out of a sphere head nested under a cylinder body. Then you added a NavMesh Agent component to move the player around the scene, using raycasting to find the point on the floor that the player clicked. In this lab, you’ll pick up where the last one left off. You’ll add GameObjects to the scene, including stairs and obstacles so you can see how Unity’s navigation AI handles them. Then you’ll add a moving obstacle to really put that NavMesh Agent through its paces.

So go ahead and **open the Unity project** that you saved at the end of the last Unity Lab. If you’ve been saving up the Unity Labs to do them back to back, then you’re probably ready to jump right in! But if not, take a few minutes and flip through the last Unity Lab again—and also look through the code that you wrote for it.

![Images](assets/pg652.png)

###### Note

If you’re using our book because you’re preparing to be a professional developer, being able to go back and read and refactor the code in your old projects is a really important skill—and not just for game development!

# Add a platform to your scene

Let’s do a little experimentation with Unity’s navigation system. To help us do that, we’ll add more GameObjects to build a platform with stairs, a ramp, and an obstacle. Here’s what it will look like:

![Images](assets/pg653-1.png)

It’s a little easier to see what’s going on if we switch to an **isometric** view, or a view that doesn’t show perspective. In a **perspective** view, objects that are further away look small, while closer objects look large. In an isometric view, objects are always the same size no matter how far away they are from the camera.

![Images](assets/pg653-2.png)

###### Note

**Sometimes it’s easier to see what’s going on in your scene if you switch to an isometric view. You can always reset the layout if you lose track of the view.**

**Add 10 GameObjects** to your scene. **Create a new material called *Platform*** in your Materials folder with albedo color CC472F, and add it to all of the GameObjects except for Obstacle, which uses a **new material called *8 Ball*** with the 8 Ball Texture map from the first Unity Lab. This table shows their names, types, and positions:

![Images](assets/pg653-3.png)

# Use bake options to make the platform walkable

Use Shift-click to select all of the new GameObjects that you added to the scene, then use Control-click (or Command-click on a Mac) to deselect Obstacle. Go to the Navigation window and click the Object button, then **make them all walkable by** checking Navigation Static and setting the Navigation Area to Walkable. **Make the Obstacle GameObject not walkable** by selecting it, clicking Navigation Static, and setting Navigation Area to Not Walkable.

![Images](assets/pg654-1.png)

Now follow the same steps that you used before to **bake the NavMesh**: click the Bake button at the top of the Navigation window to switch to the Bake view, then click the Bake button at the bottom.

![Images](assets/pg654-2.png)

It looks like it worked! The NavMesh now appears on top of the platform, and there’s space around the obstacle. Try running the game. Click on top of the platform and see what happens.

Hmm, hold on. Things aren’t working the way we expected them to. When you click on top of the platform, the player goes ***under*** it. If you look closely at the NavMesh that’s displayed when you’re viewing the Navigation window, you’ll see that it also has space around the stairs and ramp, but doesn’t actually include either of them in the NavMesh. The player has no way to get to the point you clicked on, so the AI gets it as close as it can.

![Images](assets/pg654-3.png)

# Include the stairs and ramp in your NavMesh

An AI that couldn’t get your player up or down a ramp or stairs wouldn’t be very intelligent. Luckily, Unity’s pathfinding system can handle both of those cases. We just need to make a few small adjustments to the options when we bake the NavMesh. Let’s start with the stairs. Go back to the Bake window and notice that the default value of Step Height is 0.4\. Take a careful look at the measurements for your steps—they’re all 0.5 units tall. So to tell the navigation system to include steps that are 0.5 units, **change the Step Height to 0.5**. You’ll see the picture of the step in the diagram get taller, and the number above it change from the default 0.4 value to 0.5.

We still need to include the ramp in the NavMesh. When you created the GameObjects for the platform, you gave the ramp an X rotation of –46, which means that it’s a 46-degree incline. The Max Slope setting defaults to 45, which means it will only include ramps, hills, or other slopes with at most a 45-degree incline. So **change Max Slope to 46**, then **bake the NavMesh again**. Now it will include the ramp and stairs.

![Images](assets/pg655.png)

Start your game and test out your new NavMesh changes.

# Fix height problems in the NavMesh

Now that we’ve got control of the camera, we can get a good look at what’s going on under the platform—and something doesn’t look quite right. Start your game, then rotate the camera and zoom in so you can get a clear view of the obstacle sticking out under the platform. Click the floor on one side of the obstacle, then the other. It looks like the player is going right through the obstacle! And it goes right through the end of the ramp, too.

![Images](assets/pg657-1.png)

But if you move the player back to the top of the platform, it avoids the obstacle just fine. What’s going on?

Look closely at the parts of the NavMesh above and below the obstacle. Notice any differences between them?

![Images](assets/pg657-2.png)

Go back to the part of the last lab where you set up the NavMesh Agent component—specifically, the part where you set the Height to 3\. Now you just need to do the same for the NavMesh. Go back to the Bake options in the Navigation window and **set the Agent Height to 3, then bake your mesh again**.

![Images](assets/pg657-3.png)

This created a gap in the NavMesh under the obstacle and expanded the gap under the ramp. Now the player doesn’t hit either the obstacle or the ramp when moving around under the platform.

# Add a NavMesh Obstacle

You already added a static obstacle in the middle of your platform: you created a stretched-out capsule and marked it non-walkable, and when you baked your NavMesh it had a hole around the obstacle so the player has to walk around it. What if you want an obstacle that moves? Try moving the obstacle—the NavMesh doesn’t change! It still has a hole *where the obstacle was*, not where it currently is. If you bake it again, it just creates a hole around the obstacle’s new location. To add an obstacle that moves, add a **NavMesh Obstacle component** to a GameObject.

Let’s do that right now. **Add a Cube to your scene** with position (–5.75, 1, –1) and scale (2, 2, 0.25). Create a new material for it with a dark gray color (333333) and name your new GameObject *Moving Obstacle*. This will act as a kind of gate at the bottom of the ramp that can move up out of the way of the player or down to block it.

![Images](assets/pg658-1.png)

We just need one more thing. Click the Add Component button at the bottom of the Inspector window and choose Navigation >> Nav Mesh Obstacle to **add a NavMesh Obstacle component** to your Cube GameObject.

![Images](assets/pg658-2.png)

If you leave all of the options at their default settings, you get an obstacle that the NavMesh Agent can’t get through. Instead, the Agent hits it and stops. **Check the Carve box**—this causes the obstacle to ***create a moving hole in the NavMesh*** that follows the GameObject. Now your Moving Obstacle GameObject can block the player from navigating up and down the ramp. Since the NavMesh height is set to 3, if the obstacle is less than 3 units above the floor it will create a hole in the NavMesh underneath it. If it goes above that height, the hole disappears.

###### Note

**The Unity Manual has thorough—and readable!—explanations for the various components. Click the Open Reference button (![Images](assets/pg658-3.png) ) at the top of the Nav Mesh Obstacle panel in the Inspector to open up the manual page. Take a minute to read it—it does a great job of explaining the options.**

# Add a script to move the obstacle up and down

This script uses the **OnMouseDrag** method. It works just like the OnMouseDown method you used in the last lab, except that it’s called when the GameObject is dragged.

![Images](assets/pg659-1.png)

###### Note

**The first if statement keeps the block from moving below the floor, and the second keeps it from moving too high. Can you figure out how they work?**

**Drag your script onto the Moving Obstacle GameObject** and run the game—uh-oh, something’s wrong. You can click and drag the obstacle up and down, but it also moves the player. Fix this by **adding a tag** to the GameObject.

![Images](assets/pg659-3.png)

Then **modify your MoveToClick script** to check for the tag:

![Images](assets/pg659-2.png)

Run your game again. If you click on the obstacle you can drag it up and down, and it stops when it hits the floor or gets too high. Click anywhere else, and the player moves just like before. Now you can **experiment with the NavMesh Obstacle options** (this is easier if you reduce the Speed in the Player’s NavMesh Agent):

*   Start your game. Click on *Moving Obstacle* in the Hierarchy window and **uncheck the Carve option**. Move your player to the top of the ramp, then click at the bottom of the ramp—the player will bump into the obstacle and stop. Drag the obstacle up, and the player will continue moving.

*   Now **check Carve** and try the same thing. As you move the obstacle up and down, the player will recalculate its route, taking the long way around to avoid the obstacle if it’s down, and changing course in real time as you move the obstacle.

# Get creative!

Can you find ways to improve your game and get practice writing code? Here are some ideas to help you get creative:

*   Build out the scene—add more ramps, stairs, platforms, and obstacles. Find creative ways to use materials. Search the web for new texture maps. Make it look interesting!

*   Make the NavMesh Agent move faster when the player holds down the Shift key. Search for “KeyCode” in the Scripting Reference to find the left/right Shift key codes.

*   You used OnMouseDown, Rotate, RotateAround, and Destroy in the last lab. See if you can use them to create obstacles that rotate or disappear when you click them.

*   We don’t actually have a game just yet, just a player navigating around a scene. Can you find a way to **turn your program into a timed obstacle course**?

***You already know enough about Unity to start building interesting games—and that’s a great way to get practice so you can keep getting better as a developer.***

> **This is your chance to experiment. Using your creativity is a really effective way to quickly build up your coding skills.**

# Downloadable exercise: Animal match boss battle

![Images](assets/com25.png)

If you’ve played a lot of video games (and we’re pretty sure that you have!), then you’ve had to play through a whole lot of boss battles—those fights at the end of a level or section where you face off against an opponent that’s bigger and stronger than what you’ve seen so far. We have one last challenge for you before the end of the book—consider it the *Head First C#* boss battle.

In [#start_building_with_chash_build_somethin](ch01.html#start_building_with_chash_build_somethin) you built an animal matching game. It was a great start, but it’s missing... something. Can you figure out how to turn your animal matching game into a memory game? Go to our GitHub page and download the PDF for this project—or if you want to play this boss battle in Hard mode, just dive right in and see if you can do it on your own.

![Images](assets/pg661.png)

**There’s so much more downloadable material! The book may be over, but we can keep the learning going. We’ve put together even more downloadable material on important C# topics. We’ve also continued the Unity learning path with additional Unity Labs and even a Unity boss battle.**

**We hope you’ve learned a lot—and even more importantly, we hope your C# learning journey is just beginning. Great developers never stop learning.**

**Head to our GitHub page for more: [https://github.com/head-first-csharp/fourth-edition](https://github.com/head-first-csharp/fourth-edition).**

# Thank you for reading our book!

Pat yourself on the back—this is a real accomplishment! We hope this journey has been as rewarding for you as it has been for us, and that you’ve enjoyed all of the projects and the code that you’ve written along the way.

# But wait, there’s more! Your journey’s just begun...

In some chapters we gave you additional projects that you could download from our GitHub page: [https://github.com/head-first-csharp/fourth-edition](https://github.com/head-first-csharp/fourth-edition).

![Images](assets/pg662-1.png)

###### Note

Check out these great C# and .Net resources!

Get connected with the .NET Developer Community: [https://dotnet.microsoft.com/platform/community](https://dotnet.microsoft.com/platform/community).

Watch live streams and chat with the team that builds .NET and C#: [https://dotnet.microsoft.com/platform/community/standup](https://dotnet.microsoft.com/platform/community/standup).

Learn more in the docs: [https://docs.microsoft.com/en-us/dotnet](https://docs.microsoft.com/en-us/dotnet).

The GitHub page contains **lots of additional material**. There’s still more to learn, and more projects to do!

***Keep your C# learning journey going*** by downloading PDFs that continue the *Head First C#* story and cover **essential C# topics**, including:

*   Event handlers

*   Delegates

*   The MVVM pattern (including a retro arcade game project)

*   ***...and more!***

And while you’re there, there’s **more to learn about Unity**. You can download:

*   PDF versions of all of the Unity Labs in this book

*   ***Even more Unity Labs*** that cover physics, collisions, and more!

*   A **Unity Lab boss battle** to put your Unity development skills to the test

*   A complete **Unity Lab project** to create a game from the ground up

# And check out these essential (and amazing!) books by some of our friends and colleagues, also published by O’REILLY

![Images](assets/pg662-2.png)