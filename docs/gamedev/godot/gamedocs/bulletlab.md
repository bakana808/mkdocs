BULLETLAB is a bullet-hell game and tool for designing shmup patterns.
Inspired by [BulletML](http://www.asahi-net.or.jp/~cs8k-cyu/bulletml/index_e.html).

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

#### Example: A pattern firing 3 bullets downwards in a spread

```json
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

Most numerical parameters for pattern actions can be substituted with a [Godot Expression](https://docs.godotengine.org/en/stable/tutorials/scripting/evaluating_expressions.html). This gives access to most math functions (such as `randf()`).
For some actions, additional variables are also given to use in the expression.

### Speed Variables

These variables are provided to actions with a  `speed` parameter, if using an expression.
| Variable | Description |
|---|---|
| `LAST` | The speed of the last bullet this pattern has fired.

### Angle Variables

These variables are provided to actions with an `angle` parameter, if using an expression.
| Variable | Description |
|---|---|
| `SHIP` | The angle to the player's ship.
| `LAST` | The angle of the last bullet this pattern as fired.

### Relative vs. Absolute

Expressions can be defined as relative or absolute by adding `rel;` or `abs;` to the beginning of the expression. All speed and angle parameters are relative unless specified otherwise.

If the parent to the pattern with this action is a Pattern, relative parameters are relative to the last bullet fired by the parent.
If the parent to the pattern with this action is a Bullet, relative parameters are relative to that parent bullet.

#### Example: Fire a bullet 

## Actions

###  Wait

Wait for a specified amount of frames.

```json
{"wait": < int | expr >}
```

#### Example: Wait 5 frames
```json
{"wait": 5}
```

#### Example: Wait randomly between 1 and 5 frames
```json
{"wait": "randf() * 4 + 1"}
```

#### Example: A pattern firing a 3-bullet burst at the player
```json
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

Fire a bullet.

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

