# TwinCAT 3 Başlangıç Seviyesi PLC Programlama Eğitimi

Bu depo, Beckhoff TwinCAT 3 platformu üzerinde PLC programlamaya yeni başlayanlar için hazırlanmış temel seviye uygulama ve kodlama rehberidir. İçerik, kurulum aşamasından başlayarak yapılandırılmış veri tiplerine (DUT/Struct) kadar uzanan modüler bir müfredatı barındırır.

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
* **Kurulum:** İndirilen çalıştırılabilir dosya (`setup.exe`) açılır. Sistemde önceden kurulu bir Visual Studio sürümü yoksa, kurulum esnasında sunulan **Complete** seçeneği ile entegre "Microsoft Visual Studio Shell" (örneğin 2013 Shell) bileşeni seçilerek arayüzün tam kurulumu gerçekleştirilir. İşlem bitiminde sistem yeniden başlatılır.

### 2. Donanım Bağlantısı ve IP Ayarları
Mühendislik bilgisayarı ile Beckhoff Endüstriyel PC (IPC) veya CX serisi kontrolör arasında iletişim kurmak için statik veya dinamik IP eşleşmesi sağlanmalıdır.
* **Ağ Ayarları:** IPC'nin IP adresi ile bilgisayarın IP adresi aynı alt ağda (subnet) olmalıdır (Örn: `192.168.1.X`). Ping testi ile fiziksel erişim doğrulanmalıdır.
* **Hedef Seçimi (Choose Target):** TwinCAT projesinde sol menüden `System > Choose Target > Search (Ethernet)` adımları izlenir. **Broadcast Search** yapılarak ağdaki cihazlar taranır.
* **Route Ekleme:** Bulunan cihaza tıklanıp `Add Route` denilir. İşletim sistemine bağlı olarak varsayılan şifre girilir (Windows 7/XP/10 için genelde `1` veya cihaza özel tanımlı şifre; Windows CE için şifre alanı boş bırakılır). Bağlantı başarılı olduğunda cihaz adının yanında "X" işareti belirir.

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