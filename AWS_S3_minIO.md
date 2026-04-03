# ☁️ AWS S3 & MinIO: Kapsamlı Teknik Rehber


[![AWS S3](https://img.shields.io/badge/AWS-S3-FF9900?style=flat-square&logo=amazon-aws&logoColor=white)](https://aws.amazon.com/s3/)
[![MinIO](https://img.shields.io/badge/MinIO-Object_Storage-C72E49?style=flat-square&logo=minio&logoColor=white)](https://min.io/)
[![Python](https://img.shields.io/badge/Python-3.9+-3776AB?style=flat-square&logo=python&logoColor=white)](https://python.org)
[![Docker](https://img.shields.io/badge/Docker-Ready-2496ED?style=flat-square&logo=docker&logoColor=white)](https://docker.com)
[![License](https://img.shields.io/badge/License-MIT-green?style=flat-square)](LICENSE)

---

## 📋 İçindekiler

- [Object Storage Nedir?](#-object-storage-nedir)
- [AWS S3](#-aws-s3)
  - [Temel Kavramlar](#temel-kavramlar)
  - [Storage Sınıfları](#storage-sınıfları)
  - [Güvenlik](#güvenlik)
  - [Versioning & Lifecycle](#versioning--lifecycle-management)
  - [Pre-signed URL](#pre-signed-url)
- [MinIO](#-minio)
  - [MinIO Nedir?](#minio-nedir)
  - [Dağıtık Mimari](#dağıtık-distributed-mimari)
  - [Güvenlik & Erişim](#güvenlik--erişim-yönetimi)
- [S3 vs MinIO Karşılaştırması](#-s3-vs-minio-karşılaştırması)
- [Kod Örnekleri (Python)](#-kod-örnekleri-python)
- [Pratik Uygulama](#-pratik-uygulama)
- [DevOps & Setup](#-devops--setup)
- [Best Practices](#-best-practices)
- [Mimari Diyagramlar](#-mimari-diyagramlar)
- [Quick Start](#-quick-start)
- [Cheat Sheet](#-cheat-sheet)

---

## 📦 Object Storage Nedir?

**Object Storage** (Nesne Depolama), verileri dosya sistemi (file system) ya da blok depolama (block storage) hiyerarşisi yerine düz bir alan (flat namespace) içinde **object** olarak saklayan bir depolama paradigmasıdır.

### Geleneksel Depolama vs Object Storage

| Özellik | File System (NFS/EFS) | Block Storage (EBS) | Object Storage (S3/MinIO) |
|---|---|---|---|
| Veri birimi | Dosya + Dizin ağacı | Blok (sabit boyut) | Object (key + value + metadata) |
| Hiyerarşi | Ağaç yapısı | Yok | Düz namespace (simüle edilmiş prefix) |
| Ölçeklenebilirlik | Orta | Düşük | Sınırsız (petabayt+) |
| Erişim protokolü | POSIX, SMB | iSCSI, FC | HTTP/HTTPS (REST API) |
| Metadata | Sınırlı | Yok | Zengin, özelleştirilebilir |
| Maliyet | Orta-yüksek | Yüksek | Düşük |
| Kullanım senaryosu | OS, uygulama dosyaları | Veritabanı | Medya, backup, log, statik içerik |

### Object Storage Nasıl Çalışır?

Her **object** üç temel bileşenden oluşur:

```
┌─────────────────────────────────────────────────┐
│                    OBJECT                        │
│  ┌──────────────┐  ┌──────────────────────────┐ │
│  │   DATA       │  │       METADATA           │ │
│  │  (Binary)    │  │  content-type: image/png │ │
│  │              │  │  last-modified: ...       │ │
│  └──────────────┘  │  custom-tag: production  │ │
│                    └──────────────────────────┘ │
│  KEY: "uploads/2024/profile/user-42.png"        │
└─────────────────────────────────────────────────┘
```

- **Key**: Objenin benzersiz tanımlayıcısı (path benzeri, ancak gerçek dizin yok)
- **Data**: Ham binary içerik
- **Metadata**: System ve kullanıcı tanımlı anahtar-değer çiftleri

### S3-Compatible Sistemlerin Mantığı

Amazon S3, object storage için **de facto standart API** haline gelmiştir. Bir sistem "S3-compatible" ise şu anlama gelir:

- AWS S3 REST API'sini (endpoint, HTTP metot, header, imza) **birebir destekler**
- `boto3`, `aws-cli`, `s3cmd` gibi araçlar **değiştirilmeden** çalışır
- Sadece endpoint URL ve credential değiştirilir; uygulama kodu aynı kalır

Bu mimari sayesinde MinIO, Ceph, Wasabi, Backblaze B2 gibi onlarca farklı sistem, aynı istemci koduyla kullanılabilir.

---

## ☁️ AWS S3

Amazon Simple Storage Service (S3), 2006'dan beri AWS'nin temel object storage hizmetidir. Global ölçekte milyarlarca objeyi barındırır.

### Temel Kavramlar

#### Bucket

**Bucket**, S3'teki en üst düzey container'dır. Tüm objeler bir bucket içinde saklanır.

- Her bucket adı **global olarak benzersiz** olmalıdır (tüm AWS hesapları arasında)
- Bucket adları: küçük harf, rakam, tire — 3-63 karakter
- Bir bucket belirli bir **AWS Region**'a bağlıdır ve bölgeyi sonradan değiştiremezsiniz
- Hesap başına varsayılan limit: 100 bucket (artırılabilir)

```
s3://my-company-assets/
    ├── images/
    │   ├── logo.png          → Key: "images/logo.png"
    │   └── banner.jpg        → Key: "images/banner.jpg"
    ├── videos/
    │   └── intro.mp4         → Key: "videos/intro.mp4"
    └── documents/
        └── report-2024.pdf   → Key: "documents/report-2024.pdf"
```

> **Not:** S3'te gerçek bir dizin yapısı yoktur. `/` karakteri key'in bir parçasıdır. AWS Console ve SDK'lar bunu sizi kolaylık için dizin gibi gösterir.

#### Object

Her object şu bileşenlerden oluşur:

| Bileşen | Açıklama | Limit |
|---|---|---|
| Key | Benzersiz tanımlayıcı (path) | 1024 byte |
| Value (Body) | Binary içerik | 5 TB |
| Version ID | Versioning aktifken oluşur | - |
| Metadata | System + user-defined | 2 KB |
| Tags | Anahtar-değer etiketler | 10 tag |
| ACL | Erişim kontrol listesi | - |

#### Key Tasarımı ve Performans

S3 anahtarları prefix'e göre partition eder. **Aynı prefix ile başlayan çok sayıda obje** tek bir partition'a düşerek **hot partition** sorununa yol açabilir.

```python
# ❌ Kötü — Tüm objeler aynı prefix'i paylaşır
logs/2024-01-01-00001.log
logs/2024-01-01-00002.log
logs/2024-01-01-00003.log

# ✅ İyi — Hash prefix ile dağıtım sağlanır
a3f2/logs/2024-01-01-00001.log
b71c/logs/2024-01-01-00002.log
9de4/logs/2024-01-01-00003.log
```

> **2018 sonrası güncelleme:** AWS, S3'ü prefix başına **3.500 PUT/s** ve **5.500 GET/s** destekleyecek şekilde iyileştirdi. Randomized prefix artık zorunlu değil, ancak büyük ölçekte hâlâ iyi pratik olarak öneriliyor.

#### Region

S3 bucket'ları belirli bir AWS Region'da yaşar:

- **Veri residency**: Uyumluluk gereksinimleri için verinin hangi coğrafyada olduğu önemlidir
- **Latency**: İstemciye yakın region seçilmeli
- **Cross-region replication (CRR)**: Farklı region'lara otomatik çoğaltma yapılabilir
- **Transfer ücreti**: Aynı region içi transfer ücretsiz; cross-region ücretlidir

```
Bölgeler: us-east-1, eu-west-1, ap-southeast-1 ...
Endpoint: https://s3.{region}.amazonaws.com/{bucket}/{key}
Örnek:    https://s3.eu-central-1.amazonaws.com/my-bucket/image.png
```

#### IAM (Identity and Access Management)

AWS'de S3 erişimi **IAM** ile kontrol edilir. Üç katman mevcuttur:

```
┌─────────────────────────────────────────────────────┐
│                  Erişim Kontrol Katmanları           │
│                                                     │
│  1. IAM Policy     → Kullanıcı/Role bazlı          │
│  2. Bucket Policy  → Resource (bucket) bazlı       │
│  3. ACL            → Object bazlı (legacy)          │
└─────────────────────────────────────────────────────┘
```

---

### Storage Sınıfları

S3, farklı erişim paternlerine göre optimize edilmiş storage sınıfları sunar:

| Sınıf | Kullanım Senaryosu | Erişim | Dayanıklılık | Aylık Maliyet (GB) |
|---|---|---|---|---|
| **S3 Standard** | Sık erişilen veriler | Anlık | 99.999999999% | ~$0.023 |
| **S3 Intelligent-Tiering** | Değişken erişim paterni | Anlık | 99.999999999% | ~$0.023 + izleme |
| **S3 Standard-IA** | Seyrek erişilen, hızlı erişim gerekli | Anlık | 99.999999999% | ~$0.0125 |
| **S3 One Zone-IA** | Seyrek, tek AZ, düşük öncelikli | Anlık | 99.999999999% (tek AZ) | ~$0.01 |
| **S3 Glacier Instant** | Arşiv, ms erişim | Milisaniye | 99.999999999% | ~$0.004 |
| **S3 Glacier Flexible** | Uzun vadeli arşiv | 1-12 saat | 99.999999999% | ~$0.0036 |
| **S3 Glacier Deep Archive** | 7-10 yıl arşiv | 12-48 saat | 99.999999999% | ~$0.00099 |

**Akıllı Tier Geçiş Örneği:**

```
Yeni object → Standard (30 gün erişilmezse) → Standard-IA
                                             → (90 gün) → Glacier
                                             → (365 gün) → Deep Archive
```

---

### Güvenlik

#### IAM Policy

Kullanıcı veya role'e atanan JSON policy'ler:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ReadOnlyAccess",
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::my-company-assets",
        "arn:aws:s3:::my-company-assets/*"
      ]
    },
    {
      "Sid": "DenyDeleteInProduction",
      "Effect": "Deny",
      "Action": "s3:DeleteObject",
      "Resource": "arn:aws:s3:::my-company-assets/production/*"
    }
  ]
}
```

#### Bucket Policy

Bucket'a doğrudan atanan resource-based policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadForStaticWebsite",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-website-bucket/*"
    },
    {
      "Sid": "AllowSpecificVPC",
      "Effect": "Deny",
      "Principal": "*",
      "Action": "s3:*",
      "Resource": [
        "arn:aws:s3:::my-private-bucket",
        "arn:aws:s3:::my-private-bucket/*"
      ],
      "Condition": {
        "StringNotEquals": {
          "aws:SourceVpc": "vpc-0123456789abcdef0"
        }
      }
    }
  ]
}
```

#### Şifreleme

| Tür | Açıklama | Anahtar Yönetimi |
|---|---|---|
| **SSE-S3** | S3 yönetimli | AWS |
| **SSE-KMS** | AWS KMS anahtarı | KMS (denetim logu) |
| **SSE-C** | Müşteri sağlar | Müşteri |
| **Client-side** | Uygulama şifreler | Müşteri |

```python
# SSE-KMS ile upload
s3.put_object(
    Bucket='my-bucket',
    Key='sensitive-data.json',
    Body=json_content,
    ServerSideEncryption='aws:kms',
    SSEKMSKeyId='arn:aws:kms:eu-central-1:123456789:key/abc-def'
)
```

#### Block Public Access

Tüm bucket'lar için önerilen yapılandırma:

```python
s3.put_public_access_block(
    Bucket='my-bucket',
    PublicAccessBlockConfiguration={
        'BlockPublicAcls': True,
        'IgnorePublicAcls': True,
        'BlockPublicPolicy': True,
        'RestrictPublicBuckets': True
    }
)
```

---

### Versioning & Lifecycle Management

#### Versioning

Versioning aktifken, her PUT işlemi yeni bir **version ID** oluşturur. Silinen objeler silinmez, **delete marker** eklenir.

```python
# Versioning etkinleştirme
s3.put_bucket_versioning(
    Bucket='my-bucket',
    VersioningConfiguration={'Status': 'Enabled'}
)

# Tüm versiyonları listeleme
response = s3.list_object_versions(Bucket='my-bucket', Prefix='uploads/')
for version in response.get('Versions', []):
    print(f"Key: {version['Key']}, VersionId: {version['VersionId']}, Latest: {version['IsLatest']}")

# Belirli bir versiyonu indirme
s3.download_file(
    Bucket='my-bucket',
    Key='config.json',
    Filename='config_old.json',
    ExtraArgs={'VersionId': 'Tg8r4nPZvK9cFXxxx'}
)
```

#### Lifecycle Policies

```json
{
  "Rules": [
    {
      "ID": "TransitionToGlacierAndExpire",
      "Status": "Enabled",
      "Filter": {"Prefix": "logs/"},
      "Transitions": [
        {
          "Days": 30,
          "StorageClass": "STANDARD_IA"
        },
        {
          "Days": 90,
          "StorageClass": "GLACIER"
        }
      ],
      "Expiration": {
        "Days": 365
      },
      "NoncurrentVersionExpiration": {
        "NoncurrentDays": 30
      }
    }
  ]
}
```

---

### Pre-signed URL

Pre-signed URL'ler, AWS credential'ı olmayan kullanıcılara **geçici ve güvenli** erişim sağlar.

```python
import boto3
from datetime import timedelta

s3 = boto3.client('s3', region_name='eu-central-1')

# Download için pre-signed URL (1 saat geçerli)
download_url = s3.generate_presigned_url(
    ClientMethod='get_object',
    Params={
        'Bucket': 'my-bucket',
        'Key': 'private/report-2024.pdf'
    },
    ExpiresIn=3600  # saniye
)

# Upload için pre-signed URL (doğrudan tarayıcıdan yükleme)
upload_url = s3.generate_presigned_url(
    ClientMethod='put_object',
    Params={
        'Bucket': 'my-bucket',
        'Key': 'uploads/user-42/avatar.png',
        'ContentType': 'image/png'
    },
    ExpiresIn=900  # 15 dakika
)

print(f"İndir: {download_url}")
print(f"Yükle: {upload_url}")
```

**Multipart upload için pre-signed URL** (büyük dosyalar):

```python
# 1. Multipart upload başlat
response = s3.create_multipart_upload(Bucket='my-bucket', Key='large-video.mp4')
upload_id = response['UploadId']

# 2. Her part için ayrı pre-signed URL üret
presigned_parts = []
for part_number in range(1, num_parts + 1):
    url = s3.generate_presigned_url(
        'upload_part',
        Params={
            'Bucket': 'my-bucket',
            'Key': 'large-video.mp4',
            'UploadId': upload_id,
            'PartNumber': part_number
        },
        ExpiresIn=7200
    )
    presigned_parts.append({'PartNumber': part_number, 'Url': url})

# 3. İstemci bu URL'lere PUT yapar, ETag'leri toplar
# 4. Complete multipart upload
```

---

## 🐳 MinIO

### MinIO Nedir?

**MinIO**, Apache License 2.0 altında yayımlanan, yüksek performanslı, S3-uyumlu bir open-source object storage sistemidir. Go dilinde yazılmıştır.

**Temel özellikleri:**

- **S3-compatible**: AWS S3 API'sini tam destekler — endpoint değiştirerek `boto3`, `aws-cli`, `s3cmd` kullanılır
- **Yüksek performans**: NVMe SSD üzerinde **325 GB/s okuma**, **165 GB/s yazma** (benchmark)
- **Küçük footprint**: Tek binary, ~100 MB RAM ile çalışır
- **Kubernetes-native**: MinIO Operator ile K8s üzerinde çalışır
- **AGPL-3.0 / Commercial**: Açık kaynak veya enterprise lisans

**MinIO neden kullanılır?**

- On-premise veya özel cloud ortamlarında S3 uyumlu depolama
- Geliştiriciler için yerel S3 emülatörü (test/dev ortamı)
- Veri egemenliği (data sovereignty) gereksinimleri
- AWS maliyetlerini azaltmak için hybrid cloud stratejisi
- Edge computing ve IoT senaryoları

---

### S3 ile Uyumluluğu Nasıl Çalışır?

MinIO, AWS S3 REST API spesifikasyonunu implemente eder. İstemci perspektifinden fark yoktur:

```
AWS S3 İstemcisi          MinIO Sunucusu
─────────────────         ──────────────────────
boto3.client('s3')   →    http://localhost:9000
PUT /bucket/key      →    Aynı HTTP metodlar
SigV4 Auth           →    Aynı imzalama şeması
Response format      →    Aynı XML/JSON yanıtlar
```

```python
# AWS S3
s3 = boto3.client('s3',
    region_name='eu-central-1',
    aws_access_key_id='AKIA...',
    aws_secret_access_key='...'
)

# MinIO — sadece endpoint_url eklendi!
s3 = boto3.client('s3',
    endpoint_url='http://localhost:9000',
    aws_access_key_id='minioadmin',
    aws_secret_access_key='minioadmin',
    region_name='us-east-1'
)

# Geri kalan kod AYNI
s3.upload_file('file.txt', 'my-bucket', 'file.txt')
```

---

### Dağıtık (Distributed) Mimari

MinIO, veri güvenliği için **Erasure Coding** kullanır. RAID benzeri ama daha gelişmiş bir yöntemdir.

#### Erasure Coding

```
N adet drive → K data shards + M parity shards
N = K + M

Örnek: 16 drive = 8 data + 8 parity
→ 8 drive arızalansa bile veri kaybolmaz!
```

#### Deployment Modları

**1. Single Node Single Drive (SNSD) — Dev/Test**
```bash
minio server /data
```

**2. Single Node Multi Drive (SNMD) — Küçük prod**
```bash
minio server /data{1...4}  # 4 dizin, erasure coding aktif
```

**3. Multi Node Multi Drive (MNMD) — Distributed prod**
```bash
# Her node'da çalıştırılır:
minio server http://minio{1...4}.example.com/data{1...4}
```

```
┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐
│   Node 1    │  │   Node 2    │  │   Node 3    │  │   Node 4    │
│  /data{1-4} │  │  /data{1-4} │  │  /data{1-4} │  │  /data{1-4} │
│  16 drives  │  │  16 drives  │  │  16 drives  │  │  16 drives  │
└─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘
        ↑_________________↑_________________↑_________________↑
                    Erasure Set (64 drives total)
                    32 data + 32 parity = tolerate 32 drive loss
```

#### MinIO Load Balancing

```nginx
# nginx.conf — MinIO cluster için load balancer
upstream minio_cluster {
    least_conn;
    server minio1.internal:9000;
    server minio2.internal:9000;
    server minio3.internal:9000;
    server minio4.internal:9000;
}

server {
    listen 9000;
    location / {
        proxy_pass http://minio_cluster;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_buffering off;
        client_max_body_size 0;
    }
}
```

---

### Güvenlik & Erişim Yönetimi

#### Policy Tabanlı Erişim

MinIO, IAM benzeri policy sistemi kullanır:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:GetObject", "s3:PutObject"],
      "Resource": ["arn:aws:s3:::uploads/*"]
    },
    {
      "Effect": "Allow",
      "Action": ["s3:ListBucket"],
      "Resource": ["arn:aws:s3:::uploads"]
    }
  ]
}
```

```bash
# MinIO client ile policy uygula
mc alias set local http://localhost:9000 minioadmin minioadmin
mc admin policy create local upload-only upload-policy.json
mc admin user add local appuser strongpassword123
mc admin policy attach local upload-only --user appuser
```

#### TLS Yapılandırması

```bash
# MinIO için self-signed sertifika
mkdir -p ~/.minio/certs
openssl req -newkey rsa:4096 -x509 -sha256 -days 3650 \
    -nodes -out ~/.minio/certs/public.crt \
    -keyout ~/.minio/certs/private.key \
    -subj "/CN=minio.example.com"

# MinIO TLS ile başlat
MINIO_VOLUMES="/data" \
MINIO_ROOT_USER="admin" \
MINIO_ROOT_PASSWORD="strongpassword" \
minio server --certs-dir ~/.minio/certs
```

#### OIDC / SSO Entegrasyonu

```bash
# Keycloak ile OIDC entegrasyonu
mc admin config set local identity_openid \
    config_url="https://keycloak.example.com/auth/realms/myrealm/.well-known/openid-configuration" \
    client_id="minio-client" \
    client_secret="my-secret" \
    scopes="openid,profile,email" \
    redirect_uri="https://minio.example.com/oauth_callback"
```

---

## ⚖️ S3 vs MinIO Karşılaştırması

### Genel Karşılaştırma

| Özellik | AWS S3 | MinIO |
|---|---|---|
| **Tür** | Managed cloud service | Self-hosted / managed |
| **API Uyumluluğu** | Native | S3-compatible |
| **Kurulum** | Yok (hazır) | Gerekli |
| **Ölçeklenebilirlik** | Sınırsız (AWS yönetir) | Donanım ile sınırlı |
| **Performans** | Bölge/tier'a bağlı | NVMe'de son derece yüksek |
| **Maliyet modeli** | Kullandıkça öde | Donanım + operasyon |
| **Veri egemenliği** | AWS datacenter | Kendi ortamınız |
| **SLA** | %99.99 | Kendiniz sağlarsınız |
| **Global dağıtım** | 30+ region | Manuel setup |
| **Lisans** | Proprietary | AGPL-3.0 / Commercial |
| **Destek** | AWS Support | MinIO Inc. / Topluluk |

### Maliyet Analizi

**AWS S3 (aylık 10 TB standart veri + 1 M istek):**
```
Storage     : 10,240 GB × $0.023 = $235.52
GET istekleri: 1,000,000 × $0.0004/1000 = $0.40
PUT istekleri: 100,000 × $0.005/1000 = $0.50
Egress (1 TB): 1,024 GB × $0.09 = $92.16
─────────────────────────────────────────
Toplam ≈ $328/ay
```

**MinIO (self-hosted, 10 TB):**
```
Donanım      : ~$500-2000 (one-time, 3-5 yıl kullanım)
Elektrik     : ~$20-50/ay
Yönetim      : DevOps süresi (şirkete göre değişir)
─────────────────────────────────────────
Büyük ölçekte maliyet avantajı belirginleşir
```

### Hangi Senaryoda Hangisi?

| Senaryo | Tercih | Gerekçe |
|---|---|---|
| Startup, hızlı başlangıç | **AWS S3** | Sıfır operasyon yükü |
| Global CDN ile statik içerik | **AWS S3** | CloudFront entegrasyonu |
| Veri egemenliği zorunlu | **MinIO** | Kendi datacenter'ınızda |
| On-premise Kubernetes | **MinIO** | K8s-native operator |
| Dev/test S3 emülasyonu | **MinIO** | Ücretsiz, yerel |
| 100 TB+ kendi donanımınız | **MinIO** | Maliyet avantajı |
| Yüksek throughput media | **MinIO** | NVMe performansı |
| Multi-region replikasyon | **AWS S3** | Native destek |
| ML/AI veri gölü | **Her ikisi** | Hybrid yaklaşım |
| Compliance (HIPAA, GDPR) | **MinIO** | Veri konumu kontrolü |

---

## 🐍 Kod Örnekleri (Python)

### Kurulum

```bash
pip install boto3 minio
```

### boto3 ile AWS S3

```python
import boto3
from botocore.exceptions import ClientError
import os

# İstemci oluştur
s3 = boto3.client(
    's3',
    region_name='eu-central-1',
    aws_access_key_id=os.environ['AWS_ACCESS_KEY_ID'],
    aws_secret_access_key=os.environ['AWS_SECRET_ACCESS_KEY']
)

# ─── Bucket İşlemleri ───────────────────────────────────────

def create_bucket(bucket_name: str, region: str = 'eu-central-1'):
    """Bucket oluştur."""
    try:
        if region == 'us-east-1':
            s3.create_bucket(Bucket=bucket_name)
        else:
            s3.create_bucket(
                Bucket=bucket_name,
                CreateBucketConfiguration={'LocationConstraint': region}
            )
        print(f"✅ Bucket oluşturuldu: {bucket_name}")
    except ClientError as e:
        if e.response['Error']['Code'] == 'BucketAlreadyOwnedByYou':
            print(f"ℹ️  Bucket zaten mevcut: {bucket_name}")
        else:
            raise

def list_buckets():
    """Tüm bucket'ları listele."""
    response = s3.list_buckets()
    for bucket in response['Buckets']:
        print(f"  📦 {bucket['Name']} — {bucket['CreationDate']}")

# ─── Upload İşlemleri ───────────────────────────────────────

def upload_file(local_path: str, bucket: str, key: str, extra_args: dict = None):
    """Dosya yükle."""
    try:
        s3.upload_file(
            Filename=local_path,
            Bucket=bucket,
            Key=key,
            ExtraArgs=extra_args or {}
        )
        print(f"✅ Yüklendi: {local_path} → s3://{bucket}/{key}")
    except ClientError as e:
        print(f"❌ Hata: {e}")
        raise

def upload_with_progress(local_path: str, bucket: str, key: str):
    """İlerleme çubuğu ile yükleme."""
    import os
    file_size = os.path.getsize(local_path)
    uploaded = [0]

    def progress_callback(bytes_amount):
        uploaded[0] += bytes_amount
        percent = (uploaded[0] / file_size) * 100
        print(f"\r  İlerleme: {percent:.1f}%", end='', flush=True)

    s3.upload_file(
        Filename=local_path,
        Bucket=bucket,
        Key=key,
        Callback=progress_callback
    )
    print(f"\n✅ Tamamlandı!")

# ─── Download İşlemleri ─────────────────────────────────────

def download_file(bucket: str, key: str, local_path: str):
    """Dosya indir."""
    try:
        s3.download_file(Bucket=bucket, Key=key, Filename=local_path)
        print(f"✅ İndirildi: s3://{bucket}/{key} → {local_path}")
    except ClientError as e:
        if e.response['Error']['Code'] == '404':
            print(f"❌ Dosya bulunamadı: {key}")
        else:
            raise

# ─── Listeleme ──────────────────────────────────────────────

def list_objects(bucket: str, prefix: str = ''):
    """Objeleri listele (paginator ile büyük listeler)."""
    paginator = s3.get_paginator('list_objects_v2')
    pages = paginator.paginate(Bucket=bucket, Prefix=prefix)

    objects = []
    for page in pages:
        for obj in page.get('Contents', []):
            objects.append({
                'key': obj['Key'],
                'size': obj['Size'],
                'modified': obj['LastModified']
            })
    return objects

# ─── Silme ──────────────────────────────────────────────────

def delete_objects_by_prefix(bucket: str, prefix: str):
    """Prefix ile toplu silme."""
    objects = list_objects(bucket, prefix)
    if not objects:
        print("Silinecek obje yok.")
        return

    delete_list = [{'Key': obj['key']} for obj in objects]
    # Batch delete — en fazla 1000 obje
    for i in range(0, len(delete_list), 1000):
        batch = delete_list[i:i+1000]
        response = s3.delete_objects(
            Bucket=bucket,
            Delete={'Objects': batch}
        )
        print(f"✅ {len(batch)} obje silindi.")

# ─── Pre-signed URL ─────────────────────────────────────────

def generate_download_url(bucket: str, key: str, expires_in: int = 3600) -> str:
    """İndirme için geçici URL."""
    return s3.generate_presigned_url(
        'get_object',
        Params={'Bucket': bucket, 'Key': key},
        ExpiresIn=expires_in
    )

def generate_upload_url(bucket: str, key: str, content_type: str, expires_in: int = 900) -> str:
    """Yükleme için geçici URL."""
    return s3.generate_presigned_url(
        'put_object',
        Params={'Bucket': bucket, 'Key': key, 'ContentType': content_type},
        ExpiresIn=expires_in
    )
```

### MinIO Python SDK ile Aynı İşlemler

```python
from minio import Minio
from minio.error import S3Error
import os
from datetime import timedelta

# İstemci oluştur
client = Minio(
    endpoint='localhost:9000',
    access_key=os.environ['MINIO_ACCESS_KEY'],
    secret_key=os.environ['MINIO_SECRET_KEY'],
    secure=False  # HTTPS için True
)

def create_bucket_minio(bucket_name: str):
    """MinIO'da bucket oluştur."""
    if not client.bucket_exists(bucket_name):
        client.make_bucket(bucket_name)
        print(f"✅ Bucket oluşturuldu: {bucket_name}")
    else:
        print(f"ℹ️  Bucket mevcut: {bucket_name}")

def upload_file_minio(local_path: str, bucket: str, key: str):
    """Dosya yükle."""
    client.fput_object(bucket, key, local_path)
    print(f"✅ Yüklendi: {local_path} → minio://{bucket}/{key}")

def download_file_minio(bucket: str, key: str, local_path: str):
    """Dosya indir."""
    client.fget_object(bucket, key, local_path)
    print(f"✅ İndirildi: {key} → {local_path}")

def list_objects_minio(bucket: str, prefix: str = ''):
    """Objeleri listele."""
    objects = client.list_objects(bucket, prefix=prefix, recursive=True)
    result = []
    for obj in objects:
        result.append({
            'key': obj.object_name,
            'size': obj.size,
            'modified': obj.last_modified
        })
    return result

def generate_presigned_url_minio(bucket: str, key: str, expires: int = 3600) -> str:
    """Pre-signed URL üret."""
    return client.presigned_get_object(
        bucket, key,
        expires=timedelta(seconds=expires)
    )
```

### Ortak Abstraction Katmanı

Aynı kodla hem S3 hem MinIO kullanın:

```python
from abc import ABC, abstractmethod
from typing import Optional, List, Dict
import boto3
from minio import Minio
import os

class ObjectStorageBase(ABC):
    """Object storage için soyut temel sınıf."""

    @abstractmethod
    def create_bucket(self, name: str) -> None: ...

    @abstractmethod
    def upload(self, bucket: str, key: str, local_path: str) -> None: ...

    @abstractmethod
    def download(self, bucket: str, key: str, local_path: str) -> None: ...

    @abstractmethod
    def list_objects(self, bucket: str, prefix: str = '') -> List[Dict]: ...

    @abstractmethod
    def delete(self, bucket: str, key: str) -> None: ...

    @abstractmethod
    def presigned_url(self, bucket: str, key: str, expires_in: int = 3600) -> str: ...


class S3Storage(ObjectStorageBase):
    """AWS S3 implementasyonu."""

    def __init__(self, region: str = 'eu-central-1'):
        self.client = boto3.client(
            's3',
            region_name=region,
            aws_access_key_id=os.environ['AWS_ACCESS_KEY_ID'],
            aws_secret_access_key=os.environ['AWS_SECRET_ACCESS_KEY']
        )
        self.region = region

    def create_bucket(self, name: str) -> None:
        try:
            config = {} if self.region == 'us-east-1' else {
                'CreateBucketConfiguration': {'LocationConstraint': self.region}
            }
            self.client.create_bucket(Bucket=name, **config)
        except self.client.exceptions.BucketAlreadyOwnedByYou:
            pass

    def upload(self, bucket: str, key: str, local_path: str) -> None:
        self.client.upload_file(local_path, bucket, key)

    def download(self, bucket: str, key: str, local_path: str) -> None:
        self.client.download_file(bucket, key, local_path)

    def list_objects(self, bucket: str, prefix: str = '') -> List[Dict]:
        paginator = self.client.get_paginator('list_objects_v2')
        result = []
        for page in paginator.paginate(Bucket=bucket, Prefix=prefix):
            for obj in page.get('Contents', []):
                result.append({'key': obj['Key'], 'size': obj['Size']})
        return result

    def delete(self, bucket: str, key: str) -> None:
        self.client.delete_object(Bucket=bucket, Key=key)

    def presigned_url(self, bucket: str, key: str, expires_in: int = 3600) -> str:
        return self.client.generate_presigned_url(
            'get_object',
            Params={'Bucket': bucket, 'Key': key},
            ExpiresIn=expires_in
        )


class MinIOStorage(ObjectStorageBase):
    """MinIO implementasyonu."""

    def __init__(self, endpoint: str, secure: bool = False):
        self.client = Minio(
            endpoint=endpoint,
            access_key=os.environ['MINIO_ACCESS_KEY'],
            secret_key=os.environ['MINIO_SECRET_KEY'],
            secure=secure
        )

    def create_bucket(self, name: str) -> None:
        if not self.client.bucket_exists(name):
            self.client.make_bucket(name)

    def upload(self, bucket: str, key: str, local_path: str) -> None:
        self.client.fput_object(bucket, key, local_path)

    def download(self, bucket: str, key: str, local_path: str) -> None:
        self.client.fget_object(bucket, key, local_path)

    def list_objects(self, bucket: str, prefix: str = '') -> List[Dict]:
        objs = self.client.list_objects(bucket, prefix=prefix, recursive=True)
        return [{'key': o.object_name, 'size': o.size} for o in objs]

    def delete(self, bucket: str, key: str) -> None:
        self.client.remove_object(bucket, key)

    def presigned_url(self, bucket: str, key: str, expires_in: int = 3600) -> str:
        from datetime import timedelta
        return self.client.presigned_get_object(
            bucket, key, expires=timedelta(seconds=expires_in)
        )


# ─── Factory ─────────────────────────────────────────────────

def get_storage(provider: str = None) -> ObjectStorageBase:
    """
    Ortam değişkenine göre doğru storage implementasyonunu döndür.
    STORAGE_PROVIDER=s3 | minio
    """
    provider = provider or os.environ.get('STORAGE_PROVIDER', 's3')
    if provider == 's3':
        return S3Storage(region=os.environ.get('AWS_REGION', 'eu-central-1'))
    elif provider == 'minio':
        return MinIOStorage(
            endpoint=os.environ.get('MINIO_ENDPOINT', 'localhost:9000'),
            secure=os.environ.get('MINIO_SECURE', 'false').lower() == 'true'
        )
    raise ValueError(f"Bilinmeyen provider: {provider}")


# ─── Kullanım ─────────────────────────────────────────────────

if __name__ == '__main__':
    # Provider'ı .env'den oku — kod değişmez!
    storage = get_storage()

    storage.create_bucket('my-app-bucket')
    storage.upload('my-app-bucket', 'uploads/photo.jpg', '/tmp/photo.jpg')
    url = storage.presigned_url('my-app-bucket', 'uploads/photo.jpg', expires_in=1800)
    print(f"Erişim URL: {url}")
```

---

## 🏗️ Pratik Uygulama

### FastAPI ile File Upload Servisi

```python
# app/main.py
from fastapi import FastAPI, UploadFile, File, HTTPException, BackgroundTasks
from fastapi.responses import RedirectResponse
import uuid
import tempfile
import os
from storage import get_storage

app = FastAPI(title="File Upload Service", version="1.0.0")
storage = get_storage()
BUCKET = os.environ.get('STORAGE_BUCKET', 'uploads')

@app.on_event("startup")
async def startup():
    storage.create_bucket(BUCKET)

@app.post("/upload", summary="Dosya yükle")
async def upload_file(
    file: UploadFile = File(...),
    folder: str = "general"
):
    # Dosya tipi kontrolü
    allowed_types = {'image/jpeg', 'image/png', 'application/pdf', 'video/mp4'}
    if file.content_type not in allowed_types:
        raise HTTPException(status_code=415, detail=f"Desteklenmeyen dosya tipi: {file.content_type}")

    # Boyut kontrolü (50 MB)
    MAX_SIZE = 50 * 1024 * 1024
    content = await file.read()
    if len(content) > MAX_SIZE:
        raise HTTPException(status_code=413, detail="Dosya 50 MB limitini aştı")

    # Benzersiz key oluştur
    extension = file.filename.rsplit('.', 1)[-1].lower() if '.' in file.filename else ''
    unique_key = f"{folder}/{uuid.uuid4().hex}.{extension}"

    # Geçici dosyaya yaz ve storage'a yükle
    with tempfile.NamedTemporaryFile(delete=False, suffix=f".{extension}") as tmp:
        tmp.write(content)
        tmp_path = tmp.name

    try:
        storage.upload(BUCKET, unique_key, tmp_path)
    finally:
        os.unlink(tmp_path)

    # Pre-signed URL döndür (1 saat geçerli)
    url = storage.presigned_url(BUCKET, unique_key, expires_in=3600)

    return {
        "key": unique_key,
        "url": url,
        "size": len(content),
        "content_type": file.content_type
    }

@app.get("/download/{key:path}", summary="Dosya indir (redirect)")
async def download_file(key: str):
    url = storage.presigned_url(BUCKET, key, expires_in=300)
    return RedirectResponse(url=url)

@app.get("/files", summary="Dosyaları listele")
async def list_files(folder: str = ""):
    objects = storage.list_objects(BUCKET, prefix=folder)
    return {"files": objects, "count": len(objects)}

@app.delete("/files/{key:path}", summary="Dosya sil")
async def delete_file(key: str):
    storage.delete(BUCKET, key)
    return {"message": f"Silindi: {key}"}
```

### Gerçek Dünya Senaryoları

**1. Medya Platformu:**
```
Kullanıcı → Uygulama (pre-signed URL üret) → Tarayıcı/App S3'e direkt yükler
         → Metadata veritabanına kaydet
         → CDN (CloudFront/MinIO presigned) üzerinden servis et
```

**2. Yedekleme Sistemi:**
```python
import subprocess
import datetime

def backup_postgres_to_s3(db_name: str, bucket: str):
    timestamp = datetime.datetime.utcnow().strftime('%Y%m%d_%H%M%S')
    dump_file = f"/tmp/{db_name}_{timestamp}.sql.gz"
    key = f"backups/postgres/{db_name}/{timestamp}.sql.gz"

    # pg_dump | gzip
    subprocess.run(
        f"pg_dump {db_name} | gzip > {dump_file}",
        shell=True, check=True
    )

    storage = get_storage()
    storage.upload(bucket, key, dump_file)
    os.unlink(dump_file)
    print(f"✅ Yedek alındı: s3://{bucket}/{key}")
```

**3. Log Aggregation:**
```python
import logging
import json

class S3LogHandler(logging.Handler):
    """Logları batch halinde S3'e yazar."""
    
    def __init__(self, bucket: str, prefix: str = 'logs/'):
        super().__init__()
        self.storage = get_storage()
        self.bucket = bucket
        self.prefix = prefix
        self.buffer = []

    def emit(self, record):
        log_entry = {
            'timestamp': self.formatTime(record),
            'level': record.levelname,
            'logger': record.name,
            'message': self.format(record)
        }
        self.buffer.append(json.dumps(log_entry))
        if len(self.buffer) >= 100:
            self.flush()

    def flush(self):
        if not self.buffer:
            return
        import datetime, tempfile
        ts = datetime.datetime.utcnow().strftime('%Y/%m/%d/%H%M%S')
        key = f"{self.prefix}{ts}_{id(self)}.ndjson"
        content = '\n'.join(self.buffer)
        with tempfile.NamedTemporaryFile(mode='w', suffix='.ndjson', delete=False) as f:
            f.write(content)
            tmp = f.name
        self.storage.upload(self.bucket, key, tmp)
        os.unlink(tmp)
        self.buffer.clear()
```

---

## 🔧 DevOps & Setup

### MinIO Docker Kurulumu

**Tek node (development):**
```bash
docker run -d \
  --name minio \
  -p 9000:9000 \
  -p 9001:9001 \
  -v $(pwd)/minio-data:/data \
  -e MINIO_ROOT_USER=minioadmin \
  -e MINIO_ROOT_PASSWORD=minioadmin123 \
  quay.io/minio/minio server /data --console-address ":9001"
```

**Docker Compose (production-ready):**
```yaml
# docker-compose.yml
version: '3.8'

services:
  minio:
    image: quay.io/minio/minio:latest
    container_name: minio
    command: server /data{1...4} --console-address ":9001"
    environment:
      MINIO_ROOT_USER: ${MINIO_ROOT_USER}
      MINIO_ROOT_PASSWORD: ${MINIO_ROOT_PASSWORD}
      MINIO_VOLUMES: "/data{1...4}"
    ports:
      - "9000:9000"
      - "9001:9001"
    volumes:
      - minio_data1:/data1
      - minio_data2:/data2
      - minio_data3:/data3
      - minio_data4:/data4
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:9000/minio/health/live"]
      interval: 30s
      timeout: 10s
      retries: 3
    restart: unless-stopped

  # MinIO'yu başlatma ve yapılandırma
  minio-init:
    image: quay.io/minio/mc:latest
    depends_on:
      minio:
        condition: service_healthy
    entrypoint: >
      /bin/sh -c "
      mc alias set local http://minio:9000 $${MINIO_ROOT_USER} $${MINIO_ROOT_PASSWORD};
      mc mb --ignore-existing local/uploads;
      mc mb --ignore-existing local/backups;
      mc anonymous set download local/public;
      echo 'MinIO initialized!';
      "
    environment:
      MINIO_ROOT_USER: ${MINIO_ROOT_USER}
      MINIO_ROOT_PASSWORD: ${MINIO_ROOT_PASSWORD}

volumes:
  minio_data1:
  minio_data2:
  minio_data3:
  minio_data4:
```

**MinIO Kubernetes (Operator ile):**
```bash
# MinIO Operator kurulumu
kubectl apply -k github.com/minio/operator

# Tenant oluşturma
kubectl apply -f - <<EOF
apiVersion: minio.min.io/v2
kind: Tenant
metadata:
  name: minio-tenant
  namespace: minio-operator
spec:
  image: quay.io/minio/minio:latest
  pools:
    - servers: 4
      volumesPerServer: 4
      volumeClaimTemplate:
        spec:
          accessModes:
            - ReadWriteOnce
          resources:
            requests:
              storage: 100Gi
          storageClassName: fast-ssd
  requestAutoCert: true
EOF
```

### AWS S3 Bağlantı Ayarları

```bash
# AWS CLI yapılandırması
aws configure
# AWS Access Key ID: AKIA...
# AWS Secret Access Key: ...
# Default region: eu-central-1
# Default output format: json

# Profil ile çalışma (birden fazla hesap)
aws configure --profile production
aws s3 ls --profile production

# S3 bucket oluşturma ve yapılandırma
aws s3 mb s3://my-company-assets --region eu-central-1
aws s3api put-bucket-versioning \
    --bucket my-company-assets \
    --versioning-configuration Status=Enabled
aws s3api put-bucket-encryption \
    --bucket my-company-assets \
    --server-side-encryption-configuration '{
        "Rules": [{
            "ApplyServerSideEncryptionByDefault": {
                "SSEAlgorithm": "aws:kms"
            }
        }]
    }'
```

### .env Konfigürasyon Örneği

```bash
# .env — versiyon kontrolüne EKLEME, .gitignore'a ekle!

# ─── Storage Provider ───────────────────────────
# Değer: s3 | minio
STORAGE_PROVIDER=minio
STORAGE_BUCKET=my-app-uploads

# ─── AWS S3 ─────────────────────────────────────
AWS_ACCESS_KEY_ID=AKIAIOSFODNN7EXAMPLE
AWS_SECRET_ACCESS_KEY=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
AWS_REGION=eu-central-1
AWS_S3_BUCKET=my-company-assets

# ─── MinIO ──────────────────────────────────────
MINIO_ENDPOINT=localhost:9000
MINIO_ACCESS_KEY=minioadmin
MINIO_SECRET_KEY=minioadmin123
MINIO_SECURE=false
MINIO_BUCKET=uploads

# ─── Application ────────────────────────────────
MAX_UPLOAD_SIZE_MB=50
PRESIGNED_URL_EXPIRES=3600
LOG_LEVEL=INFO
```

```python
# config.py — .env yükleme
from pydantic_settings import BaseSettings
from typing import Literal

class Settings(BaseSettings):
    storage_provider: Literal['s3', 'minio'] = 's3'
    storage_bucket: str = 'uploads'

    # AWS
    aws_access_key_id: str = ''
    aws_secret_access_key: str = ''
    aws_region: str = 'eu-central-1'

    # MinIO
    minio_endpoint: str = 'localhost:9000'
    minio_access_key: str = 'minioadmin'
    minio_secret_key: str = 'minioadmin'
    minio_secure: bool = False

    max_upload_size_mb: int = 50
    presigned_url_expires: int = 3600

    class Config:
        env_file = '.env'

settings = Settings()
```

---

## ✅ Best Practices

### Güvenli Erişim Yönetimi

```python
# ✅ En az yetki prensibi (Principle of Least Privilege)
# Okuma-yazma yapacak servis için minimal policy:

READ_WRITE_POLICY = {
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": ["s3:PutObject", "s3:GetObject", "s3:DeleteObject"],
            "Resource": "arn:aws:s3:::my-bucket/uploads/*"
        },
        {
            "Effect": "Allow",
            "Action": ["s3:ListBucket"],
            "Resource": "arn:aws:s3:::my-bucket",
            "Condition": {"StringLike": {"s3:prefix": ["uploads/*"]}}
        }
    ]
}

# ❌ Asla kullanma: AdministratorAccess veya s3:*
```

### Multipart Upload (Büyük Dosyalar)

S3 ve MinIO, 5 MB'tan büyük parçalarla multipart upload destekler. 100 MB+ dosyalar için zorunludur.

```python
import boto3
import os
from boto3.s3.transfer import TransferConfig

def upload_large_file(local_path: str, bucket: str, key: str):
    """
    Büyük dosya için optimize edilmiş multipart upload.
    - 100 MB üzeri dosyalar için multipart kullanır
    - 8 thread ile paralel yükleme
    - Otomatik retry mekanizması
    """
    s3 = boto3.client('s3')

    config = TransferConfig(
        multipart_threshold=100 * 1024 * 1024,    # 100 MB üstü multipart
        max_concurrency=8,                          # 8 paralel thread
        multipart_chunksize=16 * 1024 * 1024,      # 16 MB chunk boyutu
        use_threads=True
    )

    file_size = os.path.getsize(local_path)
    uploaded = [0]

    def callback(bytes_transferred):
        uploaded[0] += bytes_transferred
        pct = uploaded[0] / file_size * 100
        print(f"\r  📤 {pct:.1f}% ({uploaded[0]/(1024**2):.1f} MB / {file_size/(1024**2):.1f} MB)", end='')

    s3.upload_file(
        Filename=local_path,
        Bucket=bucket,
        Key=key,
        Config=config,
        Callback=callback
    )
    print(f"\n✅ Yükleme tamamlandı!")
```

### Performans Optimizasyonu

| Teknik | Açıklama | Kazanım |
|---|---|---|
| **Transfer Acceleration** | AWS edge network üzerinden yükleme | Küresel hız artışı |
| **Multipart upload** | Paralel parça yükleme | Büyük dosyalarda ~8x hız |
| **Connection pooling** | Boto3 session paylaşımı | CPU ve bağlantı tasarrufu |
| **S3 Select** | S3'te SQL benzeri sorgulama | Transfer azaltma |
| **Byte-range fetching** | Partial object download | Bant genişliği tasarrufu |
| **CloudFront** | CDN önbellekleme | Gecikme azaltma |
| **VPC Endpoint** | AWS ağ içi trafik | Daha düşük gecikme + ücretsiz egress |

```python
# ✅ Session ve resource paylaşımı (connection pooling)
import boto3
from functools import lru_cache

@lru_cache(maxsize=1)
def get_s3_resource():
    """S3 resource'u paylaşımlı ve thread-safe kullan."""
    session = boto3.Session()
    return session.resource('s3')

# Byte-range ile kısmi indirme (streaming)
def download_range(bucket: str, key: str, start: int, end: int) -> bytes:
    s3 = boto3.client('s3')
    response = s3.get_object(
        Bucket=bucket,
        Key=key,
        Range=f'bytes={start}-{end}'
    )
    return response['Body'].read()
```

### Yaygın Hatalar ve Anti-Pattern'ler

```python
# ❌ Anti-pattern 1: Her istekte yeni client oluşturma
def bad_upload(files):
    for f in files:
        s3 = boto3.client('s3')  # Her iterasyonda yeni bağlantı!
        s3.upload_file(f, 'bucket', f)

# ✅ Doğrusu: Client'ı dışarıda oluştur
s3 = boto3.client('s3')
def good_upload(files):
    for f in files:
        s3.upload_file(f, 'bucket', f)

# ❌ Anti-pattern 2: Büyük listeyi tek seferde çekme
response = s3.list_objects(Bucket='bucket')  # Maks 1000 obje!

# ✅ Doğrusu: Paginator kullan
paginator = s3.get_paginator('list_objects_v2')
for page in paginator.paginate(Bucket='bucket'):
    process(page.get('Contents', []))

# ❌ Anti-pattern 3: Public bucket ve geniş ACL
s3.put_object(Bucket='bucket', Key='secret.pdf', ACL='public-read')

# ✅ Doğrusu: Pre-signed URL ile geçici erişim
url = s3.generate_presigned_url('get_object',
    Params={'Bucket': 'bucket', 'Key': 'secret.pdf'},
    ExpiresIn=300)

# ❌ Anti-pattern 4: Senkron upload, büyük dosya
with open('10gb-file.tar.gz', 'rb') as f:
    s3.put_object(Bucket='b', Key='f', Body=f.read())  # Belleği patlatır!

# ✅ Doğrusu: upload_file ile streaming
s3.upload_file('10gb-file.tar.gz', 'b', 'f')

# ❌ Anti-pattern 5: Hata yönetimi yok
s3.download_file('bucket', 'maybe-exists.txt', '/tmp/out.txt')

# ✅ Doğrusu: ClientError yakala
from botocore.exceptions import ClientError
try:
    s3.download_file('bucket', 'maybe-exists.txt', '/tmp/out.txt')
except ClientError as e:
    if e.response['Error']['Code'] == '404':
        print("Dosya bulunamadı")
    else:
        raise
```

---

## 📊 Mimari Diyagramlar

### Object Storage Genel Mimarisi

```mermaid
graph TB
    subgraph "İstemciler"
        WEB[Web Uygulaması]
        MOB[Mobil Uygulama]
        CLI[CLI / Script]
        SVC[Backend Servisi]
    end

    subgraph "Erişim Katmanı"
        LB[Load Balancer / CDN]
        GW[API Gateway]
        AUTH[Auth Service]
    end

    subgraph "Object Storage"
        direction TB
        BUCKET1[(Bucket: uploads)]
        BUCKET2[(Bucket: backups)]
        BUCKET3[(Bucket: public)]
    end

    subgraph "Depolama Katmanı"
        direction LR
        DISK1[💾 Disk 1]
        DISK2[💾 Disk 2]
        DISK3[💾 Disk 3]
        DISK4[💾 Disk 4]
    end

    WEB --> LB
    MOB --> LB
    CLI --> GW
    SVC --> GW

    LB --> BUCKET3
    GW --> AUTH
    AUTH --> BUCKET1
    AUTH --> BUCKET2

    BUCKET1 --> DISK1
    BUCKET1 --> DISK2
    BUCKET2 --> DISK3
    BUCKET2 --> DISK4
    BUCKET3 --> DISK1
    BUCKET3 --> DISK3

    style BUCKET1 fill:#FF9900,color:#fff
    style BUCKET2 fill:#C72E49,color:#fff
    style BUCKET3 fill:#2496ED,color:#fff
```

### Upload Akış Diyagramı

```mermaid
sequenceDiagram
    participant C as İstemci
    participant API as Backend API
    participant AUTH as Auth Service
    participant S3 as S3 / MinIO
    participant DB as Veritabanı

    C->>API: POST /upload/request {filename, size, type}
    API->>AUTH: Token doğrula
    AUTH-->>API: Kullanıcı kimliği
    API->>S3: Pre-signed URL üret (PUT, 15 dk)
    S3-->>API: Presigned URL + Key
    API-->>C: {uploadUrl, fileKey}

    Note over C,S3: İstemci doğrudan S3'e yükler (API bypass)

    C->>S3: PUT {uploadUrl} (binary data)
    S3-->>C: 200 OK + ETag

    C->>API: POST /upload/confirm {fileKey, etag}
    API->>S3: Head Object (boyut, metadata doğrula)
    S3-->>API: Object metadata
    API->>DB: Dosya kaydı oluştur
    DB-->>API: Record ID
    API-->>C: {fileId, accessUrl}
```

### MinIO Distributed Mimari

```mermaid
graph TB
    CLIENT[İstemci / SDK] --> HAP[HAProxy / nginx]

    HAP --> N1
    HAP --> N2
    HAP --> N3
    HAP --> N4

    subgraph "MinIO Cluster (4 Node)"
        N1[Node 1<br/>4x NVMe]
        N2[Node 2<br/>4x NVMe]
        N3[Node 3<br/>4x NVMe]
        N4[Node 4<br/>4x NVMe]

        N1 <--> N2
        N2 <--> N3
        N3 <--> N4
        N4 <--> N1
        N1 <--> N3
        N2 <--> N4
    end

    subgraph "Erasure Coding (EC:8)"
        EC[Data: 8 shards<br/>Parity: 8 shards<br/>Tolerate: 8 drive loss]
    end

    N1 -.-> EC
    N2 -.-> EC
    N3 -.-> EC
    N4 -.-> EC
```

---

##  Quick Start

### 5 Dakikada MinIO + Python

```bash
# 1. MinIO başlat
docker run -d --name minio \
  -p 9000:9000 -p 9001:9001 \
  -e MINIO_ROOT_USER=minioadmin \
  -e MINIO_ROOT_PASSWORD=minioadmin \
  quay.io/minio/minio server /data --console-address ":9001"

# 2. Bağımlılıkları kur
pip install boto3 minio python-dotenv

# 3. .env dosyası oluştur
cat > .env << 'EOF'
STORAGE_PROVIDER=minio
MINIO_ENDPOINT=localhost:9000
MINIO_ACCESS_KEY=minioadmin
MINIO_SECRET_KEY=minioadmin
MINIO_SECURE=false
STORAGE_BUCKET=test-bucket
EOF
```

```python
# quickstart.py
from dotenv import load_dotenv
load_dotenv()

from storage import get_storage  # Yukarıdaki abstraction sınıfı

storage = get_storage()
bucket = 'test-bucket'

# Bucket oluştur
storage.create_bucket(bucket)

# Dosya yükle
with open('/tmp/test.txt', 'w') as f:
    f.write('Hello, Object Storage!')

storage.upload(bucket, 'hello/test.txt', '/tmp/test.txt')

# Listele
files = storage.list_objects(bucket)
print(f"Bucket içeriği: {files}")

# URL al
url = storage.presigned_url(bucket, 'hello/test.txt', expires_in=300)
print(f"Erişim URL (5 dk): {url}")

# MinIO Console: http://localhost:9001 (minioadmin/minioadmin)
```

### AWS S3 Quick Start

```bash
# AWS CLI ile hızlı başlangıç
aws configure  # Credentials gir

# Bucket oluştur
aws s3 mb s3://my-quickstart-bucket --region eu-central-1

# Dosya yükle
echo "Hello S3!" > test.txt
aws s3 cp test.txt s3://my-quickstart-bucket/hello/test.txt

# Listele
aws s3 ls s3://my-quickstart-bucket --recursive

# İndir
aws s3 cp s3://my-quickstart-bucket/hello/test.txt ./downloaded.txt

# Sil
aws s3 rm s3://my-quickstart-bucket/hello/test.txt
aws s3 rb s3://my-quickstart-bucket  # Bucket sil (boş olmalı)
```

---

## 📋 Cheat Sheet

### AWS S3 CLI Komutları

| İşlem | Komut |
|---|---|
| Bucket listele | `aws s3 ls` |
| Bucket oluştur | `aws s3 mb s3://bucket-name` |
| Dosya yükle | `aws s3 cp file.txt s3://bucket/key` |
| Klasör yükle | `aws s3 cp ./dir s3://bucket/prefix/ --recursive` |
| Dosya indir | `aws s3 cp s3://bucket/key file.txt` |
| Sync (sadece değişenler) | `aws s3 sync ./local s3://bucket/prefix` |
| Listele | `aws s3 ls s3://bucket/prefix/ --recursive` |
| Sil | `aws s3 rm s3://bucket/key` |
| Toplu sil | `aws s3 rm s3://bucket/prefix/ --recursive` |
| Bucket sil | `aws s3 rb s3://bucket --force` |
| Pre-signed URL | `aws s3 presign s3://bucket/key --expires-in 3600` |
| Bucket boyutu | `aws s3 ls s3://bucket --recursive --human-readable --summarize` |

### MinIO CLI (mc) Komutları

| İşlem | Komut |
|---|---|
| Alias ekle | `mc alias set local http://localhost:9000 user pass` |
| Bucket listele | `mc ls local` |
| Bucket oluştur | `mc mb local/bucket-name` |
| Dosya yükle | `mc cp file.txt local/bucket/key` |
| Dosya indir | `mc cp local/bucket/key ./file.txt` |
| Listele | `mc ls local/bucket/prefix/` |
| Sil | `mc rm local/bucket/key` |
| Mirror | `mc mirror ./local local/bucket` |
| İzle (watch) | `mc watch local/bucket` |
| Policy ayarla | `mc anonymous set download local/bucket` |
| Admin bilgisi | `mc admin info local` |
| Kullanıcı ekle | `mc admin user add local user pass` |

### Python boto3 Hızlı Referans

```python
import boto3

s3 = boto3.client('s3')

# Temel işlemler
s3.create_bucket(Bucket='b', CreateBucketConfiguration={'LocationConstraint': 'eu-west-1'})
s3.upload_file('local.txt', 'b', 'key.txt')
s3.download_file('b', 'key.txt', 'local.txt')
s3.delete_object(Bucket='b', Key='key.txt')
s3.copy_object(CopySource={'Bucket': 'src', 'Key': 'k'}, Bucket='dst', Key='k')

# Listeleme (paginator ile)
paginator = s3.get_paginator('list_objects_v2')
for page in paginator.paginate(Bucket='b', Prefix='logs/'):
    for obj in page.get('Contents', []):
        print(obj['Key'], obj['Size'])

# Pre-signed URL
url = s3.generate_presigned_url('get_object',
    Params={'Bucket': 'b', 'Key': 'k'}, ExpiresIn=3600)

# Metadata okuma
meta = s3.head_object(Bucket='b', Key='k')
print(meta['ContentLength'], meta['ContentType'], meta['LastModified'])

# Tag işlemleri
s3.put_object_tagging(Bucket='b', Key='k',
    Tagging={'TagSet': [{'Key': 'env', 'Value': 'prod'}]})

# Versioning
s3.put_bucket_versioning(Bucket='b',
    VersioningConfiguration={'Status': 'Enabled'})
```

### Storage Sınıfı Seçim Kılavuzu

```
Erişim Frekansı?
├── Günlük / Sürekli     → S3 Standard
├── Haftalık             → S3 Intelligent-Tiering
├── Aylık                → S3 Standard-IA
├── Yıllık               → S3 Glacier Instant Retrieval
└── 7+ yıl arşiv        → S3 Glacier Deep Archive

Erişim Hızı Kritik mi?
├── Evet, ms gerekli     → Standard / Intelligent-Tiering / Glacier Instant
└── Hayır, saatler kabul → Glacier Flexible / Deep Archive
```

### Güvenlik Kontrol Listesi

- [ ] Block Public Access tüm bucket'larda aktif
- [ ] Versioning kritik bucket'larda aktif
- [ ] Sunucu tarafı şifreleme (SSE-S3 veya SSE-KMS) aktif
- [ ] Bucket policy VPC endpoint kısıtlaması içeriyor
- [ ] IAM kullanıcıları için en az yetki prensibi uygulandı
- [ ] CloudTrail / MinIO audit log aktif
- [ ] MFA Delete aktif (kritik veriler için)
- [ ] Lifecycle policy ile eski versiyonlar temizleniyor
- [ ] Cross-region replikasyon felaket kurtarma için değerlendirildi
- [ ] S3 Access Analyzer ile yanlış yapılandırma kontrol ediliyor

---

## 📚 Kaynaklar

- [AWS S3 Resmi Dokümantasyonu](https://docs.aws.amazon.com/s3/)
- [MinIO Resmi Dokümantasyonu](https://min.io/docs/)
- [AWS boto3 Dokümantasyonu](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/s3.html)
- [MinIO Python SDK](https://min.io/docs/minio/linux/developers/python/minio-py.html)
- [S3 Pricing Calculator](https://aws.amazon.com/s3/pricing/)
- [MinIO GitHub Repository](https://github.com/minio/minio)

---

<div align="center">

**Bu doküman bir referans rehberidir.**

[![Made with ❤️](https://img.shields.io/badge/Made_with-❤️-red?style=flat-square)](https://github.com)

</div>
