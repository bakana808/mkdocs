## Method 1: Rendering at a Low Resolution and Upscaling

In this method, we display a low-resolution SubViewport and upscale it.
In this example we will be rendering the game at 640x360 and displaying it at 1920x1080.
This resolution was chosen since 640x360 cleanly translates to 3x3 pixels on a 1080p screen, and 2x2 pixels on a 720p screen (both common resolutions). 

Render Resolution = (640, 360)
Display Resolution = (1920, 1080)

### Project Settings

In your project ensure your display resolution (1920, 1080) is correct. This is in `Display > Window > Viewport Width/Height` in your Project Settings.

![[Pasted image 20230412172322.png]]

### Display Scene

Create the scene that your SubViewport will be displayed in. Use a Node2D as your root node, then add a SubViewportContainer and a SubViewport.

![[Pasted image 20230412171418.png]]

For the SubViewportContainer, set the scale in the inspector (`Layout > Transform > Scale`) to (3, 3).

![[Pasted image 20230412172842.png]]

Also, set the texture filter to Nearest (`CanvasItem > Texture > Filter`). This will create a sharp image when upscaling.

![[Pasted image 20230412173123.png]]

For the SubViewport, set the size in the inspector (`SubViewport > Size`) to your render resolution (640, 360).

![[Pasted image 20230412171737.png]]

### Render Scene

Create a second scene that will contain your game world. Use a Node2D as the root node again.
