#

![](https://github.com/mzthr78/docs/blob/master/dev/godot/image/animated_sprite_01.png)
![](https://github.com/mzthr78/docs/blob/master/dev/godot/image/animated_sprite_02.png)
![](https://github.com/mzthr78/docs/blob/master/dev/godot/image/animated_sprite_03.png)

```
extends CharacterBody2D

@onready var anim_sprite = $AnimatedSprite2D

func _ready() -> void:
	anim_sprite.play("default")
```
