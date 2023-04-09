# Creating an FPS Player Controller

This is a guide on how to create an FPS player controller from scratch in Godot 4.0. This controller will feature:

- A first-person camera that is controlled by the mouse
- Walking and strafing with the WASD keys
- Jumping with the Spacebar
- Accurate collision with floors, walls, and ramps in the world (stairs TBD)

We will be implementing Quake-style movement physics in this controller.

## Input Mapping Setup

## Creating the Main Scene

Our main scene will contain 

## 


``` gdscript linenums="1"
extends CharacterBody3D

#var velocity = Vector3.ZERO

var MAX_AIR_SPEED = 2
var MAX_GROUND_SPEED = 16
var MAX_ACCEL = 10 * MAX_GROUND_SPEED
var MAX_AIR_ACCEL = 100
const FRICTION = 6
const CROUCH_FRICTION = 2
const STOP_SPEED = 2

var total_pitch = 0.0  # reference to current pitch
var total_yaw = 0.0

var wishdir = Vector3.ZERO        # player's intended movement direction

var ticks_grounded = 0  # number of ticks this player was on the floor

@onready
var crouch_tween = create_tween()

@onready
var looktween = create_tween()

func right():
	Vector3.RIGHT.rotated(Vector3.UP, $camera.rotation.y).normalized()

func _ready():
	Input.set_mouse_mode(Input.MOUSE_MODE_CAPTURED)

func _input(event):
	if event is InputEventMouseMotion:
		# update camera
		var yaw = event.relative.x * -0.002
		var pitch = event.relative.y * -0.002
		
		pitch = clamp(pitch, -(PI/2) - total_pitch, (PI/2) - total_pitch)
		total_pitch += pitch
		total_yaw += yaw

		$look.rotation.y += yaw
		$look.rotate($look.transform.basis.x, pitch)
		
func get_yaw():
	return rotation.y
	
func fix_angle():
	pass
	#$look.rotate_y(rotation.y)
	#rotation.y = 0
	#rotation.z = 0
	#rotation.x = 0
	#total_pitch = $camera.rotation.x
		
func _process(_delta):

	# viewmodel angle smoothing
	var a = $viewmodel.transform.basis.get_rotation_quaternion()
	var b = $camera.transform.basis.get_rotation_quaternion()
	$viewmodel.transform.basis = Basis(a.slerp(b, 0.2))
	
	wishdir = Vector3.ZERO
	
	if Input.is_action_pressed("move_forward"):
		wishdir += Vector3.FORWARD
		
	if Input.is_action_pressed("move_back"):
		wishdir += Vector3.BACK
		
	if Input.is_action_pressed("move_left"):
		wishdir += Vector3.LEFT
		
	if Input.is_action_pressed("move_right"):
		wishdir += Vector3.RIGHT

	if wishdir != Vector3.ZERO:
		wishdir = wishdir.rotated(Vector3.UP, $camera.rotation.y).normalized()
		wishdir.y = 0

	if Input.is_action_just_pressed("quit"):
		get_tree().quit()
	
func get_speed() -> float:
	return Vector2(velocity.x, velocity.z).length()
	
func friction(delta):
	var friction
	if Input.is_action_pressed("crouch"):
		friction = CROUCH_FRICTION
	else:
		friction = FRICTION
		
	var speed = get_speed()
	if speed < STOP_SPEED: speed = STOP_SPEED
	
	var new_speed = speed - (speed * friction * delta)
	if new_speed < 0: new_speed = 0
	
	velocity = velocity * (new_speed / speed)
	
# Q2/3 style movement    
func accelerate(wishdir: Vector3, delta: float):
	
	var wishspeed
	var accel
	
	if is_on_floor():
		if not Input.is_action_pressed("jump"):
			friction(delta)
		wishspeed = MAX_GROUND_SPEED
		accel = MAX_ACCEL
	else:
		wishspeed = MAX_AIR_SPEED
		accel = MAX_AIR_ACCEL
	
	var currentspeed = velocity.dot(wishdir)
	var addspeed = clamp(wishspeed - currentspeed, 0, accel * delta)

	velocity += addspeed * wishdir

	
# get the player's intended camera angle
func get_wishlook() -> Basis:
	if not tether:
		return $look.transform.basis.orthonormalized()
	else:
		var rail_rotation = tether.get_follow_angle()
		var wishang = tether.follow.transform.basis.orthonormalized()
		wishang = wishang.rotated(wishang.y, total_yaw -rail_rotation.y)
		wishang = wishang.rotated(wishang.x, total_pitch -rail_rotation.x)
		return wishang

 
func _physics_process(delta):
	
	var wishang = $look.transform.basis.orthonormalized()
	
	if Input.is_action_pressed("crouch"):
		crouch_tween.stop()
		crouch_tween.tween_property($look, "position:y", 0.7, 0.1)
		crouch_tween.play()
	else:
		crouch_tween.stop()
		crouch_tween.tween_property($look, "position:y", 2.0, 0.1)
		crouch_tween.play()
		
	# camera tilt
	var local = $look.to_local(position + velocity)
	wishang = wishang.rotated(wishang.z, -local.x * 0.002)

	if not is_on_floor():
		velocity.y -= 25 * delta
			
	elif is_on_floor():
		if Input.is_action_pressed("jump"):
			floor_snap_length = 0.0
			var norm = $camera/down.get_collision_normal()
			velocity += norm * Vector3(0, 10, 0).dot(norm)
			velocity.y = max(velocity.y, 10)
		else:
			velocity.y = -0.5
			
	var camera_smoothing = 0.1
	
	if tether:
		var rail_rotation = tether.get_follow_angle()
		wishang = tether.follow.transform.basis.orthonormalized()
		wishang = wishang.rotated(wishang.y, total_yaw -rail_rotation.y)
		wishang = wishang.rotated(wishang.x, total_pitch -rail_rotation.x)
		camera_smoothing = 0.1
		#wishang *= Quat(Vector3.UP, total_yaw)
		#wishang *= Quat(Vector3.RIGHT, total_pitch)
		
	if not tether:
		accelerate(wishdir, delta)         
		move_and_slide()
		
		
	$camera.transform.basis = wishang

```