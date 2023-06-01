# Pixelization

## Method 1: Rendering at a Low Resolution then Upscaling

In this method, we display a low-resolution SubViewport and upscale it. This is a slightly more complicated way of achieving a "pixelization" effect, but offers a performance advantage over a pixelization post-processing effect, as we will be actually rendering the game at the lower resolution.

In this example we will be rendering the game at 640x360 and displaying it at 1920x1080 (3x pixelization).
This resolution was chosen since 640x360 cleanly translates to 3x3 pixels on a 1080p screen, and 2x2 pixels on a 720p screen (both common resolutions). 

Render Resolution = (640, 360)
Display Resolution = (1920, 1080)

### Project Settings

In your project ensure your display resolution (1920, 1080) is correct. This is in `Display > Window > Viewport Width/Height` in your Project Settings.

![[Pasted image 20230412172322.png]]

### Display Scene

Create the scene with a Node2D as your root node, then add a SubViewportContainer and a SubViewport. The SubViewport will contain your actual in-game world.

![[Pasted image 20230412171418.png]]

For the SubViewportContainer, set the scale in the inspector (`Layout > Transform > Scale`) to (3, 3).

![[Pasted image 20230412172842.png]]

Also, set the texture filter to Nearest (`CanvasItem > Texture > Filter`). This will create a sharp image when upscaling.

![[Pasted image 20230412173123.png]]

For the SubViewport, set the size in the inspector (`SubViewport > Size`) to your render resolution (640, 360).

![[Pasted image 20230412171737.png]]

Save the scene with your preferred name. For this example we will save it as `display.tscn`.
### Render/World Scene

Create a second scene, using a Node2D as the root node again.
This scene will contain the nodes you need to build your actual game world.

![[Pasted image 20230412173706.png]]

Save the scene as is. For this example we will save it as `world.tscn`.

In your display scene, drag the `world.tscn` scene we just created into the SubViewport node.

![[Pasted image 20230412173826.png]]

At this point the setup is done. Add whatever nodes that need to be rendered low-res into your `world.tscn`. Keep in mind that since you're working with a lower resolution, things will look small in the editor. (The 2D editor uses pixels as its units of distance)
## Tip: Use Guides to Show Screen Size

It may be helpful to see what the size of one full screen in our world scene looks like. The default viewport guides aren't accurate here anymore since we're working with a lower resolution.
Create guides by clicking and holding the editor ruler and dragging out. Create horizontal guides lining up at 0px and 640px and vertical guides lining up at 0px and 360px.

![[Pasted image 20230412174818.png]]

## Example Use

To show an example of what the world scene looks like upscaled, we will add the default icon sprite, tiled using a TextureRect.

![[Pasted image 20230412175142.png]]

After saving `world.tscn`, this is what is shown in `display.tscn`.

![[Pasted image 20230412175248.png]]