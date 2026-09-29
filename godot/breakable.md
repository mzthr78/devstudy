(壊せるオブジェクトのBlenderでの作り方は[こちら](https://github.com/mzthr78/devstudy/blob/master/blender/destruction.md))

(Godot4.7.2)

![](https://github.com/mzthr78/devstudy/blob/master/godot/image/breakable/breakable000.gif)
![](https://github.com/mzthr78/devstudy/blob/master/godot/image/breakable/breakable001.png)
![](https://github.com/mzthr78/devstudy/blob/master/godot/image/breakable/breakable002.png)

(ざっくり). 
RigidBodyに破片のメッシュを追加してそれを親に追加して動かして消す

main.gd
```
extends Node3D

@export var speed: float = 3
@export var min_lifetime: float = .6
@export var max_lifetime: float = .8

func _input(event: InputEvent) -> void:
	if event.is_action_pressed("ui_select"):
		explode()

func explode():
	var parent = get_parent()
	queue_free()
	
	for child in $cube_breakable.get_children():
		if child is MeshInstance3D:
			var fragment: Fragment = Fragment.new(child)
			parent.add_child(fragment)
			
			fragment.linear_velocity = Vector3(fragment.global_transform.origin - $Marker3D.global_transform.origin).normalized() * speed
			fragment.lifetime = randf_range(min_lifetime, max_lifetime)
```

fragment.gd
```
extends RigidBody3D
class_name Fragment

var collision: CollisionShape3D
var lifetime = 0

func _init(source: MeshInstance3D) -> void:
	global_transform = source.global_transform
	# ここで　print(global_transform)　しても出力できない？
	
	collision = CollisionShape3D.new()
	collision.shape = source.mesh.create_convex_shape()
	add_child(collision)
	
	var mesh: MeshInstance3D = source.duplicate()
	mesh.transform = global_transform
	add_child(mesh)
	
func _process(delta: float) -> void:
	lifetime -= delta
	if lifetime <= 0:
		queue_free()
```
