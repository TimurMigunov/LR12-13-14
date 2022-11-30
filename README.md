# LR12-13-14
Мигунов Т.У.

ЭВТ-70

Игровой Движок: Unity.

лабораторная работа №12-13-14

Тема: SpaceShooter

Ход работы:

1.Выполнение работы

Настройка сцены, иерархии, использованные объекты и скрипты.

_________________![1](https://user-images.githubusercontent.com/119228138/204782911-f8b40af2-ed7b-40d0-8a72-913984b18841.png)


_________________Рисунок 12.1 –Создание и настройка сцены, и использования объектов.

```1)	Скрипт  DamageHandler.cs

public class DamageHandler : MonoBehaviour
{
    public int health = 1;
    public float invulnPeriod = 0;
    float invulnTimer = 0;
    int correctLayer;
    SpriteRenderer spriteRend;
    void Start()
    {
        correctLayer = gameObject.layer;
        spriteRend = GetComponent<SpriteRenderer>();
        if(spriteRend == null)
        {
            spriteRend = transform.GetComponentInChildren<SpriteRenderer>();
            if(spriteRend == null)
            {
                Debug.LogError("Object ' " + gameObject.name + "' has no sprite render.");
            }
            
        }
    }
    private void OnTriggerEnter2D()
    {
            health--;
            invulnTimer = invulnPeriod;
            gameObject.layer = 10;
    }
    void Update()
    {
        if(invulnTimer >0)
        {
        invulnTimer -= Time.deltaTime;
            if (invulnTimer <= 0)
            {
                gameObject.layer = correctLayer;
                if(spriteRend != null)
                {
                    spriteRend.enabled = true;
                }
                
            }
            else
            {
                if (spriteRend != null)
                {
                    spriteRend.enabled = !spriteRend.enabled;
                }
            }
        }
        if (health <= 0)
        {
            Die();
        }
    }
    void Die()
    {
        Destroy(gameObject);
    }
}
2)Скрипт EnemyShooting.cs
public class EnemyShooting : MonoBehaviour
{
    public Vector3 bulletOffset = new Vector3(0, 0.5f, 0);
    public GameObject bulletPrefab;
    int bulletLayer;
    public float fireDelay = 0.25f;
    float cooldownTimer = 0;
    Transform player;
    void Start()
    {
        bulletLayer = gameObject.layer;
    }
    void Update()
    {
        if (player == null)
        {
            GameObject go = GameObject.FindWithTag("Player");
            if (go != null)
            {
                player = go.transform;
            }
        }
        cooldownTimer -= Time.deltaTime;
        if (cooldownTimer <= 0 && player != null && Vector3.Distance(transform.position, player.position) < 4)
        {
            cooldownTimer = fireDelay;
            Vector3 offset = transform.rotation * bulletOffset;
            GameObject bulletGO = (GameObject)Instantiate(bulletPrefab, transform.position + offset, transform.rotation);
            bulletGO.layer = bulletLayer;
        }
    }
}
3) Скрипт EnemySpawner.cs
public class EnemySpawner : MonoBehaviour
{
    public GameObject enemyPrefab;
    float spawnDistance = 12f;
    float enemyRate = 5;
    float nextEnemy = 1;
    void Update()
    {
        nextEnemy -= Time.deltaTime; 
        if(nextEnemy <= 0)
        {
            nextEnemy = enemyRate;
            enemyRate *= 0.9f;
            if (enemyRate < 2)
                enemyRate = 2;
            Vector3 offset = Random.onUnitSphere;
            offset.z = 0;
            offset = offset.normalized * spawnDistance;
            Instantiate(enemyPrefab, transform.position + offset, Quaternion.identity);
        }
    }
}
4) Скрипт FacesPlayer.cs
public class FacesPlayer : MonoBehaviour
{
    public float rotSpeed = 90f;
    Transform player;
    void Update()
    {
        if(player == null)
        {
            GameObject go = GameObject.FindWithTag ("Player");
            if(go != null)
            {
                player = go.transform; 
            }
        }
        if (player == null)
            return;
        Vector3 dir = player.position - transform.position;
        dir.Normalize();
        float zAngle = Mathf.Atan2(dir.y, dir.x) * Mathf.Rad2Deg - 90;
        Quaternion desiredRot = Quaternion.Euler(0, 0, zAngle);
        transform.rotation = Quaternion.RotateTowards(transform.rotation, desiredRot, rotSpeed * Time.deltaTime);
        
    }
}
5) Скрипт MoveForward.cs
public class MoveForward : MonoBehaviour
{
    public float maxSpeed = 5f;
    void Update()
    {
        Vector3 pos = transform.position;
        Vector3 velocity = new Vector3(0,  maxSpeed * Time.deltaTime, 0);
        pos += transform.rotation * velocity;
        transform.position = pos;
    }
}
6) Скрипт PlayerMovement.cs
public class PlayerMovement : MonoBehaviour
{
    public float maxSpeed = 5f;
    public float rotSpeed = 180f;
    float shipBoundaryRadius = 0.5f;
    void Update()
    {
        Quaternion rot = transform.rotation;
        float z = rot.eulerAngles.z;
        z -= Input.GetAxis("Horizontal") * rotSpeed * Time.deltaTime;
        rot = Quaternion.Euler( 0, 0, z );
        transform.rotation = rot;
        Vector3 pos = transform.position;
        Vector3 velocity = new Vector3(0, Input.GetAxis("Vertical") * maxSpeed * Time.deltaTime, 0);
        pos += rot * velocity;
        if(pos.y+shipBoundaryRadius > Camera.main.orthographicSize)
        {
            pos.y = Camera.main.orthographicSize - shipBoundaryRadius;
        }
        if (pos.y - shipBoundaryRadius < -Camera.main.orthographicSize)
        {
            pos.y = -Camera.main.orthographicSize + shipBoundaryRadius;
        }
        float screenRatio = (float)Screen.width / (float)Screen.height;
        float widthOrtho = Camera.main.orthographicSize * screenRatio;
        if (pos.x + shipBoundaryRadius > widthOrtho)
        {
            pos.x = widthOrtho - shipBoundaryRadius;
        }
        if (pos.x - shipBoundaryRadius < -widthOrtho)
        {
            pos.x = -widthOrtho + shipBoundaryRadius;
        }
        transform.position = pos;
    }
}
7)Скрипт PlayerShooting.cs
public class PlayerShooting : MonoBehaviour
{
    public Vector3 bulletOffset = new Vector3(0, 0.5f, 0);
    public GameObject bulletPrefab;
    int bulletLayer;
    public float fireDelay = 0.25f;
    float cooldownTimer = 0;
    void Start()
    {
        bulletLayer = gameObject.layer;
    }
    void Update()
    {
        cooldownTimer -= Time.deltaTime;
        if (Input.GetButton("Fire1") && cooldownTimer <= 0)
        {
            cooldownTimer = fireDelay;
            Vector3 offset = transform.rotation * bulletOffset;
            GameObject bulletGO = (GameObject)Instantiate(bulletPrefab, transform.position + offset, transform.rotation);
            bulletGO.layer = gameObject.layer;
        }
    }
}
8) Скрипт PlayerSpawner.cs
public class PlayerSpawner : MonoBehaviour
{
    public GameObject playerPrefab;
    GameObject playerInstance;
    public int numLives = 4;
    float respawnTimer;
    void Start()
    {
        SpawnPLayer();
    }
    void SpawnPLayer()
    {
        numLives--;
        respawnTimer = 1;
        playerInstance = (GameObject)Instantiate(playerPrefab, transform.position, Quaternion.identity);
    }
    void Update()
    {
        if (playerInstance == null && numLives > 0)
        {
            respawnTimer -= Time.deltaTime;
            if (respawnTimer <= 0)
            {
                SpawnPLayer();
            }
        }
    }
    void OnGUI()
    {
        if (numLives > 0 || playerInstance!= null)
        {
            GUI.Label(new Rect(0, 0, 100, 50), "Lives Left: " + numLives);
        }
        else
        {
            GUI.Label(new Rect(Screen.width / 2 - 50, Screen.height/2 - 25, 100, 50), "Game Over, Man!");
        }
    }
}
9) Скрипт SelfDestruct.cs
public class SelfDestruct : MonoBehaviour
{
    public float timer = 1f;
    void Update()
    {
        timer -= Time.deltaTime;
        if (timer <= 0)
        {
            Destroy(gameObject);
        }
    }
}
```
10) Растановка Объектов

_____________![image](https://user-images.githubusercontent.com/119228138/204783564-3b223d1d-20b9-4e7d-990d-13bb7dc8bcd6.png)

_____________Рис.12.2 Расставленные объекты

11)Настройка объектов

_____________![image](https://user-images.githubusercontent.com/119228138/204786168-f44cd882-e276-4d9a-b1b0-831fee70bfda.png)

_____________Рис.12.3 Настройка Main Camera

_____________![image](https://user-images.githubusercontent.com/119228138/204786461-ef47e7d7-b597-4eba-98f2-27faafee09ca.png)

_____________Рис.12.4 Настройка PlayerSpawnSpot

_____________![image](https://user-images.githubusercontent.com/119228138/204786650-5a4c32bd-d6ab-4582-93bc-83736911f934.png)

_____________Рис.12.4 Настройка EnemySpawner

Вывод: В ходе проделанной работы - Разработана игра SpaceShooter.
[ЛР12,13,14. Разработка Space Shooter.zip](https://github.com/TimurMigunov/LR12-13-14/files/10122760/12.13.14.Space.Shooter.zip)
