## Dithering

### CIELAB Delta E Method
```glsl
ColorPairResult res = find_closest_colors(color, palette, palette_size);

index_pixel += 0.26; 
float diff = res.dist_1 / res.dist_12;

vec2 uv;
if (diff < index_pixel) { uv = res.uv_1; }
else { uv = res.uv_2; }

return get_palette_color_from_uv(uv);
```
### Bisqwit-based Method
```glsl
ColorResult res = find_closest_color(color.rgb);

if( res.color != color.rgb ) {
	//float threshold = 1.0 / sqrt(float(palette_size));
	color.rgb += vec3((index_pixel - 0.5) * COLOR_DITHER);
	color = clamp(color, 0.0, 1.0);
	res = find_closest_color(color.rgb);
}

return res.color;
```