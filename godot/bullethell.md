# Bullet Hell

![](https://github.com/mzthr78/devstudy/blob/master/godot/image/bullethell/bullethell000.gif)

部品構成\
![](https://github.com/mzthr78/devstudy/blob/master/godot/image/bullethell/bullethell001.png)
![](https://github.com/mzthr78/devstudy/blob/master/godot/image/bullethell/bullethell002.png)
![](https://github.com/mzthr78/devstudy/blob/master/godot/image/bullethell/bullethell003.png)

- メイン
  - Player(Area2D)
    - Sprite2D
    - CollisionShape2D
  - Enemy(Area2D)
    - Sprite2D
    - CollisionShape2D
  - PlayerLifeLabel
  - ProgressBar
  - InfoLabel
  - PlayerExplosion
  - EnemyExplosion
- 弾
  - Bullet(Area2D)
    - Sprite2D
    - CollisionShape2D
- 発射パターン制御用
  - turret(Node2D)

> [!WARNING]
> 制御がきちゃないっす（たぶん設計がよくない）

bullethell.gd
```gdscript:bullethell.gd
extends Node2D
class_name BulletHellManager

signal enemy_destroyed
signal player_destroyed

@onready var player: Area2D = $Player
@onready var player_sprite: Sprite2D = $Player/Sprite2D
@onready var player_explosion: GPUParticles2D = $PlayerExplosion

@onready var enemy: Area2D = $Enemy
@onready var enemy_sprite: Sprite2D = $Enemy/Sprite2D
@onready var enemy_explosion: GPUParticles2D = $EnemyExplosion

@onready var progress_bar: ProgressBar = $ProgressBar
@onready var player_life_label: Label = $PlayerLifeLabel
@onready var info_label: Label = $InfoLabel

var enemy_life = 100.0
var player_life = 3

var is_ready = false
var is_gameover = false
var is_clear = false

var timer = 3

func _ready() -> void:
	progress_bar.value = 100
	progress_bar.modulate.a = 0
	
	enemy.connect("enemy_go", _on_enemy_go.bind())
	
	Info("")
	
func _process(delta: float) -> void:
	progress_bar.value = lerp(progress_bar.value, enemy_life, 5 * delta)
	if player_life > 0:
		player_life_label.text = "🤍".repeat(player_life)
	else:
		player_life_label.text = ""
	
	if is_ready && progress_bar.modulate.a < 1:
		progress_bar.modulate.a = lerp(progress_bar.modulate.a, 1.0, 5 * delta)
		
	if is_gameover || is_clear:
		timer -= delta
		
	if timer <= 0:
		if is_gameover:
			Info("GAME OVER")
		else:
			Info("CLEAR!")
			
func Info(info: String = ""):
	info_label.visible = (info != "")
	info_label.text = info

func _on_enemy_go():
	is_ready = true

func _on_player_hit():
	if !is_ready || is_gameover: return
	
	player_life -= 1
	
	if player_life <= 0:
		player_explosion.global_position = player.global_position
		player_sprite.visible = false
		player_explosion.one_shot = true
		player_explosion.emitting = true
		
		player_destroyed.emit()
		
		is_gameover = true

		# こうしちゃうと撃った弾がピタって止まっちゃう
		#player.process_mode = Node.PROCESS_MODE_DISABLED
		
func _on_enemy_hit():
	if !is_ready: return
	
	enemy_life -= 10
	
	if enemy_life <= 0:
		enemy_explosion.global_position = enemy.global_position
		enemy_sprite.visible = false
		enemy_explosion.one_shot = true
		enemy_explosion.emitting = true
		
		enemy_destroyed.emit()
		
		is_clear = true
		
		# こうしちゃうと撃った弾がピタって止まっちゃう
		#enemy.process_mode = Node.PROCESS_MODE_DISABLED
```

player.gd
```gdscript:player.gd
extends Area2D

var bullet_scene: PackedScene = load("res://bullethell/bullet.tscn")

@onready var bullet_hell: BulletHellManager = $".."

var speed = 250.0

var is_alive = true

func _ready() -> void:
	global_position = Vector2(float(get_window().size.x) / 2, float(get_window().size.y) / 4 * 3)
	bullet_hell.connect("player_destroyed", _on_player_destroyed.bind())

func _input(event: InputEvent) -> void:
	if !is_alive: return
	
	if event.is_action_pressed("ui_accept"):
		fire()
		
func _physics_process(delta: float) -> void:
	var direction: Vector2 = Input.get_vector("ui_left", "ui_right", "ui_up", "ui_down")
	
	position += direction * speed * delta

func fire():
	var bullet = bullet_scene.instantiate()
	bullet.global_position = global_position
	bullet.rad = - PI / 2
	bullet.add_to_group("player")
	add_child(bullet)
	
func _on_player_destroyed():
	is_alive = false
```

enemy.gd
```gdscript:enemygd
extends Area2D

@onready var bullet_hell: BulletHellManager = $".."
@onready var player: Area2D = $"../Player"

var bullet_scene: PackedScene = load("res://bullethell/bullet.tscn")

signal enemy_go

var speed = 100

var inc = 0
	
var interval1 = .5
var angle1 = 0
var timer1 = 0

var interval2 = .05
var angle2 = 0
var timer2 = 0

var interval3 = 5
var timer3 = interval3
var isHoge = false

var pattern: int = 0

var turret: Turret #= Fire.new(self, .1, deg_to_rad(90))

var is_destroyed = false

func _ready() -> void:
	bullet_hell.connect("enemy_destroyed", _on_enemy_destroyed.bind())

	global_position.x = get_window().size.x / 2.0
	global_position.y = -100
	
	pattern = 0
	
	targetX = get_window().size.x / 2.0
	targetY = get_window().size.y / 3.0
	
	turret = Turret.new(self, 0, .1, deg_to_rad(90.0))
	#rotation += deg_to_rad(30)
	
var targetX = 0.0
var targetY = 0.0

var timer = 0
var count = 0
var angle = 0

var is_active = false
var turret_pattern = 0

func _process(delta: float) -> void:
	if is_destroyed: 
		turret.enabled = false
		return
	
	look_at(player.global_position)
	
	inc += delta
	
	global_position.x = lerp(global_position.x, targetX, delta)
	global_position.y = lerp(global_position.y, targetY, delta)
	
	timer -= delta
	
	match(pattern):
		0:
			if abs(targetY - global_position.y) < 20:
				enemy_go.emit()
				
				timer = .5
				count = 5
				
				pattern = 1
				
				turret.interval = .5
				turret.pattern = 3
				turret.speed = 500

				turret.enabled = true
		1:
			# ここは変更
			if timer <= 0:
				timer = .5
				count += 1
				
			if count >= 5:
				targetX = get_window().size.x / 8.0
				timer = .5
				pattern = 2
			
		2:
			if abs(targetX - global_position.x) < 10:
				targetX = get_window().size.x - get_window().size.x / 8.0
				timer = .2
				pattern = 3
				count = 0
				angle = 120
	
				turret.interval = .01
				turret.pattern = 2
				turret.speed = 800
		3:
			if abs(targetX - global_position.x) < 10:
				targetX = get_window().size.x / 8.0
				timer = 0
				pattern = 1
	
				turret.interval = .5
				turret.pattern = 3
				turret.speed = 500

	# ∞の動きはこれ
	#inc += delta
	#global_position.x = get_window().size.x / 2 + sin(inc) * (get_window().size.x / 3)
	#global_position.y = get_window().size.y / 3 + sin(inc * 2) * (get_window().size.y / 4)
	
func _on_enemy_destroyed():
	is_destroyed = true
```

bullet.gd
```gdscript:bullet.gd
extends Area2D
class_name Bullet

signal player_hit
signal enemy_hit

@onready var manager: BulletHellManager

var dir: Vector2 = Vector2.ZERO	
var speed: float = 500
var rad: float = 0

func _ready() -> void:
	self.dir = Vector2.RIGHT.rotated(self.rad) * speed
	top_level = true
	
	manager = get_node("/root/BulletHell")
	
	self.connect("area_entered", _on_area_entered.bind())
	
	connect("player_hit", manager._on_player_hit.bind())
	connect("enemy_hit", manager._on_enemy_hit.bind())

func _process(delta: float) -> void:
	global_position += dir * delta
	
	if global_position.x < -10 || global_position.x > get_window().size.x + 10 \
	|| global_position.y < -10 || global_position.y > get_window().size.y + 10:
		
		queue_free()
		
func _on_area_entered(area: Area2D):
	if is_in_group("player") && area.name == "Enemy":
		enemy_hit.emit()
		queue_free()
		
	if is_in_group("enemy") && area.name == "Player":
		player_hit.emit()
		queue_free()
```

turret.gd
```gdscript:turret.gd
extends Node2D
class_name Turret

var parent: Node2D
var bullet_scene: PackedScene = load("res://bullethell/bullet.tscn")
#@onready var player: Area2D = $Player # これはできなさそう
var player: Area2D

var interval = 0
var timer = 0
var rad = 0
var pattern = 0
var speed = 300

var enabled: bool = true:
	set(new_value):
		if new_value:
			self.process_mode = Node.PROCESS_MODE_INHERIT
		else:
			self.process_mode = Node.PROCESS_MODE_DISABLED
	get:
		return enabled

func _init(parent_obj, turret_pattern = 0, fire_interval = 0.0, launch_rad = 0.0) -> void:
	
	self.interval = fire_interval
	self.timer = fire_interval
	self.parent = parent_obj
	self.rad = launch_rad
	self.pattern = turret_pattern
	
	self.parent.add_child(self)
	
	process_mode = Node.PROCESS_MODE_DISABLED

func _ready() -> void:
	player = get_node("/root/BulletHell/Player")

func _process(delta: float) -> void:
	timer -= delta
	
	if timer > 0: return
	
	match(self.pattern):
		0:
			rad = deg_to_rad(90)
			fire(rad, speed)
	
		1:
			rad = (player.global_position - parent.global_position).angle()
			fire(rad, speed)
		
		2:
			rad += PI / 12
			fire(rad, speed)
	
		3:
			for i in range(24):
				rad = PI / 12 * i
				fire(rad, speed)
		
	timer = interval

func ChangePattern(turret_pattern):
	self.pattern = turret_pattern

func SetInterval(fire_interval = 0):
	self.interval = fire_interval

func fire(launch_rad = 0, launch_speed = 300) -> void:
	var bullet: Bullet = bullet_scene.instantiate()
	bullet.global_position = parent.global_position #Vector2(100, 100)
	bullet.rad = launch_rad #deg_to_rad(90)
	bullet.speed = launch_speed
	bullet.add_to_group("enemy")
	parent.add_child(bullet)
		
```
