## Dithering

### CIELAB Delta E Method
```glsl
ColorPairResult res = find_closest_colors(color, palette, palette_size);
float dist_1 = res.dist_1; // dist from pal color to original color
float dist_12 = res.dist_12; // dist from pal color to 2nd closest pal color
vec2 uv_1 = res.uv_1; // UV of pal color
vec2 uv_2 = res.uv_2; // UV of 2nd closest pal color

index_pixel += 0.26; 
float diff = dist_1 / dist_12;

vec2 uv;
if (diff < index_pixel) { uv = uv_1; }
else { uv = uv_2; }

return get_palette_color_from_uv(uv);
```
### Bisqwit-based Method
```glsl
ColorResult res = find_closest_color(color.rgb);
vec3 pal_color = res.color;

if( pal_color != color.rgb ) {
	//float threshold = 1.0 / sqrt(float(palette_size));
	color.rgb += vec3((index_pixel - 0.5) * COLOR_DITHER);
	color = clamp(color, 0.0, 1.0);
	pal_color = find_closest_color(color.rgb).color;
}

return pal_color;
```