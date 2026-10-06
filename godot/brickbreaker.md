# ブロック崩し

![](https://github.com/mzthr78/devstudy/blob/master/godot/image/brickbreaker/brickbreaker000.gif)

メインシーン\
![](https://github.com/mzthr78/devstudy/blob/master/godot/image/brickbreaker/brickbreaker001.png)
パドル(CharacterBody2D)とボール(CharacterBody2D)と壁(画面上左右にコライダー)と穴？（Area2D)とメッセージ表示用ラベルを配置。


ブロックシーン\
![](https://github.com/mzthr78/devstudy/blob/master/godot/image/brickbreaker/brickbreaker002.png)
ブロック(RigidBody2D)

メインスクリプト(brickbreaker.gd)
- ブロックの生成
- メッセージ表示
- 残ブロック数
- ミス（残ライフ）
```
extends Node2D

@onready var brick_scene = preload("res://brickbreaker/brick.tscn")
@onready var abyss: Area2D = $Abyss
@onready var ball: CharacterBody2D = $Ball
@onready var paddle: CharacterBody2D = $Paddle
@onready var label: Label = $Info

var life: int = 3
var count: int = 5 * 3

func _ready() -> void:
	label.text = "Press Space to start"
	
	abyss.connect("body_entered", _on_abyss_entered) # area_enteredじゃなくてbody_entered
	
	for j in range(3):
		for i in range(5):
			var brick: Brick = brick_scene.instantiate()
			
			var x = 350 + 100 * i
			var y = 100 + 50 * j
			
			brick.position = Vector2(x, y)
			brick.connect("broken", _on_brick_broken)
			add_child(brick)

func _input(event: InputEvent) -> void:
	if event.is_action_pressed("ui_accept"):
		ball.velocity = (paddle.position - ball.position).normalized() * ball.speed
		
		label.visible = false


func _on_brick_broken() -> void:
	count -= 1
	
	if count <= 0:
		label.text = "CLEAR!"
		label.visible = true

		get_tree().paused = true
	
func _on_abyss_entered(body: Node2D) -> void:
	if body.name != "Ball": return
	
	life -= 1
	
	if life > 0:
		label.text = "Press Space to start"
		
		await get_tree().create_timer(0.5).timeout
		
		ball.position = Vector2(592.0, 360.0)
		ball.velocity = Vector2.ZERO
	else:
		label.text = "GAME OVER"
		label.visible = true
		get_tree().paused = true
		
	label.visible = true
```

パドルスクリプト(paddle.gd)
- パドルの移動
```
extends CharacterBody2D

#const SPEED: float = 700
const SPEED: float = 10

func _physics_process(_delta: float) -> void:
	var direction = Input.get_axis("ui_left", "ui_right")
	
	if direction:
		velocity.x = direction * SPEED
	else:
		velocity.x = move_toward(velocity.x, 0, SPEED)
		
	velocity.y = 0
	
	move_and_collide(velocity)
```

ボールスクリプト(ball.gd)
- 衝突処理
```
extends CharacterBody2D
class_name Ball

const speed = 10
var interval: float = 0.0

func _physics_process(delta: float) -> void:
	var collision = move_and_collide(velocity)
	
	if collision:
		var collider = collision.get_collider()
		
		if collider is Brick:
			(collider as Brick).destroy()
			
	# ぶつかったとき反射しないことがあるので間隔を調整
	if collision && interval <= 0:
		velocity = velocity.bounce(collision.get_normal())
		interval = 0.08
	else:
		interval -= delta

		# こういう方法もあるらしい
		#move_and_collide(collision.get_remainder().bounce(collision.get_normal()))
```

ブロックスクリプト(brick.gd)
- 壊れたことを通知
```
extends RigidBody2D
class_name Brick

signal broken

func destroy():
	broken.emit()
	queue_free()
```
