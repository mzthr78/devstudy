# tilemaplayer

TileMapLayerを追加\
![](https://github.com/mzthr78/devstudy/blob/master/godot/image/tilemaplayer/tilemaplayer_001.png)

新規TileSet\
![](https://github.com/mzthr78/devstudy/blob/master/godot/image/tilemaplayer/tilemaplayer_002.png)

下のTileSetに画像を追加\
![](https://github.com/mzthr78/devstudy/blob/master/godot/image/tilemaplayer/tilemaplayer_003.png)

TileSetでタイルを選択\
![](https://github.com/mzthr78/devstudy/blob/master/godot/image/tilemaplayer/tilemaplayer_004.png)

TileMapに切り替えてタイルを並べる\
![](https://github.com/mzthr78/devstudy/blob/master/godot/image/tilemaplayer/tilemaplayer_005.png)

コードから並べるならこんな
```
extends Node2D

@onready var tilemaplayer = $TileMapLayer

# Called when the node enters the scene tree for the first time.
func _ready() -> void:
	tilemaplayer.clear()
	
	tilemaplayer.set_cell(Vector2i(0, 0), 0, Vector2i(0, 0))
	tilemaplayer.set_cell(Vector2i(1, 0), 0, Vector2i(1, 0))
	tilemaplayer.set_cell(Vector2i(2, 0), 0, Vector2i(2, 0))


# Called every frame. 'delta' is the elapsed time since the previous frame.
func _process(delta: float) -> void:
	pass
```
