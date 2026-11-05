# 3D Player Basic(move, view)

```cs:player.cs
using Unity.VisualScripting;
using UnityEngine;
using UnityEngine.InputSystem;

// なんかCursorがどのCursorかわからなくなったらこう書くらしい
using Cursor = UnityEngine.Cursor;

public class Player : MonoBehaviour
{
    private Rigidbody rb;
    //[SerializeField] Camera camera;
    [SerializeField] Transform neck;

    [SerializeField] float moveSpeed = 5.0f;
    [SerializeField] float mouseSensitivity = 100f;
    [SerializeField] private float jumpPower = 5f;

    private Vector2 inputDir;
    private Vector2 lookDir;
    private float rotationX = 0f;

    private bool isGround = true;

    // Start is called once before the first execution of Update after the MonoBehaviour is created
    void Start()
    {
        Cursor.lockState = CursorLockMode.Locked;
        //Cursor.visible = false;

        rb = GetComponent<Rigidbody>();
    }

    // Update is called once per frame
    void Update()
    {
        // これは古い書き方
        /*
        float moveH = Input.GetAxis("Horizontal");
        float moveV = Input.GetAxis("Vertical");
        Vector3 move = new Vector3(moveH, 0f, moveV) * moveSpeed;
        rb.linearVelocity = new Vector3(move.x, rb.linearVelocity.y, move.z);
        */

        Vector3 move = (transform.right * inputDir.x + transform.forward * inputDir.y) * moveSpeed;
        rb.linearVelocity = new Vector3(move.x, rb.linearVelocity.y, move.z);

        Vector2 mouseXY = lookDir * mouseSensitivity * Time.deltaTime;

        transform.Rotate(0, mouseXY.x, 0); // または Vector3.up * mouseXY.x とか

        rotationX -= mouseXY.y;
        rotationX = Mathf.Clamp(rotationX, -90f, 90f);
        neck.localRotation = Quaternion.Euler(rotationX, 0f, 0f);
    }

    void OnMove(InputValue value)
    {
        inputDir = value.Get<Vector2>();
    }

    void OnLook(InputValue value)
    {
        lookDir = value.Get<Vector2>();
    }

    void OnJump(InputValue value)
    {
        if (!isGround) return;
        rb.AddForce(Vector3.up * jumpPower, ForceMode.Impulse);
    }

    void OnCollisionEnter(Collision collision)
    {
        // ほんとはここで「地面」かどうかを判定する
        isGround = true;
    }

    void OnCollisionExit(Collision collision)
    {
        isGround = false;
    }
}
```
