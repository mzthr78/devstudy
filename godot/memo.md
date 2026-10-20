# メモ・備忘録・小ネタとかそういうの

## 一時停止(Pause)

```gdscript
get_tree().paused = true
```

## CanvasItemの色とか変えられる？

```gdscript
(CanvasItem.)modulate = Color(1, 0, 0, 1)

(CanvasItem.)modulate.a = 0 # で、透明度変えたり
```
