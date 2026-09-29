(Unity6.5)

![](https://github.com/mzthr78/devstudy/blob/master/unity/image/breakable/breakable000.gif)
![](https://github.com/mzthr78/devstudy/blob/master/unity/image/breakable/breakable001.png)

breakable.cs
```
using System;
using System.Runtime.Serialization;
using Mono.Cecil.Cil;
using UnityEngine;
using UnityEngine.Rendering.Universal;

public class breakable : MonoBehaviour
{
    [SerializeField] GameObject empty;
    [SerializeField] float speed = 1;
    Boolean isExplosion = false;

    void Awake()
    {
        for (int i = 0; i < transform.childCount; i++)
        {
            Transform tf = transform.GetChild(i);
            tf.gameObject.AddComponent<fragment>();

            Rigidbody rb = tf.gameObject.AddComponent<Rigidbody>();
            rb.useGravity = true;
            rb.isKinematic = true;

            MeshCollider mc = tf.gameObject.AddComponent<MeshCollider>();
            mc.convex = true;
        }
    }

    void Update()
    {
        if (Input.GetKey("space") && !isExplosion)
        {
            isExplosion = true;
            
            for (int i = 0; i < transform.childCount; i++)
            {
                Transform tf = transform.GetChild(i);
                tf.GetComponent<fragment>().isExplosion = true;

                Rigidbody rb = tf.gameObject.GetComponent<Rigidbody>();
                rb.isKinematic = false;
                rb.linearVelocity = (tf.position - empty.transform.position).normalized * speed;
            }
        }
    }
}
```

fragment.cs
```
using System;
using UnityEngine;
using UnityEngine.InputSystem.Controls;

public class fragment : MonoBehaviour
{
    public Boolean isExplosion;
    float lifetime = 2;

    // Update is called once per frame
    void Update()
    {
        if (!isExplosion) return;

        lifetime -= Time.deltaTime;

        if (lifetime <= 0)
        {
            Destroy(gameObject);
        }
    }
}
```

