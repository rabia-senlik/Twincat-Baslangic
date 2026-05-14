# TwinCAT 3 Başlangıç Seviyesi PLC Programlama Eğitimi

Bu depo, Beckhoff TwinCAT 3 platformu üzerinde PLC programlamaya yeni başlayanlar için hazırlanmış temel seviye uygulama ve kodlama rehberidir. İçerik, kurulum aşamasından başlayarak yapılandırılmış veri tiplerine (DUT/Struct) kadar uzanan modüler bir müfredatı barındırır.

---

## Önerilen Depo Klasör Yapısı

Projenin modüler ve standartlara uygun olması adına yerel dizininizde `twincat-baslangic` adında bir ana klasör açıp içerisini aşağıdaki hiyerarşiye göre düzenlemeniz önerilir:

```plaintext
twincat-baslangic/
│
├── README.md
├── .gitignore
└── src/
    └── TwinCAT_Baslangic_Projesi/
        ├── TwinCAT_Baslangic_Projesi.sln
        └── TwinCAT_Baslangic_Projesi/
            ├── TwinCAT_Baslangic_Projesi.tsproj
            └── PLC_Projesi/
                ├── PLC_Projesi.plcproj
                ├── DUTs/
                │   └── Eksen_Ozellikleri.TcDUT
                ├── GVLs/
                │   └── GVL.TcGVL
                └── POUs/
                    ├── MAIN.TcPOU
                    ├── ACT_KILITLEME.TcPOU
                    ├── POU_OR_LOGIC.TcPOU
                    ├── F_GerilimOkuma.TcPOU
                    └── FB_MotorKontrol.TcPOU
```

---

## İçindekiler
1. [TwinCAT 3 İndirme ve Kurulum](#1-twincat-3-indirme-ve-kurulum)
2. [Donanım Bağlantısı ve IP Ayarları](#2-donanım-bağlantısı-ve-ip-ayarları)
3. [Free Run Modu ile I/O Testi](#3-free-run-modu-ile-io-testi)
4. [İlk PLC Projesi ve Değişken Tanımlama](#4-ilk-plc-projesi-ve-değişken-tanımlama)
5. [Fiziksel I/O Linkleme](#5-fiziksel-io-linkleme)
6. [Action ve Program Blokları (POU) Oluşturma](#6-action-ve-program-blokları-pou-oluşturma)
7. [Global Değişken Listesi (GVL) ve Kütüphane Yönetimi](#7-global-değişken-listesi-gvl-ve-kütüphane-yönetimi)
8. [Analog Sinyal İşleme ve Ölçekleme](#8-analog-sinyal-işleme-ve-ölçekleme)
9. [Fonksiyon (FUN) ve Fonksiyon Bloğu (FB) Mimarisi](#9-fonksiyon-fun-ve-fonksiyon-bloğu-fb-mimarisi)
10. [Yapılandırılmış Veri Tipleri (DUT / Struct)](#10-yapılandırılmış-veri-tipleri-dut--struct)

---

### 1. TwinCAT 3 İndirme ve Kurulum
* **İndirme:** [beckhoff.com](https://www.beckhoff.com) adresine giderek arama bölümüne "TwinCAT 3" yazılır veya "Yazılım > TwinCAT 3" dizini takip edilir. Dosyayı indirebilmek için kurumsal veya kişisel bir e-posta adresiyle ücretsiz kayıt olunması ve e-posta onayının tamamlanması gerekmektedir.
* **Kurulum:** İndirilen çalıştırılabilir dosya (`setup.exe`) açılır. Sistemde önceden kurulu bir Visual Studio sürümü yoksa, kurulum esnasında sunulan **Complete** seçeneği ile entegre "Microsoft Visual Studio Shell" bileşeni seçilerek arayüzün tam kurulumu gerçekleştirilir. İşlem bitiminde sistem yeniden başlatılır.

### 2. Donanım Bağlantısı ve IP Ayarları
Mühendislik bilgisayarı ile Beckhoff Endüstriyel PC (IPC) veya CX serisi kontrolör arasında iletişim kurmak için statik veya dinamik IP eşleşmesi sağlanmalıdır.
* **Ağ Ayarları:** IPC'nin IP adresi ile bilgisayarın IP adresi aynı alt ağda (subnet) olmalıdır (Örn: `192.168.1.X`). Ping testi ile fiziksel erişim doğrulanmalıdır.
* **Hedef Seçimi (Choose Target):** TwinCAT projesinde sol menüden `System > Choose Target > Search (Ethernet)` adımları izlenir. **Broadcast Search** yapılarak ağdaki cihazlar taranır.
* **Route Ekleme:** Bulunan cihaza tıklanıp `Add Route` denilir. İşletim sistemine bağlı olarak varsayılan şifre girilir (Windows için genelde `1` veya cihaza özel tanımlı şifre; Windows CE için şifre alanı boş bırakılır). Bağlantı başarılı olduğunda cihaz adının yanında "X" işareti belirir.

### 3. Free Run Modu ile I/O Testi
Henüz hiçbir PLC kodu yazmadan sahadaki sensörleri ve aktüatörleri test etmek için kullanılır.
* **Aktivasyon:** Cihaz hedef olarak seçildikten sonra üst menüden **Reload Devices** butonuna basılır ve gelen "Activate Free Run" uyarısına onay verilir. Görev çubuğundaki TwinCAT ikonunun kırmızı-mavi yanıp sönmesi bu moda geçildiğini gösterir.
* **Kullanım:** `I/O > Devices` altından ilgili giriş/çıkış modülünün kanallarına (`Channel`) ulaşılır. Çıkış modüllerinin `Online` sekmesinden `Write` komutu ile lojik 1/0 veya analog voltaj değerleri zorlanarak (force) saha elemanları test edilir.

### 4. İlk PLC Projesi ve Değişken Tanımlama
* Projeye sağ tıklanıp `Add New Item > Standard PLC Project` seçilir.
* Değişkenler her zaman `VAR` ile `END_VAR` blokları arasında tanımlanır.
* **Örnek Kod (ST - Structured Text):**
  ```iecst
  VAR
      nCounter : INT;
  END_VAR
  
  nCounter := nCounter + 1;
  ```
* Kodu yüklemek için önce sistem Run Mode'a alınır (yeşil ikon), ardından Login komutu ile derlenen kod denetleyiciye indirilir ve Start butonuna basılarak döngü başlatılır.

### 5. Fiziksel I/O Linkleme
Yazılımdaki değişkenleri fiziksel modüllerle eşleştirmek için donanım adresleme etiketleri kullanılır.
* **Adresleme:**
  * Dijital Giriş: `AT %I* : BOOL;`
  * Dijital Çıkış: `AT %Q* : BOOL;`
  * Analog Giriş: `AT %I* : INT;`
  * Analog Çıkış: `AT %Q* : INT;`
* **Linkleme İşlemi:** Kodu derledikten (Build) sonra proje ağacında oluşan değişken çift tıklanır. Variable sekmesinden Link to... denilerek I/O konfigürasyonundaki ilgili terminal kanalı seçilir. Değişikliklerin donanıma işlenmesi için Activate Configuration komutu çalıştırılmalıdır.

### 6. Action ve Program Blokları (POU) Oluşturma
Proje okunabilirliğini artırmak için kodlar mantıksal parçalara bölünür.
* **Action:** Ana programa (MAIN) sağ tıklanıp Add > Action denilerek oluşturulur. Ana programın yerel değişkenlerini kullanır. Çağırmak için ana programda `ACTION_ADI();` yazılır.
* **POU (Program):** POUs klasörüne sağ tıklanıp Add > POU (Tipi: Program) seçilir. Kendi yerel değişken belleğine sahiptir. Ana programda işlenebilmesi için MAIN içerisinde çağrılmalıdır.

### 7. Global Değişken Listesi (GVL) ve Kütüphane Yönetimi
* **GVL:** Tüm POU'lardan doğrudan erişilebilen genel değişkenler tanımlamak için kullanılır. GVLs klasörüne sağ tıklanıp eklenir. Tanımlanan bir değişkene program içerisinden `GVL.DegiskenAdi` şeklinde erişilir. `VAR_GLOBAL` etiketinin yanındaki attribute 'qualified_only' satırı silinirse, POU içerisinde doğrudan `DegiskenAdi` yazılarak da kullanılabilir.
* **Kütüphane Ekleme:** References klasörüne sağ tıklanıp Add Library seçilir. İletişim, hareket kontrolü veya sistem fonksiyonları projeye dahil edilir.

### 8. Analog Sinyal İşleme ve Ölçekleme
12 bit veya 16 bit çözünürlüğe sahip analog terminallerden okunan ham değerler (0..32767) mühendislik birimlerine dönüştürülmelidir.
* **Analog Giriş Ölçekleme (0-10V):**
  ```iecst
  rGerilimInput := (INT_TO_REAL(nInput) / 32767.0) * 10.0;
  ```
* **Analog Çıkış Ölçekleme (0-10V):**
  ```iecst
  nOutput := REAL_TO_INT((rGerilimOutput / 10.0) * 32767.0);
  ```

### 9. Fonksiyon (FUN) ve Fonksiyon Bloğu (FB) Mimarisi
* **Fonksiyon (FUN):** Çağrıldığında giriş parametrelerini işler ve geriye tek bir sonuç döndürür. İçsel bir hafızası (durum koruması) yoktur. Her çağrıldığında belleği sıfırlanır.
* **Fonksiyon Bloğu (FB):** Kendi dahili belleğine sahiptir. İşlemlerin durumunu (state) döngüler arasında korur. Kullanılabilmesi için ana programda bir kopyası (örneği / instance) tanımlanmalıdır.

### 10. Yapılandırılmış Veri Tipleri (DUT / Struct)
Birden fazla değişkeni tek bir çatı altında toplamak için kullanılır. Gelişmiş veri yönetimi sağlar.
* DUTs klasörüne sağ tıklanıp Add > DUT (Tipi: Structure) seçilir.
* Tanımlanan yapı, program içerisinde yeni bir veri tipi olarak deklare edilir ve alt elemanlarına nokta (.) operatörü ile erişilir (Örn: `Eksen1.rGercekPozisyon`).

---

## Kaynak Kodlar ve Proje Dosyaları

Aşağıdaki bloklar, projenin derlenebilmesi için gerekli olan temel TwinCAT kaynak kodlarını içermektedir. İlgili başlıklar altındaki XML bloklarını projenizin ilgili dizinlerine doğrudan nesne dosyası olarak kaydedebilirsiniz.

### `src/TwinCAT_Baslangic_Projesi/PLC_Projesi/DUTs/Eksen_Ozellikleri.TcDUT`
```xml
<TcPlcObject Version="1.1.0.1" ProductVersion="3.1.4024.12">
  <DUT Name="Eksen_Ozellikleri" Id="{b1a2c3d4-e5f6-7a8b-9c0d-1e2f3a4b5c6d}">
    <Declaration><![CDATA[TYPE Eksen_Ozellikleri :
STRUCT
    bBasla          : BOOL;
    rHedefPozisyon  : REAL;
    rHedefHiz       : REAL;
    rGercekPozisyon : REAL;
    rGercekHiz      : REAL;
END_STRUCT
END_TYPE
]]></Declaration>
  </DUT>
</TcPlcObject>
```

### `src/TwinCAT_Baslangic_Projesi/PLC_Projesi/GVLs/GVL.TcGVL`
```xml
<TcPlcObject Version="1.1.0.1" ProductVersion="3.1.4024.12">
  <GVL Name="GVL" Id="{a1b2c3d4-e5f6-7a8b-9c0d-1e2f3a4b5c6d}">
    <Declaration><![CDATA[{attribute 'qualified_only'}
VAR_GLOBAL
    nTest1 : INT;
    nTest2 : INT;
END_VAR]]></Declaration>
  </GVL>
</TcPlcObject>
```

### `src/TwinCAT_Baslangic_Projesi/PLC_Projesi/POUs/F_GerilimOkuma.TcPOU`
```xml
<TcPlcObject Version="1.1.0.1" ProductVersion="3.1.4024.12">
  <POU Name="F_GerilimOkuma" Id="{c1d2e3f4-a5b6-7c8d-9e0f-1a2b3c4d5e6f}" SpecialFunc="None">
    <Declaration><![CDATA[FUNCTION F_GerilimOkuma : REAL
VAR_INPUT
    nHamDeger : INT;
END_VAR
]]></Declaration>
    <Implementation>
      <ST><![CDATA[// 0-32767 integer değerini 0.0 - 10.0V aralığına dönüştürür
F_GerilimOkuma := (INT_TO_REAL(nHamDeger) / 32767.0) * 10.0;
]]></ST>
    </Implementation>
  </POU>
</TcPlcObject>
```

### `src/TwinCAT_Baslangic_Projesi/PLC_Projesi/POUs/FB_MotorKontrol.TcPOU`
```xml
<TcPlcObject Version="1.1.0.1" ProductVersion="3.1.4024.12">
  <POU Name="FB_MotorKontrol" Id="{f1e2d3c4-b5a6-7f8e-9d0c-1b2a3f4e5d6c}" SpecialFunc="None">
    <Declaration><![CDATA[FUNCTION_BLOCK FB_MotorKontrol
VAR_INPUT
    bStart : BOOL;
    bStop  : BOOL;
END_VAR
VAR_OUTPUT
    bMotorRun : BOOL;
END_VAR
]]></Declaration>
    <Implementation>
      <ST><![CDATA[// Start ve Stop butonları ile basit mühürleme lojiği
IF bStart THEN
    bMotorRun := TRUE;
END_IF

IF bStop THEN
    bMotorRun := FALSE;
END_IF
]]></ST>
    </Implementation>
  </POU>
</TcPlcObject>
```

### `src/TwinCAT_Baslangic_Projesi/PLC_Projesi/POUs/MAIN.TcPOU`
```xml
<TcPlcObject Version="1.1.0.1" ProductVersion="3.1.4024.12">
  <POU Name="MAIN" Id="{e1f2a3b4-c5d6-7e8f-9a0b-1c2d3e4f5a6b}" SpecialFunc="None">
    <Declaration><![CDATA[PROGRAM MAIN
VAR
    // Bölüm 4: Temel Sayaç
    nCounter : INT;

    // Bölüm 5: Donanım Linkleme Değişkenleri
    bDigitalInput  AT %I* : BOOL;
    bDigitalOutput AT %Q* : BOOL;
    nAnalogInput   AT %I* : INT;
    nAnalogOutput  AT %Q* : INT;

    // Bölüm 8: Analog Ölçekleme
    rGerilimInput  : REAL;
    rGerilimOutput : REAL;

    // Bölüm 9: Fonksiyon ve FB Örneklemesi
    rKanal1_Voltaj : REAL;
    rKanal2_Voltaj : REAL;
    nHamKanal2     AT %I* : INT;
    
    fbMotor1       : FB_MotorKontrol;
    bSahaStart     : BOOL;
    bSahaStop      : BOOL;
    bSahaMotor     : BOOL;

    // Bölüm 10: Yapılandırılmış Veri Tipi
    stEksen1       : Eksen_Ozellikleri;
    stEksen2       : Eksen_Ozellikleri;
END_VAR
]]></Declaration>
    <Implementation>
      <ST><![CDATA[// --- BÖLÜM 4: Temel Sayaç Lojiği ---
nCounter := nCounter + 1;

// --- BÖLÜM 5: I/O Eşitleme ---
bDigitalOutput := bDigitalInput;

// --- BÖLÜM 7: Global Değişken Kullanımı ---
GVL.nTest1 := GVL.nTest1 + 1;

// --- BÖLÜM 8: Analog Çıkış Yazma ve Giriş Okuma Dönüşümleri ---
// İstenen Real voltajı integer sinyale çevirip fiziksel çıkışa atama
nAnalogOutput := REAL_TO_INT((rGerilimOutput / 10.0) * 32767.0);

// Fiziksel girişten okunan integer sinyali Real voltaja dönüştürme
rGerilimInput := (INT_TO_REAL(nAnalogInput) / 32767.0) * 10.0;

// --- BÖLÜM 9: Fonksiyon (FUN) Kullanımı ---
rKanal1_Voltaj := F_GerilimOkuma(nHamDeger := nAnalogInput);
rKanal2_Voltaj := F_GerilimOkuma(nHamDeger := nHamKanal2);

// --- BÖLÜM 9: Fonksiyon Bloğu (FB) Kullanımı ---
fbMotor1(
    bStart    := bSahaStart, 
    bStop     := bSahaStop, 
    bMotorRun => bSahaMotor
);

// --- BÖLÜM 10: Structure (DUT) Veri Yapısı Kullanımı ---
stEksen1.rGercekPozisyon := 1250.5;
stEksen1.rHedefPozisyon  := stEksen1.rGercekPozisyon + 500.0;

// İki yapının verilerini birebir eşitleme
stEksen2 := stEksen1;
]]></ST>
    </Implementation>
  </POU>
</TcPlcObject>
```

---

## .gitignore Dosyası

TwinCAT ve Visual Studio derleme kalıntılarını ve kullanıcıya özel geçici proje dosyalarını depodan uzak tutmak için deponun ana dizinindeki `.gitignore` dosyasına eklenmesi gereken içerik:

```plaintext
# TwinCAT ignore listesi
*.tclrs
*.bak
*.sou
*.suo
*.user
*.docstates
*.compileinfo
*_Boot/
*_CompileInfo/
*~
*.tmp
.vs/
bin/
obj/
```
