# 穴掘り法

![](https://github.com/mzthr78/devstudy/blob/master/godot/image/maze/maze_generate_dfs.png)

たぶん。やり方だけ見てコードは自分で考えてみたから合ってるかわからん。

(スタック＆ループ)
```
extends Node2D

@onready var tilemaplayer = $TileMapLayer

var sizeX = 71
var sizeY = 39

const DIR = [
	Vector2i(0, -1), # UP
	Vector2i(1, 0), # RIGHT
	Vector2i(0, 1), # DOWN
	Vector2i(-1, 0), # LEFT
]

# Called when the node enters the scene tree for the first time.
func _ready() -> void:
	tilemaplayer.clear()
	
	var grid: Array = []

	# gridの初期化
	for y in range(sizeY):
		var tmp: Array = []
		for x in range(sizeX):
			tmp.append(1)
		grid.append(tmp)
	
	# この書き方はいいのか？
	grid = dig(grid)
	
	# 描画(grid -> tilemap)
	for y in range(grid.size()):
		for x in range(grid[y].size()):
			tilemaplayer.set_cell(Vector2i(x, y), 0, getPanel(grid[y][x]))

func dig(grid: Array) -> Array:
	var dir = [0, 1, 2, 3]
	
	var memo = []

	var current = Vector2i(1, 1)
	grid[current.y][current.x] = 0
	memo.push_front(current)
	
	while(current):
		dir.shuffle()
		var isDig = false
		for i in dir:
			var to1 = current + DIR[i]
			var to2 = current + DIR[i] * 2
			
			if to2.x < 0 || to2.y < 0 || to2.x > sizeX - 2 || to2.y > sizeY - 2: continue
			
			if grid[to2.y][to2.x] == 1: 
				grid[to1.y][to1.x] = 0
				grid[to2.y][to2.x] = 0
				memo.push_front(to2)
				current = to2
				isDig = true
				break
			
		if !isDig:
			current = memo.pop_front()
	
	return grid

func getPanel(index: int = 0) -> Vector2i:
	# "2"固定じゃあかん
	return Vector2i(index % 2, int(index / float(2))) # Warning出さないためのfloat()とint()
```

(再帰処理版)
```
extends Node2D

@onready var tilemaplayer = $TileMapLayer

var sizeX = 71
var sizeY = 39

const DIR = [
	Vector2i(0, -1), # UP
	Vector2i(1, 0), # RIGHT
	Vector2i(0, 1), # DOWN
	Vector2i(-1, 0), # LEFT
]

# Called when the node enters the scene tree for the first time.
func _ready() -> void:
	tilemaplayer.clear()
	
	var grid: Array = []

	# gridの初期化
	for y in range(sizeY):
		var tmp: Array = []
		for x in range(sizeX):
			tmp.append(1)
		grid.append(tmp)
		
	init_grid()
	
	digr(Vector2i(1, 1))

	for y in range(ggrid.size()):
		for x in range(ggrid[y].size()):
			tilemaplayer.set_cell(Vector2i(x, y), 0, getPanel(ggrid[y][x]))
	

var ggrid: Array = []

func init_grid():
	for y in range(sizeY):
		var tmp: Array = []
		for x in range(sizeX):
			tmp.append(1)
		ggrid.append(tmp)

func digr(current: Vector2i):
	ggrid[current.y][current.x] = 0 # ここは下にして初期値は外に書いたほうがいいかも
	var dir = [0, 1, 2, 3]
	dir.shuffle()
	for i in dir:
		var to1 = current + DIR[i]
		var to2 = current + DIR[i] * 2
		
		if to2.x < 0 || to2.y < 0 || to2.x > sizeX - 2 || to2.y > sizeY - 2: continue
		
		if ggrid[to2.y][to2.x] == 1:
			ggrid[to1.y][to1.x] = 0
			#ggrid[to2.y][to2.x] = 0 #
		
			digr(to2)
	
func getPanel(index: int = 0) -> Vector2i:
	# "2"固定じゃあかん
	return Vector2i(index % 2, int(index / float(2))) # Warning出さないためのfloat()とint()
```
