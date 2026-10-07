# ブロック崩し

![](https://github.com/mzthr78/devstudy/blob/master/unity/image/brickbreaker/brickbreaker000.gif)
![](https://github.com/mzthr78/devstudy/blob/master/unity/image/brickbreaker/brickbreaker001.png)

- BrickBreaker(Empty) ･･･ 全体管理用
- Paddle(2D Object -> Sprites -> Square) ･･･ パドル
- Ball(2D Object -> Sprites -> Circle) ･･･ ボール
- Wall(Empty)
    - Game Object(Empty) ･･･ 壁(左)
    - Game Object (1)(Empty) ･･･ 壁(左)
    - Game Object (2)(Empty) ･･･ 壁(上)
    - Game Object (3)(Empty) ･･･ 壁(下)

(Prefab)
- Brick(2D Object -> Sprites -> Square)

![](https://github.com/mzthr78/devstudy/blob/master/unity/image/brickbreaker/brickbreaker002.png)
![](https://github.com/mzthr78/devstudy/blob/master/unity/image/brickbreaker/brickbreaker003.png)
![](https://github.com/mzthr78/devstudy/blob/master/unity/image/brickbreaker/brickbreaker004.png)
![](https://github.com/mzthr78/devstudy/blob/master/unity/image/brickbreaker/brickbreaker005.png)
![](https://github.com/mzthr78/devstudy/blob/master/unity/image/brickbreaker/brickbreaker006.png)
![](https://github.com/mzthr78/devstudy/blob/master/unity/image/brickbreaker/brickbreaker007.png)
![](https://github.com/mzthr78/devstudy/blob/master/unity/image/brickbreaker/brickbreaker008.png)

brickbreaker.cs
```
using System;
using System.Collections;
using TMPro;
using UnityEditor.VersionControl;
using UnityEngine;
using UnityEngine.Categorization;

public class brickbreaker : MonoBehaviour
{
    [SerializeField] GameObject ball;
    //[SerializeField] GameObject paddle;
    [SerializeField] GameObject brickPrefab;
    [SerializeField] GameObject abyss;
    [SerializeField] TextMeshProUGUI label;

    int count = 15;
    int life = 3;

    // Start is called once before the first execution of Update after the MonoBehaviour is created
    void Start()
    {
        abyss.GetComponent<abyss>().OnFall += HandleBallFall;

        for (int j = 0; j < 3; j++)
        {
            for (int i = 0; i < 5; i++)
            {
                GameObject brick = Instantiate(brickPrefab, new Vector3(-3.8f + 2 * i, 1.6f + j, 0), Quaternion.identity);

                brick.GetComponent<brick>().OnDestroy += HandleBrickDestroy;
            }
        }
    }

    // Update is called once per frame
    void Update()
    {
        // 旧式InputSystem
        if (Input.GetKeyDown(KeyCode.Space)) // GetKeyだと連続して反応しちゃう？
        {
            Info("");
            ball.GetComponent<ball>().Fire();
        } 
    }

    void HandleBrickDestroy()
    {
        count -= 1;

        if (count <= 0)
        {
            Info("Clear!");
            Time.timeScale = 0f;
            return;
        }
    }

    void HandleBallFall()
    {
        life -= 1;

        if (life <= 0)
        {
            Info("GAME OVER");
            Time.timeScale = 0f;
            return;
        }

        StartCoroutine(Reset());
    }

    private IEnumerator Reset()
    {
        yield return new WaitForSeconds(1);

        ball.transform.position = Vector3.zero;
        ball.GetComponent<Rigidbody2D>().linearVelocity = Vector3.zero;

        Info("Press Space to start");
    }

    private void Info(String message = "")
    {
        label.enabled = (message != "");
        label.text = message;
    }
}
```

paddle.cs
```
using Unity.VisualScripting;
using UnityEngine;
using UnityEngine.InputSystem;

public class paddle : MonoBehaviour
{
    Rigidbody2D rb;

    float moveH = 0;
    float speed = 10f;

    void Awake()
    {
        rb = GetComponent<Rigidbody2D>();
    }

    void Update()
    {
        // たぶんこれは旧式InputSystem
        moveH = Input.GetAxis("Horizontal");
    }

    void FixedUpdate()
    {
        //rb.AddForce(new Vector2(moveH, 0), ForceMode2D.Impulse); 
        rb.linearVelocityX = moveH * speed; // こっちのほうがいいかな
    }

    /*
    private void OnMove(InputValue value)
    {
        moveH = value.Get<Vector2>().x;
    }

    private void OnJump(InputValue value)
    {
        Debug.Log("hoge!");
    }
    */
}
```

ball.cs
```
using UnityEngine;

public class ball : MonoBehaviour
{
    [SerializeField] GameObject paddle;

    Rigidbody2D rb;
    float speed = 10f;

    float minSpeed = 9f;
    float maxSpeed = 15f;

    void Awake()
    {
        rb = GetComponent<Rigidbody2D>();
    }

    public void Fire()
    {
        Vector2 direction = (paddle.transform.position - transform.position).normalized;
        rb.AddForce(direction * speed, ForceMode2D.Impulse);
    }

    void OnCollisionEnter2D(Collision2D collision)
    {
        Vector2 direction = rb.linearVelocity.normalized;
        float speed = Mathf.Clamp(rb.linearVelocity.magnitude, minSpeed, maxSpeed);

        rb.linearVelocity = direction * speed;
    }

    void OnTriggerEnter(Collider other)
    {
        Debug.Log(other.name);
    }
}
```

brick.cs
```
using System;
using UnityEngine;

public class brick : MonoBehaviour
{
    public event Action OnDestroy;

    void OnCollisionEnter2D(Collision2D collision)
    {
        OnDestroy.Invoke();
        Destroy(gameObject);
    }
}
```
