## Color Palette Mapping Shader

This shader provides the function `find_closest_color(src)`.  This function will return the closest color in the palette by color distance function `color_distance(a, b)` (different color distance function implementations discussed below).

The process
``` glsl linenums="1"
uniform sampler2D palette_texture;  // an image of 8 colors laid out horizontally
uniform int palette_size = 8;  // the number of colors in the palette

vec3 find_closest_color(vec3 color) {

    vec3 final_color = vec3(0); 
	float final_distance = 100000;
    
    for( int u = 0; u < palette_size; u += 1 ) {

        vec2 pal_uv = vec2(float(u) / float(palette_size), 0.0);
        vec3 pal_color = texture(palette_texture, pal_uv).rgb;
        
        if( color_distance(pal_color.rgb, color.rgb) < color_distance(final_color.rgb, color.rgb) ) {
            final_color = pal_color;
        }
    }

    return color;
}

// example usage
func fragment() {
	COLOR.rgb = find_closest_color(COLOR.rgb);
}
```

#### Color Masking
The following additions to the shader allow for defining color masks. Color masks are used to limit the output color to a subset of the given palette. This is represented by a bitfield, which is stored into the uniform int `palette_mask`. For example, if we have an 8-color palette and the `palette_mask` is `15`, or `00001111` in binary, the output color is limited to only the first 4 colors of the palette.
Note that the binary number has as many digits as the number of colors in the palette, and is in reverse order of the color order of the palette texture (the right-most digit corresponds to the left-most color, etc.).

Additional note: It is possible to input binary numbers in the Godot inspector by prefixing the number with `0b`. For example, setting `palette_mask` to `0b11111111` under the shader parameters will output `256`.
``` glsl linenums="1" hl_lines="3 10"
uniform sampler2D palette_texture;  // an image of 8 colors laid out horizontally
uniform int palette_size = 8;  // the number of colors in the palette
uniform int palette_mask = 256;  // 0b11111111

vec3 find_closest_color(vec3 src) {

    vec3 color = vec3(-20.0); 
    
    for( int u = 0; u < palette_size; u += 1 ) {
        if( bitfieldExtract(palette_mask, int(u), 1) == 0u ) { continue; }

        vec2 pal_uv = vec2(float(u) / float(palette_size), 0.0);
        vec3 pal_color = texture(palette_texture, pal_uv).rgb;
        
        if( color_distance(pal_color.rgb, src.rgb) < color_distance(color.rgb, src.rgb) ) {
            color = pal_color;
        }
    }

	return color;
}

// example usage
func fragment() {
	COLOR.rgb = find_closest_color(COLOR.rgb);
}
```

### Palette Blending
The following additions to the shader allow for blending between two palettes. One palette is used as the "palette key" which is used in determining the initial palette color. Then, the UV of the palette color is used to retrieve colors from `palette_a` and `palette_b`. The final output color is then determined
``` glsl linenums="1" hl_lines="1 2 3 4 11 17 21 25 26 27"
uniform sampler2D palette_key;  // an image of 8 colors laid out horizontally
uniform sampler2D palette_a;
uniform sampler2D palette_b;
uniform float palette_mix = 0.0;
uniform int palette_size = 8;  // the number of colors in the palette
uniform int palette_mask = 256;  // 0b11111111

vec3 find_closest_color(vec3 src) {

    vec3 color = vec3(-20.0); 
    vec2 uv = vec2(-1.0); // UV of the closest color
    
    for(int u = 0; u < palette_size; u += 1) {
        if( bitfieldExtract(palette_mask, int(u), 1) == 0u ) { continue; }

        vec2 pal_uv = vec2(float(u) / float(palette_size), 0.0);
        vec3 pal_color = texture(palette_key, pal_uv).rgb;
        
        if( _distance(pal_color.rgb, src.rgb) < _distance(color.rgb, src.rgb) ) {
            color = pal_color;
            uv = pal_uv;
        }
    }

	vec3 color_a = texture(palette_a, uv).rgb;
	vec3 color_b = texture(palette_b, uv).rgb;
	return mix(color_a, color_b, palette_mix);
}

// example usage
func fragment() {
	COLOR.rgb = find_closest_color(COLOR.rgb);
}
```
### 2 Colors
```glsl
struct ColorPairResult {
    vec3 color_a;
    vec2 uv_1;
    vec3 color_b;
    vec2 uv_2;
    float dist_1;
    float dist_12;
};

/*
* Test a color against a color palette, and return a result struct containing:
* - The UV of the closest palette color found
* - The UV of the 2nd closest palette color found
* - The color distance between the closest palette color and the tested color
* - The color distance between the closest palette color and the 2nd closest palette color
*/
ColorPairResult find_closest_colors(vec3 color) {
    
    //bool[8] mask = {pal_1, pal_2, pal_3, pal_4, pal_5, pal_6, pal_7, pal_8};
    
    float max_dist_1 = 10000.0;
    float max_dist_2 = 10000.0;

    vec2 uv_1 = vec2(-1.0); // UV of the closest color
    vec2 uv_2 = vec2(-1.0); // UV of the 2nd closest color
    
    vec3 color_a = vec3(-20, 0, 0);
    vec3 color_b = vec3(-20, 0, 0);
    
    for(float u = 0.0; u < float(palette_size); u += 1.0) {
        if( bitfieldExtract(palette_mask, int(u), 1) == 0u ) { continue; }
        //if( !mask[int(u)] ) { continue; }

        vec2 pal_uv = vec2(u / float(palette_size), 0.0);
        vec3 pal_color = texture(PALETTE_KEY, pal_uv).rgb;
        float dist = _distance(pal_color, color);
        float closest_dist = _distance(color_a, color);
        
        if( dist < closest_dist ) {

            max_dist_2 = max_dist_1;
            uv_2 = uv_1;
            color_b = color_a;

            max_dist_1 = closest_dist;
            uv_1 = pal_uv;
            color_a = pal_color;
        }
        else if ( dist < _distance(color_b.rgb, color.rgb) ) {
            
            max_dist_2 = dist;
            uv_2 = pal_uv;
            color_b = pal_color;
        }
    }

    ColorPairResult results;
    results.color_a = color_a; // the closest color
    results.color_b = color_b; // the second closest color
    results.uv_1 = uv_1; // UV of the closest color
    results.uv_2 = uv_2; // UV of the second closest color
    // distance between the tested color and the closest color
    results.dist_1 = max_dist_1; 
    // distance between the closest color and second closest color
    results.dist_12 = _distance(color_a.rgb, color_b.rgb);
    return results;
}
```
## Color Conversions
### RGB to CIELAB
```glsl
//
// Port of https://github.com/antimatter15/rgb-lab/blob/master/color.js#L28-L47
// Converts an RGB color to CIE L*ab format.
//
vec3 rgb_to_lab(vec3 rgb) {
    float r = rgb.r / 255.0, g = rgb.g / 255.0, b = rgb.b / 255.0;
    float x, y, z;

    r = (r > 0.04045) ? pow((r + 0.055) / 1.055, 2.4) : r / 12.92;
    g = (g > 0.04045) ? pow((g + 0.055) / 1.055, 2.4) : g / 12.92;
    b = (b > 0.04045) ? pow((b + 0.055) / 1.055, 2.4) : b / 12.92;

    x = (r * 0.4124 + g * 0.3576 + b * 0.1805) / 0.95047;
    y = (r * 0.2126 + g * 0.7152 + b * 0.0722) / 1.00000;
    z = (r * 0.0193 + g * 0.1192 + b * 0.9505) / 1.08883;

    x = (x > 0.008856) ? pow(x, 1.0/3.0) : (7.787 * x) + 16.0/116.0;
    y = (y > 0.008856) ? pow(y, 1.0/3.0) : (7.787 * y) + 16.0/116.0;
    z = (z > 0.008856) ? pow(z, 1.0/3.0) : (7.787 * z) + 16.0/116.0;

    return vec3((116.0 * y) - 16.0, 500.0 * (x - y), 200.0 * (y - z));
}
```
## Color Distance
### CIELAB Delta E Distance
```glsl
float distance_lab(vec3 a, vec3 b) {
	distance(rgb_to_lab(a), rgb_to_lab(b));
}

//
// Port of https://github.com/antimatter15/rgb-lab/blob/master/color.js#L52-L68
// Gets the delta E between two LAB colors.
//
float distance_lab(vec3 rgb_a, vec3 rgb_b) {
    
    const float WHT_L = 1.0, WHT_C = 1.0, WHT_H = 1.0;
    
    vec3 lab1 = rgb_to_lab(rgb_a);
    vec3 lab2 = rgb_to_lab(rgb_b);
    
    float xC1 = sqrt( pow(lab1.y, 2.0) + pow(lab1.z, 2.0) );
    float xC2 = sqrt( pow(lab2.y, 2.0) + pow(lab2.z, 2.0) );
    
    float xDL = lab2.x - lab1.x;
    float xDC = xC2 - xC1;
    float xDE = distance(lab1, lab2);

    float xDH = (xDE * xDE) - (xDL * xDL) - (xDC * xDC);
    if( xDH > 0.0 ) {
        xDH = sqrt(xDH);
    }
    else {
        xDH = 0.0;
    }
    
    float xSC = 1.0 + 0.045 * xC1;
    float xSH = 1.0 + 0.015 * xC1;
    
    xDL /= WHT_L;
    xDC /= WHT_C * xSC;
    xDH /= WHT_H * xSH;

    return sqrt(xDL*xDL + xDC*xDC + xDH*xDH);
}
```
### Luma-corrected Color Distance
```glsl
// https://gist.github.com/yyny/3b54ff26ab4f8fd0aad468dd144191c2
// Luma-corrected RGB color distance
float distance_luma(vec3 a, vec3 b) {
    vec3 diff = b - a;
    float luma1 = (a.x*0.299 + a.y*0.587 + a.z*0.114);
    float luma2 = (b.x*0.299 + b.y*0.587 + b.z*0.114);
    float dluma = luma2 - luma1;
    
    return (sq(diff.x) * 0.299 + sq(diff.y) * 0.587 + sq(diff.z) * 0.114) * sq(dluma) * 0.75;
}
```
## Dithering

### CIELAB Delta E
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