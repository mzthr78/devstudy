![](https://github.com/mzthr78/devstudy/blob/master/unity/image/bullethell/bullethell000.gif)
![](https://github.com/mzthr78/devstudy/blob/master/unity/image/bullethell/bullethell001.png)
![](https://github.com/mzthr78/devstudy/blob/master/unity/image/bullethell/bullethell002.png)
![](https://github.com/mzthr78/devstudy/blob/master/unity/image/bullethell/bullethell003.png)
![](https://github.com/mzthr78/devstudy/blob/master/unity/image/bullethell/bullethell004.png)

BulletHellManager.gd
```gdscript:BulletHellManager.gd
using System;
using System.Collections;
using System.Reflection.Emit;
using TMPro;
using Unity.VisualScripting;
using UnityEngine;
using UnityEngine.UI;

public class BulletHellManager : MonoBehaviour
{
    [SerializeField] GameObject Player;
    [SerializeField] GameObject Enemy;

    [SerializeField] TextMeshProUGUI InfoText;
    [SerializeField] Slider enemyLifeSlider;
    
    void Start()
    {
        InfoText.alpha = 0;

        Player.GetComponent<BHPlayer>().OnDestroy += OnPlayerDestroyed;
        Enemy.GetComponent<Enemy>().OnDestroy += OnEnemyDestroyed;
    }

    void Info(string info = "")
    {
        InfoText.enabled = (info != "");
        InfoText.text = info;
    }

    void OnPlayerDestroyed()
    {
        GameOver();
    }

    void OnEnemyDestroyed()
    {
        Info("CLEAR!");
        InfoText.alpha = 1;
    }

    void GameOver()
    {
        Info("GAME OVER");
        InfoText.alpha = 0;
        StartCoroutine(Wait());
    }

    IEnumerator Wait()
    {
        yield return new WaitForSeconds(1.5f);
        StartCoroutine(GameOverCoroutine());
    }

    IEnumerator GameOverCoroutine()
    {
        float timer = 0f;
        float duration = 3f;

        while (timer < duration)
        {
            timer += Time.deltaTime;
            InfoText.alpha = Mathf.Clamp01(timer / duration);
            yield return null;
        }
        InfoText.alpha = 1f;
    }
}
```

Player.gd
```gdscript:Player.gd
using System;
using Unity.VisualScripting;

//using System.Numerics;
using UnityEngine;
using UnityEngine.UI;

public class BHPlayer : MonoBehaviour
{
    [SerializeField] GameObject gameManager;
    [SerializeField] GameObject bulletPrefab;
    [SerializeField] ParticleSystem explosionParticle;
    [SerializeField] Image[] lifeImage;

    public event Action OnDestroy;

    float moveH = 0;
    float moveV = 0;

    float speed = .2f;

    float screenHalfHeight = 0f;
    float screenHalfWidth = 0f;

    int life = 3;

    // Start is called once before the first execution of Update after the MonoBehaviour is created
    void Start()
    {
        screenHalfHeight = Camera.main.orthographicSize;
        screenHalfWidth = screenHalfHeight * Camera.main.aspect;

        explosionParticle.Stop();
    }

    float timer = .5f;

    // Update is called once per frame
    void Update()
    {
        moveH = Input.GetAxis("Horizontal");
        moveV = Input.GetAxis("Vertical");

        if (Input.GetKeyDown("space"))
        {
            GameObject bullet = Instantiate(bulletPrefab, transform.position, Quaternion.identity);
            bullet.GetComponent<Bullet>().dir = new Vector3(0, 1, 0);
            bullet.GetComponent<Bullet>().speed = 20f;
            bullet.GetComponent<Bullet>().tag = "Player";
        }
    }

    void FixedUpdate()
    {

        transform.position = transform.position + new Vector3(moveH, moveV, 0) * speed;

        // 画面外に出られないように
        // ちゃんとやるなら自機の大きさの分を足したり引いたり
        transform.position = new Vector3(
            Mathf.Clamp(transform.position.x, -screenHalfWidth, screenHalfWidth),
            Mathf.Clamp(transform.position.y, -screenHalfHeight, screenHalfHeight),
            0
        );
    }

    void OnTriggerEnter2D(Collider2D collision)
    {
        if (collision.name.StartsWith("Bullet") && collision.tag == "Enemy") {

            life -= 1;

            lifeImage[life].enabled = false;

            if (life <= 0)
            {
                explosionParticle.transform.position = this.transform.position;
                explosionParticle.Play();

                OnDestroy.Invoke();

                Destroy(gameObject);
            }
        }
    }
}
```

Enemy.gd
```gdscript:Enemy.gd
using System;
using System.Collections;
using NUnit.Framework;
using Unity.VisualScripting;
using UnityEngine;
using UnityEngine.UI;

public class Enemy : MonoBehaviour
{
    [SerializeField] GameObject gameManager;
    [SerializeField] GameObject bulletPrefab;
    [SerializeField] ParticleSystem explosionParticle;
    [SerializeField] Slider lifeSlider;
    
    public event Action OnDestroy;

    GameObject bullet;
    
    float timer = .3f;
    float angle = 0;
    int movePattern = -1;
    int bulletPattern = 0;

    int movePatternCount = 3;
    int bulletPatternCount = 2;

    int targetX = 0;
    int targetY = 2;
    Vector3 target;
    int flip = 0;

    int life = 100;

    bool isStart = false;

    // Start is called once before the first execution of Update after the MonoBehaviour is created
    void Start()
    {
        transform.position = new Vector3(0, 10, 0);
        target = new Vector3(0, targetY, 0);
        explosionParticle.Stop();
        lifeSlider.value = 100;
        lifeSlider.GetComponent<CanvasGroup>().alpha = 0;
     }

    void Update()
    {
        timer -= Time.deltaTime;

        if (timer <= 0)
        {
            switch (bulletPattern)
            {
                case 0:
                    break;
                case 1:
                    bullet = Instantiate(bulletPrefab, transform.position, Quaternion.identity);
                    bullet.GetComponent<Bullet>().dir = Quaternion.Euler(0, 0, angle) * new Vector3(0, -1, 0); //new Vector3(0, -1, 0);
                    bullet.GetComponent<Bullet>().speed = 30f;
                    bullet.GetComponent<Bullet>().tag = "Enemy";

                    timer = .03f;
                    angle += 30;

                    break;
                case 2:
                    for (int i = 0; i < 12; i++)
                    {
                        bullet = Instantiate(bulletPrefab, transform.position, Quaternion.identity);
                        bullet.GetComponent<Bullet>().dir = Quaternion.Euler(0, 0, angle + 30 * i) * new Vector3(0, -1, 0); //new Vector3(0, -1, 0);
                        bullet.GetComponent<Bullet>().speed = 30f;
                        bullet.GetComponent<Bullet>().tag = "Enemy";
                    }
                    timer = .3f;
                    break;
            }
        }
    }

    void FixedUpdate()
    {
        Vector3 pos = transform.position;

        switch (movePattern)
        {
            case -1:
                transform.position = Vector3.Lerp(transform.position, target, Time.deltaTime * 2);
                break;
            case 0:
                break;
            case 1:
                transform.position = Vector3.Lerp(transform.position, target, Time.deltaTime * 2);
                break;
            case 2:
                transform.position = Vector3.Lerp(transform.position, target, Time.deltaTime * 2);
                break;
        }
        
        if (Vector3.Distance(pos, target) < 0.05f)
        {
            if (!isStart)
            {
                StartCoroutine(LifeSliderFadeIn());
                isStart = true;
            }

            flip += 1; // こんな変な計算を
            flip %= 2; // しなくてもええんやで

            movePattern = flip + 1;
            bulletPattern = flip + 1;

            targetX = 6 * (flip == 0 ? 1 : -1);
            target = new Vector3(targetX, targetY, 0);
        }
    }

    void OnTriggerEnter2D(Collider2D collision)
    {
        if (collision.name == "Bullet" && collision.tag == "Enemy") return;

        //gameManager.GetComponent<BulletHellManager>().Hoge();

        if (collision.name.StartsWith("Bullet") && collision.tag == "Player")
        {
            life -= 10;
            lifeSlider.value -= 10;

            if (life <= 0)
            {
                explosionParticle.transform.position = this.transform.position;
                explosionParticle.Play();

                OnDestroy.Invoke();

                Destroy(gameObject);
            }
        }
    }

    IEnumerator LifeSliderFadeIn()
    {
        float timer = 0f;
        float duration = 3f;

        while (timer < duration)
        {
            timer += Time.deltaTime;
            lifeSlider.GetComponent<CanvasGroup>().alpha = Mathf.Clamp01(timer / duration);
            yield return null;
        }
        lifeSlider.GetComponent<CanvasGroup>().alpha = 1f;
    }
}
```

Bullet.gd
```gdscript:Bullet.gd
using System;
using UnityEngine;

public class Bullet : MonoBehaviour
{
    public float speed = 10f;
    public Vector3 dir = Vector3.zero;

    void FixedUpdate()
    {
        transform.position += dir.normalized * Time.deltaTime * speed;
    }

    void OnBecameInvisible()
    {
        Destroy(gameObject);
    }
}
```
