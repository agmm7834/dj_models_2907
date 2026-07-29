# Django `models.py` — Noldan Model Yaratishni O'rganamiz

> Bu qo'llanma Django'ni endi boshlagan talabalar uchun yozilgan. Har bir tushuncha
> **sodda til** va **ko'p kod misollari** bilan tushuntirilgan. Ketma-ket o'qing va
> har bir kodni o'zingiz yozib ko'ring — faqat o'qish bilan o'rganilmaydi.

---

## Mundarija

1. [Model nima?](#1-model-nima)
2. [Birinchi modelimiz](#2-birinchi-modelimiz)
3. [Maydon (field) turlari](#3-maydon-field-turlari)
4. [`max_length` — matn uzunligi](#4-max_length--matn-uzunligi)
5. [`default` — standart qiymat](#5-default--standart-qiymat)
6. [`null` va `blank` — eng ko'p chalkashtiriladigan juftlik](#6-null-va-blank--eng-kop-chalkashtiriladigan-juftlik)
7. [`auto_now_add` va `auto_now` — vaqtni avtomatik yozish](#7-auto_now_add-va-auto_now--vaqtni-avtomatik-yozish)
8. [`choices` — tayyor variantlar ro'yxati](#8-choices--tayyor-variantlar-royxati)
9. [`unique`, `verbose_name`, `help_text`, `db_index`](#9-unique-verbose_name-help_text-db_index)
10. [Bog'lanishlar: `ForeignKey`, `OneToOneField`, `ManyToManyField`](#10-boglanishlar)
11. [`related_name` — teskari tomonga nom berish](#11-related_name--teskari-tomonga-nom-berish)
12. [`on_delete` — bog'langan obyekt o'chsa nima bo'ladi?](#12-on_delete--boglangan-obyekt-ochsa-nima-boladi)
13. [`__str__` metodi](#13-__str__-metodi)
14. [`class Meta` — model sozlamalari](#14-class-meta--model-sozlamalari)
15. [Migratsiya — modelni bazaga yuborish](#15-migratsiya--modelni-bazaga-yuborish)
16. [Shell'da sinab ko'ramiz](#16-shellda-sinab-koramiz)
17. [To'liq loyiha misoli: Blog](#17-toliq-loyiha-misoli-blog)
18. [Tez-tez uchraydigan xatolar](#18-tez-tez-uchraydigan-xatolar)
19. [Shpargalka](#19-shpargalka)

---

## 1. Model nima?

**Model — bu ma'lumotlar bazasidagi jadvalning Python ko'rinishi.**

Oddiy qilib aytganda: siz SQL yozishni bilmasangiz ham bo'ladi. Siz Python klassi
yozasiz, Django esa uni jadvalga aylantiradi.

Taqqoslab ko'ring:

**SQL'da shunday yozilardi:**

```sql
CREATE TABLE student (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    full_name VARCHAR(100) NOT NULL,
    age INTEGER NOT NULL
);
```

**Django'da esa shunchaki:**

```python
from django.db import models


class Student(models.Model):
    full_name = models.CharField(max_length=100)
    age = models.IntegerField()
```

Ikkalasi ham bir xil natija beradi. Ammo ikkinchisi ancha oson, to'g'rimi?

### Tushunish uchun jadval

| Django | Ma'lumotlar bazasi |
|--------|--------------------|
| Klass (`class Student`) | Jadval (`student`) |
| Maydon (`full_name = ...`) | Ustun (`full_name` column) |
| Obyekt (`Student(...)`) | Qator (row) |

---

## 2. Birinchi modelimiz

Model `models.py` faylida yoziladi. Odatda bu fayl ilova (app) papkasida turadi:

```
myproject/
├── manage.py
├── myproject/
│   └── settings.py
└── blog/              ← bizning ilovamiz
    ├── models.py      ← modellar shu yerda!
    ├── views.py
    └── admin.py
```

`blog/models.py`:

```python
from django.db import models


class Student(models.Model):
    full_name = models.CharField(max_length=100)
    age = models.IntegerField()
    email = models.EmailField()
```

### Kod nimani anglatadi?

```python
from django.db import models       # 1. models modulini import qilamiz

class Student(models.Model):       # 2. models.Model'dan meros olamiz — MAJBURIY!
    full_name = models.CharField(  # 3. maydon nomi = maydon turi(sozlamalar)
        max_length=100
    )
```

> ⚠️ **Muhim:** `models.Model`dan meros olishni unutmang. Aks holda Django buni
> model deb tanimaydi va jadval yaratilmaydi.

### `id` maydonini yozmadikmi?

Yozmadik — chunki **Django uni o'zi qo'shadi**. Har bir modelda avtomatik
`id` degan maydon paydo bo'ladi (1, 2, 3, ... deb o'sib boradi).

Ya'ni yuqoridagi model aslida shunday:

```python
class Student(models.Model):
    # id = models.AutoField(primary_key=True)   ← Django buni o'zi qo'shadi
    full_name = models.CharField(max_length=100)
    age = models.IntegerField()
    email = models.EmailField()
```

---

## 3. Maydon (field) turlari

Django'da juda ko'p maydon turi bor. Mana eng ko'p ishlatiladiganlari:

### 3.1. Matn uchun

```python
class TextExamples(models.Model):
    # Qisqa matn — ism, sarlavha, telefon raqam
    title = models.CharField(max_length=200)

    # Uzun matn — maqola matni, izoh. max_length shart emas!
    body = models.TextField()

    # Email — Django formatni tekshiradi
    email = models.EmailField()

    # Havola (link)
    website = models.URLField()

    # URL uchun chiroyli manzil: "mening-birinchi-maqolam"
    slug = models.SlugField(max_length=200)
```

**`CharField` va `TextField` farqi:**

| | `CharField` | `TextField` |
|---|---|---|
| Uzunlik | Cheklangan (`max_length` majburiy) | Cheksiz |
| Admin panelda | Bitta qatorli input | Katta textarea |
| Qachon ishlatiladi | Ism, sarlavha, telefon | Maqola, izoh, tavsif |

### 3.2. Raqamlar uchun

```python
class NumberExamples(models.Model):
    # Butun son: 1, 42, -7
    age = models.IntegerField()

    # Faqat musbat butun son: 0, 1, 42 (manfiy bo'lolmaydi)
    view_count = models.PositiveIntegerField()

    # Katta son (masalan, million+)
    population = models.BigIntegerField()

    # Kasrli son — PUL UCHUN SHUNI ISHLATING!
    price = models.DecimalField(max_digits=10, decimal_places=2)
    # max_digits=10  → jami 10 ta raqam
    # decimal_places=2 → shundan 2 tasi verguldan keyin
    # Ya'ni: 99999999.99 gacha

    # Kasrli son (ilmiy hisob-kitob uchun)
    temperature = models.FloatField()
```

> 💡 **Maslahat:** Pul uchun **hech qachon** `FloatField` ishlatmang! `FloatField`
> yaxlitlash xatoliklariga olib keladi (`0.1 + 0.2 = 0.30000000000000004`).
> Pul uchun har doim `DecimalField`.

### 3.3. Sana va vaqt uchun

```python
class DateExamples(models.Model):
    birth_date = models.DateField()          # faqat sana: 2005-03-15
    created_at = models.DateTimeField()      # sana + vaqt: 2005-03-15 14:30:00
    lesson_time = models.TimeField()         # faqat vaqt: 14:30:00
    duration = models.DurationField()        # davomiylik: 2 soat 30 daqiqa
```

### 3.4. Ha/Yo'q uchun

```python
class BooleanExamples(models.Model):
    is_active = models.BooleanField(default=True)      # True yoki False
    is_published = models.BooleanField(default=False)
```

### 3.5. Fayl va rasm uchun

```python
class FileExamples(models.Model):
    # Rasm — Pillow kutubxonasi kerak: pip install Pillow
    avatar = models.ImageField(upload_to='avatars/')

    # Har qanday fayl
    resume = models.FileField(upload_to='documents/')
```

`upload_to='avatars/'` — fayl `media/avatars/` papkasiga saqlanadi.

### Barcha turlar bir joyda

```python
from django.db import models


class AllFieldTypes(models.Model):
    """Maydon turlarini eslab qolish uchun bitta model."""

    # --- Matn ---
    char_field = models.CharField(max_length=100)
    text_field = models.TextField()
    email_field = models.EmailField()
    url_field = models.URLField()
    slug_field = models.SlugField()

    # --- Raqam ---
    integer_field = models.IntegerField()
    positive_field = models.PositiveIntegerField()
    decimal_field = models.DecimalField(max_digits=10, decimal_places=2)
    float_field = models.FloatField()

    # --- Sana / vaqt ---
    date_field = models.DateField()
    datetime_field = models.DateTimeField()
    time_field = models.TimeField()

    # --- Mantiqiy ---
    boolean_field = models.BooleanField(default=False)

    # --- Fayl ---
    image_field = models.ImageField(upload_to='images/')
    file_field = models.FileField(upload_to='files/')
```

---

## 4. `max_length` — matn uzunligi

`max_length` — matn maydoni **maksimal nechta belgi** sig'dira olishini bildiradi.

```python
class Person(models.Model):
    first_name = models.CharField(max_length=50)
```

Bu degani: `first_name` ga eng ko'pi bilan **50 ta belgi** yozish mumkin.

### `CharField` uchun `max_length` MAJBURIY

```python
# ❌ XATO — ishlamaydi!
name = models.CharField()

# ✅ TO'G'RI
name = models.CharField(max_length=100)
```

Agar `max_length` yozmasangiz, Django shunday xato beradi:

```
ERRORS:
blog.Person.name: (fields.E120) CharFields must define a 'max_length' attribute.
```

### `TextField` uchun `max_length` SHART EMAS

```python
# ✅ To'g'ri — TextField'ga max_length kerak emas
content = models.TextField()
```

### Amaliy misollar

```python
class Contact(models.Model):
    # Ism-familiya uchun 100 ta yetarli
    full_name = models.CharField(max_length=100)

    # Telefon: +998901234567 → 13 ta belgi, 20 ta xotirjam
    phone = models.CharField(max_length=20)

    # Maqola sarlavhasi — uzunroq bo'lishi mumkin
    title = models.CharField(max_length=250)

    # Mamlakat kodi: UZ, US, RU → 2 ta yetarli
    country_code = models.CharField(max_length=2)

    # Parol hash — juda uzun bo'ladi
    password_hash = models.CharField(max_length=255)
```

### Nima bo'ladi agar chegaradan oshsa?

```python
# Model: name = models.CharField(max_length=5)

Person.objects.create(name="Abdulla")   # 7 ta belgi — 5 dan ko'p!
# → Xatolik beradi (baza turiga qarab)
```

Formalarda esa Django foydalanuvchiga chiroyli xabar ko'rsatadi:
*"Ensure this value has at most 5 characters."*

---

## 5. `default` — standart qiymat

`default` — agar qiymat berilmasa, **avtomatik qo'yiladigan qiymat**.

```python
class Product(models.Model):
    name = models.CharField(max_length=100)
    price = models.DecimalField(max_digits=10, decimal_places=2, default=0)
    is_available = models.BooleanField(default=True)
    stock = models.PositiveIntegerField(default=0)
    description = models.TextField(default="Tavsif kiritilmagan")
```

Endi obyekt yaratganda:

```python
# Faqat name berdik
p = Product.objects.create(name="Noutbuk")

print(p.name)          # Noutbuk
print(p.price)         # 0            ← default ishladi
print(p.is_available)  # True         ← default ishladi
print(p.stock)         # 0            ← default ishladi
print(p.description)   # Tavsif kiritilmagan
```

`default`ni bekor qilish uchun shunchaki qiymat bering:

```python
p = Product.objects.create(
    name="Telefon",
    price=500,
    is_available=False,
)
print(p.price)         # 500          ← default o'rniga bizniki
print(p.is_available)  # False
```

### `default` bilan turli xil misollar

```python
from django.utils import timezone


class Order(models.Model):
    # Oddiy qiymatlar
    quantity = models.PositiveIntegerField(default=1)
    discount = models.DecimalField(max_digits=5, decimal_places=2, default=0.00)
    status = models.CharField(max_length=20, default="yangi")
    is_paid = models.BooleanField(default=False)

    # Bo'sh matn
    note = models.TextField(default="")
```

### ⚠️ Eng katta xato: `default` ga funksiyani chaqirib qo'yish

```python
# ❌ XATO — timezone.now() BIR MARTA, migratsiya paytida hisoblanadi.
#           Barcha yozuvlar bir xil vaqtga ega bo'ladi!
created_at = models.DateTimeField(default=timezone.now())

# ✅ TO'G'RI — qavssiz! Django funksiyani har safar o'zi chaqiradi.
created_at = models.DateTimeField(default=timezone.now)
```

Farqni yodda tuting:
- `timezone.now()` — **qiymat** (bitta aniq vaqt)
- `timezone.now` — **funksiya** (har safar yangi vaqt beradi)

### O'zgaruvchan (mutable) default — ro'yxat va lug'at

```python
# ❌ XATO — hamma obyekt bitta ro'yxatni bo'lishadi!
tags = models.JSONField(default=[])

# ✅ TO'G'RI — funksiya orqali
tags = models.JSONField(default=list)
settings = models.JSONField(default=dict)
```

---

## 6. `null` va `blank` — eng ko'p chalkashtiriladigan juftlik

Bu ikkisi eng ko'p savol tug'diradigan mavzu. Sodda qilib:

| Parametr | Kimga tegishli | Ma'nosi |
|----------|----------------|---------|
| `null=True` | **Ma'lumotlar bazasiga** | Ustunda `NULL` (bo'shlik) turishi mumkin |
| `blank=True` | **Formaga / admin panelga** | Maydonni to'ldirmasdan saqlash mumkin |

**Yodda tuting:**
- `null` → baza haqida
- `blank` → forma (validatsiya) haqida

### Misol bilan tushunamiz

```python
class Profile(models.Model):
    # 1. MAJBURIY maydon — bo'sh qoldirib bo'lmaydi
    username = models.CharField(max_length=50)

    # 2. Ixtiyoriy MATN — faqat blank=True
    bio = models.TextField(blank=True)

    # 3. Ixtiyoriy RAQAM/SANA — ikkalasi ham
    age = models.IntegerField(null=True, blank=True)
    birth_date = models.DateField(null=True, blank=True)
```

### Oltin qoida 🏆

> **Matn maydonlari (`CharField`, `TextField`) uchun `null=True` YOZMANG!**
> Faqat `blank=True` yozing.

Sababi: matn maydoni bo'sh bo'lganda ikki xil holat paydo bo'lardi —
`NULL` va `""` (bo'sh satr). Bu chalkashlik keltiradi. Django hujjatlari ham
shuni tavsiya qiladi.

```python
# ❌ Yomon
bio = models.TextField(null=True, blank=True)

# ✅ Yaxshi
bio = models.TextField(blank=True)
```

### To'liq taqqoslash jadvali

```python
class Example(models.Model):
    # ┌──────────────────────────────────────────────────────────┐
    # │ Hech narsa yozilmagan → MAJBURIY                          │
    # └──────────────────────────────────────────────────────────┘
    a = models.CharField(max_length=50)
    # Bazada: NOT NULL
    # Formada: to'ldirish shart

    # ┌──────────────────────────────────────────────────────────┐
    # │ Faqat blank=True → matn uchun ixtiyoriy                   │
    # └──────────────────────────────────────────────────────────┘
    b = models.CharField(max_length=50, blank=True)
    # Bazada: NOT NULL, lekin "" (bo'sh satr) saqlanadi
    # Formada: bo'sh qoldirish mumkin

    # ┌──────────────────────────────────────────────────────────┐
    # │ null=True + blank=True → raqam/sana uchun ixtiyoriy       │
    # └──────────────────────────────────────────────────────────┘
    c = models.IntegerField(null=True, blank=True)
    # Bazada: NULL bo'lishi mumkin
    # Formada: bo'sh qoldirish mumkin

    # ┌──────────────────────────────────────────────────────────┐
    # │ Faqat null=True → kamdan-kam kerak bo'ladi                │
    # └──────────────────────────────────────────────────────────┘
    d = models.IntegerField(null=True)
    # Bazada: NULL bo'lishi mumkin
    # Formada: HAR DOIM to'ldirish shart (chalkash!)
```

### Amaliy misol: talaba profili

```python
class Student(models.Model):
    # Majburiy — bularsiz talaba bo'lmaydi
    first_name = models.CharField(max_length=50)
    last_name = models.CharField(max_length=50)
    student_id = models.CharField(max_length=20)

    # Ixtiyoriy matnlar — blank=True
    middle_name = models.CharField(max_length=50, blank=True)
    address = models.TextField(blank=True)
    phone = models.CharField(max_length=20, blank=True)

    # Ixtiyoriy raqam/sana — null=True + blank=True
    gpa = models.DecimalField(
        max_digits=3,
        decimal_places=2,
        null=True,
        blank=True,
    )
    graduation_date = models.DateField(null=True, blank=True)

    # Ixtiyoriy rasm
    photo = models.ImageField(upload_to='students/', null=True, blank=True)
```

### Qiymatlarni tekshirish

```python
s = Student.objects.create(
    first_name="Ali",
    last_name="Valiyev",
    student_id="2024001",
)

print(s.middle_name)      # "" ← bo'sh satr (blank=True)
print(s.gpa)              # None ← NULL (null=True)
print(s.graduation_date)  # None

# Tekshirish
if not s.middle_name:
    print("Otasining ismi kiritilmagan")

if s.gpa is None:
    print("GPA hali hisoblanmagan")
```

---

## 7. `auto_now_add` va `auto_now` — vaqtni avtomatik yozish

Bu ikkisi `DateField` va `DateTimeField` uchun ishlatiladi.

| Parametr | Qachon yoziladi |
|----------|-----------------|
| `auto_now_add=True` | **Faqat bir marta** — obyekt birinchi yaratilganda |
| `auto_now=True` | **Har safar** — obyekt saqlangan sayin yangilanadi |

```python
class Post(models.Model):
    title = models.CharField(max_length=200)
    content = models.TextField()

    created_at = models.DateTimeField(auto_now_add=True)  # yaratilgan vaqt
    updated_at = models.DateTimeField(auto_now=True)      # oxirgi tahrir vaqti
```

### Amalda qanday ishlaydi?

```python
# 1. Yaratamiz
post = Post.objects.create(title="Salom", content="Birinchi post")

print(post.created_at)   # 2026-07-29 10:00:00
print(post.updated_at)   # 2026-07-29 10:00:00   ← ikkalasi bir xil

# 2. Bir soatdan keyin tahrirlaymiz
post.title = "Salom Dunyo"
post.save()

print(post.created_at)   # 2026-07-29 10:00:00   ← O'ZGARMADI!
print(post.updated_at)   # 2026-07-29 11:00:00   ← YANGILANDI!
```

### ⚠️ Muhim: bu maydonlarni qo'lda o'zgartirib bo'lmaydi

```python
from datetime import datetime

post = Post.objects.create(
    title="Test",
    content="...",
    created_at=datetime(2020, 1, 1),   # ← E'TIBORSIZ QOLDIRILADI!
)

print(post.created_at)   # 2026-07-29 ... (hozirgi vaqt)
```

`auto_now_add` va `auto_now` maydonlari:
- Admin panelda **tahrirlab bo'lmaydi** (avtomatik `editable=False` bo'ladi)
- Formalarda **ko'rinmaydi**

### Agar vaqtni qo'lda ham o'zgartirmoqchi bo'lsangiz?

`default=timezone.now` ishlating:

```python
from django.utils import timezone


class Event(models.Model):
    title = models.CharField(max_length=200)

    # Avtomatik hozirgi vaqt, LEKIN kerak bo'lsa o'zgartirsa bo'ladi
    starts_at = models.DateTimeField(default=timezone.now)
```

```python
# Standart holat
e1 = Event.objects.create(title="Uchrashuv")
print(e1.starts_at)   # hozirgi vaqt

# Qo'lda o'zgartirish — ISHLAYDI!
e2 = Event.objects.create(
    title="Konferensiya",
    starts_at=timezone.datetime(2026, 12, 31, 18, 0),
)
print(e2.starts_at)   # 2026-12-31 18:00:00
```

### Uchtasining farqi bir jadvalda

| | `auto_now_add=True` | `auto_now=True` | `default=timezone.now` |
|---|---|---|---|
| Yaratilganda yoziladi | ✅ | ✅ | ✅ |
| Saqlanganda yangilanadi | ❌ | ✅ | ❌ |
| Qo'lda o'zgartirsa bo'ladi | ❌ | ❌ | ✅ |
| Admin panelda ko'rinadi | ❌ | ❌ | ✅ |

### Deyarli har bir modelda ishlatiladigan shablon

```python
class Article(models.Model):
    title = models.CharField(max_length=200)
    body = models.TextField()

    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)


class Comment(models.Model):
    text = models.TextField()

    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
```

> 💡 Bu juftlikni **har bir modelga** qo'shishni odat qiling. Keyinchalik
> "bu yozuv qachon qo'shilgan edi?" degan savol albatta paydo bo'ladi.

---

## 8. `choices` — tayyor variantlar ro'yxati

Ba'zan maydon faqat ma'lum qiymatlarni qabul qilishi kerak. Masalan, holat:
"yangi", "jarayonda", "tugallangan".

```python
class Task(models.Model):
    STATUS_CHOICES = [
        ('new', 'Yangi'),
        ('in_progress', 'Jarayonda'),
        ('done', 'Tugallangan'),
    ]

    title = models.CharField(max_length=200)
    status = models.CharField(
        max_length=20,
        choices=STATUS_CHOICES,
        default='new',
    )
```

Har bir variant ikkitalik:
- Birinchisi (`'new'`) — **bazada saqlanadigan** qiymat
- Ikkinchisi (`'Yangi'`) — **foydalanuvchi ko'radigan** matn

```python
t = Task.objects.create(title="Uy vazifasi")

print(t.status)                  # new         ← bazadagi qiymat
print(t.get_status_display())    # Yangi       ← chiroyli ko'rinishi
```

> `get_<maydon_nomi>_display()` — Django avtomatik yaratadigan metod.

### Zamonaviy usul: `TextChoices`

Django 3.0+ da chiroyliroq yozish mumkin:

```python
class Task(models.Model):
    class Status(models.TextChoices):
        NEW = 'new', 'Yangi'
        IN_PROGRESS = 'in_progress', 'Jarayonda'
        DONE = 'done', 'Tugallangan'

    title = models.CharField(max_length=200)
    status = models.CharField(
        max_length=20,
        choices=Status.choices,
        default=Status.NEW,
    )
```

Foydalanish:

```python
# Yaratish
t = Task.objects.create(title="Loyiha", status=Task.Status.IN_PROGRESS)

# Solishtirish — matn yozmaymiz, xato qilish ehtimoli kam!
if t.status == Task.Status.DONE:
    print("Tugallangan!")

# Filtrlash
Task.objects.filter(status=Task.Status.NEW)
```

### Yana misollar

```python
class Product(models.Model):
    class Category(models.TextChoices):
        PHONE = 'phone', 'Telefon'
        LAPTOP = 'laptop', 'Noutbuk'
        TABLET = 'tablet', 'Planshet'

    class Size(models.TextChoices):
        SMALL = 'S', 'Kichik'
        MEDIUM = 'M', "O'rta"
        LARGE = 'L', 'Katta'

    name = models.CharField(max_length=100)
    category = models.CharField(max_length=20, choices=Category.choices)
    size = models.CharField(max_length=1, choices=Size.choices, default=Size.MEDIUM)


class Student(models.Model):
    class Grade(models.IntegerChoices):   # raqamlar uchun
        FRESHMAN = 1, "1-kurs"
        SOPHOMORE = 2, "2-kurs"
        JUNIOR = 3, "3-kurs"
        SENIOR = 4, "4-kurs"

    name = models.CharField(max_length=100)
    grade = models.IntegerField(choices=Grade.choices, default=Grade.FRESHMAN)
```

---

## 9. `unique`, `verbose_name`, `help_text`, `db_index`

### `unique=True` — takrorlanmas qiymat

```python
class User(models.Model):
    username = models.CharField(max_length=50, unique=True)
    email = models.EmailField(unique=True)
    phone = models.CharField(max_length=20, unique=True, blank=True)
```

```python
User.objects.create(username="ali", email="ali@mail.uz")
User.objects.create(username="ali", email="vali@mail.uz")
# → IntegrityError: UNIQUE constraint failed
```

### `verbose_name` — maydonning chiroyli nomi

Admin panelda va formalarda ko'rinadigan nom:

```python
class Book(models.Model):
    title = models.CharField(max_length=200, verbose_name="Kitob nomi")
    author = models.CharField(max_length=100, verbose_name="Muallif")
    pages = models.PositiveIntegerField(verbose_name="Sahifalar soni")
    published_year = models.PositiveIntegerField(verbose_name="Nashr yili")
```

`verbose_name` bo'lmasa, Django maydon nomini o'zi chiroyli qiladi:
`published_year` → "Published year".

Birinchi pozitsion argument sifatida ham yozish mumkin:

```python
title = models.CharField("Kitob nomi", max_length=200)   # bir xil natija
```

### `help_text` — foydalanuvchiga izoh

Forma ostida kichkina yozuv sifatida chiqadi:

```python
class Product(models.Model):
    name = models.CharField(
        max_length=100,
        help_text="Mahsulot nomini to'liq kiriting",
    )
    price = models.DecimalField(
        max_digits=10,
        decimal_places=2,
        help_text="Narxni so'mda kiriting, masalan: 150000.00",
    )
    sku = models.CharField(
        max_length=20,
        unique=True,
        help_text="Noyob kod, masalan: PRD-001",
    )
```

### `db_index=True` — qidiruvni tezlashtirish

Agar maydon bo'yicha tez-tez qidirsangiz (`filter`), index qo'shing:

```python
class Order(models.Model):
    order_number = models.CharField(max_length=50, db_index=True)
    status = models.CharField(max_length=20, db_index=True)
    created_at = models.DateTimeField(auto_now_add=True, db_index=True)
```

> `unique=True` qo'ysangiz, index avtomatik yaratiladi — alohida `db_index`
> yozish shart emas.

### `editable=False` — formada ko'rsatmaslik

```python
class Log(models.Model):
    message = models.TextField()
    ip_address = models.CharField(max_length=45, editable=False)
```

### Hammasi birga

```python
class Course(models.Model):
    code = models.CharField(
        max_length=10,
        unique=True,
        db_index=True,
        verbose_name="Kurs kodi",
        help_text="Masalan: CS101",
    )
    title = models.CharField(
        max_length=200,
        verbose_name="Kurs nomi",
    )
    description = models.TextField(
        blank=True,
        verbose_name="Tavsif",
        help_text="Kurs haqida qisqacha ma'lumot",
    )
    credits = models.PositiveIntegerField(
        default=3,
        verbose_name="Kreditlar soni",
    )
    is_active = models.BooleanField(
        default=True,
        verbose_name="Faol",
    )
    created_at = models.DateTimeField(auto_now_add=True)
```

---

## 10. Bog'lanishlar

Haqiqiy loyihalarda modellar bir-biri bilan bog'langan bo'ladi:
- Maqolaning **muallifi** bor
- Talabaning **guruhi** bor
- Mahsulotning **kategoriyasi** bor

Django'da 3 xil bog'lanish bor:

| Turi | Ma'nosi | Misol |
|------|---------|-------|
| `ForeignKey` | Ko'pdan → birga | Ko'p maqola → bitta muallif |
| `OneToOneField` | Birdan → birga | Bitta user → bitta profil |
| `ManyToManyField` | Ko'pdan → ko'pga | Ko'p talaba ↔ ko'p kurs |

### 10.1. `ForeignKey` — ko'pdan birga (eng ko'p ishlatiladi)

```python
class Author(models.Model):
    name = models.CharField(max_length=100)


class Book(models.Model):
    title = models.CharField(max_length=200)
    author = models.ForeignKey(
        Author,                      # 1. Qaysi modelga bog'lanamiz
        on_delete=models.CASCADE,    # 2. Muallif o'chsa nima bo'ladi (MAJBURIY)
        related_name='books',        # 3. Teskari tomondan murojaat nomi
    )
```

Endi:
- **Bitta muallif** — ko'p kitob yozishi mumkin ✅
- **Bitta kitob** — faqat bitta muallifga tegishli ✅

Ishlatish:

```python
# Muallif yaratamiz
author = Author.objects.create(name="Abdulla Qodiriy")

# Uning kitoblarini yaratamiz
Book.objects.create(title="O'tkan kunlar", author=author)
Book.objects.create(title="Mehrobdan chayon", author=author)

# Kitobdan muallifga
book = Book.objects.get(title="O'tkan kunlar")
print(book.author.name)          # Abdulla Qodiriy

# Muallifdan kitoblariga (related_name orqali!)
print(author.books.count())      # 2
for b in author.books.all():
    print(b.title)
# O'tkan kunlar
# Mehrobdan chayon
```

### 10.2. `OneToOneField` — birdan birga

```python
from django.contrib.auth.models import User


class Profile(models.Model):
    user = models.OneToOneField(
        User,
        on_delete=models.CASCADE,
        related_name='profile',
    )
    bio = models.TextField(blank=True)
    avatar = models.ImageField(upload_to='avatars/', null=True, blank=True)
    birth_date = models.DateField(null=True, blank=True)
```

Har bir foydalanuvchida **faqat bitta** profil bo'ladi:

```python
user = User.objects.create_user(username="ali", password="12345")
Profile.objects.create(user=user, bio="Talabaman")

# Ikki tomonga ham murojaat
print(user.profile.bio)          # Talabaman
print(profile.user.username)     # ali

# Ikkinchi profil yaratib bo'lmaydi!
Profile.objects.create(user=user, bio="Yana bitta")
# → IntegrityError
```

### 10.3. `ManyToManyField` — ko'pdan ko'pga

```python
class Course(models.Model):
    title = models.CharField(max_length=200)


class Student(models.Model):
    name = models.CharField(max_length=100)
    courses = models.ManyToManyField(
        Course,
        related_name='students',
        blank=True,             # M2M da null=True KERAK EMAS!
    )
```

- Bitta talaba — ko'p kursga yozilishi mumkin ✅
- Bitta kursda — ko'p talaba bo'lishi mumkin ✅

```python
math = Course.objects.create(title="Matematika")
physics = Course.objects.create(title="Fizika")

ali = Student.objects.create(name="Ali")

# Kurs qo'shish
ali.courses.add(math)
ali.courses.add(physics)
# yoki bir vaqtda:
ali.courses.add(math, physics)

# Ko'rish
print(ali.courses.count())              # 2
for c in ali.courses.all():
    print(c.title)

# Teskari tomondan
print(math.students.count())            # 1
for s in math.students.all():
    print(s.name)

# O'chirish
ali.courses.remove(physics)
print(ali.courses.count())              # 1

# Hammasini o'chirish
ali.courses.clear()

# Butunlay almashtirish
ali.courses.set([math, physics])
```

> ⚠️ `ManyToManyField` da **`null=True` yozmang** — hech qanday ta'siri yo'q.
> Faqat `blank=True` yozing (formada bo'sh qoldirish uchun).

### 10.4. O'ziga o'zi bog'lanish

Masalan, kategoriyaning ota-kategoriyasi bo'lishi mumkin:

```python
class Category(models.Model):
    name = models.CharField(max_length=100)
    parent = models.ForeignKey(
        'self',                  # ← o'ziga bog'lanish
        on_delete=models.CASCADE,
        null=True,
        blank=True,
        related_name='children',
    )
```

```python
elektronika = Category.objects.create(name="Elektronika")
telefonlar = Category.objects.create(name="Telefonlar", parent=elektronika)
noutbuklar = Category.objects.create(name="Noutbuklar", parent=elektronika)

print(elektronika.children.count())     # 2
print(telefonlar.parent.name)           # Elektronika
```

### 10.5. Model hali yozilmagan bo'lsa — satr sifatida

```python
class Book(models.Model):
    title = models.CharField(max_length=200)
    # Author klassi pastda yozilgan — matn sifatida yozamiz
    author = models.ForeignKey('Author', on_delete=models.CASCADE)


class Author(models.Model):
    name = models.CharField(max_length=100)
```

---

## 11. `related_name` — teskari tomonga nom berish

Bu — bog'lanishning **teskari tomonidan** murojaat qilish uchun ishlatiladigan nom.

### `related_name`siz

```python
class Author(models.Model):
    name = models.CharField(max_length=100)


class Book(models.Model):
    title = models.CharField(max_length=200)
    author = models.ForeignKey(Author, on_delete=models.CASCADE)
    # related_name yo'q!
```

Django avtomatik `<model_nomi_kichik_harflarda>_set` yaratadi:

```python
author = Author.objects.get(id=1)

print(author.book_set.all())        # ← chalkash nom
print(author.book_set.count())
```

### `related_name` bilan

```python
class Book(models.Model):
    title = models.CharField(max_length=200)
    author = models.ForeignKey(
        Author,
        on_delete=models.CASCADE,
        related_name='books',       # ← o'zimiz nom berdik
    )
```

```python
author = Author.objects.get(id=1)

print(author.books.all())           # ← chiroyli va tushunarli!
print(author.books.count())
print(author.books.filter(title__startswith="O"))
```

### Nomlash qoidasi

`related_name` — **teskari tomondan nima olinishini** bildiradi, shuning uchun
odatda **ko'plikda** yoziladi:

```python
class Comment(models.Model):
    post = models.ForeignKey(Post, on_delete=models.CASCADE,
                             related_name='comments')     # post.comments


class Order(models.Model):
    customer = models.ForeignKey(Customer, on_delete=models.CASCADE,
                                 related_name='orders')   # customer.orders


class Lesson(models.Model):
    course = models.ForeignKey(Course, on_delete=models.CASCADE,
                               related_name='lessons')    # course.lessons
```

`OneToOneField` uchun esa **birlikda**:

```python
class Profile(models.Model):
    user = models.OneToOneField(User, on_delete=models.CASCADE,
                                related_name='profile')   # user.profile
```

### `related_name` QACHON MAJBURIY bo'ladi?

Agar bitta modelda **bir xil modelga ikkita bog'lanish** bo'lsa:

```python
# ❌ XATO — Django ikkalasini ham "message_set" deb nomlamoqchi bo'ladi
class Message(models.Model):
    sender = models.ForeignKey(User, on_delete=models.CASCADE)
    receiver = models.ForeignKey(User, on_delete=models.CASCADE)
    text = models.TextField()
```

Django xato beradi:
```
ERRORS:
chat.Message.receiver: (fields.E304) Reverse accessor clashes with
'Message.sender'.
```

```python
# ✅ TO'G'RI — har biriga alohida nom
class Message(models.Model):
    sender = models.ForeignKey(
        User,
        on_delete=models.CASCADE,
        related_name='sent_messages',
    )
    receiver = models.ForeignKey(
        User,
        on_delete=models.CASCADE,
        related_name='received_messages',
    )
    text = models.TextField()
    created_at = models.DateTimeField(auto_now_add=True)
```

Endi:

```python
ali = User.objects.get(username="ali")

print(ali.sent_messages.count())        # yuborgan xabarlari
print(ali.received_messages.count())    # olgan xabarlari
```

### Teskari tomondan filtrlash

`related_name` faqat obyektdan emas, so'rovlarda ham ishlaydi:

```python
# Kamida bitta kitobi bor mualliflar
Author.objects.filter(books__isnull=False).distinct()

# "Django" so'zi bor kitob yozgan mualliflar
Author.objects.filter(books__title__icontains="Django")

# Izohi bor postlar
Post.objects.filter(comments__isnull=False).distinct()
```

### `related_name='+'` — teskari bog'lanish kerak bo'lmasa

```python
class Log(models.Model):
    user = models.ForeignKey(User, on_delete=models.CASCADE, related_name='+')
    # Endi user.log_set ishlamaydi — bu ataylab qilingan
```

---

## 12. `on_delete` — bog'langan obyekt o'chsa nima bo'ladi?

`ForeignKey` va `OneToOneField` uchun `on_delete` **majburiy**.

Savol: *"Muallif o'chirilsa, uning kitoblariga nima bo'ladi?"*

### `CASCADE` — birga o'chadi

```python
author = models.ForeignKey(Author, on_delete=models.CASCADE)
```

Muallif o'chsa → **barcha kitoblari ham o'chadi**.

```python
author = Author.objects.get(name="Ali")
print(Book.objects.count())     # 5
author.delete()
print(Book.objects.count())     # 2  ← Ali'ning 3 ta kitobi ham o'chdi
```

### `PROTECT` — o'chirishga ruxsat bermaydi

```python
author = models.ForeignKey(Author, on_delete=models.PROTECT)
```

```python
author.delete()
# → ProtectedError: Cannot delete some instances of model 'Author'
```

Avval kitoblarni o'chirish kerak. Muhim ma'lumotlarni himoya qilish uchun.

### `SET_NULL` — bo'sh qoladi

```python
author = models.ForeignKey(
    Author,
    on_delete=models.SET_NULL,
    null=True,               # ← null=True MAJBURIY!
)
```

Muallif o'chsa → kitob qoladi, lekin `author = None` bo'ladi.

```python
author.delete()
book = Book.objects.first()
print(book.author)          # None  ← kitob o'chmadi!
```

### `SET_DEFAULT` — standart qiymatga o'tadi

```python
author = models.ForeignKey(
    Author,
    on_delete=models.SET_DEFAULT,
    default=1,               # id=1 bo'lgan "Noma'lum muallif"
)
```

### Qaysi birini tanlash?

| Holat | Tanlov |
|-------|--------|
| Postning izohlari | `CASCADE` (post o'chsa, izohlar keraksiz) |
| Foydalanuvchi profili | `CASCADE` |
| Buyurtmaning mijozi | `PROTECT` (buyurtma tarixi yo'qolmasin) |
| Maqolaning kategoriyasi | `SET_NULL` (kategoriya o'chsa ham maqola qolsin) |

```python
class Post(models.Model):
    title = models.CharField(max_length=200)

    # Kategoriya o'chsa — post qolsin, kategoriyasiz
    category = models.ForeignKey(
        'Category',
        on_delete=models.SET_NULL,
        null=True,
        blank=True,
        related_name='posts',
    )

    # Muallif o'chsa — postlari ham o'chsin
    author = models.ForeignKey(
        User,
        on_delete=models.CASCADE,
        related_name='posts',
    )


class Comment(models.Model):
    # Post o'chsa — izohlar ham o'chsin
    post = models.ForeignKey(
        Post,
        on_delete=models.CASCADE,
        related_name='comments',
    )
    text = models.TextField()
```

---

## 13. `__str__` metodi

`__str__` — obyekt matn sifatida qanday ko'rinishini belgilaydi.

### `__str__`siz

```python
class Student(models.Model):
    name = models.CharField(max_length=100)
```

```python
s = Student.objects.create(name="Ali")
print(s)                        # Student object (1)   ← foydasiz!
print(Student.objects.all())    # <QuerySet [<Student: Student object (1)>]>
```

Admin panelda ham shunday chiqadi — hech narsa tushunarli emas.

### `__str__` bilan

```python
class Student(models.Model):
    name = models.CharField(max_length=100)

    def __str__(self):
        return self.name
```

```python
s = Student.objects.create(name="Ali")
print(s)                        # Ali   ← ajoyib!
print(Student.objects.all())    # <QuerySet [<Student: Ali>]>
```

### Turli xil misollar

```python
class Person(models.Model):
    first_name = models.CharField(max_length=50)
    last_name = models.CharField(max_length=50)

    def __str__(self):
        return f"{self.first_name} {self.last_name}"


class Book(models.Model):
    title = models.CharField(max_length=200)
    author = models.ForeignKey(Author, on_delete=models.CASCADE)

    def __str__(self):
        return f"{self.title} — {self.author.name}"


class Order(models.Model):
    number = models.CharField(max_length=20)
    total = models.DecimalField(max_digits=10, decimal_places=2)

    def __str__(self):
        return f"Buyurtma #{self.number} ({self.total} so'm)"


class Comment(models.Model):
    post = models.ForeignKey(Post, on_delete=models.CASCADE)
    text = models.TextField()

    def __str__(self):
        # Uzun matnni qisqartiramiz
        return f"{self.text[:50]}..."
```

> 💡 **`__str__` ni HAR BIR modelga yozing.** Bu 2 qatorlik ish, lekin admin
> panelda ishlashni ancha osonlashtiradi.

---

## 14. `class Meta` — model sozlamalari

`Meta` — modelning **maydonlariga emas, o'ziga** tegishli sozlamalar.

```python
class Post(models.Model):
    title = models.CharField(max_length=200)
    created_at = models.DateTimeField(auto_now_add=True)

    class Meta:
        ordering = ['-created_at']              # tartiblash
        verbose_name = "Maqola"                 # birlikdagi nom
        verbose_name_plural = "Maqolalar"       # ko'plikdagi nom

    def __str__(self):
        return self.title
```

### `ordering` — standart tartib

```python
class Meta:
    ordering = ['-created_at']       # yangi → eski (minus = teskari)
```

Endi har safar:

```python
Post.objects.all()      # avtomatik yangi → eski tartibda keladi
```

Turli xil variantlar:

```python
class Meta:
    ordering = ['name']                    # A → Z
    # ordering = ['-name']                 # Z → A
    # ordering = ['-created_at']           # yangi → eski
    # ordering = ['last_name', 'first_name']  # avval familiya, keyin ism
    # ordering = ['-priority', 'title']    # avval muhimlar, keyin alifbo
```

### `verbose_name` va `verbose_name_plural`

Admin panelda "Posts" o'rniga "Maqolalar" ko'rsatish uchun:

```python
class Category(models.Model):
    name = models.CharField(max_length=100)

    class Meta:
        verbose_name = "Kategoriya"
        verbose_name_plural = "Kategoriyalar"
```

> Buni yozmasangiz, Django `Category` → "Categorys" deb noto'g'ri ko'plik
> yasaydi. Shuning uchun har doim yozgan ma'qul.

### `unique_together` — bir nechta maydon birgalikda takrorlanmasin

```python
class Enrollment(models.Model):
    student = models.ForeignKey(Student, on_delete=models.CASCADE)
    course = models.ForeignKey(Course, on_delete=models.CASCADE)
    enrolled_at = models.DateTimeField(auto_now_add=True)

    class Meta:
        unique_together = ['student', 'course']
        # Bitta talaba bitta kursga IKKI MARTA yozila olmaydi
```

```python
Enrollment.objects.create(student=ali, course=math)   # ✅ OK
Enrollment.objects.create(student=ali, course=math)   # ❌ IntegrityError
Enrollment.objects.create(student=ali, course=physics)  # ✅ OK
```

### `db_table` — jadval nomini o'zgartirish

```python
class Post(models.Model):
    title = models.CharField(max_length=200)

    class Meta:
        db_table = 'blog_posts'     # standart: appname_post
```

### To'liq `Meta` misoli

```python
class Product(models.Model):
    name = models.CharField(max_length=200)
    sku = models.CharField(max_length=50)
    price = models.DecimalField(max_digits=10, decimal_places=2)
    created_at = models.DateTimeField(auto_now_add=True)

    class Meta:
        ordering = ['-created_at', 'name']
        verbose_name = "Mahsulot"
        verbose_name_plural = "Mahsulotlar"
        unique_together = ['name', 'sku']
        db_table = 'shop_products'

    def __str__(self):
        return f"{self.name} ({self.sku})"
```

---

## 15. Migratsiya — modelni bazaga yuborish

Modelni yozdingiz. Lekin baza hali u haqda bilmaydi! Migratsiya kerak.

### Uch qadam

```bash
# 1. Modelni yozing (models.py)

# 2. Migratsiya faylini yarating
python manage.py makemigrations

# 3. Bazaga qo'llang
python manage.py migrate
```

### Nima bo'ladi?

```bash
$ python manage.py makemigrations
Migrations for 'blog':
  blog/migrations/0001_initial.py
    - Create model Post
    - Create model Comment
```

Django `blog/migrations/0001_initial.py` faylini yaratdi. Bu faylda SQL'ga
aylanadigan ko'rsatmalar bor.

```bash
$ python manage.py migrate
Operations to perform:
  Apply all migrations: blog
Running migrations:
  Applying blog.0001_initial... OK
```

Endi jadval bazada bor!

### Modelni o'zgartirsangiz — yana takrorlang

```python
# Yangi maydon qo'shdik
class Post(models.Model):
    title = models.CharField(max_length=200)
    content = models.TextField()
    views = models.PositiveIntegerField(default=0)   # ← YANGI
```

```bash
python manage.py makemigrations
python manage.py migrate
```

### ⚠️ Mavjud jadvalga majburiy maydon qo'shsangiz

Django so'raydi:

```
It is impossible to add a non-nullable field 'views' to post without
specifying a default.
 1) Provide a one-off default now
 2) Quit and manually define a default
```

Buni oldini olish uchun **yangi maydonga `default` yoki `null=True` bering**:

```python
views = models.PositiveIntegerField(default=0)          # ✅
subtitle = models.CharField(max_length=200, blank=True, default="")  # ✅
rating = models.IntegerField(null=True, blank=True)     # ✅
```

### Foydali buyruqlar

```bash
# Migratsiyalar holatini ko'rish
python manage.py showmigrations

# Qanday SQL yaratilishini ko'rish
python manage.py sqlmigrate blog 0001

# Modellarda xato bor-yo'qligini tekshirish
python manage.py check
```

### Modelni admin panelga qo'shish

`blog/admin.py`:

```python
from django.contrib import admin
from .models import Post, Comment


admin.site.register(Post)
admin.site.register(Comment)
```

Chiroyliroq variant:

```python
from django.contrib import admin
from .models import Post


@admin.register(Post)
class PostAdmin(admin.ModelAdmin):
    list_display = ['title', 'author', 'is_published', 'created_at']
    list_filter = ['is_published', 'created_at']
    search_fields = ['title', 'content']
```

Admin panelga kirish uchun:

```bash
python manage.py createsuperuser
python manage.py runserver
# http://127.0.0.1:8000/admin/
```

---

## 16. Shell'da sinab ko'ramiz

```bash
python manage.py shell
```

```python
>>> from blog.models import Post, Comment
>>> from django.contrib.auth.models import User

# --- YARATISH ---
>>> user = User.objects.create_user(username="ali", password="12345")

>>> post = Post.objects.create(
...     title="Django o'rganamiz",
...     content="Bu birinchi maqolam",
...     author=user,
... )

# Ikkinchi usul
>>> p = Post(title="Ikkinchi post", content="...", author=user)
>>> p.save()

# --- O'QISH ---
>>> Post.objects.all()                      # hammasi
>>> Post.objects.count()                    # nechta
>>> Post.objects.first()                    # birinchisi
>>> Post.objects.last()                     # oxirgisi
>>> Post.objects.get(id=1)                  # aniq bittasi

# --- FILTRLASH ---
>>> Post.objects.filter(author=user)
>>> Post.objects.filter(is_published=True)
>>> Post.objects.filter(title__icontains="django")     # ichida bor
>>> Post.objects.filter(title__startswith="Django")    # bilan boshlanadi
>>> Post.objects.filter(views__gt=100)                 # katta (>)
>>> Post.objects.filter(views__gte=100)                # katta yoki teng (>=)
>>> Post.objects.filter(views__lt=100)                 # kichik (<)
>>> Post.objects.exclude(is_published=True)            # BUNDAN TASHQARI

# --- TARTIBLASH ---
>>> Post.objects.order_by('title')          # A → Z
>>> Post.objects.order_by('-created_at')    # yangi → eski

# --- YANGILASH ---
>>> post = Post.objects.get(id=1)
>>> post.title = "Yangi sarlavha"
>>> post.save()

# Ko'pini birdan
>>> Post.objects.filter(author=user).update(is_published=True)

# --- O'CHIRISH ---
>>> post = Post.objects.get(id=1)
>>> post.delete()

>>> Post.objects.filter(is_published=False).delete()

# --- BOG'LANISHLAR ---
>>> post = Post.objects.get(id=2)
>>> post.author.username                    # postdan userga
'ali'
>>> user.posts.count()                      # userdan postlariga (related_name!)
1
>>> post.comments.all()                     # postning izohlari
```

---

## 17. To'liq loyiha misoli: Blog

Endi o'rganganlarimizni bitta to'liq loyihada birlashtiramiz.

`blog/models.py`:

```python
from django.contrib.auth.models import User
from django.db import models


class Category(models.Model):
    """Maqola kategoriyasi: Texnologiya, Sport, Sayohat..."""

    name = models.CharField(
        max_length=100,
        unique=True,
        verbose_name="Nomi",
    )
    slug = models.SlugField(
        max_length=100,
        unique=True,
        help_text="URL uchun, masalan: texnologiya",
    )
    description = models.TextField(
        blank=True,
        verbose_name="Tavsif",
    )
    created_at = models.DateTimeField(auto_now_add=True)

    class Meta:
        ordering = ['name']
        verbose_name = "Kategoriya"
        verbose_name_plural = "Kategoriyalar"

    def __str__(self):
        return self.name


class Tag(models.Model):
    """Teg: #django, #python, #web"""

    name = models.CharField(max_length=50, unique=True)
    slug = models.SlugField(max_length=50, unique=True)

    class Meta:
        ordering = ['name']
        verbose_name = "Teg"
        verbose_name_plural = "Teglar"

    def __str__(self):
        return f"#{self.name}"


class Post(models.Model):
    """Blog maqolasi."""

    class Status(models.TextChoices):
        DRAFT = 'draft', 'Qoralama'
        PUBLISHED = 'published', 'Chop etilgan'
        ARCHIVED = 'archived', 'Arxivlangan'

    # --- Asosiy maydonlar ---
    title = models.CharField(
        max_length=250,
        verbose_name="Sarlavha",
    )
    slug = models.SlugField(
        max_length=250,
        unique=True,
        help_text="URL manzili uchun",
    )
    content = models.TextField(
        verbose_name="Matn",
    )
    summary = models.TextField(
        blank=True,
        max_length=500,
        verbose_name="Qisqacha mazmun",
        help_text="Ro'yxatda ko'rinadigan qisqa matn",
    )
    cover = models.ImageField(
        upload_to='posts/covers/',
        null=True,
        blank=True,
        verbose_name="Muqova rasmi",
    )

    # --- Bog'lanishlar ---
    author = models.ForeignKey(
        User,
        on_delete=models.CASCADE,
        related_name='posts',
        verbose_name="Muallif",
    )
    category = models.ForeignKey(
        Category,
        on_delete=models.SET_NULL,
        null=True,
        blank=True,
        related_name='posts',
        verbose_name="Kategoriya",
    )
    tags = models.ManyToManyField(
        Tag,
        blank=True,
        related_name='posts',
        verbose_name="Teglar",
    )

    # --- Holat va statistika ---
    status = models.CharField(
        max_length=20,
        choices=Status.choices,
        default=Status.DRAFT,
        db_index=True,
        verbose_name="Holati",
    )
    views = models.PositiveIntegerField(
        default=0,
        verbose_name="Ko'rishlar soni",
    )
    is_featured = models.BooleanField(
        default=False,
        verbose_name="Tavsiya etilgan",
    )

    # --- Vaqtlar ---
    created_at = models.DateTimeField(auto_now_add=True, db_index=True)
    updated_at = models.DateTimeField(auto_now=True)
    published_at = models.DateTimeField(null=True, blank=True)

    class Meta:
        ordering = ['-created_at']
        verbose_name = "Maqola"
        verbose_name_plural = "Maqolalar"

    def __str__(self):
        return self.title


class Comment(models.Model):
    """Maqolaga izoh."""

    post = models.ForeignKey(
        Post,
        on_delete=models.CASCADE,
        related_name='comments',
        verbose_name="Maqola",
    )
    author = models.ForeignKey(
        User,
        on_delete=models.CASCADE,
        related_name='comments',
        verbose_name="Muallif",
    )
    parent = models.ForeignKey(
        'self',
        on_delete=models.CASCADE,
        null=True,
        blank=True,
        related_name='replies',
        verbose_name="Javob berilayotgan izoh",
    )

    text = models.TextField(verbose_name="Izoh matni")
    is_approved = models.BooleanField(
        default=False,
        verbose_name="Tasdiqlangan",
    )

    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    class Meta:
        ordering = ['created_at']
        verbose_name = "Izoh"
        verbose_name_plural = "Izohlar"

    def __str__(self):
        return f"{self.author.username}: {self.text[:40]}"


class Like(models.Model):
    """Maqolaga qo'yilgan layk."""

    post = models.ForeignKey(
        Post,
        on_delete=models.CASCADE,
        related_name='likes',
    )
    user = models.ForeignKey(
        User,
        on_delete=models.CASCADE,
        related_name='likes',
    )
    created_at = models.DateTimeField(auto_now_add=True)

    class Meta:
        # Bitta user bitta postga faqat bir marta layk qo'yadi
        unique_together = ['post', 'user']
        verbose_name = "Layk"
        verbose_name_plural = "Layklar"

    def __str__(self):
        return f"{self.user.username} → {self.post.title}"
```

### Bu modellar bilan ishlash

```python
from django.contrib.auth.models import User
from blog.models import Category, Tag, Post, Comment, Like

# --- Tayyorlash ---
ali = User.objects.create_user(username="ali", password="12345")
vali = User.objects.create_user(username="vali", password="12345")

tech = Category.objects.create(name="Texnologiya", slug="texnologiya")

django_tag = Tag.objects.create(name="django", slug="django")
python_tag = Tag.objects.create(name="python", slug="python")

# --- Maqola yaratish ---
post = Post.objects.create(
    title="Django modellari haqida",
    slug="django-modellari",
    content="Django'da model yaratish juda oson...",
    summary="Qisqacha: modellar haqida",
    author=ali,
    category=tech,
    status=Post.Status.PUBLISHED,
)

post.tags.add(django_tag, python_tag)

# --- Izoh qo'shish ---
c1 = Comment.objects.create(post=post, author=vali, text="Juda foydali!")

# Izohga javob (parent orqali)
Comment.objects.create(post=post, author=ali, parent=c1, text="Rahmat!")

# --- Layk ---
Like.objects.create(post=post, user=vali)

# --- Ma'lumot olish ---
print(post.author.username)          # ali
print(post.category.name)            # Texnologiya
print(post.tags.count())             # 2
print(post.comments.count())         # 2
print(post.likes.count())            # 1
print(post.get_status_display())     # Chop etilgan

print(c1.replies.count())            # 1  ← javoblar

# Teskari tomondan
print(ali.posts.count())             # 1
print(tech.posts.count())            # 1
print(django_tag.posts.count())      # 1
print(vali.comments.count())         # 1

# --- Filtrlash ---
Post.objects.filter(status=Post.Status.PUBLISHED)
Post.objects.filter(category__name="Texnologiya")
Post.objects.filter(tags__name="django")
Post.objects.filter(author__username="ali")
Post.objects.filter(comments__isnull=False).distinct()
```

---

## 18. Tez-tez uchraydigan xatolar

### ❌ 1. `models.Model` dan meros olmaslik

```python
class Post:                          # ❌ jadval yaratilmaydi
    title = models.CharField(max_length=100)

class Post(models.Model):            # ✅
    title = models.CharField(max_length=100)
```

### ❌ 2. `CharField` da `max_length` yo'q

```python
name = models.CharField()                    # ❌
name = models.CharField(max_length=100)      # ✅
```

### ❌ 3. `ForeignKey` da `on_delete` yo'q

```python
author = models.ForeignKey(User)                              # ❌
author = models.ForeignKey(User, on_delete=models.CASCADE)    # ✅
```

### ❌ 4. Matn maydoniga `null=True`

```python
bio = models.TextField(null=True, blank=True)     # ❌ chalkashlik
bio = models.TextField(blank=True)                # ✅
```

### ❌ 5. `default` ga funksiyani chaqirish

```python
created = models.DateTimeField(default=timezone.now())   # ❌ qavs bor
created = models.DateTimeField(default=timezone.now)     # ✅ qavssiz
```

### ❌ 6. Pul uchun `FloatField`

```python
price = models.FloatField()                                       # ❌
price = models.DecimalField(max_digits=10, decimal_places=2)      # ✅
```

### ❌ 7. `SET_NULL` bilan `null=True` yozmaslik

```python
category = models.ForeignKey(Category, on_delete=models.SET_NULL)             # ❌
category = models.ForeignKey(Category, on_delete=models.SET_NULL, null=True)  # ✅
```

### ❌ 8. `ManyToManyField` ga `null=True`

```python
tags = models.ManyToManyField(Tag, null=True, blank=True)   # ❌ null foydasiz
tags = models.ManyToManyField(Tag, blank=True)              # ✅
```

### ❌ 9. Migratsiyani unutish

Modelni o'zgartirdingiz, lekin `makemigrations` + `migrate` qilmadingiz →
`no such column` xatosi chiqadi.

### ❌ 10. Bir modelga ikkita FK, lekin `related_name` yo'q

```python
# ❌
sender = models.ForeignKey(User, on_delete=models.CASCADE)
receiver = models.ForeignKey(User, on_delete=models.CASCADE)

# ✅
sender = models.ForeignKey(User, on_delete=models.CASCADE,
                           related_name='sent')
receiver = models.ForeignKey(User, on_delete=models.CASCADE,
                             related_name='received')
```

---

## 19. Shpargalka

### Maydon turlari

```python
models.CharField(max_length=100)                       # qisqa matn
models.TextField()                                     # uzun matn
models.EmailField()                                    # email
models.URLField()                                      # havola
models.SlugField(max_length=100)                       # URL uchun matn

models.IntegerField()                                  # butun son
models.PositiveIntegerField()                          # musbat butun son
models.DecimalField(max_digits=10, decimal_places=2)   # pul
models.FloatField()                                    # kasrli son

models.BooleanField(default=False)                     # True/False

models.DateField()                                     # sana
models.DateTimeField()                                 # sana + vaqt
models.TimeField()                                     # vaqt

models.ImageField(upload_to='images/')                 # rasm
models.FileField(upload_to='files/')                   # fayl

models.ForeignKey(Model, on_delete=models.CASCADE)     # ko'pdan birga
models.OneToOneField(Model, on_delete=models.CASCADE)  # birdan birga
models.ManyToManyField(Model)                          # ko'pdan ko'pga
```

### Parametrlar

```python
max_length=100          # matnning maksimal uzunligi (CharField uchun majburiy)
default=0               # standart qiymat
null=True               # bazada NULL bo'lishi mumkin (raqam/sana uchun)
blank=True              # formada bo'sh qoldirish mumkin (matn uchun)
unique=True             # takrorlanmas qiymat
choices=[...]           # tanlanadigan variantlar
db_index=True           # tez qidiruv uchun index
verbose_name="Nomi"     # admin paneldagi chiroyli nom
help_text="Izoh"        # formadagi yordamchi matn
editable=False          # formada ko'rsatilmasin

auto_now_add=True       # yaratilganda bir marta yoziladi
auto_now=True           # har saqlanganda yangilanadi

on_delete=models.CASCADE      # bog'langani o'chsa — bu ham o'chsin
on_delete=models.PROTECT      # o'chirishga ruxsat bermasin
on_delete=models.SET_NULL     # NULL bo'lsin (null=True kerak!)
related_name='items'          # teskari tomondan murojaat nomi
```

### Model shabloni (nusxa olib ishlating)

```python
from django.db import models


class MyModel(models.Model):
    # --- Tanlovlar ---
    class Status(models.TextChoices):
        ACTIVE = 'active', 'Faol'
        INACTIVE = 'inactive', 'Nofaol'

    # --- Majburiy maydonlar ---
    name = models.CharField(max_length=200, verbose_name="Nomi")

    # --- Ixtiyoriy maydonlar ---
    description = models.TextField(blank=True, verbose_name="Tavsif")
    priority = models.IntegerField(null=True, blank=True)

    # --- Bog'lanishlar ---
    owner = models.ForeignKey(
        'auth.User',
        on_delete=models.CASCADE,
        related_name='my_models',
    )

    # --- Holat ---
    status = models.CharField(
        max_length=20,
        choices=Status.choices,
        default=Status.ACTIVE,
    )
    is_active = models.BooleanField(default=True)

    # --- Vaqtlar ---
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    class Meta:
        ordering = ['-created_at']
        verbose_name = "Mening modelim"
        verbose_name_plural = "Mening modellarim"

    def __str__(self):
        return self.name
```

### Buyruqlar

```bash
python manage.py makemigrations      # migratsiya fayli yaratish
python manage.py migrate             # bazaga qo'llash
python manage.py showmigrations      # holatni ko'rish
python manage.py check               # xatolarni tekshirish
python manage.py shell               # interaktiv rejim
python manage.py createsuperuser     # admin yaratish
python manage.py runserver           # serverni ishga tushirish
```

---

## Yakuniy maslahat

O'rganish tartibi:

1. ✅ Oddiy model yozing (2-3 ta maydon) → `makemigrations` → `migrate`
2. ✅ Admin panelga qo'shing va ma'lumot kiriting
3. ✅ `__str__` qo'shing va farqini ko'ring
4. ✅ `null`, `blank`, `default` bilan o'ynang
5. ✅ `ForeignKey` qo'shing va `related_name` ni sinab ko'ring
6. ✅ `shell` da filtrlashni mashq qiling
7. ✅ Kichik loyiha yozing (To-do, Kutubxona, Do'kon)

**Eng muhimi:** kodni o'qib emas, **yozib** o'rganing. Har bir misolni
o'z loyihangizda takrorlang, ataylab xato qiling va xato xabarlarini o'qing —
Django xatolari juda tushunarli yozilgan.

Omad! 🚀
