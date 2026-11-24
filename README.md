[English](README.en.md)

# Serverless P2P Encrypted Video Call (WebRTC)

Üçüncü parti medya sunucularına (SFU/MCU), veritabanlarına veya bulut kayıt sistemlerine ihtiyaç duymadan, iki cihaz arasında **doğrudan uçtan uca şifreli (P2P)** ses ve görüntü iletişimi sağlayan minimalist WebRTC görüntülü görüşme uygulaması.

---

## Temel Özellikler

* **%100 Doğrudan P2P Medya Akışı:** Ses ve görüntü paketleri DTLS/SRTP protokolleri ile şifrelenerek doğrudan iki cihaz arasında akar; hiçbir merkezi sunucu üzerinden geçmez.
* **Sıfır Kayıt (Zero-Logging):** Sinyalleşme sunucusu (`server.js`) yalnızca bağlantı el sıkışması için gerekli olan SDP ve ICE adaylarını bellekte anlık olarak iletir. Diske, veritabanına veya tarayıcı önbelleğine hiçbir veri kaydedilmez.
* **Sıfır Kurulumlu İstemci:** Karşı tarafın herhangi bir uygulama, eklenti veya VPN kurmasına gerek yoktur. Oluşturulan HTTPS bağlantısı üzerinden iOS Safari, Android Chrome ve masaüstü tarayıcılarda doğrudan çalışır.
* **Akıllı Arayüz & Mobil Uyum:**
  * Kesilmesiz (`object-fit: contain`) tam ekran karşı taraf görüntüsü.
  * Sağ alt köşede ayna modlu Picture-in-Picture (PiP) yerel kamera önizlemesi.
  * Görüşme sırasında 3 saniye hareketsizlikte kontrolleri otomatik gizleme (auto-hide).
  * Mobil cihazlar için tek tuşla ön/arka kamera geçişi (`facingMode`) ve yankı/gürültü engelleme.

---

## Gereksinimler (Yalnızca Ev Sahibi Bilgisayar İçin)

* [Node.js](https://nodejs.org/) (v18 veya üzeri)
* [Cloudflare Tunnel (`cloudflared`)](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/downloads/)

---

## Kurulum ve Çalıştırma

### 1. Projeyi İndirin ve Bağımlılıkları Yükleyin

Proje dizininde terminali açın ve bağımlılıkları kurun:

```bash
npm install
```

### 2. Sinyalleşme Sunucusunu Başlatın

İlk terminal penceresinde yerel HTTP ve WebSocket sinyalleşme sunucusunu başlatın:

```bash
node server.js
```

Çıktı: `Sunucu 3000 portunda hazir.`

### 3. Güvenli HTTPS Tüneli Oluşturun

İkinci bir terminal penceresinde yerel sunucuyu SSL/HTTPS sertifikalı olarak dış dünyaya açmak için Cloudflare Tunnel başlatın:

```bash
cloudflared tunnel --url http://localhost:3000
```

### 4. Görüşmeyi Başlatın

1. Terminal çıktısında üretilen `https://<rastgele-alt-alan-adi>.trycloudflare.com` adresini kopyalayın.
2. Linki hem kendi tarayıcınızda açın hem de görüşmek istediğiniz kişiye gönderin. (İstemci WebSocket adresini mevcut URL üzerinden dinamik olarak algılar; kod üzerinde herhangi bir düzenleme yapmanıza gerek yoktur.)
3. İki tarafta da "Bağlantıya hazır" durumu göründüğünde **Ara** ve **Gelen Aramayı Kabul Et** butonlarıyla P2P görüşmeyi başlatın.

---

## Proje Yapısı

```plaintext
├── index.html      # Tek dosyalık WebRTC istemci arayüzü ve sinyalleşme mantığı
├── server.js       # 2 kişilik oda kapasiteli minimalist HTTP + WebSocket sinyalleşme sunucusu
├── package.json    # Node.js proje bağımlılıkları (ws)
├── package-lock.json
├── .gitignore      # node_modules dışlama kuralı
└── README.md       # Proje dokümantasyonu
```

---

## Ağ ve Güvenlik Duvarı Notları

* **CGNAT / Kurumsal Ağlar:** Doğrudan P2P bağlantısı UDP protokolü üzerinden kurulur. Katı güvenlik duvarına (Symmetric NAT) sahip kurumsal, okul veya halka açık Wi-Fi ağları UDP trafiğini engelleyebilir; bu durumlarda standart ev ağı veya mobil erişim noktası (hotspot) kullanılması önerilir.
* **Hotspot NAT Loopback:** Aynı cihazın mobil erişim noktasını (hotspot) paylaşırken hem paylaşan telefon hem de o ağa bağlı bilgisayar arasında P2P bağlantı kurmaya çalışmak yerel ağ döngüsüne (NAT Hairpinning) takılabilir. Testlerin farklı ağlara bağlı iki cihaz arasında yapılması gerekir.

---

## Güvenlik Notları

* **Bağlantıyı paylaşırken dikkatli olun:** `trycloudflare.com` adresini yalnızca görüşeceğiniz kişiye gönderin. Odada şifre yoktur; bağlantıyı bilen ve oda boşken giren herkes görüşmeye katılabilir. Adres her çalıştırmada değişir, ancak bu linki README, commit mesajı veya issue gibi herkese açık yerlere yapıştırmayın.
* **Sinyalleşme sunucusu:** Medya akışı P2P ve DTLS/SRTP ile şifrelidir; ancak bağlantı kurulum mesajları (SDP/ICE) sizin sunucunuz ve Cloudflare Tunnel üzerinden geçer. "Sıfır kayıt" yalnızca bu sunucunun kod davranışı için geçerlidir.
