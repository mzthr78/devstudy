![](https://github.com/mzthr78/devstudy/blob/master/unity/image/rollaball/rollaball000.gif)
![](https://github.com/mzthr78/devstudy/blob/master/unity/image/rollaball/rollaball001.png)

ball.cs
```
using UnityEngine;
using UnityEngine.InputSystem;

public class ball : MonoBehaviour
{
    [SerializeField] rollaball gm;
    [SerializeField] Camera camera;
    private Vector3 cameraOffset;

    Vector3 move = Vector3.zero;
    float speed = 5;

    Rigidbody rb;

    void Awake()
    {
        cameraOffset = camera.transform.position - transform.position;
    }

    void Start()
    {
        rb = GetComponent<Rigidbody>();   
    }

    void FixedUpdate()
    {
        rb.AddForce(move * speed);
    }

    void LateUpdate()
    {
        camera.transform.position = transform.position + cameraOffset;
    }

    void OnMove(InputValue value)
    {
        move = new Vector3(value.Get<Vector2>().x, 0, value.Get<Vector2>().y);
    }

    void OnTriggerEnter(Collider other)
    {
        if (other.tag == "Collectible")
        {
            Destroy(other.gameObject);
            gm.DecCount();
        }
    }
}
```

rollaball.cs
```
using System;
using TMPro;
using UnityEditor.Rendering;
using UnityEngine;

public class rollaball : MonoBehaviour
{
    [SerializeField] GameObject collectiblePrefab;
    [SerializeField] TextMeshProUGUI timeText;
    [SerializeField] TextMeshProUGUI leftText;
    [SerializeField] TextMeshProUGUI clearText;
    public int count = 8;

    void Awake()
    {
        clearText.enabled = false;

        for (int i = 0; i < count; i++)
        {
            GameObject tmp = Instantiate(collectiblePrefab, new Vector3(0, 0.5f, 4.5f), Quaternion.identity);
            tmp.transform.RotateAround(Vector3.zero, Vector3.up, 360 / count * i);
        }
    }

    public void DecCount()
    {
        count -= 1;

        /*
        if (count <= 0)
        {
            Time.timeScale = 0;
        }
        */
    }

    void Update()
    {
        leftText.text = count.ToString();

        if (count <= 0) {
            clearText.enabled = true;
            return;
        }

        timeText.text = Time.time.ToString();
    }
}
```
