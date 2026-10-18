# メモ・備忘録・小ネタとかそういうの

## 一時停止(Pause)

```cs
Time.timeScale = 0.0f
```

> [!WARNING]
> これで止まるのはFixedUpdate()だけ。Update()は動く。\
> deltaTimeはゼロになる。
