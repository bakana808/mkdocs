This shader is a fix for if you are using a bitmap font (image imported as font data), and the font characters are always off by half a pixel.

![[Pasted image 20230410031457.png]]

``` gdshader
shader_type canvas_item;

uniform vec2 offset = vec2(0, -0.5);

void vertex() {
    VERTEX += (SCREEN_MATRIX * vec4(TEXTURE_PIXEL_SIZE, 0, 1)).xy * offset;
}
```