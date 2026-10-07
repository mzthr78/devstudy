# 玉転がし(3D)

![](https://github.com/mzthr78/devstudy/blob/master/godot/image/rollaball/rollaball000.gif)
![](https://github.com/mzthr78/devstudy/blob/master/godot/image/rollaball/rollaball001.png)

ボール(ball.gd)
```
extends RigidBody3D

var direction = Vector3.FORWARD
var smooth_speed = 2.5

@onready var marker: Marker3D = $Marker3D

var speed = 3
var force_factor = 0.01

func _ready() -> void:
	marker.top_level = true

#func _physics_process(delta: float) -> void:
func _physics_process(_delta: float) -> void:
	marker.global_transform.origin = global_transform.origin
	
	var input_dir := Input.get_vector("ui_left", "ui_right", "ui_up", "ui_down")
	#var direction = (transform.basis * Vector3(input_dir.x, 0, input_dir.y)).normalized()
	var force = Vector3(input_dir.x, 0, input_dir.y) * force_factor
	
	if input_dir:
		apply_impulse(force * speed, position)
```

全体管理(rollaball.gd)
```
extends Node3D
class_name rollaball

var collectible_scene = load("res://rollaball/collectible.tscn")

@onready var TimeLabel: Label = $Control/Label2
@onready var CountLabel: Label = $Control/Label4
@onready var ClearLabel: Label = $Control/Label5

var count = 8
var radius = 3
@onready var start_time = Time.get_unix_time_from_system()

var isPaused = false

# Called when the node enters the scene tree for the first time.
func _ready() -> void:
	ClearLabel.visible = false
	CountLabel.text = str(count)
	
	for i in range(8):
		var collectible: Area3D = collectible_scene.instantiate()
		var angle = deg_to_rad(int(360.0 / count) * i)
		collectible.position = Vector3(0, 0.5, 0) + Vector3(cos(angle), 0, sin(angle)) * radius
		#collectible.look_at(Vector3(0, 0.5, 0)) # これは効かない
		collectible.connect("collected", _on_collected)
		
		add_child(collectible)
		
	#self.connect("collected", _on_collected)
func _process(delta: float) -> void:
	if isPaused: return
	TimeLabel.text = "%.5f" % (Time.get_unix_time_from_system() - start_time)
	
func _input(event: InputEvent) -> void:
	# 録画用に追加
	if event.is_action_pressed("ui_cancel"):
		isPaused = !isPaused
		
		
func _on_collected():
	count -= 1
	CountLabel.text = str(count)
	
	if count <= 0:
		ClearLabel.visible = true
		get_tree().paused = true
```


アイテム(collectible.gd)
```
extends Area3D

signal collected

# Called when the node enters the scene tree for the first time.
func _ready() -> void:
	self.connect("body_entered", _on_body_entered.bind())
	
	look_at(Vector3(0, 0.5, 0))
	
func _on_body_entered(body: Node3D) -> void:
	if body.name != "Ball": return
	collected.emit()
	queue_free()
```
