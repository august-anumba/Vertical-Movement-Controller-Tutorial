# Vertical Movement Controller Tutorial

This tutorial shows how to create a vertical movement controller in Unity.

## 1. Adding to the `PlayerMovement` C# Script

Since we already implemented the ground check in horizontal movement tutorial, its going to be simple to also implement vertical movement into the script.

Firstly we will need a few new floats like `jumpForce`, `jumpCooldown` and `airMultiplier` in addition to a `readyToJump` bool:

```.cs
    public float jumpForce;
    public float jumpCooldown;
    public float airMultiplier;
    bool readyToJump;
```
We now create a new function called `jump()` and before we apply any forces we need to reset the Y velocity by using the following script:

```.cs
        rb.velocity = new Vector3(rb.velocity.x, 0f, rb.velocity.z);
```

We can now add upwards / vertical force to our `rigidBody` with:

```.cs
        rb.AddForce(transform.up * jumpForce, ForceMode.Impulse);
```

The `jump` function should now look like this:

```.cs
private void Jump()
    {
        rb.velocity = new Vector3(rb.velocity.x, 0f, rb.velocity.z);

        rb.AddForce(transform.up * jumpForce, ForceMode.Impulse);

    }
```

We also create a `resetJump` function and use it to set the bool `readyToJump` to true:

```.cs
private void ResetJump()
    {
        readyToJump = true;
    }
```

Next we need to bind a key to call the `jump` and make the player jump, we will use the Spacebar under `KeyCode.Space`.

```.cs
[Header("Keybinds")]
    public KeyCode jumpKey = KeyCode.Space;
```

Then in the `MyInput` function we check if the player is ready to jump and grounded, if this is all true we can call the `jump` function and and set `readyToJump` to false.
We also invoke the 5th, oh sorry i meant invoke the `ResetJump` function using `jumpCooldown` as a delay, this way you can continuously jump by holding down the jump key:

```.cs
if(Input.GetKey(jumpKey) && readyToJump && grounded)
        {
            readyToJump = false;

            Jump();

            Invoke(nameof(ResetJump), jumpCooldown);
```
We will also want to adject the air control a bit so inside of the `MovePlayer` function you want to add these if statements and if you're in the air multiply your move speed with the air multiplier.

```.cs
       if(grounded)
            rb.AddForce(moveDirection.normalized * moveSpeed * 10f, ForceMode.Force);
        
        else if(!grounded)
            rb.AddForce(moveDirection.normalized * moveSpeed * 10f * airMultiplier, ForceMode.Force);
```
 Now back in unity we now set the values of the variables:
 
 ![image](https://github.com/august-anumba/Vertical-Movement-Controller-Tutorial/assets/146851823/40ec00e6-6798-47b3-b5ff-6441c1bcd844)

You can use the pictured values as a guide and adject as seen fit.

I encountered some errors and bugs with the jumping, to fix them i needed to adjust the grounded = Physics.Raycast calculation to:

```.cs
grounded = Physics.Raycast(transform.position + new Vector3(0, 0.05f, 0), Vector3.down, playerHeight * 0.05f, whatIsGround);
```
and that fixed the bug where the player could essentially double jump while holding spacebar.

The finished script should look like this;

```.cs
using System.Collections;
using System.Collections.Generic;
using UnityEditor.Experimental.GraphView;
using UnityEngine;

public class PlayerMovement : MonoBehaviour
{
    [Header("Movement")]
    public float moveSpeed;

    public float groundDrag;

    public float jumpForce;
    public float jumpCooldown;
    public float airMultiplier;
    bool readyToJump;

    [Header("Keybinds")]
    public KeyCode jumpKey = KeyCode.Space;

    [Header("Ground Check")]
    public float playerHeight;
    public LayerMask whatIsGround;
    bool grounded;

    public Transform orientation;

    float horizontalInput;
    float verticalInput;

    Vector3 moveDirection;

    Rigidbody rb;


    private void Start()
    {
        readyToJump = true;
        rb = GetComponent<Rigidbody>();
        rb.freezeRotation = true;
    }

    private void Update()
    {
        grounded = Physics.Raycast(transform.position + new Vector3(0, 0.05f, 0), Vector3.down, playerHeight * 0.05f, whatIsGround);

        MyInput();
        SpeedControl();

        if (grounded)
            rb.drag = groundDrag;
        else
            rb.drag = 0;
        
    }

    private void FixedUpdate()
    {
        MovePlayer();
    }

    private void MyInput()
    {
        horizontalInput = Input.GetAxisRaw("Horizontal");
        verticalInput = Input.GetAxisRaw("Vertical");

        if(Input.GetKey(jumpKey) && readyToJump && grounded)
        {
            readyToJump = false;

            Jump();

            Invoke(nameof(ResetJump), jumpCooldown);
        }
    }

    private void MovePlayer()
    {
        moveDirection = orientation.forward * verticalInput + orientation.right * horizontalInput;

        if(grounded)
            rb.AddForce(moveDirection.normalized * moveSpeed * 10f, ForceMode.Force);
        
        else if(!grounded)
            rb.AddForce(moveDirection.normalized * moveSpeed * 10f * airMultiplier, ForceMode.Force);
    }       

    private void SpeedControl()
    {
        Vector3 flatVel = new Vector3(rb.velocity.x, 0f, rb.velocity.z);

        if(flatVel.magnitude > moveSpeed)
        {
            Vector3 limitedVel = flatVel.normalized * moveSpeed;
            rb.velocity = new Vector3(limitedVel.x, rb.velocity.y, limitedVel.z);
        }


    }

    private void Jump()
    {
        rb.velocity = new Vector3(rb.velocity.x, 0f, rb.velocity.z);

        rb.AddForce(transform.up * jumpForce, ForceMode.Impulse);

    }

    private void ResetJump()
    {
        readyToJump = true;
    }
}

```
