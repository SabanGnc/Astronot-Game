# Astronot-Game

## Dash, Dance, W-A-S-D, Jump, mouse control

```
using UnityEngine;

public class AstronotControl : MonoBehaviour
{
    public float moveSpeed = 5.0f;
    public float sensitivity = 2.0f;
    public float jumpForce = 5.0f;
    private bool isGrounded = true;
    private Rigidbody rb;
    private float rotationX = 0;
    public float dashSpeed = 10.0f;
    public float dashDuration = 0.2f;
    private bool isDashing = false;
    private float dashTimer = 0.0f;

    private Animator animator;

    void Start()
    {
        rb = GetComponent<Rigidbody>();
        animator = GetComponent<Animator>();
    }

    void Update()
    {
        Cursor.visible = false;
        Move();
        RotateCamera();
        HandleJump();
        HandleDash();
        HandleSillyDance();
        HandleThrillerDance();
        HandleFlairDance();
        HandleBreakDance(); // Yeni eklenen fonksiyon
        UpdateAnimator();
    }

    private void Move()
    {
        float horizontalInput = Input.GetAxis("Horizontal");
        float verticalInput = Input.GetAxis("Vertical");

        Vector3 movement = new Vector3(horizontalInput, 0, verticalInput) * moveSpeed * Time.deltaTime;
        transform.Translate(movement);
    }

    private void RotateCamera()
    {
        float mouseX = Input.GetAxis("Mouse X");
        float mouseY = Input.GetAxis("Mouse Y");

        rotationX -= mouseY * sensitivity;
        rotationX = Mathf.Clamp(rotationX, -90, 90);

        Camera.main.transform.localRotation = Quaternion.Euler(rotationX, 0, 0);
        transform.rotation *= Quaternion.Euler(0, mouseX * sensitivity, 0);
    }

    private void HandleJump()
    {
        if (isGrounded && Input.GetKeyDown(KeyCode.Space))
        {
            Jump();
        }
    }

    private void Jump()
    {
        rb.velocity = new Vector3(rb.velocity.x, 0, rb.velocity.z);
        rb.AddForce(Vector3.up * jumpForce, ForceMode.Impulse);
        isGrounded = false;
    }

    private void OnCollisionEnter(Collision collision)
    {
        if (collision.contacts[0].normal == Vector3.up)
        {
            isGrounded = true;
        }
    }

    private void HandleDash()
    {
        if (Input.GetKeyDown(KeyCode.Q) && !isDashing)
        {
            StartDash();
        }

        if (isDashing)
        {
            dashTimer += Time.deltaTime;
            if (dashTimer >= dashDuration)
            {
                isDashing = false;
                dashTimer = 0.0f;
                rb.velocity = Vector3.zero;
            }
        }
    }

    private void StartDash()
    {
        isDashing = true;
        rb.velocity = transform.forward * dashSpeed;
        animator.SetBool("IsDashing", true);
    }

    private void HandleSillyDance()
    {
        if (Input.GetKeyDown(KeyCode.Alpha1))
        {
            if (!animator.GetBool("IsSillyDance"))
            {
                animator.SetBool("IsSillyDance", true);
            }
            else
            {
                animator.SetBool("IsSillyDance", false);
            }
        }
    }

    private void HandleThrillerDance()
    {
        if (Input.GetKeyDown(KeyCode.Alpha2))
        {
            if (!animator.GetBool("IsThrillerDance"))
            {
                animator.SetBool("IsThrillerDance", true);
            }
            else
            {
                animator.SetBool("IsThrillerDance", false);
            }
        }
    }

    private void HandleFlairDance()
    {
        if (Input.GetKeyDown(KeyCode.Alpha3))
        {
            if (!animator.GetBool("IsFlairDance"))
            {
                animator.SetBool("IsFlairDance", true);
            }
            else
            {
                animator.SetBool("IsFlairDance", false);
            }
        }
    }

    private void HandleBreakDance()
    {
        if (Input.GetKeyDown(KeyCode.Alpha4))
        {
            if (!animator.GetBool("IsBreakDance"))
            {
                animator.SetBool("IsBreakDance", true);
            }
            else
            {
                animator.SetBool("IsBreakDance", false);
            }
        }
    }

    private void UpdateAnimator()
    {
        bool isWalking = Input.GetAxis("Vertical") != 0;
        animator.SetBool("IsWalking", isWalking);

        bool isRunning = isWalking && Input.GetKey(KeyCode.LeftShift);
        animator.SetBool("IsRunning", isRunning);

        animator.SetBool("IsJumping", !isGrounded);

        if (!isDashing)
        {
            animator.SetBool("IsDashing", false);
        }
    }
}

```

## Health Manager

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;

public class HealthManager : MonoBehaviour
{
    public Slider healthSlider;
    public Color lowHealthColor;
    public int maxHealth = 100;
    private int currentHealth;
    public GameObject deathPanel;

    public bool isDying = false;
    private float deathTimer = 0.0f;

    void Start()
    {
        currentHealth = maxHealth;
        UpdateHealthBar();
        deathPanel.SetActive(false);
    }

    void Update()
    {
        if (isDying)
        {
            deathTimer += Time.deltaTime;

            if (deathTimer >= 4.0f)
            {
                ShowDeathPanel();
            }
        }
    }

    void UpdateHealthBar()
    {
        healthSlider.value = (float)currentHealth / maxHealth;

        if (healthSlider.value <= 0.3f)
        {
            healthSlider.fillRect.GetComponentInChildren<UnityEngine.UI.Image>().color = lowHealthColor;
        }
        else
        {
            healthSlider.fillRect.GetComponentInChildren<UnityEngine.UI.Image>().color = Color.green;
        }

        if (currentHealth <= 0 && !isDying)
        {
            isDying = true;
            GetComponent<CharacterController>().enabled = false;
        }
    }

    public void TakeDamage(int damageAmount)
    {
        currentHealth -= damageAmount;
        if (currentHealth <= 0 && !isDying)
        {
            currentHealth = 0;
            UpdateHealthBar();
        }
        else
        {
            UpdateHealthBar();
        }
    }

    void ShowDeathPanel()
    {
        deathPanel.SetActive(true);
    }

    public void RestartGame()
    {
    }
}

```

## Enemy, Düşmanların Yapay Zekası 

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Enemy : MonoBehaviour
{
    public float speed = 3.0f;
    public int damage = 10;
    public float attackDistance = 1.5f; // Düşmanın saldırı yapabileceği mesafe
    public float retreatTime = 2.0f; // Geri çekilme süresi

    private Transform player;
    private Animator animator;
    private bool isAttacking = false;

    void Start()
    {
        player = GameObject.FindGameObjectWithTag("Player").transform;
        animator = GetComponent<Animator>();

        if (animator != null)
        {
            animator.SetBool("IsWalking", true);
        }
    }

    void Update()
    {
        if (player != null)
        {
            if (player.GetComponent<HealthManager>().isDying)
            {
                isAttacking = false;
                return;
            }

            Vector3 direction = player.position - transform.position;
            direction.y = 0;
            direction.Normalize();

            float distanceToPlayer = Vector3.Distance(transform.position, player.position);

            if (distanceToPlayer > attackDistance)
            {
                // Geri çekilme durumu
                if (isAttacking)
                {
                    transform.Translate(-direction * speed * Time.deltaTime, Space.World);
                    return;
                }

                transform.Translate(direction * speed * Time.deltaTime, Space.World);
                isAttacking = false;

                // Düşman yürürken oyuncuya doğru dönsün
                if (direction != Vector3.zero)
                {
                    transform.forward = direction;
                }
            }
            else
            {
                if (!isAttacking)
                {
                    StartCoroutine(Attack());
                }
            }
        }
    }

    IEnumerator Attack()
    {
        isAttacking = true;

        if (animator != null)
        {
            animator.SetBool("IsWalking", false);
            animator.SetTrigger("Attack");
        }

        yield return new WaitForSeconds(0.5f); // Saldırı animasyonu için gerekli zaman

        // Saldırı sonrası hasar verme
        if (player != null)
        {
            HealthManager healthManager = player.GetComponent<HealthManager>();
            if (healthManager != null)
            {
                healthManager.TakeDamage(damage);
            }
        }

        if (animator != null)
        {
            animator.SetBool("IsWalking", true);
        }

        yield return new WaitForSeconds(retreatTime); // Geri çekilme süresi

        isAttacking = false;
    }

    private void OnTriggerEnter(Collider other)
    {
        if (other.CompareTag("PlayerProjectile"))
        {
            HealthManager healthManager = GetComponent<HealthManager>();
            if (healthManager != null)
            {
                healthManager.TakeDamage(damage);
            }
        }
    }
}

```


