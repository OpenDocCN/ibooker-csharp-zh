# Unity Lab #5: Raycasting

When you set up a scene in Unity, you’re creating a virtual 3D world for the characters in your game to move around in. But in most games, most things in the game aren’t directly controlled by the player. So how do these objects find their way around a scene?

The goal of labs 5 and 6 is to get you familiar with Unity’s **pathfinding and navigation system**, a sophisticated AI system that lets you create characters that can find their way around the worlds that you create. In this lab, you’ll build a scene out of GameObjects and use navigation to move a character around it.

You’ll use **raycasting** to write code that’s responsive to the geometry of the scene, **capture input,** and use it to move a GameObject to the point where the player clicked. Just as importantly, you’ll **get practice writing C# code** with classes, fields, references, and other topics we’ve discussed.

# Create a new Unity project and start to set up the scene

*Before you begin, close any Unity project that you have open. Also close Visual Studio—we’ll let Unity open it for us. Create a new Unity project using the 3D template, set your layout to Wide so it matches our screenshots, and give it a name like **Unity Labs 5 and 6** so you can come back to it later.*

Start by creating a play area that the player will navigate around. Right-click inside the Hierarchy window and **create a Plane** (GameObject >>3D Object >> Plane). Name your new Plane GameObject *Floor.*

Right-click on the Assets folder in the Project window and **create a folder inside it called Materials**. Then right-click on the new Materials folder you created and choose **Create >> Material**. Call the new material *FloorMaterial*. Let’s keep this material simple for now—we’ll just make it a color. Select Floor in the Project window, then click on the white box to the right of the word Albedo in the Inspector.

![Images](assets/pg578-1.png)

In the Color window, use the outer ring to choose a color for the floor. We used a color with number 4E51CB in the screenshot—you can type that into the Hexadecimal box.

Drag the material from the **Project window onto the Plane GameObject in the Hierarchy window**. Your floor plane should now be the color that you selected.

![Images](assets/pg578-2.png)![Images](assets/pg578-3.png)

###### Note

Think about it and take a guess. Then use the Inspector window to try various Y scale values and see if the plane acts the way you expected. (Don’t forget to set them back!)

###### Note

**A Plane is a flat square object that’s 10 units long by 10 units wide (in the X-Z plane), and 0 units tall (in the Y plane). Unity creates it so that the center of the plane is at point (0,0,0). This center point of the plane determines it position in the scene. Just like our other objects, you can move a plane around the scene by using the Inspector or the tools to change its position and rotation. You can also change its scale, but since it has no height, you can only change the X and Z scale—any positive number you put into the Y scale will be ignored.**

**The objects that you can create using the 3D Object menu (planes, spheres, cubes, cylinders, and a few other basic shapes) are called primitive objects. You can learn more about them by opening the Unity Manual from the Help menu and searching for the “Primitive and placeholder objects” help page. Take a minute and open up that help page right now. Read what it says about planes, spheres, cubes, and cylinders.**

# Set up the camera

In the last two Unity Labs you learned that a GameObject is essentially a “container” for components, and that the Main Camera has just three components: a Transform, a Camera, and an Audio Listener. That makes sense, because all a camera really needs to do is be at a location and record what it sees and hears. Have a look at the camera’s Transform component in the Inspector window.

![Images](assets/pg579-1.png)

Notice how the position is (0, 1, –10). Click on the Z label in the Position line and drag up and down. You’ll see the camera fly back and forth in the scene window. Take a close look at the box and four lines in front of the camera. They represent the camera’s **viewport**, or the visible area on the player’s screen.

**Move the camera around the scene and rotate it using the Move tool (W) and Rotate tool (E)**, just like you did with other GameObjects in your scene. The Camera Preview window will update in real time, showing you what the camera sees. Keep an eye on the Camera Preview while you move the camera around. The floor will appear to move as it flies in and out of the camera’s view.

Use the context menu in the Inspector window to reset the Main Camera’s Transform component. Notice how it ***doesn’t reset the camera to its original position***—it resets both the camera’s position and its rotation to (0, 0, 0). You’ll see the camera intersecting the plane in the Scene window.

![Images](assets/pg579-2.png)

Now let’s point the camera straight down. Start by clicking on the X label next to Rotation and dragging up and down. You’ll see the viewport in the camera preview move. Now **set the camera’s X rotation to 90** in the Inspector window to point it straight down.

You’ll notice that there’s nothing in the Camera Preview anymore, which makes sense because the camera is looking straight down below the infinitely thin plane. **Click on the Y position label in the Transform component and drag up** until you see the entire plane in the Camera Preview.

Now **select Floor in the Hierarchy window**. Notice that the Camera Preview disappears—it only appears when the camera is selected. You can also switch between the Scene and Game windows to see what the camera sees.

Use the Plane’s Transform component in the Inspector window to **set the Floor GameObject’s scale to (4, 1, 2)** so that it’s twice as long as it is wide. Since a Plane is 10 units wide and 10 units long, this scale will make it 40 units long and 20 units wide. The plane will completely fill up the viewport again, so move the Camera further up along the Y axis until the entire plane is in view.

![Images](assets/pg579-3.png)

###### Note

You can switch between the Scene and Game windows to see what the camera sees.

# Create a GameObject for the player

Your game will need a player to control. We’ll create a simple humanoid-ish player that has a cylinder for a body and a sphere for a head. Make sure you don’t have any objects selected by clicking the scene (or the empty space) in the Hierarchy window.

**Create a Cylinder GameObject** (3D Object >> Cylinder)—you’ll see a cylinder appear in the middle of the scene. Change its name to *Player*, then **choose Reset from the context menu** for the Transform component to make sure it has all of its default values. Next, **create a Sphere GameObject** (3D Object >> Sphere). Change its name to *Head*, and reset its Transform component as well. They’ll each have a separate line in the Hierarchy window.

But we don’t want separate GameObjects—we want a single GameObject that’s controlled by a single C# script. This is why Unity has the concept of **parenting**. Click on Head in the Hierarchy window and **drag it onto Player**. This makes Player the parent of Head. Now the Head GameObject is **nested** under Player.

![Images](assets/pg580-1.png)

Select Head in the Hierarchy window. It was created at (0, 0, 0) like all of the other spheres you created. You can see the outline of the sphere, but you can’t see the sphere itself because it’s hidden by the plane and the cylinder. Use the Transform component in the Inspector window to **change the Y position of the sphere to 1.5**. Now the sphere appears above the cylinder, just the right place for the player’s head.

Now select Player in the Hierarchy window. Since its Y position is 0, half of the cylinder is hidden by the plane. **Set its Y position to 1**. The cylinder pops up above the plane. Notice how it took the Head sphere along with it. Moving Player causes Head to move along with it because moving a parent GameObject moves its children too—in fact, *any* change that you make to its Transform component will automatically get applied to the children. If you scale it down, its children will scale, too.

Switch to the Game window—your player is in the middle of the play area.

![Images](assets/pg580-2.png)

###### Note

When you modify the Transform component for a GameObject that has nested children, the children will move, rotate, and scale along with it.

# Introducing Unity’s navigation system

One of the most basic things that video games do is move things around. Players, enemies, characters, items, obstacles...all of these things can move. That’s why Unity is equipped with a sophisticated artificial intelligence–based navigation and pathfinding system to help GameObjects move around your scenes. We’ll take advantage of the navigation system to make our player move toward a target.

Unity’s navigation and pathfinding system lets your characters intelligently find their way around a game world. To use it, you need to set up basic pieces to tell Unity where the player can go:

*   First, you need to tell Unity exactly where your characters are allowed to go. You do this by **setting up a NavMesh**, which contains all of the information about the walkable areas in the scene: slopes, stairs, obstacles, and even points called off-mesh links that let you set up specific player actions like opening a door.

*   Second, you **add a NavMesh Agent component** to any GameObject that needs to navigate. This component automatically moves the GameObject around the scene, using its AI to find the most efficient path to a target and avoiding obstacles and, optionally, other NavMesh Agents.

*   It can sometimes take a lot of computation for Unity to navigate complex NavMeshes. That’s why Unity has a Bake feature, which lets you set up a NavMesh in advance **and precompute (or bake)** the geometric details to make the agents work more efficiently.

![Images](assets/pg581.png)

###### Note

Unity provides a sophisticated AI navigation and pathfinding system that can move your GameObjects around a scene in real time by finding an efficient path that avoids obstacles.

# Set up the NavMesh

Let’s set up a NavMesh that just consists of the Floor plane. We’ll do this using the Navigation window. **Choose AI >> Navigation from the Window menu** to add the Navigation window to your Unity workspace. It should show up as a tab in the same panel as the Inspector window. Then use the Navigation window to **mark the Floor GameObject *navigation static* and *walkable:***

*   Press the **Object button** at the top of the Navigation window.

*   **Select the Floor plane** in the Hierarchy window.

*   Check the **Navigation Static box**. This tells Unity to include the Floor when baking the NavMesh.

*   **Select Walkable** from the Navigation Area dropdown. This tells Unity that the Floor plane is a surface that any GameObject with a NavMesh Agent can navigate.

![Images](assets/pg582-1.png)

Since the only walkable area in this game will be the floor, we’re done in the Object section. For a more complex scene with many walkable surfaces or nonwalkable obstacles, each individual GameObject needs to be marked appropriately.

**Click the Bake button** at the top of the Navigation window to see the bake options.

Now **click the *other* Bake button** at the *bottom* of the Navigation window. It will briefly change to Cancel and then switch back to Bake. Did you notice that something changed in the Scene window? Switch back and forth between the Inspector and Navigation windows. When the Navigation window is active, the Scene window shows the NavMesh Display and highlights the NavMesh as a blue overlay on top of the GameObjects that are part of the baked NavMesh. In this case, it highlights the plane that you marked as navigation static and walkable.

Your NavMesh is now set up.

![Images](assets/pg582-2.png)

# Make your player automatically navigate the play area

Let’s add a NavMesh Agent to your Player GameObject. **Select Player** in the Hierarchy window, then go back to the Inspector window, click the **Add Component** button, and choose **Navigation >> NavMesh Agent** to add the NavMesh Agent component. The cylinder body is 2 units tall and the sphere head is 1 unit tall, so you want your agent to be 3 units tall—so set the Height to 3\. Now the NavMesh Agent is ready to move the Player GameObject around the NavMesh.

**Create a Scripts folder and add a script called *MoveToClick.cs***. This script will let you click on the play area and tells the NavMesh Agent to move the GameObject to that spot. You learned about private fields in [#encapsulation_keep_your_privateshellippr](ch05.html#encapsulation_keep_your_privateshellippr). This script will use one to store a reference to the NavMeshAgent. Your GameObject’s code will need a reference to its agent so it can tell the agent where to go, so you’ll call the GetComponent method to get that reference and save it in a **private NavMeshAgent field** called `agent`:

[PRE0]

![Images](assets/pg583-1.png)

The navigation system uses classes in the UnityEngine.AI namespace, so you’ll need to **add this `using` line** to the top of your *MoveToClick.cs* file:

![Images](assets/pg583-2.png)![Images](assets/pg584-2.png)

**Yes! We’re using a really useful tool called raycasting.**

In the second Unity Lab, you used Debug.DrawRay to explore how 3D vectors work by drawing a ray that starts at (0, 0, 0). Your MoveToClick script’s Update method actually does something similar to that. It uses the **Physics.Raycast method** to “cast” a ray—just like the one you used to explore vectors—that starts at the camera and goes through the point where the user clicked, and **checks if the ray hit the floor**. If it did, then the Physics.Raycast method provides the location on the floor where it hit. Then the script sets the **NavMesh Agent’s destination field**, which causes the NavMesh Agent to **automatically move the player** toward that location.