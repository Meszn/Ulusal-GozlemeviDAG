# 🧠 Memento Design Pattern

<div align="center">

![Design Pattern](https://img.shields.io/badge/Design%20Pattern-Behavioral-blueviolet?style=for-the-badge&logo=abstract&logoColor=white)
![Difficulty](https://img.shields.io/badge/Difficulty-Intermediate-orange?style=for-the-badge)
![GoF](https://img.shields.io/badge/GoF-Gang%20of%20Four-darkgreen?style=for-the-badge)
![Languages](https://img.shields.io/badge/Languages-Java%20%7C%20Python%20%7C%20C%23-informational?style=for-the-badge)

> *"Geçmişi hatırlayan, geleceği değiştirebildiği için ona hükmeder."*

**Bir nesnenin iç durumunu dışarıya sızdırmadan kaydedip geri yükleyen davranışsal tasarım deseni.**

[📖 Tanım](#-tanım-ve-amaç) • [🏗️ Yapı](#️-yapı-ve-bileşenler) • [💡 Örnekler](#-gerçek-hayat-senaryoları) • [💻 Kod](#-kod-örnekleri) • [⚖️ Karşılaştırma](#️-diğer-desenlerle-karşılaştırma) • [🎓 Mülakat](#-mülakat-soruları)

</div>

---

## 📋 İçindekiler

- [📖 Tanım ve Amaç](#-tanım-ve-amaç)
- [🎯 Ne Zaman Kullanılır?](#-ne-zaman-kullanılır)
- [🏗️ Yapı ve Bileşenler](#️-yapı-ve-bileşenler)
- [📐 UML Diyagramı](#-uml-diyagramı)
- [🔄 Akış Diyagramı](#-akış-diyagramı)
- [💡 Gerçek Hayat Senaryoları](#-gerçek-hayat-senaryoları)
- [💻 Kod Örnekleri](#-kod-örnekleri)
  - [☕ Java](#-java---metin-editörü-undoredo)
  - [🐍 Python](#-python---oyun-kayıt-sistemi)
  - [🔷 C#](#-c---form-geri-alma-sistemi)
- [✅ Avantajlar](#-avantajlar)
- [❌ Dezavantajlar](#-dezavantajlar)
- [🔗 SOLID Prensipleri ile İlişkisi](#-solid-prensipleri-ile-ilişkisi)
- [⚖️ Diğer Desenlerle Karşılaştırma](#️-diğer-desenlerle-karşılaştırma)
- [🎓 Mülakat Soruları](#-mülakat-soruları)
- [📚 Kaynaklar](#-kaynaklar)

---

## 📖 Tanım ve Amaç

**Memento Pattern** (Hatıra/Anı Deseni), bir nesnenin iç durumunu **kapsülleme ilkesini bozmadan** dışarıya kaydetmeye ve gerektiğinde bu durumu geri yüklemeye olanak tanıyan bir **davranışsal (behavioral) tasarım deseni**dir.

GoF (Gang of Four) kitabında şu şekilde tanımlanmıştır:

> *"Without violating encapsulation, capture and externalize an object's internal state so that the object can be restored to this state later."*

### 🎯 Temel Hedefler

| Hedef | Açıklama |
|-------|----------|
| 🔒 **Kapsülleme Koruma** | Nesnenin iç detayları dışarıya sızdırılmaz |
| ⏪ **Geri Alma (Undo)** | Önceki duruma dönme imkânı sağlar |
| ⏩ **İleri Alma (Redo)** | İptal edilen işlemi tekrar uygulama |
| 📸 **Snapshot** | Belirli anların anlık görüntüsünü alır |
| 🛡️ **Güvenli Saklama** | Durum bilgisi güvenli şekilde dışarıda tutulur |

---

## 🎯 Ne Zaman Kullanılır?

Memento desenini şu durumlarda kullanmalısınız:

```
✅ Bir nesnenin önceki durumuna geri dönme ihtiyacı varsa
✅ Nesnenin iç yapısını ifşa etmeden anlık görüntü almak istiyorsanız
✅ Durum geçmişini takip etmeniz gerekiyorsa
✅ İşlem geri alma / tekrarlama (undo/redo) mekanizması kuruyorsanız
✅ Transaction tabanlı işlemlerde rollback gerekiyorsa
```

```
❌ Nesne durumu çok büyük ve sık değişiyorsa (bellek sorunu)
❌ Tüm geçmiş tutulmak zorunda değilse basit flag yeterlidir
❌ Performans kritik sistemlerde dikkatli olunmalıdır
```

---

## 🏗️ Yapı ve Bileşenler

Memento deseni **3 temel bileşenden** oluşur:

```
┌─────────────────────────────────────────────────────────┐
│                    MEMENTO PATTERN                      │
│                                                         │
│   ┌─────────────┐     creates    ┌─────────────────┐   │
│   │  Originator │───────────────▶│    Memento      │   │
│   │             │                │                 │   │
│   │ -state      │◀───────────────│ -state (private)│   │
│   │ +save()     │   restores     │ +getState()     │   │
│   │ +restore()  │                └─────────────────┘   │
│   └─────────────┘                         ▲            │
│                                           │  stores    │
│                                  ┌────────┴────────┐   │
│                                  │    Caretaker    │   │
│                                  │                 │   │
│                                  │ -mementos[]     │   │
│                                  │ +backup()       │   │
│                                  │ +undo()         │   │
│                                  └─────────────────┘   │
└─────────────────────────────────────────────────────────┘
```

### 🔑 Bileşen Açıklamaları

#### 1️⃣ Originator (Kaynak Nesne)
> Durumu kaydedilmek ve geri yüklenmek istenen **asıl nesne**.

- Kendi `state`'ini tutar
- `save()` metodu ile Memento nesnesi üretir
- `restore(memento)` metodu ile durumunu geri yükler
- Kapsülleme ilkesine uygun şekilde state'ini korur

#### 2️⃣ Memento (Hatıra Nesnesi)
> Originator'ın belirli bir andaki **durumunun anlık görüntüsü**.

- Originator'ın state'ini saklar
- Dışarıdan erişim **kısıtlıdır** (private/immutable)
- Yalnızca Originator tarafından okunabilir olmalıdır
- Genellikle **immutable** (değiştirilemez) tasarlanır

#### 3️⃣ Caretaker (Bakıcı/Yönetici)
> Memento nesnelerini **saklayan ve yöneten** nesne.

- Memento'ların listesini/stack'ini tutar
- Memento'nun içeriğini **görmez ve değiştirmez**
- Undo/Redo operasyonlarını tetikler
- Bellek yönetiminden sorumludur

---

## 📐 UML Diyagramı

```
┌──────────────────────────────────────────────────────────────────────┐
│                         «interface»                                  │
│                           Memento                                    │
│                    ──────────────────                                │
│                    + getName(): String                               │
│                    + getDate(): String                               │
└──────────────────────────────────────────────────────────────────────┘
                                △
                                │ implements
                                │
┌──────────────────┐   creates  │  ┌──────────────────────────────────┐
│    Originator    │────────────┼─▶│        ConcreteMemento           │
│ ──────────────── │            │  │ ──────────────────────────────── │
│ - state: String  │            │  │ - state: String                  │
│ ──────────────── │            │  │ - name: String                   │
│ + save():Memento │            │  │ - date: Date                     │
│ + restore(m)     │            │  │ ──────────────────────────────── │
│ + doSomething()  │◀───────────┘  │ + ConcreteMemento(state)         │
└──────────────────┘  restores     │ + getName(): String              │
                                   │ + getDate(): String              │
                                   │ + getState(): String  «package»  │
                                   └──────────────────────────────────┘
                                                    △
                                                    │ stores
                                                    │
                          ┌─────────────────────────┴────────┐
                          │           Caretaker              │
                          │ ──────────────────────────────── │
                          │ - mementos: List<Memento>        │
                          │ - originator: Originator         │
                          │ ──────────────────────────────── │
                          │ + backup()                       │
                          │ + undo()                         │
                          │ + showHistory()                  │
                          └──────────────────────────────────┘
```

---

## 🔄 Akış Diyagramı

```
     Kullanıcı İşlemi
           │
           ▼
    ┌─────────────┐
    │  Değişiklik │
    │  Yapmak     │
    │  İstiyor    │
    └──────┬──────┘
           │
           ▼
    ┌─────────────┐        ┌─────────────────────┐
    │  Caretaker  │        │     Originator       │
    │  backup()   │───────▶│  createMemento()     │
    │  çağırır    │        │  state'i paketle     │
    └─────────────┘        └──────────┬──────────┘
                                      │
                                      ▼
                           ┌──────────────────────┐
                           │   ConcreteMemento     │
                           │   Oluşturulur         │
                           │   (state kopyalanır)  │
                           └──────────┬───────────┘
                                      │
                                      ▼
                           ┌──────────────────────┐
                           │   Caretaker Stack'e  │
                           │   eklenir             │
                           └──────────┬───────────┘
                                      │
           ┌───────────────────────────┘
           │
           ▼
    ┌─────────────┐
    │  Originator │
    │  değişiklik │
    │  yapar      │
    └──────┬──────┘
           │
           │    ◄── Kullanıcı UNDO ister
           ▼
    ┌─────────────┐        ┌─────────────────────┐
    │  Caretaker  │        │     Originator       │
    │  undo()     │───────▶│  restore(memento)    │
    │  çağırır    │        │  state geri yüklenir │
    └─────────────┘        └─────────────────────┘
```

---

## 💡 Gerçek Hayat Senaryoları

### 📝 1. Metin Editörü (Undo/Redo)

```
┌─────────────────────────────────────────────────────────┐
│  📄 Text Editor                             [_][□][X]   │
│─────────────────────────────────────────────────────────│
│  File  Edit  View  Help                                 │
│─────────────────────────────────────────────────────────│
│                                                         │
│  Merhaba Dünya!                                         │
│  Bu bir test metnidir.                                  │
│  |█                                                     │
│                                                         │
│─────────────────────────────────────────────────────────│
│  History Stack:                                         │
│  ├── [State 3] "Merhaba Dünya!\nBu bir test..."  ◄ NOW  │
│  ├── [State 2] "Merhaba Dünya!\n"                       │
│  └── [State 1] "Merhaba "                               │
│  [Ctrl+Z Undo] [Ctrl+Y Redo]                            │
└─────────────────────────────────────────────────────────┘
```

**Her tuş vuruşunda veya belirli aralıklarla state snapshot alınır. Ctrl+Z ile bir önceki snapshot'a dönülür.**

---

### 🎮 2. Oyun Kayıt Sistemi (Save/Load)

```
┌─────────────────────────────────────────────────────────┐
│  🗡️  RPG GAME - Dragon's Quest                          │
│─────────────────────────────────────────────────────────│
│  Hero: Aragorn    HP: ████████░░ 80/100                 │
│  Level: 15        XP: ████████████ 9500/10000           │
│  Gold: 2500 💰    Location: Dark Forest                 │
│─────────────────────────────────────────────────────────│
│  SAVE SLOTS:                                            │
│  ┌──────────────────────────────────────────────┐      │
│  │ Slot 1: LVL 15 | HP:80 | Dark Forest | 12:45 │      │
│  │ Slot 2: LVL 12 | HP:95 | Castle Gate | 10:30 │      │
│  │ Slot 3: LVL 8  | HP:60 | Village     | 08:15 │      │
│  └──────────────────────────────────────────────┘      │
│  [💾 SAVE] [📂 LOAD] [🗑️ DELETE]                        │
└─────────────────────────────────────────────────────────┘
```

**Her kayıt noktası bir Memento nesnesidir. Karakterin tüm özellikleri, envanteri, konumu snapshot olarak saklanır.**

---

### 📋 3. Form Geri Alma Sistemi

```
┌─────────────────────────────────────────────────────────┐
│  📝 Kullanıcı Kayıt Formu                               │
│─────────────────────────────────────────────────────────│
│  Ad:       [Ahmet              ]                        │
│  Soyad:    [Yılmaz             ]                        │
│  E-posta:  [ahmet@example.com  ]                        │
│  Telefon:  [0532 xxx xx xx     ]                        │
│─────────────────────────────────────────────────────────│
│  Form History:                                          │
│  ├── Snapshot 3: {ad:"Ahmet", soyad:"Yılmaz"...} ◄ NOW │
│  ├── Snapshot 2: {ad:"Ahmet", soyad:""}                 │
│  └── Snapshot 1: {ad:"", soyad:""}                      │
│─────────────────────────────────────────────────────────│
│  [↩ Geri Al] [↪ İleri Al] [✅ Kaydet] [❌ İptal]        │
└─────────────────────────────────────────────────────────┘
```

---

### 🏦 4. Veritabanı Transaction Rollback

```
BEGIN TRANSACTION
        │
        ▼
  [Snapshot al] ◄──── Memento oluşturulur
        │
        ▼
  İşlem 1: UPDATE balance SET amount = 5000
        │
        ▼
  İşlem 2: INSERT INTO log VALUES (...)
        │
        ▼
   Hata mı? ──YES──▶ ROLLBACK (Memento'dan restore)
        │
        NO
        ▼
  COMMIT (Memento silinir)
```

---

## 💻 Kod Örnekleri

### ☕ Java - Metin Editörü Undo/Redo

```java
import java.util.ArrayDeque;
import java.util.Deque;

// ─────────────────────────────────────────────
// MEMENTO: Editörün belirli anının snapshot'ı
// ─────────────────────────────────────────────
public class EditorMemento {
    private final String content;
    private final int cursorPosition;
    private final long timestamp;

    // Package-private constructor: yalnızca Editor erişebilir
    EditorMemento(String content, int cursorPosition) {
        this.content = content;
        this.cursorPosition = cursorPosition;
        this.timestamp = System.currentTimeMillis();
    }

    // Yalnızca Editor bu metotları çağırabilir (package-private)
    String getContent() { return content; }
    int getCursorPosition() { return cursorPosition; }

    @Override
    public String toString() {
        return String.format("[%d] '%s' (cursor: %d)", 
            timestamp, content.substring(0, Math.min(20, content.length())), 
            cursorPosition);
    }
}

// ─────────────────────────────────────────────
// ORIGINATOR: Asıl editör nesnesi
// ─────────────────────────────────────────────
public class TextEditor {
    private StringBuilder content = new StringBuilder();
    private int cursorPosition = 0;

    public void type(String text) {
        content.insert(cursorPosition, text);
        cursorPosition += text.length();
        System.out.println("✏️  Yazıldı: '" + text + "' → " + getContent());
    }

    public void delete(int count) {
        if (cursorPosition >= count) {
            content.delete(cursorPosition - count, cursorPosition);
            cursorPosition -= count;
            System.out.println("🗑️  Silindi " + count + " karakter → " + getContent());
        }
    }

    public void moveCursor(int position) {
        this.cursorPosition = Math.max(0, Math.min(position, content.length()));
    }

    // Memento oluştur (snapshot al)
    public EditorMemento save() {
        System.out.println("💾 Snapshot alındı: " + getContent());
        return new EditorMemento(content.toString(), cursorPosition);
    }

    // Memento'dan geri yükle
    public void restore(EditorMemento memento) {
        this.content = new StringBuilder(memento.getContent());
        this.cursorPosition = memento.getCursorPosition();
        System.out.println("⏪ Geri yüklendi: " + getContent());
    }

    public String getContent() { return content.toString(); }
    public int getCursorPosition() { return cursorPosition; }
}

// ─────────────────────────────────────────────
// CARETAKER: Geçmişi yöneten sınıf
// ─────────────────────────────────────────────
public class EditorHistory {
    private final Deque<EditorMemento> undoStack = new ArrayDeque<>();
    private final Deque<EditorMemento> redoStack = new ArrayDeque<>();
    private final TextEditor editor;

    public EditorHistory(TextEditor editor) {
        this.editor = editor;
    }

    public void backup() {
        undoStack.push(editor.save());
        redoStack.clear(); // Yeni işlem yapılınca redo geçmişi temizlenir
    }

    public void undo() {
        if (undoStack.isEmpty()) {
            System.out.println("⚠️  Geri alınacak işlem yok!");
            return;
        }
        // Mevcut durumu redo stack'e kaydet
        redoStack.push(editor.save());
        // Önceki durumu geri yükle
        editor.restore(undoStack.pop());
    }

    public void redo() {
        if (redoStack.isEmpty()) {
            System.out.println("⚠️  İleri alınacak işlem yok!");
            return;
        }
        undoStack.push(editor.save());
        editor.restore(redoStack.pop());
    }

    public void showHistory() {
        System.out.println("\n📚 Undo Geçmişi (" + undoStack.size() + " adım):");
        undoStack.forEach(m -> System.out.println("  → " + m));
    }
}

// ─────────────────────────────────────────────
// DEMO: Kullanım örneği
// ─────────────────────────────────────────────
public class MementoDemo {
    public static void main(String[] args) {
        TextEditor editor = new TextEditor();
        EditorHistory history = new EditorHistory(editor);

        System.out.println("═══════ MEMENTO PATTERN DEMO ═══════\n");

        history.backup();
        editor.type("Merhaba");

        history.backup();
        editor.type(", Dünya");

        history.backup();
        editor.type("!");

        System.out.println("\n--- UNDO ---");
        history.undo(); // "Merhaba, Dünya" olur
        history.undo(); // "Merhaba" olur

        System.out.println("\n--- REDO ---");
        history.redo(); // "Merhaba, Dünya" olur

        history.showHistory();
    }
}
```

**Çıktı:**
```
═══════ MEMENTO PATTERN DEMO ═══════

💾 Snapshot alındı: 
✏️  Yazıldı: 'Merhaba' → Merhaba
💾 Snapshot alındı: Merhaba
✏️  Yazıldı: ', Dünya' → Merhaba, Dünya
💾 Snapshot alındı: Merhaba, Dünya
✏️  Yazıldı: '!' → Merhaba, Dünya!

--- UNDO ---
💾 Snapshot alındı: Merhaba, Dünya!
⏪ Geri yüklendi: Merhaba, Dünya
💾 Snapshot alındı: Merhaba, Dünya
⏪ Geri yüklendi: Merhaba

--- REDO ---
💾 Snapshot alındı: Merhaba
⏪ Geri yüklendi: Merhaba, Dünya
```

---

### 🐍 Python - Oyun Kayıt Sistemi

```python
from __future__ import annotations
from dataclasses import dataclass, field
from datetime import datetime
from typing import List, Optional
import copy

# ─────────────────────────────────────────────
# MEMENTO: Oyun durumunun anlık görüntüsü
# ─────────────────────────────────────────────
@dataclass(frozen=True)  # Immutable memento
class GameSave:
    """Oyunun belirli bir anındaki durumunu temsil eder."""
    _character_name: str
    _level: int
    _health: int
    _max_health: int
    _experience: int
    _gold: int
    _location: str
    _inventory: tuple  # Immutable olması için tuple
    _save_date: str = field(default_factory=lambda: datetime.now().strftime("%Y-%m-%d %H:%M"))
    _slot_name: str = "Auto Save"

    def __str__(self) -> str:
        return (
            f"[{self._slot_name}] {self._character_name} | "
            f"LVL:{self._level} | HP:{self._health}/{self._max_health} | "
            f"📍{self._location} | 💰{self._gold} | {self._save_date}"
        )


# ─────────────────────────────────────────────
# ORIGINATOR: Oyun karakteri
# ─────────────────────────────────────────────
class GameCharacter:
    """Oyun karakteri - durumunu kaydedip geri yükleyebilir."""

    def __init__(self, name: str):
        self._name = name
        self._level = 1
        self._health = 100
        self._max_health = 100
        self._experience = 0
        self._gold = 0
        self._location = "Başlangıç Köyü"
        self._inventory: List[str] = []

    def save(self, slot_name: str = "Auto Save") -> GameSave:
        """Mevcut durumu bir GameSave (Memento) olarak döndürür."""
        print(f"💾 Kaydediliyor... [{slot_name}]")
        return GameSave(
            _character_name=self._name,
            _level=self._level,
            _health=self._health,
            _max_health=self._max_health,
            _experience=self._experience,
            _gold=self._gold,
            _location=self._location,
            _inventory=tuple(self._inventory),
            _slot_name=slot_name
        )

    def restore(self, save: GameSave) -> None:
        """GameSave'den durumu geri yükler."""
        self._level = save._level
        self._health = save._health
        self._max_health = save._max_health
        self._experience = save._experience
        self._gold = save._gold
        self._location = save._location
        self._inventory = list(save._inventory)
        print(f"📂 Yüklendi: {save}")

    def gain_experience(self, amount: int) -> None:
        self._experience += amount
        if self._experience >= self._level * 1000:
            self._level += 1
            self._max_health += 20
            self._health = self._max_health
            print(f"⭐ SEVİYE ATLADI! Yeni seviye: {self._level}")
        print(f"✨ {amount} XP kazanıldı. Toplam: {self._experience}")

    def take_damage(self, amount: int) -> None:
        self._health = max(0, self._health - amount)
        print(f"💔 {amount} hasar alındı. HP: {self._health}/{self._max_health}")

    def collect_gold(self, amount: int) -> None:
        self._gold += amount
        print(f"💰 {amount} altın toplandı. Toplam: {self._gold}")

    def move_to(self, location: str) -> None:
        self._location = location
        print(f"🗺️  Taşındı: {location}")

    def pick_up_item(self, item: str) -> None:
        self._inventory.append(item)
        print(f"🎒 Eşya alındı: {item}")

    def status(self) -> str:
        return (
            f"\n{'═'*50}\n"
            f"  🦸 {self._name} | LVL {self._level}\n"
            f"  ❤️  HP: {self._health}/{self._max_health}\n"
            f"  ✨ XP: {self._experience}\n"
            f"  💰 Altın: {self._gold}\n"
            f"  📍 Konum: {self._location}\n"
            f"  🎒 Envanter: {', '.join(self._inventory) or 'Boş'}\n"
            f"{'═'*50}"
        )


# ─────────────────────────────────────────────
# CARETAKER: Kayıt yuvalarını yöneten sistem
# ─────────────────────────────────────────────
class SaveManager:
    """Oyun kayıtlarını yöneten sistem - max 3 slot."""

    def __init__(self, character: GameCharacter, max_slots: int = 3):
        self._character = character
        self._max_slots = max_slots
        self._slots: List[Optional[GameSave]] = [None] * max_slots
        self._auto_saves: List[GameSave] = []

    def save_to_slot(self, slot: int, slot_name: Optional[str] = None) -> bool:
        if not 0 <= slot < self._max_slots:
            print(f"❌ Geçersiz slot: {slot}")
            return False
        name = slot_name or f"Slot {slot + 1}"
        self._slots[slot] = self._character.save(name)
        return True

    def load_from_slot(self, slot: int) -> bool:
        if not 0 <= slot < self._max_slots or self._slots[slot] is None:
            print(f"❌ Slot {slot + 1} boş veya geçersiz!")
            return False
        self._character.restore(self._slots[slot])
        return True

    def auto_save(self) -> None:
        save = self._character.save("Auto Save")
        self._auto_saves.append(save)
        if len(self._auto_saves) > 10:  # Max 10 auto save tut
            self._auto_saves.pop(0)

    def load_last_auto_save(self) -> bool:
        if not self._auto_saves:
            print("❌ Auto save bulunamadı!")
            return False
        self._character.restore(self._auto_saves[-1])
        return True

    def show_saves(self) -> None:
        print("\n📁 KAYIT YUVALARI:")
        print("─" * 60)
        for i, slot in enumerate(self._slots):
            if slot:
                print(f"  Slot {i+1}: {slot}")
            else:
                print(f"  Slot {i+1}: [Boş]")
        print(f"\n  🔄 Auto Save sayısı: {len(self._auto_saves)}")
        print("─" * 60)


# ─────────────────────────────────────────────
# DEMO
# ─────────────────────────────────────────────
if __name__ == "__main__":
    print("🎮 OYUN KAYIT SİSTEMİ - MEMENTO PATTERN\n")

    hero = GameCharacter("Aragorn")
    save_manager = SaveManager(hero)

    # Oyuna başla
    hero.move_to("Karanlık Orman")
    hero.gain_experience(500)
    hero.collect_gold(100)
    hero.pick_up_item("Demir Kılıç")
    print(hero.status())

    # Manuel kayıt
    save_manager.save_to_slot(0, "Orman Girişi")

    # Devam et
    hero.gain_experience(600)  # Seviye atlar
    hero.move_to("Ejderha Mağarası")
    hero.pick_up_item("Ejderha Kalkanı")
    save_manager.save_to_slot(1, "Ejderha Öncesi")
    print(hero.status())

    # Zor boss savaşı - büyük hasar al
    print("\n⚔️  BOSS SAVAŞI!")
    hero.take_damage(90)
    hero.take_damage(5)  # Ölüm!
    print(hero.status())

    # Kayıttan yükle!
    print("\n💀 Karakter öldü! Kayıttan yükleniyor...")
    save_manager.show_saves()
    save_manager.load_from_slot(1)
    print(hero.status())
```

---

### 🔷 C# - Form Geri Alma Sistemi

```csharp
using System;
using System.Collections.Generic;
using System.Text.Json;

namespace MementoPattern
{
    // ─────────────────────────────────────────────
    // MEMENTO: Form durumunun snapshot'ı
    // ─────────────────────────────────────────────
    public sealed class FormMemento
    {
        // Internal erişim: Yalnızca aynı assembly içindeki sınıflar erişebilir
        internal string FirstName { get; }
        internal string LastName { get; }
        internal string Email { get; }
        internal string Phone { get; }
        internal DateTime CapturedAt { get; }

        internal FormMemento(string firstName, string lastName, 
                            string email, string phone)
        {
            FirstName = firstName;
            LastName = lastName;
            Email = email;
            Phone = phone;
            CapturedAt = DateTime.Now;
        }

        public override string ToString() =>
            $"[{CapturedAt:HH:mm:ss}] {FirstName} {LastName} | {Email}";
    }

    // ─────────────────────────────────────────────
    // ORIGINATOR: Form nesnesi
    // ─────────────────────────────────────────────
    public class UserRegistrationForm
    {
        public string FirstName { get; set; } = string.Empty;
        public string LastName { get; set; } = string.Empty;
        public string Email { get; set; } = string.Empty;
        public string Phone { get; set; } = string.Empty;

        // Validator
        public bool IsValid =>
            !string.IsNullOrEmpty(FirstName) &&
            !string.IsNullOrEmpty(LastName) &&
            Email.Contains("@");

        // Snapshot al
        public FormMemento Save()
        {
            Console.WriteLine("📸 Form snapshot alındı");
            return new FormMemento(FirstName, LastName, Email, Phone);
        }

        // Snapshot'tan geri yükle
        public void Restore(FormMemento memento)
        {
            FirstName = memento.FirstName;
            LastName = memento.LastName;
            Email = memento.Email;
            Phone = memento.Phone;
            Console.WriteLine($"⏪ Form geri yüklendi: {memento}");
        }

        // Formu doldur (simüle edilmiş kullanıcı girişi)
        public void FillField(string fieldName, string value)
        {
            switch (fieldName.ToLower())
            {
                case "firstname": FirstName = value; break;
                case "lastname": LastName = value; break;
                case "email": Email = value; break;
                case "phone": Phone = value; break;
            }
            Console.WriteLine($"✏️  {fieldName}: '{value}'");
        }

        public void Display()
        {
            Console.WriteLine($"""
            
            ┌─────────────────────────────────┐
            │ 📝 Kullanıcı Kayıt Formu        │
            ├─────────────────────────────────┤
            │ Ad     : {FirstName,-25}│
            │ Soyad  : {LastName,-25}│
            │ E-posta: {Email,-25}│
            │ Telefon: {Phone,-25}│
            ├─────────────────────────────────┤
            │ Geçerli: {(IsValid ? "✅ Evet" : "❌ Hayır"),-25}│
            └─────────────────────────────────┘
            """);
        }
    }

    // ─────────────────────────────────────────────
    // CARETAKER: Form geçmişi yöneticisi
    // ─────────────────────────────────────────────
    public class FormHistoryManager
    {
        private readonly Stack<FormMemento> _undoStack = new();
        private readonly Stack<FormMemento> _redoStack = new();
        private readonly UserRegistrationForm _form;
        private const int MaxHistorySize = 20;

        public FormHistoryManager(UserRegistrationForm form)
        {
            _form = form;
        }

        public bool CanUndo => _undoStack.Count > 0;
        public bool CanRedo => _redoStack.Count > 0;

        public void SaveState()
        {
            if (_undoStack.Count >= MaxHistorySize)
            {
                // En eski kaydı at (Stack'ten dönüştürme gerekiyor)
                var items = new List<FormMemento>(_undoStack);
                items.RemoveAt(items.Count - 1);
                _undoStack.Clear();
                items.Reverse();
                items.ForEach(i => _undoStack.Push(i));
            }

            _undoStack.Push(_form.Save());
            _redoStack.Clear(); // Yeni değişiklik → redo geçmişi sıfırla
        }

        public bool Undo()
        {
            if (!CanUndo)
            {
                Console.WriteLine("⚠️  Geri alınacak işlem yok!");
                return false;
            }

            _redoStack.Push(_form.Save()); // Mevcut durumu redo'ya kaydet
            _form.Restore(_undoStack.Pop());
            return true;
        }

        public bool Redo()
        {
            if (!CanRedo)
            {
                Console.WriteLine("⚠️  İleri alınacak işlem yok!");
                return false;
            }

            _undoStack.Push(_form.Save());
            _form.Restore(_redoStack.Pop());
            return true;
        }

        public void Reset()
        {
            if (_undoStack.TryPeek(out var oldest))
            {
                // Stack'in en altındaki (ilk kayıt) state'e git
                while (_undoStack.Count > 1)
                    _redoStack.Push(_undoStack.Pop());

                _form.Restore(_undoStack.Pop());
                Console.WriteLine("🔄 Form ilk haline getirildi");
            }
        }

        public void ShowHistory()
        {
            Console.WriteLine($"\n📚 Geçmiş - Undo: {_undoStack.Count} | Redo: {_redoStack.Count}");
            int i = 1;
            foreach (var m in _undoStack)
                Console.WriteLine($"  {i++}. {m}");
        }
    }

    // ─────────────────────────────────────────────
    // DEMO
    // ─────────────────────────────────────────────
    class Program
    {
        static void Main()
        {
            Console.WriteLine("═══ FORM UNDO/REDO - MEMENTO PATTERN ═══\n");

            var form = new UserRegistrationForm();
            var history = new FormHistoryManager(form);

            // Başlangıç state'ini kaydet
            history.SaveState();

            // Kullanıcı formu dolduruyor
            form.FillField("firstname", "Ahmet");
            history.SaveState();

            form.FillField("lastname", "Yılmaz");
            history.SaveState();

            form.FillField("email", "ahmet@example.com");
            history.SaveState();

            form.FillField("phone", "0532 123 45 67");
            form.Display();
            history.ShowHistory();

            // Undo işlemleri
            Console.WriteLine("\n--- 2 ADIM GERİ AL ---");
            history.Undo();
            history.Undo();
            form.Display();

            // Redo
            Console.WriteLine("--- 1 ADIM İLERİ AL ---");
            history.Redo();
            form.Display();
        }
    }
}
```

---

## ✅ Avantajlar

| # | Avantaj | Açıklama |
|---|---------|----------|
| 1️⃣ | **Kapsülleme Korunur** | Originator'ın iç durumu dışarıya sızdırılmaz |
| 2️⃣ | **Temiz Undo/Redo** | Geri alma mekanizması net bir soyutlamayla kurulur |
| 3️⃣ | **Caretaker Bağımsızlığı** | Caretaker ne sakladığını bilmek zorunda değildir |
| 4️⃣ | **Tek Sorumluluk** | Her sınıf kendi görevine odaklanır |
| 5️⃣ | **Snapshot Güvenliği** | Memento immutable yapılabilir, bozulmaz |
| 6️⃣ | **Basit Rollback** | Transaction hatalarında kolay geri dönüş |

---

## ❌ Dezavantajlar

| # | Dezavantaj | Çözüm Önerisi |
|---|------------|---------------|
| ⚠️ | **Bellek Tüketimi** | State büyükse her snapshot maliyetlidir | Snapshot limiti koyun, eski kayıtları silin |
| ⚠️ | **Derin Kopyalama** | Referans tipler derin kopyalanmalıdır | `deepcopy` / `clone` kullanın |
| ⚠️ | **Caretaker Yükü** | Çok sayıda Memento yönetimi karmaşıklaşır | TTL veya max limit belirleyin |
| ⚠️ | **Seri Hale Getirme** | Büyük state'leri disk/ağa yazmak yavaştır | Delta/diff tabanlı snapshot kullanın |
| ⚠️ | **Dil Kısıtları** | Bazı dillerde paket-level erişim zor olabilir | Inner class veya friend class kullanın |

### 💡 Bellek Optimizasyon Stratejileri

```
Tam Snapshot          Delta (Fark) Snapshot       Hibrit
─────────────         ─────────────────────        ──────────────────
[S1: Full]            [S1: Full]                   [S1: Full]     ←─ Checkpoint
[S2: Full]            [S2: +3 chars at pos 5]      [S2: +delta]
[S3: Full]            [S3: -1 char at pos 8]       [S3: +delta]
[S4: Full]            [S4: +10 chars at end]       [S4: +delta]
                                                   [S5: Full]     ←─ Checkpoint
Memory: O(n*size)     Memory: O(n*change)          Memory: O(checkpoint + deltas)
```

---

## 🔗 SOLID Prensipleri ile İlişkisi

```
SOLID PRENSİPLERİ ve MEMENTO PATTERN
═══════════════════════════════════════════════════════════════
```

| Prensip | İlişki | Açıklama |
|---------|--------|----------|
| **S** - Single Responsibility | ✅ **Destekler** | Originator iş mantığı + state yönetimi; Caretaker sadece history yönetimi |
| **O** - Open/Closed | ✅ **Destekler** | Yeni snapshot tipi eklemek için mevcut kodu değiştirmek gerekmez |
| **L** - Liskov Substitution | ➡️ **Nötr** | Memento interface'i soyutlanırsa türetilen sınıflar kullanılabilir |
| **I** - Interface Segregation | ✅ **Destekler** | Caretaker yalnızca ihtiyaç duyduğu metotlara (save/restore) erişir |
| **D** - Dependency Inversion | ✅ **Destekler** | Caretaker somut Memento değil, soyut Memento interface'ine bağlıdır |

### 🔒 Kapsülleme (Encapsulation) ile İlişkisi

Memento, **kapsüllemenin en güçlü savunucularından** biridir:

```
❌ KÖTÜ YÖNTEM (Kapsüllemeyi bozar):
─────────────────────────────────────
Caretaker → state = originator.getState() // State doğrudan ifşa!
Caretaker → originator.setState(state)    // İç yapı maruz kalıyor!

✅ MEMENTO YÖNTEMI (Kapsülleme korunur):
─────────────────────────────────────────
Caretaker → memento = originator.save()   // Black box!
Caretaker → originator.restore(memento)   // İç yapı gizli!
```

---

## ⚖️ Diğer Desenlerle Karşılaştırma

### 🔄 Memento vs Command Pattern

| Özellik | Memento | Command |
|---------|---------|---------|
| **Undo Yöntemi** | State snapshot'ı geri yükler | Ters işlemi çalıştırır (`unexecute()`) |
| **Bellek** | Her state'in tam kopyası | Yalnızca işlem parametreleri |
| **Karmaşıklık** | Basit, state-based | Karmaşık, action-based |
| **Kullanım** | Büyük state değişimleri | Atomik, tersine çevrilebilir işlemler |
| **Birlikte** | ✅ Command + Memento = Güçlü undo | - |

```
Command Undo:                    Memento Undo:
─────────────                    ─────────────
[Cmd: addText("Hi")]             [State: "Hello World"]
  → unexecute: removeText("Hi")    → restore("Hello World")

Avantaj: Az bellek               Avantaj: Her durumu geri alabilir
Dezavantaj: Her cmd tersine      Dezavantaj: Büyük bellek kullanımı
            çevrilebilir olmalı
```

---

### 🔄 Memento vs State Pattern

| Özellik | Memento | State |
|---------|---------|-------|
| **Amaç** | Geçmiş duruma dönmek | Davranışı duruma göre değiştirmek |
| **Odak** | **Geçmiş yönetimi** | **Mevcut davranış** |
| **Durum Sayısı** | Dinamik, sınırsız snapshot | Önceden tanımlı, sonlu durumlar |
| **Caretaker** | Gerekli | Yok |

---

### 🔄 Memento vs Prototype Pattern

| Özellik | Memento | Prototype |
|---------|---------|-----------|
| **Amaç** | State kaydetme/geri yükleme | Nesne klonlama |
| **Erişim** | Kapsülleme korunur | Clone dışarıdan erişilebilir |
| **Kullanım** | Undo mekanizması | Nesne yaratma optimizasyonu |
| **Benzerlik** | Her ikisi de kopyalama içerir | - |

---

### 🔄 Karmaşık Senaryo: Command + Memento

```
En güçlü kombinasyon: Her Command işlemi öncesinde Memento alınır

┌─────────────┐    execute()    ┌───────────────┐
│   Client    │────────────────▶│    Command    │
└─────────────┘                 └───────┬───────┘
                                        │ 1. backup()
                                        ▼
                                ┌───────────────┐
                                │   Caretaker   │
                                │  (Memento     │
                                │   Manager)   │
                                └───────┬───────┘
                                        │ 2. execute()
                                        ▼
                                ┌───────────────┐
                                │  Originator   │
                                │  (değişir)    │
                                └───────────────┘
```

---

## 🎓 Mülakat Soruları

### 🟢 Başlangıç Seviyesi

> **S1: Memento Pattern nedir ve hangi problemi çözer?**

✅ **Cevap:** Memento, bir nesnenin iç durumunu dışarıya sızdırmadan kaydetmeye ve geri yüklemeye olanak tanıyan davranışsal bir tasarım desenidir. Temel problem: bir nesneyi önceki durumuna döndürme ihtiyacı, ancak bunu yaparken kapsülleme ilkesini korumak.

---

> **S2: Memento Pattern'ın 3 temel bileşeni nelerdir?**

✅ **Cevap:** 
- **Originator:** Durumu kaydedilen/yüklenen asıl nesne
- **Memento:** Originator'ın iç durumunun snapshot'ı (immutable)
- **Caretaker:** Memento nesnelerini saklayan ve yöneten nesne

---

### 🟡 Orta Seviye

> **S3: Caretaker neden Memento'nun içeriğine erişmemelidir?**

✅ **Cevap:** Caretaker'ın Memento'nun içeriğine erişmesi, Originator'ın iç yapısını ifşa eder ve kapsülleme ilkesini bozar. Caretaker yalnızca Memento'yu "saklamalı" ve "iletmeli", içeriğini analiz etmemelidir. Bu, *bilgi gizleme (information hiding)* prensibinin doğal bir uygulamasıdır.

---

> **S4: Memento'yu immutable (değiştirilemez) yapmak neden önemlidir?**

✅ **Cevap:** Mutable bir Memento, Caretaker veya başka sınıflar tarafından yanlışlıkla değiştirilebilir. Bu, geçmişin bozulmasına neden olur. Immutable Memento, snapshot'ın bütünlüğünü garanti eder. Java'da `final` alanlar, C#'ta `sealed` + `init`, Python'da `@dataclass(frozen=True)` kullanılabilir.

---

### 🔴 İleri Seviye

> **S5: Büyük state nesneleri için Memento'nun bellek verimliliğini nasıl artırırsınız?**

✅ **Cevap:** Üç yöntem:
1. **Delta/Diff Snapshot:** Tüm state yerine yalnızca değişen kısmı sakla
2. **Checkpoint + Delta Hibrit:** Belirli aralıklarla tam snapshot, arada deltalar
3. **Lazy Serialization:** Snapshot'ı hemen oluşturmak yerine, geri yüklendiğinde hesapla
4. **LRU Cache / Zaman Kısıtı:** Eski snapshot'ları otomatik sil

---

> **S6: Memento ile Command Pattern'ı birlikte nasıl kullanırsınız? Hangisi ne zaman tek başına yetersiz kalır?**

✅ **Cevap:**
- **Command tek başına:** Her işlemin tersine çevrilebilir olmasını gerektirir. Bazı işlemler (dosya silme, ağ isteği) tersine çevrilemez. Çözüm: Bu komutlar öncesinde Memento ile state al.
- **Memento tek başına:** Büyük state değişimlerinde bellek sorunları. Çözüm: Command ile sadece değişen kısımları logla.
- **Birlikte:** Her Command.execute() öncesinde Caretaker.backup() çağrılır. Undo ise Command.unexecute() yerine Memento.restore() kullanır.

---

> **S7: Thread-safe bir Memento implementasyonu nasıl yazarsınız?**

✅ **Cevap:** 
```java
// Synchronized snapshot al
public synchronized EditorMemento save() {
    return new EditorMemento(
        new String(content), // Defensive copy
        cursorPosition
    );
}

// Synchronized geri yükle
public synchronized void restore(EditorMemento m) {
    this.content = new StringBuilder(m.getContent());
    this.cursorPosition = m.getCursorPosition();
}
```
Ayrıca Caretaker'da `ConcurrentLinkedDeque` veya `synchronized` bloklar kullanılmalıdır.

---

> **S8: Memento Pattern ile Event Sourcing arasındaki fark nedir?**

✅ **Cevap:**

| | Memento | Event Sourcing |
|-|---------|----------------|
| **Saklanan** | State snapshot | Her olayın log'u |
| **Replay** | Doğrudan state yükle | Tüm olayları baştan oynat |
| **Ölçek** | Küçük/orta sistemler | Büyük, dağıtık sistemler |
| **Audit Trail** | Yok | Tam geçmiş kaydı |

---

## 📊 Özet Tablosu

```
┌──────────────────────────────────────────────────────────────────┐
│                    MEMENTO PATTERN - ÖZET                        │
├────────────────────┬─────────────────────────────────────────────┤
│ Kategori           │ Davranışsal (Behavioral)                    │
├────────────────────┼─────────────────────────────────────────────┤
│ Diğer Adlar        │ Token, Snapshot                             │
├────────────────────┼─────────────────────────────────────────────┤
│ Bileşenler         │ Originator, Memento, Caretaker              │
├────────────────────┼─────────────────────────────────────────────┤
│ Temel Amaç         │ Kapsüllemeyi bozmadan state kaydet/yükle    │
├────────────────────┼─────────────────────────────────────────────┤
│ En İyi Kullanım    │ Undo/Redo, Oyun kayıt, Transaction rollback │
├────────────────────┼─────────────────────────────────────────────┤
│ Ana Risk           │ Bellek tüketimi (büyük state'lerde)         │
├────────────────────┼─────────────────────────────────────────────┤
│ SOLID Uyumu        │ SRP ✅ OCP ✅ ISP ✅ DIP ✅                  │
├────────────────────┼─────────────────────────────────────────────┤
│ İlgili Desenler    │ Command, State, Prototype, Iterator         │
└────────────────────┴─────────────────────────────────────────────┘
```

---

## 📚 Kaynaklar

- 📗 **Design Patterns: Elements of Reusable OO Software** — Gang of Four (GoF)
- 📘 **Head First Design Patterns** — Freeman & Robson (O'Reilly)
- 📙 **Refactoring to Patterns** — Joshua Kerievsky
- 🌐 [Refactoring Guru - Memento](https://refactoring.guru/design-patterns/memento)
- 🌐 [SourceMaking - Memento](https://sourcemaking.com/design_patterns/memento)
- 🌐 [Microsoft Docs - Design Patterns](https://docs.microsoft.com/en-us/azure/architecture/patterns/)

---

<div align="center">

**⭐ Faydalı bulduysan yıldız vermeyi unutma!**

[![GitHub stars](https://img.shields.io/github/stars/?style=social)](.)
[![GitHub forks](https://img.shields.io/github/forks/?style=social)](.)


</div>
