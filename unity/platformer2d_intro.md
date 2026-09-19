#

![](https://github.com/mzthr78/docs/blob/master/dev/unity/image/platformer2d_intro.gif)


![](https://github.com/mzthr78/docs/blob/master/dev/unity/image/platformer2d_intro_000.png)

{A)\
![](https://github.com/mzthr78/docs/blob/master/dev/unity/image/platformer2d_intro_002.png)

(B)\
![](https://github.com/mzthr78/docs/blob/master/dev/unity/image/platformer2d_intro_001.png)

Physics Materialを追加. 
Create -> Physics Material\
![](https://github.com/mzthr78/docs/blob/master/dev/unity/image/platformer2d_intro_003.png)

\
![](https://github.com/mzthr78/docs/blob/master/dev/unity/image/platformer2d_intro_004.png)

\
![](https://github.com/mzthr78/docs/blob/master/dev/unity/image/platformer2d_intro_005.png)

```player.cs
using UnityEngine;
using UnityEngine.InputSystem;

public class PlayerIntro : MonoBehaviour
{
    Rigidbody2D rb;

    float moveH;
    float speed = 5f;
    float jump;

    // Start is called once before the first execution of Update after the MonoBehaviour is created
    void Start()
    {
        rb = GetComponent<Rigidbody2D>();
        jump = rb.gravityScale * 8;
    }

    // Update is called once per frame
    void FixedUpdate()
    {
        rb.linearVelocityX = moveH * speed;
    }


    private void OnMove(InputValue value)
    {
        moveH = value.Get<Vector2>().x;
    }

    private void OnJump(InputValue value)
    {
        rb.linearVelocityY = jump;
    } 

    /*
    public void Move(InputAction.CallbackContext ctx)
    {
        moveH = ctx.ReadValue<Vector2>().x;
        Debug.Log("moveH=" + moveH);
    }

    public void Jump(InputAction.CallbackContext ctx)
    {
        rb.linearVelocityY = jump;
    }
    */
}
```
