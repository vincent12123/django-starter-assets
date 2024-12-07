---

## 1. Membuat Folder Proyek dan Virtual Environment

### a. Buat Folder Proyek

Pertama, buat folder proyek dengan nama `django_starter`:

```bash
mkdir django_starter
cd django_starter
```

### b. Buat Virtual Environment

Virtual environment (venv) digunakan untuk mengelola dependensi proyek secara terpisah dari sistem Python global Anda.

```bash
python -m venv venv
```

Aktifkan virtual environment:

- **Windows:**

  ```bash
  venv\Scripts\activate
  ```

- **Unix atau MacOS:**

  ```bash
  source venv/bin/activate
  ```

### c. Buat `requirements.txt`

Buat file `requirements.txt` yang berisi daftar paket Python yang diperlukan oleh proyek Anda:

```bash
touch requirements.txt
```

Isi `requirements.txt` dengan:

```txt
Django
Pillow
django-cleanup
django-htmx
```

#### Penjelasan `requirements.txt`:

- **Django**: Framework web yang digunakan untuk membangun aplikasi web.
- **Pillow**: Library Python untuk memproses gambar, diperlukan jika Anda menangani upload gambar.
- **django-cleanup**: Mengelola file yang diunggah secara otomatis, seperti menghapus file lama saat objek dihapus atau diubah.
- **django-htmx**: Integrasi HTMX dengan Django untuk membuat interaksi frontend yang dinamis tanpa harus menulis banyak JavaScript.

### d. Instalasi Dependensi

Jalankan perintah berikut untuk menginstal semua paket yang tercantum di `requirements.txt`:

```bash
pip install -r requirements.txt
```

---

## 2. Memulai Proyek Django

### a. Inisialisasi Proyek Django

Mulai proyek Django dengan nama `a_core` di direktori saat ini:

```bash
django-admin startproject a_core .
```

> **Catatan**: Titik (`.`) di akhir perintah menunjukkan bahwa proyek akan dibuat di direktori saat ini.

### b. Setup Database

Jalankan migrasi awal untuk menyiapkan database:

```bash
python manage.py migrate
```

### c. Jalankan Server Django

Pastikan semuanya berjalan dengan baik dengan menjalankan server:

```bash
python manage.py runserver
```

Buka browser dan akses `http://127.0.0.1:8000/` untuk melihat halaman default Django.

---

## 3. Membuat Aplikasi Homepage

### a. Buat Aplikasi `a_home`

Buat aplikasi Django bernama `a_home`:

```bash
python manage.py startapp a_home
```

### b. Tambahkan Aplikasi ke `settings.py`

Buka file `a_core/settings.py` dan tambahkan `'a_home'` ke dalam daftar `INSTALLED_APPS`:

```python
INSTALLED_APPS = [
    # Aplikasi bawaan Django
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',

    # Aplikasi kita
    'a_home',  # Tambahkan ini
]
```

---

## 4. Membuat View untuk Homepage

### a. Definisikan View di `a_home/views.py`

Buka file `a_home/views.py` dan tambahkan fungsi `home_view`:

```python
from django.shortcuts import render

def home_view(request):
    return render(request, 'home.html')
```

### b. Konfigurasi URL di `a_core/urls.py`

Buka file `a_core/urls.py` dan tambahkan path untuk homepage:

```python
from django.contrib import admin
from django.urls import path
from a_home.views import home_view  # Import home_view

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', home_view, name='home'),  # Tambahkan ini
]
```

---

## 5. Mengatur Template

### a. Buat Folder `templates`

Buat folder `templates` di root direktori proyek:

```bash
mkdir templates
```

### b. Konfigurasi `settings.py` untuk Template

Buka `a_core/settings.py` dan tambahkan path ke folder `templates` dalam konfigurasi `TEMPLATES`:

```python
import os
from pathlib import Path

BASE_DIR = Path(__file__).resolve().parent.parent

TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [BASE_DIR / 'templates'],  # Tambahkan ini
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                # ... tambahkan context_processors lainnya jika diperlukan
            ],
        },
    },
]
```

### c. Menyiapkan Template Dasar

#### i. Unduh Template `base.html`

Anda dapat mengunduh template dasar dari repository GitHub berikut:

[GitHub - django-templates](https://github.com/vincent12123/django-templates)

1. **Download file ZIP dan ekstrak.**
2. **Ekstrak `Zip file` Anda:**

   Salin kontennya dan buat file `base.html` di folder `templates` proyek Anda:

    ```bash
    touch templates/base.html
    ```

Buka `base.html` di editor teks (misalnya VS Code) dan tempelkan konten yang telah disalin.

#### ii. Memisahkan Komponen ke dalam Includes

Untuk menjaga kebersihan dan modularitas template, pisahkan beberapa elemen ke dalam file terpisah.

1. **Memisahkan Pesan (Messages)**

   - **Potong Elemen Messages:** Di `base.html`, potong bagian yang menampilkan pesan (biasanya menggunakan `{% if messages %} ... {% endif %}`).
   - **Buat Folder `includes` dan File `messages.html`:**

     ```bash
     mkdir templates/includes
     touch templates/includes/messages.html
     ```

     Tempelkan elemen messages yang telah dipotong ke dalam `messages.html`.

   - **Include `messages.html` di `base.html`:**

     ```django
     {% include 'includes/messages.html' %}
     ```

2. **Memisahkan Header**

   - **Potong Elemen Header:** Di `base.html`, potong bagian header (biasanya elemen `<header>`).
   - **Buat File `header.html`:**

     ```bash
     touch templates/includes/header.html
     ```

     Tempelkan elemen header yang telah dipotong ke dalam `header.html`.

   - **Include `header.html` di `base.html`:**

     ```django
     {% include 'includes/header.html' %}
     ```

3. **Memisahkan Konten**

   - Di `base.html`, ganti elemen konten utama dengan blok Django:

     ```django
     {% block layout %}
     {% endblock %}
     ```

### d. Membuat Layout Template

1. **Buat Folder `layouts` dan File `blank.html`:**

   ```bash
   mkdir templates/layouts
   touch templates/layouts/blank.html
   ```

2. **Isi `blank.html`:**

   ```django
   {% extends 'base.html' %}

   {% block layout %}
   <content class="block w-full">
       {% block content %}
       {% endblock %}
   </content>
   {% endblock %}
   ```

### e. Membuat Template Homepage

1. **Buat File `home.html`:**

   ```bash
   touch templates/home.html
   ```

2. **Isi `home.html`:**

   ```django
   {% extends 'layouts/blank.html' %}

   {% block content %}
       <div class="max-w-4xl mx-auto px-8 py-24">
           <h1>New Project</h1>
       </div>
   {% endblock %}
   ```

### f. Menjalankan dan Memeriksa Homepage

Setelah semua langkah di atas selesai, jalankan server Django:

```bash
python manage.py runserver
```

Buka browser dan akses `http://127.0.0.1:8000/` untuk melihat homepage Anda yang telah dibuat.

---

## 6. Penjelasan Struktur Template

### a. `base.html`

`base.html` adalah template dasar yang berfungsi sebagai kerangka utama untuk semua halaman dalam aplikasi Django Anda. Di dalamnya, terdapat elemen-elemen umum seperti header, footer, dan area pesan (messages). Dengan menggunakan `{% include %}`, Anda dapat memisahkan komponen-komponen ini ke dalam file terpisah untuk meningkatkan keterbacaan dan pemeliharaan kode.

### b. `layouts/blank.html`

`blank.html` adalah template layout yang mewarisi dari `base.html`. Di dalamnya, terdapat blok `layout` yang didefinisikan ulang, serta blok `content` yang dapat diisi oleh template yang mewarisinya. Struktur ini memungkinkan Anda untuk membuat berbagai layout dengan mudah.

### c. `home.html`

`home.html` adalah template spesifik untuk halaman homepage. Template ini mewarisi dari `blank.html` dan mengisi blok `content` dengan konten khusus untuk homepage.

### **Alur Pewarisan Template:**

1. **`home.html`** mewarisi dari **`layouts/blank.html`**.
2. **`layouts/blank.html`** mewarisi dari **`base.html`**.
3. **`base.html`** menyediakan kerangka dasar, termasuk header dan messages.
4. **`blank.html`** mendefinisikan blok `layout` dan `content` yang dapat diisi oleh template turunan.
5. **`home.html`** mengisi blok `content` dengan konten spesifik untuk homepage.

Dengan struktur ini, Anda dapat dengan mudah membuat halaman-halaman lain yang memiliki layout serupa dengan hanya mewarisi dari `blank.html` atau bahkan membuat layout baru berdasarkan kebutuhan.

---

## 7. Menambahkan Logo dengan File Statis

### a. Membuat Folder `static`

Buat folder `static` di root proyek Anda:

```bash
mkdir static
```

### b. Mengonfigurasi `settings.py` untuk File Statis

Buka `a_core/settings.py` dan tambahkan konfigurasi berikut:

```python
STATIC_URL = '/static/'
STATICFILES_DIRS = [BASE_DIR / 'static']
```

### c. Menyalin File Statis ke Folder `static`

1. **Buat Folder untuk Gambar:**

   ```bash
   mkdir static/images
   ```

2. **Salin File Logo:**

   Pastikan Anda memiliki file logo (misalnya, `logo.svg`) dan salin ke `static/images/`:

   ```bash
   cp path_ke_logo/logo.svg static/images/
   ```

### d. Menampilkan Logo di Template

1. **Edit `header.html`:**

   ```django
   {% load static %}

   <header>
       <a href="/">
           <img src="{% static 'images/logo.svg' %}" alt="Logo" />
           <!-- Elemen header lainnya -->
       </a>
   </header>
   ```

2. **Pastikan Tag Static Dimuat:**

   Pastikan `{% load static %}` ada di bagian atas `header.html`.

### e. Memeriksa Logo di Browser

Setelah semua langkah di atas selesai, jalankan server Django:

```bash
python manage.py runserver
```

Buka browser dan akses `http://127.0.0.1:8000/` untuk melihat homepage Anda. Logo seharusnya tampil di header sesuai dengan yang telah Anda tambahkan.

---

## 8. Membuat Halaman Profil Pengguna

### a. Membuat Aplikasi `a_users`

```bash
python manage.py startapp a_users
```

### b. Menambahkan `a_users` ke `INSTALLED_APPS`

Buka `a_core/settings.py` dan tambahkan `'a_users'` ke dalam `INSTALLED_APPS`:

```python
INSTALLED_APPS = [
    # ...
    'a_home',
    'a_users',  # Tambahkan ini
]
```

### c. Membuat Model `Profile`

Model `Profile` akan memperluas model `User` bawaan Django untuk menyimpan informasi tambahan tentang pengguna.

**Di `a_users/models.py`:**

```python
from django.db import models
from django.contrib.auth.models import User
from django.templatetags.static import static

class Profile(models.Model):
    user = models.OneToOneField(User, on_delete=models.CASCADE)
    image = models.ImageField(upload_to='avatars/', null=True, blank=True)
    displayname = models.CharField(max_length=20, null=True, blank=True)
    info = models.TextField(null=True, blank=True)

    def __str__(self):
        return str(self.user)

    @property
    def name(self):
        return self.displayname if self.displayname else self.user.username

    @property
    def avatar(self):
        return self.image.url if self.image else static('images/avatar.svg')
```

#### Penjelasan Model `Profile`:

- **`user`**: Hubungan one-to-one dengan model `User` bawaan Django. Setiap pengguna memiliki satu profil.
- **`image`**: Field untuk menyimpan gambar profil pengguna. File gambar akan diunggah ke folder `avatars/` di dalam folder media Anda. Field ini opsional (`null=True, blank=True`).
- **`displayname`**: Nama tampilan pengguna yang bisa berbeda dari username asli. Juga opsional.
- **`info`**: Informasi tambahan tentang pengguna. Bisa digunakan untuk bio atau deskripsi singkat.
- **`__str__`**: Mengembalikan representasi string dari objek `Profile`, yaitu nama pengguna.
- **`name` (property)**: Mengembalikan `displayname` jika diisi; jika tidak, mengembalikan `username` pengguna.
- **`avatar` (property)**: Mengembalikan URL gambar avatar pengguna. Jika pengguna belum mengunggah gambar, akan mengembalikan URL default (`images/avatar.svg`).

### d. Melakukan Migrasi

Setelah membuat model, Anda perlu membuat migrasi untuk menerapkannya ke database.

```bash
python manage.py makemigrations
python manage.py migrate
```

### e. Mendaftarkan Model di Admin

**Di `a_users/admin.py`:**

```python
from django.contrib import admin
from .models import Profile

admin.site.register(Profile)
```

#### Penjelasan:

- **`from .models import Profile`**: Mengimpor model `Profile` dari file `models.py` dalam aplikasi `a_users`.
- **`admin.site.register(Profile)`**: Mendaftarkan model `Profile` ke admin site sehingga Anda dapat menambah, mengedit, atau menghapus profil pengguna melalui antarmuka admin.

### f. Membuat `signals.py`

**Signals** di Django digunakan untuk menjalankan kode tertentu ketika peristiwa tertentu terjadi, seperti saat model disimpan atau dihapus. Dalam kasus ini, kita akan menggunakan sinyal `post_save` untuk otomatis membuat objek `Profile` setiap kali pengguna baru dibuat.

**Di `a_users/signals.py`:**

```python
from django.db.models.signals import post_save
from django.dispatch import receiver
from django.contrib.auth.models import User
from .models import Profile

@receiver(post_save, sender=User)
def user_postsave(sender, instance, created, **kwargs):
    if created:
        Profile.objects.create(user=instance)
```

#### Penjelasan `signals.py`:

- **Import Statements:**
  - **`from django.dispatch import receiver`**: Mengimpor dekorator `receiver` yang digunakan untuk menghubungkan fungsi dengan sinyal tertentu.
  - **`from django.db.models.signals import post_save`**: Mengimpor sinyal `post_save` yang dipicu setelah model disimpan.
  - **`from django.contrib.auth.models import User`**: Mengimpor model `User` bawaan Django.
  - **`from .models import Profile`**: Mengimpor model `Profile` yang telah kita buat.

- **Fungsi `user_postsave`:**
  - **Dekorator `@receiver(post_save, sender=User)`**: Menghubungkan fungsi ini dengan sinyal `post_save` dari model `User`. Artinya, setiap kali objek `User` disimpan, fungsi ini akan dipanggil.
  - **Parameter:**
    - **`sender`**: Model yang mengirim sinyal, dalam hal ini `User`.
    - **`instance`**: Instans objek yang disimpan.
    - **`created`**: Boolean yang menunjukkan apakah objek baru dibuat (`True`) atau diperbarui (`False`).
    - **`**kwargs`**: Argumen tambahan.
  - **Logika Fungsi:**
    - Jika objek `User` baru dibuat (`created` adalah `True`), maka buat objek `Profile` yang terkait dengan pengguna tersebut.

### g. Mengonfigurasi `apps.py` untuk Sinyal

Setelah membuat `signals.py`, kita perlu memastikan bahwa sinyal ini diimpor dan didaftarkan saat aplikasi siap dijalankan. Ini penting karena tanpa mengimpor `signals.py`, sinyal tidak akan dipicu.

**Di `a_users/apps.py`:**

```python
from django.apps import AppConfig

class AUsersConfig(AppConfig):
    default_auto_field = 'django.db.models.BigAutoField'
    name = 'a_users'

    def ready(self):
        import a_users.signals
```

#### Penjelasan:

- **`ready` Method:**
  - Metode `ready` dipanggil oleh Django saat aplikasi siap dijalankan.
  - Di sini, kita mengimpor `a_users.signals` untuk memastikan bahwa sinyal terdaftar dan siap digunakan.

- **Kenapa Harus Mendaftarkan `signals.py` di `apps.py`:**
  - **Pendaftaran Sinyal:** Jika `signals.py` tidak diimpor, Django tidak akan mengetahui keberadaan sinyal tersebut, dan otomatisasi pembuatan profil tidak akan berjalan.
  - **Lifecycle Aplikasi:** Dengan mengimpor sinyal di dalam metode `ready`, kita memastikan bahwa sinyal didaftarkan setelah semua konfigurasi aplikasi lainnya selesai, mencegah masalah potensial terkait urutan impor.

### h. Membuat Superuser

Untuk mengakses antarmuka admin dan mengelola pengguna serta profil, Anda perlu membuat superuser.

```bash
python manage.py createsuperuser
```

### i. Menjalankan Server dan Memeriksa Admin

```bash
python manage.py runserver
```

Buka browser dan akses `http://127.0.0.1:8000/admin/`. Masuk dengan kredensial superuser yang telah Anda buat.

#### Memeriksa Model `Profile` di Admin:

1. **Navigasi ke `Profiles`:** Setelah masuk ke admin, Anda akan melihat model `Profile` di daftar aplikasi `a_users`.
2. **Mengecek Profil Pengguna:**
   - Jika Anda memiliki pengguna yang sudah dibuat, Anda akan melihat entri `Profile` yang terkait dengan setiap pengguna.
   - Jika Anda membuat pengguna baru, profilnya akan otomatis dibuat berkat sinyal yang telah kita konfigurasikan.

---

## 9. Menambahkan Avatar dan Username ke Header

### a. Mengubah `header.html`

**Di `templates/includes/header.html`:**

```django
{% load static %}

<header class="flex items-center justify-between bg-gray-800 h-20 px-8 text-white sticky top-0 z-40">
    <div>
        <a class="flex items-center gap-2" href="/">
            <img class="h-6" src="{% static 'images/logo.svg' %}" alt="Logo"/>
            <span class="text-lg font-bold">Project Title</span>
        </a>
    </div>
    <nav>
        <ul>
            {% if request.user.is_authenticated %}
                <li><a href="/">Home</a></li>
                <li>
                    <a href="{% url 'profile' %}">
                        <img class="h-8 w-8 rounded-full" src="{{ user.profile.avatar }}" alt="Avatar"/>
                        {{ user.profile.name }}
                    </a>
                </li>
            {% else %}
                <li><a href="{% url 'account_login' %}">Login</a></li>
                <li><a href="{% url 'account_signup' %}">Signup</a></li>
            {% endif %}
        </ul>
    </nav>
</header>
```

#### Penjelasan:

- **`{% load static %}`**: Memuat tag static untuk mengakses file statis.
- **Header**: Menampilkan logo, judul proyek, dan navigasi yang berbeda berdasarkan status autentikasi pengguna.
- **Autentikasi Pengguna**:
  - **Jika pengguna sudah login**: Menampilkan link "Home" dan avatar dengan nama pengguna.
  - **Jika pengguna belum login**: Menampilkan link "Login" dan "Signup".

### b. Menambahkan Dropdown Menu (Opsional)

Jika Anda ingin menambahkan menu dropdown, Anda bisa menggunakan JavaScript atau library seperti Alpine.js.

---

## 10. Membuat Halaman Profil Pengguna

### a. Membuat View untuk Profil

**Di `a_users/views.py`:**

```python
from django.shortcuts import render
from django.contrib.auth.decorators import login_required

@login_required
def profile_view(request):
    profile = request.user.profile
    return render(request, 'a_users/profile.html', {'profile': profile})
```

#### Penjelasan:

- **`@login_required`**: Dekorator ini memastikan bahwa hanya pengguna yang sudah login yang dapat mengakses halaman profil. Jika pengguna belum login, mereka akan diarahkan ke halaman login.
- **`profile = request.user.profile`**: Mengambil objek `Profile` yang terkait dengan pengguna saat ini.
- **`render(request, 'a_users/profile.html', {'profile': profile})`**: Mengembalikan respons HTTP dengan merender template `profile.html` dan mengirimkan konteks `profile`.

### b. Membuat Template `profile.html`

**Di `templates/a_users/profile.html`:**

```django
{% extends 'layouts/blank.html' %}

{% block content %}
<div class="max-w-lg mx-auto flex flex-col items-center pt-20 px-4">
    <img class="w-36 h-36 rounded-full object-cover mb-4" src="{{ profile.avatar }}" alt="Avatar"/>
    <div class="text-center">
        <h1 class="text-2xl font-bold">{{ profile.name }}</h1>
        <div class="text-gray-400 mb-2 -mt-3">@{{ profile.user.username }}</div>
        {% if profile.info %}
        <div class="mt-8">{{ profile.info|linebreaksbr }}</div>
        {% endif %}
    </div>
</div>
{% endblock %}
```

#### Penjelasan Template:

- **`{% extends 'layouts/blank.html' %}`**: Menggunakan template layout `blank.html` sebagai dasar.
- **Blok `content`**: Menampilkan avatar, nama pengguna, username, dan informasi tambahan jika tersedia.
  - **Avatar**: Menampilkan gambar avatar pengguna. Jika avatar belum diunggah, akan menampilkan gambar default.
  - **Nama dan Username**: Menampilkan nama tampilan dan username pengguna.
  - **Informasi Tambahan**: Menampilkan informasi tambahan pengguna jika tersedia. Filter `linebreaksbr` digunakan untuk mengonversi baris baru menjadi `<br>`.

### c. Menambahkan URL untuk Profil

**Di `a_core/urls.py`:**

```python
from django.contrib import admin
from django.urls import path, include
from a_home.views import home_view

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', home_view, name='home'),
    path('profile/', include('a_users.urls')),  # Tambahkan ini
]
```

**Di `a_users/urls.py`:**

```python
from django.urls import path
from .views import profile_view

urlpatterns = [
    path('', profile_view, name='profile'),
]
```

#### Penjelasan:

- **`path('profile/', include('a_users.urls'))`**: Ini berarti setiap URL yang dimulai dengan `profile/` akan diteruskan ke modul URL yang ada di aplikasi `a_users`. Misalnya, `http://127.0.0.1:8000/profile/` akan diarahkan ke URL yang didefinisikan dalam `a_users/urls.py`.
- **`a_users/urls.py`**: Mendefinisikan URL untuk `profile/` tanpa tambahan path. URL ini akan memanggil fungsi `profile_view` yang telah kita buat di `views.py`.

### d. Membuat File `urls.py` di Aplikasi `a_users`

Buat file `urls.py` di dalam direktori aplikasi `a_users` dan tambahkan kode berikut:

```python
from django.urls import path
from .views import profile_view

urlpatterns = [
    path('', profile_view, name='profile'),
]
```

#### Penjelasan:

- **`path('', profile_view, name='profile')`**: Mendefinisikan URL untuk `profile/` tanpa tambahan path. URL ini akan memanggil fungsi `profile_view` yang telah kita buat di `views.py`.

### e. Menambahkan Link ke Header

**Di `templates/includes/header.html`:**

```django
<li><a href="{% url 'profile' %}">My Profile</a></li>
```

### f. Menjalankan Server dan Memeriksa Halaman Profil

Jalankan server dan akses `http://127.0.0.1:8000/profile/` untuk melihat halaman profil Anda.

---

## Penjelasan Mengapa Menambahkan Path di `urls.py`

### a. **Mengapa Menambahkan Path di `urls.py`?**

File `urls.py` dalam setiap aplikasi Django berfungsi sebagai router yang menentukan bagaimana permintaan HTTP (URL) diarahkan ke **views** tertentu. Menambahkan path di `urls.py` memungkinkan Django untuk mengetahui view mana yang harus dipanggil ketika pengguna mengakses URL tertentu.

#### Alasan Utama:

1. **Organisasi URL yang Modular:** Dengan memisahkan URL berdasarkan aplikasi, proyek menjadi lebih terstruktur dan mudah dikelola, terutama pada proyek besar dengan banyak aplikasi.
2. **Reusabilitas Aplikasi:** Jika Anda memutuskan untuk menggunakan ulang aplikasi `a_users` di proyek lain, cukup sertakan aplikasi tersebut dan tambahkan path yang diperlukan tanpa mengubah konfigurasi global.
3. **Keterbacaan dan Pemeliharaan:** Memisahkan URL berdasarkan aplikasi membuatnya lebih mudah dibaca dan dipelihara. Setiap aplikasi bertanggung jawab atas URL-nya sendiri.

### b. **Bagaimana Cara Kerjanya?**

1. **`a_core/urls.py`:**

   - File ini adalah konfigurasi URL utama untuk proyek Django Anda.
   - Dengan menambahkan `path('profile/', include('a_users.urls'))`, Anda memberi tahu Django bahwa semua URL yang dimulai dengan `profile/` akan diarahkan ke file `urls.py` di dalam aplikasi `a_users`.

   **Contoh `a_core/urls.py`:**

   ```python
   from django.contrib import admin
   from django.urls import path, include
   from a_home.views import home_view

   urlpatterns = [
       path('admin/', admin.site.urls),
       path('', home_view, name='home'),
       path('profile/', include('a_users.urls')),  # Menambahkan path ini
   ]
   ```

2. **`a_users/urls.py`:**

   - File ini berisi konfigurasi URL khusus untuk aplikasi `a_users`.
   - Dalam contoh ini, URL kosong (`''`) di aplikasi `a_users` akan diarahkan ke view `profile_view`. Jadi, mengakses `http://127.0.0.1:8000/profile/` akan memanggil `profile_view`.

   **Contoh `a_users/urls.py`:**

   ```python
   from django.urls import path
   from .views import profile_view

   urlpatterns = [
       path('', profile_view, name='profile'),  # URL kosong di aplikasi a_users
   ]
   ```

#### Diagram Alur:

Permintaan HTTP ➔ `a_core/urls.py` ➔ `profile/` ➔ `a_users/urls.py` ➔ `profile_view` (`a_users/views.py`)

Dengan demikian, Django mengetahui bahwa ketika pengguna mengakses `http://127.0.0.1:8000/profile/`, ia harus memanggil fungsi `profile_view` yang ada di aplikasi `a_users`.

---

## Halaman Edit Profil

Berikut adalah langkah-langkah untuk membuat halaman edit profil dalam aplikasi Django:

1. **Di `a_users/forms.py`:**

   Buat file `forms.py` di dalam direktori aplikasi `a_users` dan tambahkan kode berikut:

   ```python
   from django.forms import ModelForm
   from django import forms
   from .models import Profile

   class ProfileForm(ModelForm):
       class Meta:
           model = Profile
           exclude = ['user']
           widgets = {
               'image': forms.FileInput(),
               'displayname': forms.TextInput(attrs={'placeholder': 'Add display name'}),
               'info': forms.Textarea(attrs={'rows': 3, 'placeholder': 'Add information'})
           }
   ```

   *Penjelasan:*
   - Mengimpor `ModelForm` dari `django.forms` dan `forms`.
   - Mendefinisikan `ProfileForm` yang merupakan `ModelForm` berdasarkan model `Profile`.
   - Mengecualikan field `user` dari form.
   - Mengatur widget untuk field `image`, `displayname`, dan `info` dengan placeholder dan atribut lainnya.

2. **Di `a_users/views.py`:**

   Tambahkan fungsi `profile_edit_view`:

   ```python
   from django.shortcuts import render, redirect
   from django.contrib.auth.decorators import login_required
   from .forms import ProfileForm

   @login_required
   def profile_edit_view(request):
       form = ProfileForm(instance=request.user.profile)
       
       if request.method == 'POST':
           form = ProfileForm(request.POST, request.FILES, instance=request.user.profile)
           if form.is_valid():
               form.save()
               return redirect('profile')
       return render(request, 'a_users/profile_edit.html', {'form': form})
   ```

   *Penjelasan:*
   - Mengimpor `redirect` untuk melakukan redirect setelah form disubmit.
   - Mendefinisikan fungsi `profile_edit_view` untuk menampilkan form edit profil.
   - Menginisialisasi `ProfileForm` dengan instance profil pengguna saat ini.
   - Menangani permintaan POST saat form disubmit.
   - Jika form valid, simpan perubahan dan redirect ke halaman profil.

3. **Menambahkan URL untuk Edit Profil**

   **Di `a_users/urls.py`:**

   ```python
   from django.urls import path
   from .views import profile_view, profile_edit_view

   urlpatterns = [
       path('', profile_view, name='profile'),
       path('edit/', profile_edit_view, name='profile-edit'),  # Tambahkan ini
   ]
   ```

4. **Membuat Template `profile_edit.html` di `a_users/templates/a_users/`:**

   ```django
   {% extends 'layouts/blank.html' %}

   {% block content %}
   <h1 class="mb-4">Edit your Profile</h1>

   <div class="text-center flex flex-col items-center">
       <img id="avatar" class="w-36 h-36 rounded-full object-cover my-4" src="{{ user.profile.avatar }}" alt="Avatar"/>
       <div class="text-center max-w-md">
           <h1 id="displayname">{{ user.profile.displayname|default:"" }}</h1>
           <div class="text-gray-400 mb-2 -mt-3">@{{ user.username }}</div>
       </div>
   </div>
   <form method="POST" enctype="multipart/form-data">
       {% csrf_token %}
       {{ form.as_p }}
       <button type="submit">Submit</button>
       <a class="button button-gray ml-1" href="{{ request.META.HTTP_REFERER }}">Cancel</a>
   </form>

   <script>
       // Mengupdate avatar saat file gambar dipilih
       const fileInput = document.querySelector('input[type="file"]');

       fileInput.addEventListener('change', (event) => {
           const file = event.target.files[0];
           const image = document.querySelector('#avatar');

           if (file && file.type.includes('image')) {
               const url = URL.createObjectURL(file);
               image.src = url;
           }
       });

       // Mengupdate nama tampilan secara real-time
       const displayNameInput = document.getElementById('id_displayname');
       const displayNameOutput = document.getElementById('displayname');

       displayNameInput.addEventListener('input', (event) => {
           displayNameOutput.innerText = event.target.value;
       });
   </script>

   {% endblock %}
   ```

   *Penjelasan:*
   - Menggunakan template dasar `layouts/blank.html`.
   - Menampilkan form untuk mengedit profil, termasuk gambar avatar dan nama tampilan.
   - Menggunakan JavaScript untuk memperbarui avatar dan nama tampilan secara langsung saat pengguna melakukan perubahan.

5. **Mengedit `a_users/urls.py`:**

   Sudah dilakukan pada langkah 3 di atas.

6. **Memperbarui `templates/includes/header.html`:**

   ```django
   <li><a href="{% url "profile-edit" %}">Edit Profile</a></li>
   ```

7. **Membuat Folder `media` di Root Directory:**

   Buat folder `media` di root direktori proyek Anda untuk menyimpan file media yang diupload oleh pengguna.

   ```bash
   mkdir media
   ```

8. **Mengkonfigurasi `a_core/settings.py`:**

   ```python
   STATIC_URL = '/static/'
   STATICFILES_DIRS = [BASE_DIR / 'static']

   MEDIA_URL = '/media/'
   MEDIA_ROOT = BASE_DIR / 'media'
   ```

   *Penjelasan:*
   - Menentukan URL dan root directory untuk file media.

9. **Menambahkan Path untuk File Media di `a_core/urls.py`:**

   ```python
   from django.conf import settings
   from django.conf.urls.static import static

   urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
   ```

   *Penjelasan:*
   - Menambahkan konfigurasi untuk melayani file media selama pengembangan.

10. **Menguji Upload Gambar, Nama, dan Deskripsi:**

    - Refresh halaman dan coba upload gambar serta mengisi nama dan deskripsi untuk memastikan semuanya berfungsi.

11. **Menambahkan `django-cleanup` ke `INSTALLED_APPS`:**

    Di `a_core/settings.py`:

    ```python
    INSTALLED_APPS = [
        # ... aplikasi lainnya
        'django_cleanup.apps.CleanupConfig',
    ]
    ```

    *Penjelasan:*
    - `django-cleanup` akan otomatis menghapus file lama saat file baru diupload atau saat model dihapus.

12. **Menguji Dengan Mengganti Gambar:**

    - Coba ganti gambar profil untuk memastikan `django-cleanup` berfungsi dengan benar.

13. **Mengkonfigurasi `django-allauth`:**

    Di `a_core/settings.py`, tambahkan ke `INSTALLED_APPS`:

    ```python
    INSTALLED_APPS = [
        # ... aplikasi lainnya
        'allauth',
        'allauth.account',
        'allauth.socialaccount',
    ]
    ```

    Dan di `MIDDLEWARE`:

    ```python
    MIDDLEWARE = [
        # ... middleware lainnya
        'allauth.account.middleware.AccountMiddleware',
    ]
    ```

    *Penjelasan:*
    - Menambahkan aplikasi `django-allauth` untuk manajemen autentikasi dan akun.

14. **Menambahkan Path untuk `allauth` di `a_core/urls.py`:**

    ```python
    path('accounts/', include('allauth.urls')),
    ```

    *Penjelasan:*
    - Menyertakan URL patterns dari `django-allauth`.

15. **Menjalankan Migrasi Database:**

    ```bash
    python manage.py migrate
    ```

    *Penjelasan:*
    - Menerapkan migrasi yang diperlukan oleh `django-allauth`.

16. **Memperbarui Link Logout di `templates/includes/header.html`:**

    ```django
    <li><a href="{% url "account_logout" %}">Log Out</a></li>
    ```

    *Penjelasan:*
    - Mengubah link logout agar menggunakan URL dari `django-allauth`.

17. **Menguji Fungsi Logout:**

    - Logout dari aplikasi untuk memastikan integrasi dengan `django-allauth` berhasil.

18. **Membuat Template `templates/allauth/layouts/base.html`:**

    ```django
    {% extends "layouts/box.html" %}

    {% block class %} allauth {% endblock class %}

    {% block content %}
    {% endblock content %}
    ```

    *Penjelasan:*
    - Membuat template dasar untuk halaman-halaman `allauth`.

19. **Memperbarui Link Login di `templates/includes/header.html`:**

    ```django
    <li><a href="{% url "account_login" %}">Login</a></li>
    ```

    *Penjelasan:*
    - Mengubah link login agar menggunakan URL dari `django-allauth`.

20. **Mengkonfigurasi Pengaturan Tambahan di `a_core/settings.py`:**

    ```python
    LOGIN_REDIRECT_URL = '/'

    EMAIL_BACKEND = 'django.core.mail.backends.console.EmailBackend'
    ACCOUNT_AUTHENTICATION_METHOD = 'email'
    ACCOUNT_EMAIL_REQUIRED = True

    AUTHENTICATION_BACKENDS = [
        'django.contrib.auth.backends.ModelBackend',
        'allauth.account.auth_backends.AuthenticationBackend',
    ]
    ```

    *Penjelasan:*
    - Mengatur URL redirect setelah login.
    - Menggunakan backend email console untuk pengujian.
    - Mengatur metode autentikasi menggunakan email dan mewajibkan email.

21. **Menguji Fungsi Login:**

    - Coba login dengan email dan password untuk memastikan semuanya berfungsi.

22. **Membuat Template Pesan HTML Kosong:**

    Buat folder `templates/account/messages/` dan kosongkan isi file:

    - `logged_in.txt`
    - `logged_out.txt`

    *Penjelasan:*
    - Mengosongkan pesan default yang ditampilkan setelah login dan logout.

23. **Memperbarui Link Signup di `templates/includes/header.html`:**

    ```django
    <li><a href="{% url "account_signup" %}?next={% url 'profile-onboarding' %}">Signup</a></li>
    ```

    *Penjelasan:*
    - Menambahkan parameter `next` agar setelah signup, pengguna diarahkan ke halaman onboarding profil.

24. **Menambahkan URL Onboarding di `a_users/urls.py`:**

    ```python
    from django.urls import path
    from .views import profile_view, profile_edit_view

    urlpatterns = [
        path('', profile_view, name='profile'),
        path('edit/', profile_edit_view, name='profile-edit'),
        path('onboarding/', profile_edit_view, name='profile-onboarding'),  # Tambahkan ini
    ]
    ```

    *Penjelasan:*
    - Menambahkan URL pattern untuk proses onboarding yang menggunakan `profile_edit_view`.

25. **Memperbarui `profile_edit_view` di `a_users/views.py`:**

    ```python
    from django.shortcuts import render, redirect, get_object_or_404
    from django.contrib.auth.models import User
    from django.urls import reverse
    from django.contrib.auth.decorators import login_required
    from .forms import ProfileForm

    @login_required
    def profile_edit_view(request):
        form = ProfileForm(instance=request.user.profile)
        
        if request.method == 'POST':
            form = ProfileForm(request.POST, request.FILES, instance=request.user.profile)
            if form.is_valid():
                form.save()
                return redirect('profile')
        
        if request.path == reverse('profile-onboarding'):
            onboarding = True
        else:
            onboarding = False
            
        return render(request, 'a_users/profile_edit.html', {'form': form, 'onboarding': onboarding})
    ```

    *Penjelasan:*
    - Menambahkan logika untuk menentukan apakah pengguna sedang dalam proses onboarding atau tidak.
    - Menggunakan flag `onboarding` untuk menentukan tampilan yang sesuai.

26. **Memperbarui Heading di `profile_edit.html`:**

    ```django
    {% block content %}

    {% if onboarding %}
    <h1 class="mb-4">Complete your profile</h1>
    {% else %}
    <h1 class="mb-4">Edit your profile</h1>
    {% endif %}

    <div class="text-center flex flex-col items-center">
    ```

    *Penjelasan:*
    - Mengubah judul halaman berdasarkan apakah pengguna sedang onboarding atau mengedit profil.

27. **Memperbarui Tombol Cancel dalam Form:**

    ```django
    <form method="POST" enctype="multipart/form-data">
        {% csrf_token %}
        {{ form.as_p }}
        <button type="submit">Submit</button>
       
        {% if onboarding %}
        <a class="button button-gray ml-1" href="{% url "home" %}">Skip</a>
        {% else %}
        <a class="button button-gray ml-1" href="{{ request.META.HTTP_REFERER }}">Cancel</a>
        {% endif %}
        
    </form>
    ```

    *Penjelasan:*
    - Menambahkan logika untuk tombol "Skip" saat onboarding dan "Cancel" saat mengedit profil.

28. **Menambahkan Signal di `a_users/signals.py`:**

    ```python
    from django.db.models.signals import pre_save
    from django.dispatch import receiver
    from django.contrib.auth.models import User

    @receiver(pre_save, sender=User)
    def user_presave(sender, instance, **kwargs):
        if instance.username:
            instance.username = instance.username.lower()
    ```

    *Penjelasan:*
    - Menambahkan signal `pre_save` untuk memastikan username selalu disimpan dalam huruf kecil.

29. **Restart Server dan Uji Signup:**

    - Restart server dan coba proses signup untuk memastikan perubahan berfungsi.

30. **Menambahkan URL Profil dengan `@username` di `a_core/urls.py`:**

    ```python
    from a_users.views import profile_view

    urlpatterns = [
        # ... URL patterns lainnya
        path('@<str:username>/', profile_view, name='profile'),
    ]
    ```

    *Penjelasan:*
    - Menambahkan URL pattern untuk profil pengguna yang diakses dengan `@username`.

31. **Memperbarui `profile_view` di `a_users/views.py`:**

    ```python
    from django.shortcuts import render, redirect, get_object_or_404
    from django.contrib.auth.models import User

    def profile_view(request, username=None):
        if username:
            user = get_object_or_404(User, username=username)
            profile = user.profile
        else:
            try:
                profile = request.user.profile
            except:
                return redirect('account_login')
        return render(request, 'a_users/profile.html', {'profile': profile})
    ```

    *Penjelasan:*
    - Menambahkan logika untuk mendapatkan profil berdasarkan `username` yang diberikan.
    - Jika tidak ada `username`, coba dapatkan profil pengguna yang sedang login.

32. **Menguji Akses ke `http://127.0.0.1:8000/@usernameanda`:**

    - Coba akses URL profil dengan format `@username` untuk memastikan semuanya berfungsi.

---

Dengan mengikuti langkah-langkah di atas, Anda telah berhasil memperjelas dan mengimplementasikan fitur edit profil, integrasi dengan `django-allauth`, serta menambahkan URL profil dengan format `@username` dalam aplikasi Django Anda. Struktur header yang diperbaiki di dokumen Markdown ini akan membantu dalam navigasi dan pemeliharaan dokumentasi yang lebih baik.
