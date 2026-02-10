# 🌐 OSI Referans Modeli: Ağ İletişimi Temelleri

![Network Engineering](https://img.shields.io/badge/Topic-Network_Architecture-blue?style=for-the-badge&logo=cisco)
![Standard](https://img.shields.io/badge/Standard-ISO%2FOSI-green?style=for-the-badge)

**Open Systems Interconnection (OSI)** modeli, 1984 yılında ISO (International Organization for Standardization) tarafından yayınlanan; farklı donanım ve yazılımlara sahip bilgisayarların birbiriyle nasıl iletişim kuracağını tanımlayan 7 katmanlı evrensel bir standarttır.

Bu model, karmaşık bir ağ iletişim sürecini **"Böl ve Yönet"** prensibiyle daha küçük, anlaşılır ve yönetilebilir parçalara ayırır.

---

## 🏗️ OSI Modelinin 7 Katmanı

Aşağıdan yukarıya (Donanımdan Yazılıma) doğru sıralanış şöyledir:

| # | Katman Adı (Layer) | Veri Birimi (PDU) | Temel İşlevi |
|:---:|:---|:---|:---|
| **7** | **Uygulama** (Application) | Veri (Data) | Kullanıcı arayüzü ve ağ servisleri. |
| **6** | **Sunum** (Presentation) | Veri (Data) | Veri formatlama, şifreleme, sıkıştırma. |
| **5** | **Oturum** (Session) | Veri (Data) | İletişimi başlatma, sürdürme, sonlandırma. |
| **4** | **Taşıma** (Transport) | Segment | Veri bütünlüğü, akış kontrolü (TCP/UDP). |
| **3** | **Ağ** (Network) | Paket (Packet) | Mantıksal adresleme (IP) ve Rotalama. |
| **2** | **Veri Bağlantısı** (Data Link) | Çerçeve (Frame) | Fiziksel adresleme (MAC) ve hata denetimi. |
| **1** | **Fiziksel** (Physical) | Bit | Elektrik, ışık veya radyo sinyalleri. |

---

## 1. Fiziksel Katman (Physical Layer)

İletişimin başladığı ve bittiği, tamamen donanımsal olan katmandır. Verinin **"0 ve 1" (Bit)** haline dönüştürülüp kablo veya hava yoluyla karşı tarafa iletildiği yerdir.

* **Görevi:** Sinyal üretimi, voltaj seviyeleri, kablo tipleri, konnektör yapıları.
* **Donanım:** Hub, Repeater, Ethernet Kabloları (CAT6, Fiber), Modem.
* **Örnek:** `Ethernet (IEEE 802.3)`, `Wi-Fi`, `Bluetooth`, `USB`.

> 💡 **Analoji:** Şehirlerarası bir mektubun taşındığı **otoban** veya tren raylarıdır. Yolun kendisi mektubun içinde ne olduğuyla ilgilenmez.

---

## 2. Veri Bağlantısı Katmanı (Data Link Layer)

Fiziksel katmandan gelen ham 0 ve 1'leri anlamlı veri bloklarına (**Frame/Çerçeve**) dönüştürür. Aynı ağ üzerindeki cihazların **MAC Adresi** (Media Access Control) ile birbirini bulmasını sağlar.

* **Görevi:** Fiziksel adresleme, hatadaki bozulmaları kontrol etme (CRC/FCS).
* **Donanım:** Switch (L2), Network Kartı (NIC).
* **Alt Katmanlar:**
    1.  **MAC:** Veriye donanım adresini ekler.
    2.  **LLC:** Protokolleri tanımlar ve akış kontrolü yapar.

> 💡 **Analoji:** Bir apartmandaki (Yerel Ağ) daire numaralarıdır. Postacı apartmanı bulur (IP), ama daire numarasını (MAC) bilmeden mektubu teslim edemez.

---

## 3. Ağ Katmanı (Network Layer)

Verinin kaynak bilgisayardan hedef bilgisayara en kısa ve en hızlı yoldan nasıl gideceğine (**Routing**) karar verir. Yerel ağdan çıkıp "İnternet" dediğimiz yapıya burada gireriz.

* **Görevi:** Mantıksal adresleme (IP Adresi), Paket yönlendirme (Routing).
* **Veri Birimi:** Paket (Packet).
* **Donanım:** Router (Yönlendirici).
* **Protokoller:** `IPv4`, `IPv6`, `ICMP` (Ping komutu burada çalışır).

> 💡 **Analoji:** Posta Dağıtım Merkezi. Mektubun üzerindeki **Şehir ve Posta Koduna** bakılarak hangi kamyona yükleneceğine karar verilir.

---

## 4. Taşıma Katmanı (Transport Layer)

Verinin uçtan uca (End-to-End) güvenli veya hızlı bir şekilde taşınmasından sorumludur. Veriyi parçalara (**Segment**) ayırır ve hedefte tekrar birleştirir.

* **Görevi:** Segmentasyon, Hata düzeltme, Akış kontrolü.
* **Adresleme:** Port Numaraları (Örn: Web için 80, Mail için 25).
* **İki Temel Protokol:**
    * **TCP (Transmission Control Protocol):** Güvenilirdir. Verinin gittiğini teyit eder (ACK). (Örn: Web siteleri, E-posta).
    * **UDP (User Datagram Protocol):** Hızlıdır ama garantisi yoktur. (Örn: Online oyunlar, Video konferans).

> 💡 **Analoji:** Kargo gönderim türü. **TCP**, "İadeli Taahhütlü" mektuptur (imza karşılığı teslim). **UDP**, posta kutusuna bırakılıp gidilen normal mektuptur.

---

## 5. Oturum Katmanı (Session Layer)

İki cihaz arasındaki iletişimin (Oturumun) kurulması, yönetilmesi ve sonlandırılmasını sağlar.

* **Görevi:** Diyalog kontrolü, Senkronizasyon.
* **Örnek:** Bir dosya indirirken internet kesilirse, bağlantı geldiğinde indirmenin "kaldığı yerden devam etmesi" (Checkpointing) bu katmanın özelliğidir.
* **Protokoller:** `NetBIOS`, `RPC`, `SQL`.

> 💡 **Analoji:** Bir telefon görüşmesi. "Alo" diyerek oturumu başlatmak, konuşmak ve "Görüşürüz" diyerek kapatmak.

---

## 6. Sunum Katmanı (Presentation Layer)

Verinin, uygulamanın anlayacağı dile çevrildiği "Tercüman" katmandır. Farklı formatların birbirine dönüştürülmesi işini yapar.

* **Görevi:**
    1.  **Formatlama:** (Örn: ASCII, EBCDIC, JPEG, GIF).
    2.  **Şifreleme/Deşifreleme:** (Encryption/Decryption).
    3.  **Sıkıştırma:** Veri boyutunu küçültme (ZIP, RAR).

> 💡 **Analoji:** İki farklı dil konuşan devlet başkanı arasındaki **tercüman**. Ayrıca gizli belgelerin çantaya konup kilitlenmesi (Şifreleme) burada yapılır.

---

## 7. Uygulama Katmanı (Application Layer)

Kullanıcının bilgisayar ağı ile doğrudan etkileşime girdiği en üst katmandır. Kullandığımız yazılımların ağ ile konuşan arayüzüdür.

* **Görevi:** Dosya transferi, E-posta, Web tarama, Veritabanı erişimi.
* **Protokoller:**
    * **HTTP/HTTPS:** Web sayfaları.
    * **FTP:** Dosya transferi.
    * **SMTP/POP3:** E-posta.
    * **DNS:** Alan adı çözümleme (google.com -> IP).
    * **SSH:** Uzaktan yönetim.

> 💡 **Analoji:** Web tarayıcınızın (Chrome, Firefox) kendisi değil, tarayıcının "Sunucuya bağlan" dediği andaki iletişim kurallarıdır.

---

## 🔄 Veri Akışı: Kapsülleme (Encapsulation)

Bir bilgisayardan diğerine veri gönderilirken (Örn: Bir E-posta atarken) süreç **Katman 7'den Katman 1'e** doğru işler. Her katman, veriye kendi "Başlığını" (Header) ekler. Buna **Encapsulation** denir.

1.  **Uygulama:** Kullanıcı "Gönder"e basar. (Veri)
2.  **Sunum:** Veri formatlanır (Örn: HTML).
3.  **Oturum:** Bağlantı ID'si eklenir.
4.  **Taşıma:** TCP başlığı ve Port numarası eklenir. (Segment)
5.  **Ağ:** IP adresleri eklenir. (Paket)
6.  **Veri Bağlantısı:** MAC adresleri eklenir. (Çerçeve)
7.  **Fiziksel:** 010101 sinyaline dönüşür ve kabloya verilir. (Bit)

Alıcı tarafta ise süreç tam tersine (**Decapsulation**) işler; her katman kendi açabildiği paketi açar ve veriyi yukarı iletir.

---

## 🆚 OSI Modeli vs TCP/IP Modeli

Günümüzde internette pratik olarak TCP/IP modeli kullanılır, ancak OSI referans modelidir.

| OSI Modeli (7 Katman) | TCP/IP Modeli (4 Katman) |
| :--- | :--- |
| 7. Uygulama | **Uygulama** (Application) |
| 6. Sunum | *(Bu 3 katman TCP/IP'de birleşiktir)* |
| 5. Oturum | |
| 4. Taşıma | **Taşıma** (Transport) |
| 3. Ağ | **İnternet** (Internet) |
| 2. Veri Bağlantısı | **Ağ Erişimi** (Network Access) |
| 1. Fiziksel | *(Fiziksel donanım katmanı)* |

---
*Bu dökümantasyon eğitim amaçlı hazırlanmıştır.*
