# 🌐 HTTP ve HTTPS: Web İletişiminin Temelleri

&#x20;&#x20;

**HTTP (HyperText Transfer Protocol)**, istemci (tarayıcı, mobil uygulama, script) ile sunucu arasında veri alışverişini sağlayan ve **OSI Modeli’nin 7. katmanında (Uygulama Katmanı)** çalışan temel web protokolüdür.

Bir web sitesine girdiğinizde, bir mobil uygulamadan **API isteği** attığınızda veya bir backend servisiyle haberleştiğinizde, arka planda çalışan ana mekanizma HTTP’dir.

---

## 📑 İçindekiler

1. [HTTP Nasıl Çalışır? (İstek–Cevap Döngüsü)](#1-http-nasıl-çalışır-istek-cevap-döngüsü)
2. [HTTP Metotları (CRUD İşlemleri)](#2-http-metotları-verbs)
3. [Durum Kodları (Status Codes)](#3-durum-kodları-status-codes)
4. [HTTPS: Güvenli İletişim](#4-https-neden-sonundaki-s-önemli)
5. [SSL/TLS Handshake (El Sıkışması)](#5-ssltls-handshake-nasıl-çalışır)
6. [HTTP Versiyonları (1.1 vs 2 vs 3)](#6-http-versiyonları-evrim)

---

## 1. HTTP Nasıl Çalışır? (İstek–Cevap Döngüsü)

HTTP **stateless (durumsuz)** bir protokoldür. Yani sunucu, önceki istekleri kendi başına hatırlamaz. Her istek **bağımsızdır**.

> Durum bilgisinin korunması gerekiyorsa **Cookie, Session, JWT** gibi mekanizmalar kullanılır.

İletişim her zaman **Client → Server** şeklinde başlar.

### 📤 Bir HTTP İsteğinin Anatomisi (Request)

```http
GET /index.html HTTP/1.1
Host: www.google.com
User-Agent: Mozilla/5.0 (Windows NT 10.0)
Accept: text/html
Authorization: Bearer eyJhbGciOiJIUzI1...
```

**Bileşenler:**

- **Method:** `GET` → Ne yapmak istiyorsun?
- **Path:** `/index.html` → Hangi kaynağa erişiyorsun?
- **Headers:** Kimlik bilgileri, içerik tipi, tarayıcı bilgisi
- **Body (Opsiyonel):** POST/PUT/PATCH isteklerinde gönderilen veri

---

### 📥 Bir HTTP Cevabının Anatomisi (Response)

```http
HTTP/1.1 200 OK
Date: Mon, 27 Jul 2025 12:28:53 GMT
Content-Type: text/html; charset=UTF-8
Server: gws

<html>
  <body>Merhaba Dünya!</body>
</html>
```

**Bileşenler:**

- **Status Code:** `200 OK` → İşlem sonucu
- **Headers:** Dönen içeriğin tipi, cache bilgileri
- **Body:** HTML, JSON veya başka bir veri

---

## 2. HTTP Metotları (Verbs)

RESTful API tasarımının temel taşlarıdır.

| Metot  | Görevi                     | CRUD   | Güvenli mi? |
| ------ | -------------------------- | ------ | ----------- |
| GET    | Veri almak                 | Read   | ✅ Evet      |
| POST   | Yeni veri oluşturmak       | Create | ❌ Hayır     |
| PUT    | Veriyi tamamen güncellemek | Update | ❌ Hayır     |
| PATCH  | Veriyi kısmen güncellemek  | Update | ❌ Hayır     |
| DELETE | Veriyi silmek              | Delete | ❌ Hayır     |

⚠️ **Not:** Tarayıcı adres çubuğundan yapılan tüm istekler **GET**’tir.

---

## 3. Durum Kodları (Status Codes)

Sunucunun istemciye verdiği **geri bildirimdir**.

### 1xx – Bilgilendirme

- **100 Continue:** İstek devam ediyor

### 2xx – Başarı ✅

- **200 OK:** İşlem başarılı
- **201 Created:** Yeni kaynak oluşturuldu

### 3xx – Yönlendirme ➡️

- **301 Moved Permanently:** Kalıcı yönlendirme (SEO için kritik)
- **302 Found:** Geçici yönlendirme

### 4xx – İstemci Hatası ❌

- **400 Bad Request:** Hatalı istek
- **401 Unauthorized:** Kimlik doğrulama gerekli
- **403 Forbidden:** Yetki yok
- **404 Not Found:** Kaynak bulunamadı

### 5xx – Sunucu Hatası 🔥

- **500 Internal Server Error:** Sunucu tarafında bug
- **502 Bad Gateway:** Arka servis cevap vermiyor
- **503 Service Unavailable:** Sunucu yoğun veya bakımda

---

## 4. HTTPS: Neden Sonundaki "S" Önemli?

**HTTPS**, HTTP protokolünün **SSL/TLS** ile şifrelenmiş hâlidir.

### HTTP vs HTTPS

- **HTTP:** Veri açık metin gider (dinlenebilir)
- **HTTPS:** Veri şifreli gider (okunamaz)

### HTTPS Ne Sağlar?

- 🔐 **Encryption:** Veri gizliliği
- 🧾 **Integrity:** Veri yolda değiştirilemez
- 🪪 **Authentication:** Sunucunun gerçekliği doğrulanır

---

## 5. SSL/TLS Handshake (Nasıl Çalışır?)

Tarayıcıda 🔒 simgesinin görünmesi için şu adımlar gerçekleşir:

1. **Client Hello** → Tarayıcı desteklediği algoritmaları gönderir
2. **Server Hello** → Sunucu sertifikasını yollar
3. **Sertifika Doğrulama** → CA kontrolü yapılır
4. **Anahtar Değişimi** → Session Key oluşturulur
5. **Şifreli İletişim** başlar

> Sunucunun **Private Key’i** asla paylaşılmaz.

---

## 6. HTTP Versiyonları

| Versiyon | Yıl   | Özellik                                 | Dezavantaj            |
| -------- | ----- | --------------------------------------- | --------------------- |
| HTTP/1.1 | 1997  | Metin tabanlı, her kaynak için bağlantı | Yavaş yükleme         |
| HTTP/2   | 2015  | Multiplexing, binary format             | TCP kayıpları etkiler |
| HTTP/3   | 2020+ | QUIC (UDP tabanlı)                      | Henüz yaygın değil    |

---


📌 *Bu doküman, Türkiye Ulusal Gözlemevi staj sürecimde hazırlanmıştır ve web teknolojileri, güvenli iletişim standartları için referans niteliğindedir.*

