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
spainEpicPack.Add(new PlayerCard { playerName = "Xavi", position = "OS", ovrRating = 106, cardTier = "Epic" });
        spainEpicPack.Add(new PlayerCard { playerName = "Andres Iniesta", position = "OS", ovrRating = 105, cardTier = "Epic" });
        spainEpicPack.Add(new PlayerCard { playerName = "Sergio Ramos", position = "DEF", ovrRating = 104, cardTier = "Big Time" });
        spainEpicPack.Add(new PlayerCard { playerName = "Carles Puyol", position = "DEF", ovrRating = 103, cardTier = "Big Time" });
        spainEpicPack.Add(new PlayerCard { playerName = "Iker Casillas", position = "GK", ovrRating = 102, cardTier = "Highlight" });
    }
<!DOCTYPE html>
<html lang="tr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>VFL Mobile - 3D 11v11 Match Engine</title>
    <link href="https://fonts.googleapis.com/css2?family=Teko:wght@600;700&family=Poppins:wght@400;600;800&display=swap" rel="stylesheet">
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <style>
        :root {
            --epic-gold: #ffd700;
            --bg-dark: #0a0b10;
        }

        body {
            margin: 0;
            background: #08090d;
            color: white;
            font-family: 'Poppins', sans-serif;
            text-align: center;
            overflow: hidden;
            user-select: none;
            -webkit-user-select: none;
        }

        /* SCREEN MANAGEMENT */
        .screen {
            position: absolute;
            top: 0; left: 0; width: 100%; height: 100%;
            display: flex;
            flex-direction: column;
            justify-content: space-between;
            box-sizing: border-box;
            padding: 15px;
            z-index: 2;
            transition: opacity 0.3s ease;
        }
        .hidden { display: none !important; opacity: 0; pointer-events: none; }

        /* HEADER */
        .header {
            display: flex;
            justify-content: space-between;
            align-items: center;
        }
        .user-badge {
            display: flex;
            align-items: center;
            gap: 10px;
            font-weight: 800;
            font-size: 14px;
        }
        .user-avatar {
            width: 36px; height: 36px;
            border-radius: 50%;
            background: linear-gradient(45deg, #00f2fe, #4facfe);
            display: flex; align-items: center; justify-content: center;
            border: 2px solid #fff;
        }
        .currencies { display: flex; gap: 8px; }
        .currency-box {
            background: rgba(255,255,255,0.08);
            backdrop-filter: blur(8px);
            padding: 5px 12px;
            border-radius: 20px;
            border: 1px solid rgba(255,255,255,0.15);
            font-size: 12px;
            font-weight: 600;
        }

        /* MAIN BANNER */
        .banner-container {
            margin: auto 0;
            background: linear-gradient(135deg, #
<!DOCTYPE html>
<html lang="tr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>VFL Mobile - 3D 11v11 Match Engine</title>
    <link href="https://fonts.googleapis.com/css2?family=Teko:wght@600;700&family=Poppins:wght@400;600;800&display=swap" rel="stylesheet">
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <style>
        :root {
            --epic-gold: #ffd700;
            --bg-dark: #0a0b10;
        }

        body {
            margin: 0;
            background: #08090d;
            color: white;
            font-family: 'Poppins', sans-serif;
            text-align: center;
            overflow: hidden;
            user-select: none;
            -webkit-user-select: none;
        }

        /* SCREEN MANAGEMENT */
        .screen {
            position: absolute;
            top: 0; left: 0; width: 100%; height: 100%;
            display: flex;
            flex-direction: column;
            justify-content: space-between;
            box-sizing: border-box;
            padding: 15px;
            z-index: 2;
            transition: opacity 0.3s ease;
        }
        .hidden { display: none !important; opacity: 0; pointer-events: none; }

        /* HEADER */
        .header {
            display: flex;
            justify-content: space-between;
            align-items: center;
        }
        .user-badge {
            display: flex;
            align-items: center;
            gap: 10px;
            font-weight: 800;
            font-size: 14px;
        }
        .user-avatar {
            width: 36px; height: 36px;
            border-radius: 50%;
            background: linear-gradient(45deg, #00f2fe, #4facfe);
            display: flex; align-items: center; justify-content: center;
            border: 2px solid #fff;
        }
        .currencies { display: flex; gap: 8px; }
        .currency-box {
            background: rgba(255,255,255,0.08);
            backdrop-filter: blur(8px);
            padding: 5px 12px;
            border-radius: 20px;
            border: 1px solid rgba(255,255,255,0.15);
            font-size: 12px;
            font-weight: 600;
        }

        /* MAIN BANNER */
        .banner-container {
            margin: auto 0;
            background: linear-gradient(135deg, #2a0845, #120428);
            border-radius: 20px;
            border: 2px solid rgba(255, 215, 0, 0.5);
            box-shadow: 0 0 25px rgba(255, 215, 0, 0.25);
            padding: 30px 15px;
        }
        .banner-title {
            font-family: 'Teko', sans-serif;
            font-size: 42px;
            margin: 0;
            background: linear-gradient(180deg, #fff, #ffd700);
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
        }

        /* BUTTONS */
        .btn-epic {
            background: linear-gradient(90deg, #ffd700, #ffa500);
            color: #000;
            font-family: 'Teko', sans-serif;
            font-size: 24px;
            font-weight: 700;
            padding: 10px 40px;
            border: none;
            border-radius: 30px;
            cursor: pointer;
            box-shadow: 0 0 20px rgba(255, 215, 0, 0.6);
            text-transform: uppercase;
            margin-top: 15px;
        }
        .btn-epic:active { transform: scale(0.95); }

        /* MENU GRID */
        .grid-container {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 12px;
            margin-bottom: 20px;
        }
        .menu-card {
            background: rgba(22, 26, 38, 0.85);
            border-radius: 16px;
            padding: 15px 10px;
            border: 1px solid rgba(255,255,255,0.08);
            cursor: pointer;
        }
        .menu-card h3 { margin: 5px 0; font-size: 15px; }
        .menu-card p { font-size: 11px; color: #8a8f9d; margin: 0; }

        /* 3D GAMEPLAY STAGE */
        #gameCanvas {
            position: absolute;
            top: 0; left: 0; width: 100%; height: 100%;
            z-index: 1;
        }

        /* MATCH HUD */
        .match-hud {
            position: absolute;
            top: 15px; left: 0; width: 100%;
            display: flex;
            justify-content: center;
            align-items: center;
            gap: 15px;
            z-index: 5;
            pointer-events: none;
        }
        .scoreboard {
            background: rgba(0,0,0,0.8);
            border: 2px solid #ffd700;
            padding: 6px 20px;
            border-radius: 20px;
            font-family: 'Teko', sans-serif;
            font-size: 26px;
            letter-spacing: 2px;
        }
        .btn-back {
            position: absolute;
            top: 15px; left: 15px;
            background: rgba(255,0,0,0.8);
            color: white; border: none;
            padding: 6px 12px; border-radius: 10px;
            font-weight: bold; cursor: pointer; z-index: 10;
        }

        /* ON-SCREEN CONTROLS */
        .controls-overlay {
            position: absolute;
            bottom: 0; left: 0; width: 100%; height: 100%;
            pointer-events: none;
            z-index: 5;
            display: flex;
            justify-content: space-between;
            align-items: flex-end;
            padding: 20px;
            box-sizing: border-box;
        }
        .joystick-zone {
            width: 120px; height: 120px;
            background: rgba(255,255,255,0.1);
            border: 2px solid rgba(255,255,255,0.3);
            border-radius: 50%;
            pointer-events: auto;
            position: relative;
            touch-action: none;
        }
        .joystick-knob {
            width: 50px; height: 50px;
            background: rgba(255,215,0,0.8);
            border-radius: 50%;
            position: absolute;
            top: 35px; left: 35px;
            pointer-events: none;
            box-shadow: 0 0 10px #ffd700;
        }
        .action-buttons {
            display: flex;
            gap: 15px;
            pointer-events: auto;
            padding-bottom: 20px;
        }
        .act-btn {
            width: 65px; height: 65px;
            border-radius: 50%;
            border: 2px solid #ffd700;
            background: rgba(0,0,0,0.6);
            color: #ffd700;
            font-family: 'Teko', sans-serif;
            font-size: 20px;
            font-weight: bold;
            display: flex; align-items: center; justify-content: center;
            cursor: pointer;
            box-shadow: 0 0 10px rgba(255,215,0,0.4);
        }
        .act-btn:active { background: #ffd700; color: #000; }

        /* PACK MODAL */
        .modal {
            display: none;
            position: fixed;
            top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(5, 6, 10, 0.95);
            z-index: 20;
            justify-content: center; align-items: center;
            flex-direction: column;
        }
        .fut-card {
            width: 240px; height: 360px;
            background: linear-gradient(135deg, #1e1103 0%, #4a3000 50%, #1a0f00 100%);
            border-radius: 18px;
            border: 3px solid #ffd700;
            box-shadow: 0 0 30px rgba(255, 215, 0, 0.6);
            position: relative;
            padding: 15px;
            box-sizing: border-box;
            animation: cardReveal 0.6s cubic-bezier(0.175, 0.885, 0.32, 1.275);
        }
        @keyframes cardReveal {
            0% { transform: scale(0) rotate(-10deg); opacity: 0; }
            100% { transform: scale(1) rotate(0deg); opacity: 1; }
        }
    </style>
</head>
<body>

    <!-- ANA MENÜ EKRANI -->
    <div id="mainMenu" class="screen">
        <div class="header">
            <div class="user-badge">
                <div class="user-avatar">⚽</div>
                <div>emrr38 <span style="font-size:10px; color:#ffd700; display:block;">LVL 15</span></div>
            </div>
            <div class="currencies">
                <div class="currency-box">🪙 1,450</div>
                <div class="currency-box" style="border-color: rgba(255,215,0,0.4);">💎 350</div>
            </div>
        </div>

        <div class="banner-container">
            <h2 class="banner-title">SPAIN LEGENDS</h2>
            <p style="font-size: 12px; color: #d1c4e9; margin: 0;">EPIC & BIG TIME KARTLARI</p>
            <button class="btn-epic" onclick="openPack()">PAKETİ AÇ (100 💎)</button>
        </div>

        <div class="grid-container">
            <div class="menu-card" onclick="startMatch()">
                <div style="font-size: 24px;">🏟️</div>
                <h3>11v11 MAÇ BAŞLAT</h3>
                <p>3D eFootball Simülasyonu</p>
            </div>
            <div class="menu-card" onclick="alert('Kadro ve formasyon yönetimi aktif!')">
                <div style="font-size: 24px;">🛡️</div>
                <h3>KADROM</h3>
                <p>11 Oyuncu Düzeni</p>
            </div>
        </div>
    </div>

    <!-- 3D OYUN ALANI -->
    <canvas id="gameCanvas"></canvas>

    <!-- MAÇ İÇİ HUD VE KONTROLLER -->
    <div id="gameHUD" class="screen hidden" style="pointer-events: none;">
        <button class="btn-back" onclick="endMatch()">ÇIKIŞ</button>
        <div class="match-hud">
            <div class="scoreboard">VFL 0 - 0 </div>
        </div>

        <div class="controls-overlay">
            <div class="joystick-zone" id="joystickZone">
                <div class="joystick-knob" id="joystickKnob"></div>
            </div>
            <div class="action-buttons">
                <div class="act-btn" onclick="playerPass()">PAS</div>
                <div class="act-btn" onclick="playerShoot()" style="background: rgba(255,0,0,0.5); border-color:#ff4d4d;">ŞUT</div>
            </div>
        </div>
    </div>

    <!-- PAKET AÇILIŞ MODALI -->
    <div id="packModal" class="modal" onclick="closeModal()">
        <div class="fut-card">
            <div style="display:flex; justify-content:space-between; align-items:flex-start;">
                <div style="text-align:left;">
                    <div style="font-family:'Teko'; font-size:48px; font-weight:700; color:#ffd700; line-height:35px;" id="cardOvr">106</div>
                    <div style="font-size:14px; font-weight:800; background:rgba(0,0,0,0.5); padding:2px 6px; border-radius:4px; display:inline-block;" id="cardPos">OS</div>
                </div>
            </div>
            <h2 id="cardName" style="font-family:'Teko'; font-size:32px; margin: 120px 0 0 0; text-transform:uppercase;">XAVI</h2>
            <p style="font-size: 10px; color: #ffd700; font-weight: 700; letter-spacing:2px; margin:0;" id="cardTier">EPIC • SPAIN</p>
        </div>
        <p style="margin-top: 20px; font-size: 12px; color: #aaa;">Kapatmak için ekrana dokun</p>
    </div>

    <script>
        // PAKET SİSTEMİ
        const legends = [
            { name: "Xavi", ovr: "106", pos: "OS", tier: "EPIC • SPAIN" },
            { name: "Iniesta", ovr: "105", pos: "OS", tier: "EPIC • SPAIN" },
            { name: "S. Ramos", ovr: "104", pos: "DEF", tier: "BIG TIME • SPAIN" },
            { name: "C. Puyol", ovr: "103", pos: "DEF", tier: "BIG TIME • SPAIN" },
            { name: "Casillas", ovr: "102", pos: "GK", tier: "HIGHLIGHT • SPAIN" }
        ];

        function openPack() {
            const player = legends[Math.floor(Math.random() * legends.length)];
            document.getElementById("cardName").innerText = player.name;
            document.getElementById("cardOvr").innerText = player.ovr;
            document.getElementById("cardPos").innerText = player.pos;
            document.getElementById("cardTier").innerText = player.tier;
            document.getElementById("packModal").style.display = "flex";
        }
        function closeModal() { document.getElementById("packModal").style.display = "none"; }

        // 3D THREE.JS MAÇ MOTORU
        let scene, camera, renderer, ball, players = [];
        let moveX = 0, moveZ = 0;
        let isPlaying = false;

        function init3D() {
            const canvas = document.getElementById("gameCanvas");
            scene = new THREE.Scene();
            scene.background = new THREE.Color(0x0f172a);

            camera = new THREE.PerspectiveCamera(60, window.innerWidth / window.innerHeight, 0.1, 1000);
            renderer = new THREE.WebGLRenderer({ canvas: canvas, antialias: true });
            renderer.setSize(window.innerWidth, window.innerHeight);

            // Işıklandırma
            const ambientLight = new THREE.AmbientLight(0xffffff, 0.7);
            scene.add(ambientLight);
            const dirLight = new THREE.DirectionalLight(0xffffff, 0.8);
            dirLight.position.set(20, 40, 20);
            scene.add(dirLight);

            // 11v11 Çim Saha
            const pitchGeo = new THREE.PlaneGeometry(60, 90);
            const pitchMat = new THREE.MeshBasicMaterial({ color: 0x1b4d3e });
            const pitch = new THREE.Mesh(pitchGeo, pitchMat);
            pitch.rotation.x = -Math.PI / 2;
            scene.add(pitch);

            // Saha Çizgileri ve Kaleler (Basitleştirilmiş)
            const lineGeo = new THREE.BoxGeometry(58, 0.1, 88);
            const wireMat = new THREE.MeshBasicMaterial({ color: 0xffffff, wireframe: true });
            const lines = new THREE.Mesh(lineGeo, wireMat);
            lines.position.y = 0.05;
            scene.add(lines);

            // Top
            const ballGeo = new THREE.SphereGeometry(0.6, 16, 16);
            const ballMat = new THREE.MeshStandardMaterial({ color: 0xffffff });
            ball = new THREE.Mesh(ballGeo, ballMat);
            ball.position.set(0, 0.6, 0);
            scene.add(ball);

            // Oyuncuları Oluştur (Takımlar: Kırmızı ve Mavi 11v11)
            createTeam(0xcc2929, -1); // Rakip
            createTeam(0x2952cc, 1);  // Kendi Kadromuz

            animate();
        }

        function createTeam(color, zDir) {
            const mat = new THREE.MeshStandardMaterial({ color: color });
            const geo = new THREE.CapsuleGeometry(0.8, 1.8, 4, 8);

            // 4-3-3 Formasyon Dizilimi Örneği
            const positions = [
                {x: 0, z: zDir * 40}, // Kaleci
                {x: -15, z: zDir * 25}, {x: -5, z: zDir * 28}, {x: 5, z: zDir * 28}, {x: 15, z: zDir * 25}, // Defans
                {x: -10, z: zDir * 10}, {x: 0, z: zDir * 5}, {x: 10, z: zDir * 10}, // Orta Saha
                {x: -18, z: zDir * -10}, {x: 0, z: zDir * -15}, {x: 18, z: zDir * -10}  // Forvet
            ];

            positions.forEach(pos => {
                const player = new THREE.Mesh(geo, mat);
                player.position.set(pos.x, 1.4, pos.z);
                scene.add(player);
                players.push(player);
            });
        }

        function startMatch() {
            document.getElementById("mainMenu").classList.add("hidden");
            document.getElementById("gameHUD").classList.remove("hidden");
            isPlaying = true;
            if(!scene) init3D();
        }

        function endMatch() {
            isPlaying = false;
            document.getElementById("gameHUD").classList.add("hidden");
            document.getElementById("mainMenu").classList.remove("hidden");
        }

        function playerPass() { console.log("Pas verildi!"); }
        function playerShoot() { console.log("ŞUT ÇEKİLDİ!"); ball.position.z -= 5; }

        // Joystick Kontrolleri
        const zone = document.getElementById("joystickZone");
        const knob = document.getElementById("joystickKnob");
        let activeTouch = false;
        let startX = 0, startY = 0;

        zone.addEventListener("touchstart", (e) => {
            activeTouch = true;
            startX = e.touches[0].clientX;
            startY = e.touches[0].clientY;
        });

        window.addEventListener("touchmove", (e) => {
            if (!activeTouch) return;
            let x = e.touches[0].clientX - startX;
            let y = e.touches[0].clientY - startY;
            let dist = Math.sqrt(x*x + y*y);
            let maxDist = 40;
            if (dist > maxDist) { x = (x/dist)*maxDist; y = (y/dist)*maxDist; }
            
            knob.style.transform = `translate(${x}px, ${y}px)`;
            moveX = x / maxDist;
            moveZ = y / maxDist;
        });

        window.addEventListener("touchend", () => {
            activeTouch = false;
            knob.style.transform = `translate(0px, 0px)`;
            moveX = 0; moveZ = 0;
        });

        // Animasyon ve Kamera Döngüsü
        function animate() {
            requestAnimationFrame(animate);
            if (isPlaying) {
                // Oyuncumuzu hareket ettir
                if (players.length > 0) {
                    players[10].position.x += moveX * 0.4;
                    players[10].position.z += moveZ * 0.4;

                    // eFootball Dinamik Kamera Açısı (Arkadan takip)
                    camera.position.x = players[10].position.x * 0.5;
                    camera.position.y = players[10].position.y + 12;
                    camera.position.z = players[10].position.z + 18;
                    camera.lookAt(players[10].position.x, players[10].position.y, players[10].position.z - 5);
                }
            }
            renderer.render(scene, camera);
        }

        window.addEventListener('resize', () => {
            if(camera && renderer) {
                camera.aspect = window.innerWidth / window.innerHeight;
                camera.updateProjectionMatrix();
                renderer.setSize(window.innerWidth, window.innerHeight);
            }
        });
    </script>
</body>
</html>
<!DOCTYPE html>
<html lang="tr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>VFL Mobile - 3D 11v11 Match Engine</title>
    <link href="https://fonts.googleapis.com/css2?family=Teko:wght@600;700&family=Poppins:wght@400;600;800&display=swap" rel="stylesheet">
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <style>
        :root {
            --epic-gold: #ffd700;
            --bg-dark: #0a0b10;
        }

        body {
            margin: 0;
            background: #08090d;
            color: white;
            font-family: 'Poppins', sans-serif;
            text-align: center;
            overflow: hidden;
            user-select: none;
            -webkit-user-select: none;
        }

        /* SCREEN MANAGEMENT */
        .screen {
            position: absolute;
            top: 0; left: 0; width: 100%; height: 100%;
            display: flex;
            flex-direction: column;
            justify-content: space-between;
            box-sizing: border-box;
            padding: 15px;
            z-index: 2;
            transition: opacity 0.4s ease;
        }
        .hidden { display: none !important; opacity: 0; pointer-<!DOCTYPE html>
<html lang="tr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>VFL Mobile - 3D 11v11 Match Engine</title>
    <link href="https://fonts.googleapis.com/css2?family=Teko:wght@600;700&family=Poppins:wght@400;600;800&display=swap" rel="stylesheet">
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <style>
        :root {
            --epic-gold: #ffd700;
            --bg-dark: #0a0b10;
        }

        body {
            margin: 0;
            background: #08090d;
            color: white;
            font-family: 'Poppins', sans-serif;
            text-align: center;
            overflow: hidden;
            user-select: none;
            -webkit-user-select: none;
        }

        /* SCREEN MANAGEMENT */
        .screen {
            position: absolute;
            top: 0; left: 0; width: 100%; height: 100%;
            display: flex;
            flex-direction: column;
            justify-content: space-between;
            box-sizing: border-box;
            padding: 15px;
            z-index: 2;
            transition: opacity 0.4s ease;
        }
        .hidden { display: none !important; opacity: 0; pointer-events: none; }

        /* START / TITLE SCREEN */
        #startScreen {
            z-index: 100;
            background: radial-gradient(circle at center, #2b1055 0%, #0a0b10 100%);
            justify-content: space-between;
            align-items: center;
            padding: 40px 20px;
            cursor: pointer;
        }
        .title-logo {
            font-family: 'Teko', sans-serif;
            font-size: 64px;
            font-weight: 700;
            letter-spacing: 3px;
            margin-top: 60px;
            background: linear-gradient(180deg, #ffffff 30%, #ffd700 100%);
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
            text-shadow: 0 0 30px rgba(255, 215, 0, 0.5);
        }
        .tap-to-start {
            font-family: 'Teko', sans-serif;
            font-size: 28px;
            color: #ffd700;
            letter-spacing: 2px;
            animation: blinkText 1.5s infinite alternate;
        }
        @keyframes blinkText {
            0% { opacity: 0.2; transform: scale(0.98); }
            100% { opacity: 1; transform: scale(1.05); }
        }
        .version-tag {
            font-size: 11px;
            color: #6c727f;
            letter-spacing: 1px;
        }

        /* HEADER */
        .header {
            display: flex;
            justify-content: space-between;
            align-items: center;
        }
        .user-badge {
            display: flex;
            align-items: center;
            gap: 10px;
            font-weight: 800;
            font-size: 14px;
        }
        .user-avatar {
            width: 36px; height: 36px;
            border-radius: 50%;
            background: linear-gradient(45deg, #00f2fe, #4facfe);
            display: flex; align-items: center; justify-content: center;
            border: 2px solid #fff;
        }
        .currencies { display: flex; gap: 8px; }
        .currency-box {
            background: rgba(255,255,255,0.08);
            backdrop-filter: blur(8px);
            padding: 5px 12px;
            border-radius: 20px;
            border: 1px solid rgba(255,255,255,0.15);
            font-size: 12px;
            font-weight: 600;
        }

        /* MAIN BANNER */
        .banner-container {
            margin: auto 0;
            background: linear-gradient(135deg, #2a0845, #120428);
            border-radius: 20px;
            border: 2px solid rgba(255, 215, 0, 0.5);
            box-shadow: 0 0 25px rgba(255, 215, 0, 0.25);
            padding: 30px 15px;
        }
        .banner-title {
            font-family: 'Teko', sans-serif;
            font-size: 42px;
            margin: 0;
            background: linear-gradient(180deg, #fff, #ffd700);
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
        }

        /* BUTTONS */
        .btn-epic {
            background: linear-gradient(90deg, #ffd700, #ffa500);
            color: #000;
            font-family: 'Teko', sans-serif;
            font-size: 24px;
            font-weight: 700;
            padding: 10px 40px;
            border: none;
            border-radius: 30px;
            cursor: pointer;
            box-shadow: 0 0 20px rgba(255, 215, 0, 0.6);
            text-transform: uppercase;
            margin-top: 15px;
        }
        .btn-epic:active { transform: scale(0.95); }

        /* MENU GRID */
        .grid-container {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 12px;
            margin-bottom: 20px;
        }
        .menu-card {
            background: rgba(22, 26, 38, 0.85);
            border-radius: 16px;
            padding: 15px 10px;
            border: 1px solid rgba(255,255,255,0.08);
            cursor: pointer;
        }
        .menu-card h3 { margin: 5px 0; font-size: 15px; }
        .menu-card p { font-size: 11px; color: #8a8f9d; margin: 0; }

        /* 3D GAMEPLAY STAGE */
        #gameCanvas {
            position: absolute;
            top: 0; left: 0; width: 100%; height: 100%;
            z-index: 1;
        }

        /* MATCH HUD */
        .match-hud {
            position: absolute;
            top: 15px; left: 0; width: 100%;
            display: flex;
            justify-content: center;
            align-items: center;
            gap: 15px;
            z-index: 5;
            pointer-events: none;
        }
        .scoreboard {
            background: rgba(0,0,0,0.8);
            border: 2px solid #ffd700;
            padding: 6px 20px;
            border-radius: 20px;
            font-family: 'Teko', sans-serif;
            font-size: 26px;
            letter-spacing: 2px;
        }
        .btn-back {
            position: absolute;
            top: 15px; left: 15px;
            background: rgba(255,0,0,0.8);
            color: white; border: none;
            padding: 6px 12px; border-radius: 10px;
            font-weight: bold; cursor: pointer; z-index: 10;
        }

        /* ON-SCREEN CONTROLS */
        .controls-overlay {
            position: absolute;
            bottom: 0; left: 0; width: 100%; height: 100%;
            pointer-events: none;
            z-index: 5;
            display: flex;
            justify-content: space-between;
            align-items: flex-end;
            padding: 20px;
            box-sizing: border-box;
        }
        .joystick-zone {
            width: 120px; height: 120px;
            background: rgba(255,255,255,0.1);
            border: 2px solid rgba(255,255,255,0.3);
            border-radius: 50%;
            pointer-events: auto;
            position: relative;
            touch-action: none;
        }
        .joystick-knob {
            width: 50px; height: 50px;
            background: rgba(255,215,0,0.8);
            border-radius: 50%;
            position: absolute;
            top: 35px; left: 35px;
            pointer-events: none;
            box-shadow: 0 0 10px #ffd700;
        }
        .action-buttons {
            display: flex;
            gap: 15px;
            pointer-events: auto;
            padding-bottom: 20px;
        }
        .act-btn {
            width: 65px; height: 65px;
            border-radius: 50%;
            border: 2px solid #ffd700;
            background: rgba(0,0,0,0.6);
            color: #ffd700;
            font-family: 'Teko', sans-serif;
            font-size: 20px;
            font-weight: bold;
            display: flex; align-items: center; justify-content: center;
            cursor: pointer;
            box-shadow: 0 0 10px rgba(255,215,0,0.4);
        }
        .act-btn:active { background: #ffd700; color: #000; }

        /* PACK MODAL */
        .modal {
            display: none;
            position: fixed;
            top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(5, 6, 10, 0.95);
            z-index: 20;
            justify-content: center; align-items: center;
            flex-direction: column;
        }
        .fut-card {
            width: 240px; height: 360px;
            background: linear-gradient(135deg, #1e1103 0%, #4a3000 50%, #1a0f00 100%);
            border-radius: 18px;
            border: 3px solid #ffd700;
            box-shadow: 0 0 30px rgba(255, 215, 0, 0.6);
            position: relative;
            padding: 15px;
            box-sizing: border-box;
            animation: cardReveal 0.6s cubic-bezier(0.175, 0.885, 0.32, 1.275);
        }
        @keyframes cardReveal {
            0% { transform: scale(0) rotate(-10deg); opacity: 0; }
            100% { transform: scale(1) rotate(0deg); opacity: 1; }
        }
    </style>
</head>
<body>

    <!-- OYUN BAŞLAMA EKRANI (TITLE SCREEN) -->
    <div id="startScreen" class="screen" onclick="enterGame()">
        <div class="title-logo">VFL MOBILE</div>
        <div class="tap-to-start">GİRİŞ İÇİN EKRANA DOKUN</div>
        <div class="version-tag">v1.0.4 • Season 1</div>
    </div>

    <!-- ANA MENÜ EKRANI -->
    <div id="mainMenu" class="screen hidden">
        <div class="header">
            <div class="user-badge">
                <div class="user-avatar">⚽</div>
                <div>emrr38 <span style="font-size:10px; color:#ffd700; display:block;">LVL 15</span></div>
            </div>
            <div class="currencies">
                <div class="currency-box">🪙 1,450</div>
                <div class="currency-box" style="border-color: rgba(255,215,0,0.4);">💎 350</div>
            </div>
        </div>

        <div class="banner-container">
            <h2 class="banner-title">SPAIN LEGENDS</h2>
            <p style="font-size: 12px; color: #d1c4e9; margin: 0;">EPIC & BIG TIME KARTLARI</p>
            <button class="btn-epic" onclick="openPack()">PAKETİ AÇ (100 💎)</button>
        </div>

        <div class="grid-container">
            <div class="menu-card" onclick="startMatch()">
                <div style="font-size: 24px;">🏟️</div>
                <h3>11v11 MAÇ BAŞLAT</h3>
                <p>3D eFootball Simülasyonu</p>
            </div>
            <div class="menu-card" onclick="alert('Kadro ve formasyon yönetimi aktif!')">
                <div style="font-size: 24px;">🛡️</div>
                <h3>KADROM</h3>
                <p>11 Oyuncu Düzeni</p>
            </div>
        </div>
    </div>

    <!-- 3D OYUN ALANI -->
    <canvas id="gameCanvas"></canvas>

    <!-- MAÇ İÇİ HUD VE KONTROLLER -->
    <div id="gameHUD" class="screen hidden" style="pointer-events: none;">
        <button class="btn-back" onclick="endMatch()">ÇIKIŞ</button>
        <div class="match-hud">
            <div class="scoreboard">VFL 0 - 0 </div>
        </div>

        <div class="controls-overlay">
            <div class="joystick-zone" id="joystickZone">
                <div class="joystick-knob" id="joystickKnob"></div>
            </div>
            <div class="action-buttons">
                <div class="act-btn" onclick="playerPass()">PAS</div>
                <div class="act-btn" onclick="playerShoot()" style="background: rgba(255,0,0,0.5); border-color:#ff4d4d;">ŞUT</div>
            </div>
        </div>
    </div>

    <!-- PAKET AÇILIŞ MODALI -->
    <div id="packModal" class="modal" onclick="closeModal()">
        <div class="fut-card">
            <div style="display:flex; justify-content:space-between; align-items:flex-start;">
                <div style="text-align:left;">
                    <div style="font-family:'Teko'; font-size:48px; font-weight:700; color:#ffd700; line-height:35px;" id="cardOvr">106</div>
                    <div style="font-size:14px; font-weight:800; background:rgba(0,0,0,0.5); padding:2px 6px; border-radius:4px; display:inline-block;" id="cardPos">OS</div>
                </div>
            </div>
            <h2 id="cardName" style="font-family:'Teko'; font-size:32px; margin: 120px 0 0 0; text-transform:uppercase;">XAVI</h2>
            <p style="font-size: 10px; color: #ffd700; font-weight: 700; letter-spacing:2px; margin:0;" id="cardTier">EPIC • SPAIN</p>
        </div>
        <p style="margin-top: 20px; font-size: 12px; color: #aaa;">Kapatmak için ekrana dokun</p>
    </div>

    <script>
        // BAŞLAMA EKRANINDAN GEÇİŞ
        function enterGame() {
            document.getElementById("startScreen").classList.add("hidden");
            document.getElementById("mainMenu").classList.remove("hidden");
        }

        // PAKET SİSTEMİ
        const legends = [
            { name: "Xavi", ovr: "106", pos: "OS", tier: "EPIC • SPAIN" },
            { name: "Iniesta", ovr: "105", pos: "OS", tier: "EPIC • SPAIN" },
            { name: "S. Ramos", ovr: "104", pos: "DEF", tier: "BIG TIME • SPAIN" },
            { name: "C. Puyol", ovr: "103", pos: "DEF", tier: "BIG TIME • SPAIN" },
            { name: "Casillas", ovr: "102", pos: "GK", tier: "HIGHLIGHT • SPAIN" }
        ];

        function openPack() {
            const player = legends[Math.floor(Math.random() * legends.length)];
            document.getElementById("cardName").innerText = player.name;
            document.getElementById("cardOvr").innerText = player.ovr;
            document.getElementById("cardPos").innerText = player.pos;
            document.getElementById("cardTier").innerText = player.tier;
            document.getElementById("packModal").style.display = "flex";
        }
        function closeModal() { document.getElementById("packModal").style.display = "none"; }

        // 3D THREE.JS MAÇ MOTORU
        let scene, camera, renderer, ball, players = [];
        let moveX = 0, moveZ = 0;
        let isPlaying = false;

        function init3D() {
            const canvas = document.getElementById("gameCanvas");
            scene = new THREE.Scene();
            scene.background = new THREE.Color(0x0f172a);

            camera = new THREE.PerspectiveCamera(60, window.innerWidth / window.innerHeight, 0.1, 1000);
            renderer = new THREE.WebGLRenderer({ canvas: canvas, antialias: true });
            renderer.setSize(window.innerWidth, window.innerHeight);

            // Işıklandırma
            const ambientLight = new THREE.AmbientLight(0xffffff, 0.7);
            scene.add(ambientLight);
            const dirLight = new THREE.DirectionalLight(0xffffff, 0.8);
            dirLight.position.set(20, 40, 20);
            scene.add(dirLight);

            // 11v11 Çim Saha
            const pitchGeo = new THREE.PlaneGeometry(60, 90);
            const pitchMat = new THREE.MeshBasicMaterial({ color: 0x1b4d3e });
            const pitch = new THREE.Mesh(pitchGeo, pitchMat);
            pitch.rotation.x = -Math.PI / 2;
            scene.add(pitch);

            // Saha Çizgileri ve Kaleler
            const lineGeo = new THREE.BoxGeometry(58, 0.1, 88);
            const wireMat = new THREE.MeshBasicMaterial({ color: 0xffffff, wireframe: true });
            const lines = new THREE.Mesh(lineGeo, wireMat);
            lines.position.y = 0.05;
            scene.add(lines);

            // Top
            const ballGeo = new THREE.SphereGeometry(0.6, 16, 16);
            const ballMat = new THREE.MeshStandardMaterial({ color: 0xffffff });
            ball = new THREE.Mesh(ballGeo, ballMat);
            ball.position.set(0, 0.6, 0);
            scene.add(ball);

            // Oyuncuları Oluştur (Takımlar: Kırmızı ve Mavi 11v11)
            createTeam(0xcc2929, -1); // Rakip
            createTeam(0x2952cc, 1);  // Kendi Kadromuz

            animate();
        }

        function createTeam(color, zDir) {
            const mat = new THREE.MeshStandardMaterial({ color: color });
            const geo = new THREE.CapsuleGeometry(0.8, 1.8, 4, 8);

            // 4-3-3 Formasyon Dizilimi
            const positions = [
                {x: 0, z: zDir * 40}, // Kaleci
                {x: -15, z: zDir * 25}, {x: -5, z: zDir * 28}, {x: 5, z: zDir * 28}, {x: 15, z: zDir * 25}, // Defans
                {x: -10, z: zDir * 10}, {x: 0, z: zDir * 5}, {x: 10, z: zDir * 10}, // Orta Saha
                {x: -18, z: zDir * -10}, {x: 0, z: zDir * -15}, {x: 18, z: zDir * -10}  // Forvet
            ];

            positions.forEach(pos => {
                const player = new THREE.Mesh(geo, mat);
                player.position.set(pos.x, 1.4, pos.z);
                scene.add(player);
                players.push(player);
            });
        }

        function startMatch() {
            document.getElementById("mainMenu").classList.add("hidden");
            document.getElementById("gameHUD").classList.remove("hidden");
            isPlaying = true;
            if(!scene) init3D();
        }

        function endMatch() {
            isPlaying = false;
            document.getElementById("gameHUD").classList.add("hidden");
            document.getElementById("mainMenu").classList.remove("hidden");
        }

        function playerPass() { console.log("Pas verildi!"); }
        function playerShoot() { console.log("ŞUT ÇEKİLDİ!"); ball.position.z -= 5; }

        // Joystick Kontrolleri
        const zone = document.getElementById("joystickZone");
        const knob = document.getElementById("joystickKnob");
        let activeTouch = false;
        let startX = 0, startY = 0;

        zone.addEventListener("touchstart", (e) => {
            activeTouch = true;
            startX = e.touches[0].clientX;
            startY = e.touches[0].clientY;
        });

        window.addEventListener("touchmove", (e) => {
            if (!activeTouch) return;
            let x = e.touches[0].clientX - startX;
            let y = e.touches[0].clientY - startY;
            let dist = Math.sqrt(x*x + y*y);
            let maxDist = 40;
            if (dist > maxDist) { x = (x/dist)*maxDist; y = (y/dist)*maxDist; }
            
            knob.style.transform = `translate(${x}px, ${y}px)`;
            moveX = x / maxDist;
            moveZ = y / maxDist;
        });

        window.addEventListener("touchend", () => {
            activeTouch = false;
            knob.style.transform = `translate(0px, 0px)`;
            moveX = 0; moveZ = 0;
        });

        // Animasyon ve Kamera Döngüsü
        function animate() {
            requestAnimationFrame(animate);
            if (isPlaying) {
                if (players.length > 0) {
                    players[10].position.x += moveX * 0.4;
                    players[10].position.z += moveZ * 0.4;

                    camera.position.x = players[10].position.x * 0.5;
                    camera.position.y = players[10].position.y + 12;
                    camera.position.z = players[10].position.z + 18;
                    camera.lookAt(players[10].position.x, players[10].position.y, players[10].position.z - 5);
                }
            }
            renderer.render(scene, camera);
        }

        window.addEventListener('resize', () => {
            if(camera && renderer) {
                camera.aspect = window.innerWidth / window.innerHeight;
                camera.updateProjectionMatrix();
                renderer.setSize(window.innerWidth, window.innerHeight);
            }
        });
    </script>
</body>
</html>
