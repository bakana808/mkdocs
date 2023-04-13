BULLETLAB is a bullet-hell game and tool for designing shmup patterns. Inspired by [BulletML](http://www.asahi-net.or.jp/~cs8k-cyu/bulletml/index_e.html).

# Reference

BULLETLAB parses bullet patterns using a JSON file.

Most objects read by the parser are in the form of a JSON object with a single key, with the key identifying the type of object.
For example, Patterns are in the form `{"pattern": { ... }}`.

## Patterns

Patterns describe the actions of either a top-level entity (such as an enemy ship) or a bullet.

```json
{"pattern": {
	"id": < string >,
	"repeat": < int | expr >,
	"do": [ ... ]
}}
```

| Key | Default Value | Description |
|---|---|---|
| `id` | `"_"` | The identifier of the pattern. Other patterns or bullets can refer to this pattern by identifier later in the file.
| `repeat` | `1` | The number of times to repeat the list of actions.
| `do` | `[]` | A list of action objects to perform.

Node that all of the actions in the pattern will be performed per frame (including repeats) unless there is a `Wait` action. Keep this in mind when creating a pattern with large amounts of actions.

#### Examples

```json
// Example: Fires 3 bullets downward in a spread pattern
{"pattern": {
	"id": "3-round-spread",
	"do": [
		{"fire": {"angle": 0}},
		{"fire": {"angle": 5}},
		{"fire": {"angle": -5}},
	]
}}
```

## Expressions

Most numerical parameters for pattern actions can be replaced with a [Godot Expression](https://docs.godotengine.org/en/stable/tutorials/scripting/evaluating_expressions.html). This allows users to describe values as a math expression, as well as allowing access to most math functions in Godot (such as `randf()`).
For some actions, additional variables are also given to use in the expression.

### Speed Variables

These variables are provided to actions with a  `speed` parameter, if using an expression.

| Variable | Description |
|---|---|
| `LAST` | The speed of the last bullet this pattern has fired (starting from `0`).

### Angle Variables

These variables are provided to actions with an `angle` parameter, if using an expression.

| Variable | Description |
|---|---|
| `SHIP` | The angle to the player's ship.
| `LAST` | The angle of the last bullet this pattern as fired (starting from `0`).

### Relative vs. Absolute

Expressions can be defined as relative or absolute by adding `rel;` or `abs;` to the beginning of the expression. All speed and angle parameters are relative by default if not specified.

If the parent to the pattern with this action is a Pattern, relative parameters are relative to the last bullet fired by the parent.
If the parent to the pattern with this action is a Bullet, relative parameters are relative to that parent bullet.

## Actions

###  Wait

Wait for a specified amount of frames.

```json
{"wait": < int | expr >}
```

#### Examples
```json
// Waits for 5 frames
{"wait": 5}

// Waits for a random amount between 1 and 5 frames
{"wait": "randf() * 4 + 1"}

// A pattern firing a 3-round burst of bullets at the player
{"pattern": {
	"id": "3-round-burst",
	"do": [
		{"fire": {"angle": "SHIP"}},
		{"wait": 5},
		{"fire": {"angle": "SHIP"}},
		{"wait": 5},
		{"fire": {"angle": "SHIP"}}
	]
}}
```

### Fire

Fires a bullet. This will also set the `LAST` variable in future Fire actions to allow for sequential patterns.

```json
{"fire": {
	"speed": < float | expr >,
	"angle": < float | expr >,
	"pattern": < Pattern | string >
}}
```

#### Parameters
| Key | Default Value | Description |
|---|---|---|
| `speed` | `1` | The speed of the bullet (in pixels per frame).
| `angle` | `0` | The angle or direction of the bullet (in degrees). `0` degrees is down.
| `pattern` | `none` | A pattern to run as a subpattern of this bullet. Either a Pattern object or identifier of a previously defined Pattern.

#### Examples
```json
// Fires a fast bullet at the player
{"fire": {"speed": 5, "angle": "SHIP"}}

// Fires a bullet that fires fast bullets at the player every 0.5 seconds
{"fire": {"angle": "60", "pattern": {
	"do": [
		{"wait": 30},
		{"fire": {"speed": "abs; 5", "angle": "SHIP"}}
	]
}}}

// Fires 36 bullets in a circle pattern
{"fire": {"speed": 1, "pattern": {
	"repeat": 35,
	"do": [
		{"fire": {"angle": "LAST + 10"}}
	]
}}}
```
### Speed

Sets the speed of a parent bullet. If the parent is a Pattern, this action does nothing.

```json
// sets the speed over time
{"speed": {"to": < float | expr >, "in": < float | expr >, "curve": < float | expr >}}

// sets the speed instantly (equivalent to {"to": < float | expr >, "in": 0})
{"speed": < float | expr >}
```

#### Parameters
| Key | Default Value | Description |
|---|---|---|
| `to` | required  | The target speed to set to.
| `in` | `0` | The amount of frames to interpolate the speed to the target speed. If 0, sets the speed immediately.
| `curve` | `1.0` | The curve to use for the interpolation. A value of 1.0 is linear. Uses the ease() function in Godot, see [this image](https://raw.githubusercontent.com/godotengine/godot-docs/master/img/ease_cheatsheet.png) for a visual of different curves.

### Angle

Sets the angle of a parent bullet. If the parent is a Pattern, this action does nothing.

```json
// sets the angle over time
{"angle": {"to": < float | expr >, "in": < float | expr >, "curve": < float | expr >}}

// sets the angle instantly (equivalent to {"to": < float | expr >, "in": 0, "curve": 1.0})
{"angle": < float | expr >}
```

#### Parameters
| Key | Default Value | Description |
|---|---|---|
| `to` | required  | The target angle to set to.
| `in` | `0` | The amount of frames to interpolate the angle to the target angle. If 0, sets the angle immediately.
| `curve` | `1.0` | The curve to use for the interpolation. A value of 1.0 is linear. Uses the ease() function in Godot, see [this image](https://raw.githubusercontent.com/godotengine/godot-docs/master/img/ease_cheatsheet.png) for a visual of different curves.

### Subpattern


Adds another pattern to be run alongside the pattern this action is in. Note that the pattern will not wait until the subpattern is finished. All subpatterns run in parallel to the parent pattern.

```json
{"pattern": < Pattern | string >}
```

Uses the same syntax as a normal pattern if an object is provided (see: [[#Patterns]]).
If a string is provided, uses a previously defined Pattern by identifier.

#### Examples

```json
// Fire a series of circle patterns
[
	{"pattern": {
		"id": "circle",
		"repeat": 36,
		"do": [
			{"fire": {"angle": "LAST + 10"}}
		]
	}},

	{"pattern": {
		"repeat": 10,
		"do": [
			{"pattern": "circle"},
			{"wait": 30}
		]
	}}
]
```