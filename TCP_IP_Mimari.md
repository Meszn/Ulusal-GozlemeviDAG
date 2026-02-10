# 🌐 TCP/IP Protokol Mimarisi: İnternetin Omurgası

![Protocol Suite](https://img.shields.io/badge/Protocol-TCP%2FIP-blue?style=for-the-badge&logo=internet-explorer)
![Network Engineering](https://img.shields.io/badge/Focus-Deep_Dive-red?style=for-the-badge)
![Status](https://img.shields.io/badge/Documentation-Detailed-success?style=for-the-badge)

**TCP/IP (Transmission Control Protocol / Internet Protocol)**, modern internetin üzerinde çalıştığı, cihazların birbirini nasıl bulacağını ve verinin nasıl taşınacağını belirleyen protokoller kümesidir. 1970'lerde ABD Savunma Bakanlığı (DoD) tarafından, nükleer bir savaş durumunda bile iletişimin kopmaması amacıyla tasarlanmıştır.

Bu doküman, TCP/IP modelini **paket başlıkları, el sıkışma mekanizmaları ve protokol detayları** ile ele alır.

---

## 📑 İçindekiler
1.  [TCP/IP vs OSI Karşılaştırması](#1-tcpip-ve-osi-karşılaştırması)
2.  [Katman 1: Ağ Erişimi (Network Access)](#2-katman-1-ağ-erişimi-network-access)
3.  [Katman 2: İnternet (Internet - IP)](#3-katman-2-internet-internet-layer)
4.  [Katman 3: Taşıma (Transport - TCP/UDP)](#4-katman-3-taşıma-transport-layer)
5.  [Katman 4: Uygulama (Application)](#5-katman-4-uygulama-application-layer)
6.  [TCP Mekanizması](#6-detaylı-analiz-tcp-mekanizması)
7.  [IPv4 vs IPv6](#7-ipv4-ve-ipv6-farkı)

---

## 1. TCP/IP ve OSI Karşılaştırması

TCP/IP, OSI'nin 7 katmanını daha pratik olan 4 katmanda özetler.



| OSI Modeli (7 Katman) | TCP/IP Modeli (4 Katman) | Protokoller |
| :--- | :--- | :--- |
| Uygulama, Sunum, Oturum | **4. Uygulama** | HTTP, FTP, DNS, SSH, TLS |
| Taşıma | **3. Taşıma** | TCP, UDP |
| Ağ | **2. İnternet** | IP (IPv4/IPv6), ICMP, ARP, IPsec |
| Veri Bağlantısı, Fiziksel | **1. Ağ Erişimi** | Ethernet, Wi-Fi, Fiber, MAC |

---

## 2. Katman 1: Ağ Erişimi (Network Access)

Fiziksel donanım ve verinin kabloya (veya havaya) aktarılmasıyla ilgilenir.

* **PDU (Veri Birimi):** Frame (Çerçeve)
* **Adresleme:** MAC Adresi (Fiziksel Adres) - `00:1A:2B:3C:4D:5E`
* **Örnek:** Ethernet kartının veriyi elektrik sinyaline çevirmesi.

### ARP (Address Resolution Protocol)
IP adresi bilinen bir cihazın MAC adresini öğrenmek için kullanılır.
> *Senaryo:* Bilgisayarın "192.168.1.1 (Modem)"e gitmek ister. Ancak kabloya veriyi koymak için Modemin MAC adresine ihtiyacı vardır. Ağa "Bu IP kimde?" diye bağırır (Broadcast), modem de "Benim MAC adresim bu" diye cevap döner.

---

## 3. Katman 2: İnternet (Internet Layer)

Verinin kaynak adresten hedef adrese **yönlendirilmesinden (Routing)** sorumludur.

* **PDU:** Packet (Paket)
* **Adresleme:** IP Adresi (Mantıksal Adres) - `192.168.1.5`

### IP (Internet Protocol)
Paketlerin üzerine "Gönderici IP" ve "Alıcı IP" etiketlerini yapıştırır. Güvenilir değildir (paket kaybolabilir), sadece "en iyi çabayı" (Best Effort) gösterir.



**Bir IP Paketinin Başlığında Neler Var?**
* **Source IP:** Gönderen kim?
* **Destination IP:** Kime gidiyor?
* **TTL (Time to Live):** Paket sonsuza kadar ağda dolaşmasın diye koyulan ömür sayacı. Her router geçişinde 1 azalır. 0 olunca paket imha edilir.
* **Protocol:** İçinde ne taşıyor? (TCP=6, UDP=17, ICMP=1).

### ICMP (Internet Control Message Protocol)
Ağdaki hata mesajları ve durum bilgisi için kullanılır.
* **Ping:** `Echo Request` ve `Echo Reply` mesajlarını kullanır.
* **Traceroute:** TTL değerini manipüle ederek paketin hangi routerlardan geçtiğini haritalar.

---

## 4. Katman 3: Taşıma (Transport Layer)

İki cihaz üzerindeki **uygulamalar** arasında veri akışını sağlar.

* **PDU:** Segment (TCP) / Datagram (UDP)
* **Adresleme:** Port Numarası (Hangi uygulama?) - `80, 443, 3306`

### TCP (Transmission Control Protocol)
Bağlantı tabanlıdır (Connection-oriented). Verinin **eksiksiz** ve **sıralı** gittiğini garanti eder.



**TCP Başlığı Özellikleri:**
* **Sequence Number:** Paketlerin sırasını belirler (Karışık gelse bile birleştirirken kullanılır).
* **Acknowledgement Number:** "Şu pakete kadar aldım, sıradakini gönder" teyididir.
* **Window Size:** Alıcının "Şu an tampon belleğimde X byte yer var, daha hızlı/yavaş gönder" diyerek akış kontrolü yapmasını sağlar.

### UDP (User Datagram Protocol)
Bağlantısızdır (Connectionless). "Ateşle ve Unut" mantığıyla çalışır.



* **Avantaj:** Çok hızlıdır, başlık boyutu küçüktür (8 Byte).
* **Dezavantaj:** Paket kaybolabilir, sıra karışabilir.
* **Kullanım:** DNS sorguları, VoIP, **Online Oyunlar (Unity Multiplayer)**.

---

## 5. Katman 4: Uygulama (Application Layer)

Yazılımın ağ ile konuştuğu katmandır. Veri burada anlamlı hale gelir (HTML, JSON, Resim).

* **HTTP/HTTPS:** Web tarayıcıları.
* **DNS (Domain Name System):** `google.com` -> `142.250.185.78` dönüşümünü yapar.
* **DHCP:** Ağa bağlanan cihaza otomatik IP dağıtır.
* **FTP:** Dosya transferi.
* **SMTP/IMAP:** E-posta trafiği.

---

## 6. TCP Mekanizması

### 🤝 3-Way Handshake (Bağlantı Kurma)
TCP güvenli bir boru hattı kurmak için 3 aşamalı bir el sıkışma yapar:



1.  **SYN (Synchronize):** Client -> Server ("Merhaba, seninle X portundan konuşmak istiyorum, sıra numaram 100").
2.  **SYN-ACK:** Server -> Client ("Merhaba, isteğini aldım (ACK 101). Ben de hazırım, benim sıra numaram 500 (SYN)").
3.  **ACK (Acknowledge):** Client -> Server ("Tamam, senin de hazır olduğunu duydum (ACK 501). Bağlantı kuruldu!").

### 👋 4-Way Handshake (Bağlantı Koparma)
TCP'de "Telefonu yüzüne kapatmak" yoktur, nazikçe vedalaşılır:
1.  **FIN:** Client -> "Benim işim bitti."
2.  **ACK:** Server -> "Tamam, bittiğini duydum."
3.  **FIN:** Server -> "Benim de işim bitti, kapatıyorum."
4.  **ACK:** Client -> "Tamam, kapattığını onaylıyorum."

### Akış ve Tıkanıklık Kontrolü (Flow & Congestion Control)
* **Sliding Window:** Alıcı, sunucuya "Bana tek seferde 10 paket gönder, hepsine tek tek ACK atmayayım, 10 tanesi gelince toplu ACK atarım" der. Ağ hızına göre bu pencere büyür veya küçülür.
* **Retransmission:** Gönderilen bir paketin ACK cevabı belirli bir sürede (Timeout) gelmezse, TCP o paketin kaybolduğunu varsayar ve **otomatik olarak tekrar gönderir.**

---

## 7. IPv4 ve IPv6 Farkı

IP adreslerinin tükenmesi nedeniyle yeni nesil adreslemeye geçilmektedir.

| Özellik | IPv4 | IPv6 |
| :--- | :--- | :--- |
| **Boyut** | 32-bit | 128-bit |
| **Format** | `192.168.1.1` (Decimal) | `2001:0db8:85a3::8a2e` (Hexadecimal) |
| **Adres Sayısı** | ~4.3 Milyar | Neredeyse sınırsız |
| **Konfigürasyon** | Genelde DHCP gerekir | SLAAC (Otomatik yapılandırma) destekler |
| **Header** | Değişken boyutlu, karmaşık | Sabit boyutlu, daha hızlı işlenir |


---
*Bu dokümantasyon, Türkiye Ulusal Gözlemevi staj sürecimde detaylı teknik referans olarak hazırlanmıştır.*
