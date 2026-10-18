# Vfl-mobile1
using UnityEngine;

public class StreetWallPhysics : MonoBehaviour
{
    public float bounciness = 0.85f; // Duvar esneklik katsayısı

    private void OnCollisionEnter(Collision collision)
    {
        if (collision.gameObject.CompareTag("Ball"))
        {
            Rigidbody ballRb = collision.gameObject.GetComponent<Rigidbody>();
            if (ballRb != null)
            {
                // Topun duvara geliş açısına göre sekme yönünü hesapla
                Vector3 reflectDir = Vector3.Reflect(collision.relativeVelocity, collision.contacts[0].normal);
                ballRb.velocity = reflectDir * bounciness;
            }
        }
    }
} using UnityEngine;
using System.IO;

public class OfflineSaveManager : MonoBehaviour
{
    private string savePath;

    void Awake()
    {
        savePath = Application.persistentDataPath + "/vfl_save.json";
    }

    [System.Serializable]
    public class GameData
    {
        public int playerCoins;
        public int currentLeagueLevel;
        public string teamName;
    }

    public void SaveGame(int coins, int league, string tName)
    {
        GameData data = new GameData();
        data.playerCoins = coins;
        data.currentLeagueLevel = league;
        data.teamName = tName;

        string json = JsonUtility.ToJson(data);
        File.WriteAllText(savePath, json);
        Debug.Log("Oyun çevrimdışı olarak kaydedildi!");
    }

    public GameData LoadGame()
    {
        if (File.Exists(savePath))
        {
            string json = File.ReadAllText(savePath);
            GameData data = JsonUtility.FromJson<GameData>(json);
            return data;
        }
        else
        {
            Debug.LogWarning("Kayıt dosyası bulunamadı, yeni profil oluşturuluyor.");
            return new GameData { playerCoins = 1000, currentLeagueLevel = 5, teamName = "VFL FC" };
        }
    }
} using UnityEngine;

[CreateAssetMenu(fileName = "NewPlayerCard", menuName = "VFL Mobile/Player Card")]
public class PlayerCardSO : ScriptableObject
{
    public string playerName;      // Örn: Arda Güler, Xavi, Rooney
    public int overallRating;       // Örn: 105, 106, 107
    public string position;        // Örn: OOS, OS, FW, LW, GK
    public Sprite teamOrNationLogo;// Kulüp veya Milli Takım Amblemi
    public string nationality;     // Örn: Türkiye, İspanya, İngiltere
    public Sprite playerImage;     // Oyuncunun kart görseli
} using UnityEngine;

public class MatchController : MonoBehaviour
{
    // --- RAKİP AI SİSTEMİ ---
    public void ExecuteAIBehavior(Transform aiPlayer, Transform ball, Transform goal)
    {
        float distanceToBall = Vector3.Distance(aiPlayer.position, ball.position);
        float distanceToGoal = Vector3.Distance(aiPlayer.position, goal.position);

        if (distanceToBall < 1.5f) // Top AI'daysa
        {
            if (distanceToGoal < 20f)
            {
                // Kaleye yakınsa şut çek
                Debug.Log("AI: Kaleye şut çekiliyor!");
            }
            else
            {
                // Değilse en uygun arkadaşına pas ver veya dripling yap
                Debug.Log("AI: Pas veriliyor / Hücum kuruluyor.");
            }
        }
    }

    // --- HAKEM SİSTEMİ ---
    public void CheckFoul(float tackleTacklingForce, bool isFromBehind)
    {
        // Arkadan sert müdahale veya aşırı güç kullanımı
        if (isFromBehind || tackleTacklingForce > 80f)
        {
            float cardChance = Random.Range(0f, 100f);
            
            if (cardChance > 70f)
            {
                Debug.Log("HAKEM: Kırmızı Kart!");
            }
            else if (cardChance > 30f)
            {
                Debug.Log("HAKEM: Sarı Kart!");
            }
            else
            {
                Debug.Log("HAKEM: Faul ve Serbest Vuruş!");
            }
        }
    }
} using UnityEngine;

public enum SkillType { Roulete, RainbowFlick, Elastic, StepOver }

public class SkillMoveController : MonoBehaviour
{
    private Rigidbody playerRb;
    public Transform ball;
    public Rigidbody ballRb;

    void Start()
    {
        playerRb = GetComponent<Rigidbody>();
    }

    public void PerformSkill(SkillType skill)
    {
        switch (skill)
        {
            case SkillType.RainbowFlick:
                // Topu arkadan öne doğru aşırtarak çalım atma
                ballRb.AddForce(transform.forward * 4f + Vector3.up * 8f, ForceMode.Impulse);
                Debug.Log("Sokak Becerisi: Rainbow Flick!");
                break;

            case SkillType.Roulete:
                // 360 derece dönerek topu saklama ve geçme
                transform.Rotate(0, 360, 0);
                ballRb.AddForce(transform.forward * 5f, ForceMode.Impulse);
                Debug.Log("Sokak Becerisi: Rulet!");
                break;

            case SkillType.Elastic:
                // Dış ayak-iç ayak hızlı bilek hareketi
                Vector3 curveDir = transform.right * 2f + transform.forward * 5f;
                ballRb.AddForce(curveDir, ForceMode.Impulse);
                Debug.Log("Sokak Becerisi: Elastico!");
                break;
        }
    }
} using UnityEngine;

public class PlayerController : MonoBehaviour
{
    public float moveSpeed = 6.0f;
    public float sprintSpeed = 9.0f;
    public Joystick joystick; // Mobil Joystick UI entegrasyonu
    
    private Rigidbody rb;
    private Vector3 moveInput;
    private bool isSprinting = false;

    void Start()
    {
        rb = GetComponent<Rigidbody>();
    }

    void Update()
    {
        // Mobil joystick verilerini al
        float moveX = joystick.Horizontal;
        float moveZ = joystick.Vertical;

        moveInput = new Vector3(moveX, 0, moveZ).normalized;

        // Yön dönme mantığı
        if (moveInput != Vector3.zero)
        {
            Quaternion targetRotation = Quaternion.LookRotation(moveInput);
            transform.rotation = Quaternion.Slerp(transform.rotation, targetRotation, Time.deltaTime * 12f);
        }
    }

    void FixedUpdate()
    {
        float currentSpeed = isSprinting ? sprintSpeed : moveSpeed;
        rb.MovePosition(rb.position + moveInput * currentSpeed * Time.fixedDeltaTime);
    }

    public void ToggleSprint(bool sprint)
    {
        isSprinting = sprint;
    }
}
using UnityEngine;
using UnityEngine.UI;
using System.Collections.Generic;

public class PackOpenManager : MonoBehaviour
{
    [System.Serializable]
    public class PlayerCard
    {
        public string playerName;
        public string position;
        public int ovrRating;
        public string cardTier; // "Epic", "Big Time", "Highlight"
        public Sprite cardImage; // Hazırladığın kart görseli
    }

    [Header("İspanya Efsaneleri Paketi")]
    public List<PlayerCard> spainEpicPack = new List<PlayerCard>();

    [Header("UI Görsel Bağlantıları")]
    public Image displayCardImage;  // Ekrandaki kart resmi
    public GameObject walkoutEffect; // Paket açılış efekti

    void Start()
    {
        // Oyuncu Veritabanını Oluşturma
        InitSpainLegendPack();
    }

    void InitSpainLegendPack()
    {
        spainEpicPack.Add(new PlayerCard { playerName = "Xavi", position = "OS", ovrRating = 106, cardTier = "Epic" });
        spainEpicPack.Add(new PlayerCard { playerName = "Andres Iniesta", position = "OS", ovrRating = 105, cardTier = "Epic" });
        spainEpicPack.Add(new PlayerCard { playerName = "Sergio Ramos", position = "DEF", ovrRating = 104, cardTier = "Big Time" });
        spainEpicPack.Add(new PlayerCard { playerName = "Carles Puyol", position = "DEF", ovrRating = 103, cardTier = "Big Time" });
        spainEpicPack.Add(new PlayerCard { playerName = "Iker Casillas", position = "GK", ovrRating = 102, cardTier = "Highlight" });
    }

    // Paket Açma Butonuna Basıldığında Çalışır
    public PlayerCard OpenSpainPack()
    {
        int randomIndex = Random.Range(0, spainEpicPack.Count);
        PlayerCard pulledPlayer = spainEpicPack[randomIndex];

        Debug.Log("🎉 PAKETTEN ÇIKAN EFSANE: " + pulledPlayer.playerName + " | OVR: " + pulledPlayer.ovrRating + " | Mevki: " + pulledPlayer.position);

        // Ekrandaki kart görselini güncelleme (Unity UI Entegrasyonu)
        if (displayCardImage != null && pulledPlayer.cardImage != null)
        {
            displayCardImage.sprite = pulledPlayer.cardImage;
        }

        return pulledPlayer;
    }
}
using UnityEngine;
using UnityEngine.UI;
using System.Collections.Generic;

public class PackOpenManager : MonoBehaviour
{
    [System.Serializable]
    public class PlayerCard
    {
        public string playerName;
        public string position;
        public int ovrRating;
        public string cardTier; // "Epic", "Big Time", "Highlight"
        public Sprite cardImage; // Hazırladığın kart görseli
    }

    [Header("İspanya Efsaneleri Paketi")]
    public List<PlayerCard> spainEpicPack = new List<PlayerCard>();

    [Header("UI Görsel Bağlantıları")]
    public Image displayCardImage;  // Ekrandaki kart resmi
    public GameObject walkoutEffect; // Paket açılış efekti

    void Start()
    {
        // Oyuncu Veritabanını Oluşturma
        InitSpainLegendPack();
    }

    void InitSpainLegendPack()
    {
        spainEpicPack.Add(new PlayerCard { playerName = "Xavi", position = "OS", ovrRating = 106, cardTier = "Epic" });
        spainEpicPack.Add(new PlayerCard { playerName = "Andres Iniesta", position = "OS", ovrRating = 105, cardTier = "Epic" });
        spainEpicPack.Add(new PlayerCard { playerName = "Sergio Ramos", position = "DEF", ovrRating = 104, cardTier = "Big Time" });
        spainEpicPack.Add(new PlayerCard { playerName = "Carles Puyol", position = "DEF", ovrRating = 103, cardTier = "Big Time" });
        spainEpicPack.Add(new PlayerCard { playerName = "Iker Casillas", position = "GK", ovrRating = 102, cardTier = "Highlight" });
    }

    // Paket Açma Butonuna Basıldığında Çalışır
    public PlayerCard OpenSpainPack()
    {
        int randomIndex = Random.Range(0, spainEpicPack.Count);
        PlayerCard pulledPlayer = spainEpicPack[randomIndex];

        Debug.Log("🎉 PAKETTEN ÇIKAN EFSANE: " + pulledPlayer.playerName + " | OVR: " + pulledPlayer.ovrRating + " | Mevki: " + pulledPlayer.position);

        // Ekrandaki kart görselini güncelleme (Unity UI Entegrasyonu)
        if (displayCardImage != null && pulledPlayer.cardImage != null)
        {
            displayCardImage.sprite = pulledPlayer.cardImage;
        }

        return pulledPlayer;
    }
}
using UnityEngine;
using UnityEngine.UI;
using System.Collections;
using TMPro;

public class PackWalkoutManager : MonoBehaviour
{
    [Header("UI Elementleri")]
    public GameObject walkoutPanel;
    public Image countryFlagImage;
    public TextMeshProUGUI ovrText;
    public TextMeshProUGUI positionText;
    public Image cardDisplayImage;

    [Header("Efektler")]
    public GameObject confettiEffect;
    public AudioSource walkoutSound;

    public void TriggerWalkoutSequence(PackOpenManager.PlayerCard card)
    {
        StartCoroutine(PlayWalkoutSequence(card));
    }

    private IEnumerator PlayWalkoutSequence(PackOpenManager.PlayerCard card)
    {
        // 1. Ekranı ve Efektleri Aç
        walkoutPanel.SetActive(true);
        if (walkoutSound != null) walkoutSound.Play();

        // 2. İlk Aşama: Pozisyon ve OVR Parlaması
        ovrText.text = card.ovrRating.ToString();
        positionText.text = card.position;
        ovrText.gameObject.SetActive(true);
        positionText.gameObject.SetActive(true);

        yield return new WaitForSeconds(1.5f); // 1.5 saniye bekle

        // 3. İkinci Aşama: Hazırladığın Efsane Kartı Ekrana Getir
        if (cardDisplayImage != null && card.cardImage != null)
        {
            cardDisplayImage.sprite = card.cardImage;
        }
        
        cardDisplayImage.gameObject.SetActive(true);
        if (confettiEffect != null) confettiEffect.SetActive(true);

        Debug.Log("✨ " + card.playerName + " için eFootball tarzı Walkout tamamlandı!");
    }
}
