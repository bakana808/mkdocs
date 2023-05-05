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