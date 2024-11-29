# Panduan Membuat Aplikasi Django dengan Fitur Pengguna Lengkap

Panduan ini akan memandu Anda melalui proses pembuatan aplikasi Django dengan fitur-fitur seperti halaman profil pengguna, edit profil, integrasi dengan `django-allauth` untuk manajemen akun, dan lain-lain. Berikut adalah langkah-langkah yang akan kita ikuti:

---

## **1. Membuat Folder Proyek dan Virtual Environment**

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
django-allauth
```

#### **Penjelasan `requirements.txt`:**

- **Django**: Framework web yang digunakan untuk membangun aplikasi web.
- **Pillow**: Library Python untuk memproses gambar, diperlukan jika Anda menangani upload gambar.
- **django-cleanup**: Mengelola file yang diunggah secara otomatis, seperti menghapus file lama saat objek dihapus atau diubah.
- **django-htmx**: Integrasi HTMX dengan Django untuk membuat interaksi frontend yang dinamis tanpa harus menulis banyak JavaScript.
- **django-allauth**: Paket untuk manajemen autentikasi, pendaftaran, dan login pengguna dengan berbagai metode, termasuk email dan sosial media.

### d. Instalasi Dependensi

Jalankan perintah berikut untuk menginstal semua paket yang tercantum di `requirements.txt`:

```bash
pip install -r requirements.txt
```

---

## **2. Memulai Proyek Django**

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

## **3. Membuat Aplikasi Homepage**

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

## **4. Membuat View untuk Homepage**

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

## **5. Mengatur Template**

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
                # ...
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
2. **Salin kontennya dan buat file `base.html` di folder `templates` proyek Anda:**

   ```bash
   touch templates/base.html
   ```

3. **Buka `base.html` di editor teks (misalnya VS Code) dan paste konten yang telah disalin.**

#### ii. Memisahkan Komponen ke dalam Includes

Untuk menjaga kebersihan dan modularitas template, pisahkan beberapa elemen ke dalam file terpisah.

1. **Memisahkan Pesan (Messages)**

   - **Potong Elemen Messages:** Di `base.html`, potong bagian yang menampilkan pesan (biasanya menggunakan `{% for message in messages %}`).
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

## **6. Penjelasan Struktur Template**

### a. **Hierarki Template dan Pewarisan**

- **`base.html`**: Template dasar yang berfungsi sebagai kerangka utama untuk semua halaman dalam aplikasi Anda. Di dalamnya terdapat elemen-elemen umum seperti header, footer, dan area pesan.
- **`layouts/blank.html`**: Mewarisi dari `base.html`. Template ini mendefinisikan blok `layout` dan `content` yang akan diisi oleh halaman-halaman spesifik.
- **`home.html`**: Mewarisi dari `layouts/blank.html`. Template ini mengisi blok `content` dengan konten khusus untuk homepage.

### b. **Penggunaan Blok dan Include**

- **Blok (`{% block %}`)**: Digunakan untuk mendefinisikan area yang dapat diisi atau ditimpa oleh template turunan.
- **Include (`{% include %}`)**: Digunakan untuk menyertakan template lain, seperti `header.html` dan `messages.html`, sehingga kode lebih modular dan mudah dikelola.

### c. **Alur Pewarisan Template**

1. **`home.html`** mewarisi dari **`layouts/blank.html`**.
2. **`layouts/blank.html`** mewarisi dari **`base.html`**.
3. **`base.html`** menyediakan kerangka dasar, termasuk header dan messages.
4. **`blank.html`** mendefinisikan blok `layout` dan `content` yang dapat diisi oleh template turunan.
5. **`home.html`** mengisi blok `content` dengan konten spesifik untuk homepage.

Dengan struktur ini, Anda dapat dengan mudah membuat halaman-halaman lain yang memiliki layout serupa dengan hanya mewarisi dari `blank.html` atau bahkan membuat layout baru berdasarkan kebutuhan.

---

## **7. Menambahkan Logo dengan File Statis**

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

## **8. Membuat Halaman Profil Pengguna**

### 8.1 Membuat Aplikasi `a_users`

Buat aplikasi Django bernama `a_users`:

```bash
python manage.py startapp a_users
```

### 8.2 Menambahkan `a_users` ke `INSTALLED_APPS`

Buka file `a_core/settings.py` dan tambahkan `'a_users'` ke dalam daftar `INSTALLED_APPS`:

```python
INSTALLED_APPS = [
    # Aplikasi bawaan Django
    # ...
    'a_home',
    'a_users',  # Tambahkan ini
]
```

### 8.3 Membuat Model `Profile`

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

#### **Penjelasan Model `Profile`:**

- **`user`**: Hubungan one-to-one dengan model `User` bawaan Django. Setiap pengguna memiliki satu profil.
- **`image`**: Field untuk menyimpan gambar profil pengguna. File gambar akan diunggah ke folder `avatars/` di dalam folder media Anda. Field ini opsional.
- **`displayname`**: Nama tampilan pengguna yang bisa berbeda dari username asli. Juga opsional.
- **`info`**: Informasi tambahan tentang pengguna.
- **`__str__`**: Mengembalikan representasi string dari objek `Profile`, yaitu nama pengguna.
- **`name`**: Properti yang mengembalikan `displayname` jika diisi; jika tidak, mengembalikan `username` pengguna.
- **`avatar`**: Properti yang mengembalikan URL gambar avatar pengguna. Jika pengguna belum mengunggah gambar, akan mengembalikan URL default (`images/avatar.svg`).

### 8.4 Melakukan Migrasi Database

Setelah membuat model, Anda perlu membuat migrasi untuk menerapkannya ke database.

```bash
python manage.py makemigrations
python manage.py migrate
```

### 8.5 Mendaftarkan Model di Admin

**Di `a_users/admin.py`:**

```python
from django.contrib import admin
from .models import Profile

admin.site.register(Profile)
```

### 8.6 Mengatur Sinyal di `signals.py` dan `apps.py`

**Membuat `signals.py` di `a_users`:**

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

**Mengonfigurasi `apps.py` untuk Sinyal:**

Di `a_users/apps.py`:

```python
from django.apps import AppConfig

class AUsersConfig(AppConfig):
    name = 'a_users'

    def ready(self):
        import a_users.signals
```

### 8.7 Membuat View `profile_view`

**Di `a_users/views.py`:**

```python
from django.shortcuts import render, redirect, get_object_or_404
from django.contrib.auth.decorators import login_required
from django.contrib.auth.models import User

@login_required
def profile_view(request, username=None):
    if username:
        profile = get_object_or_404(User, username=username).profile
    else:
        profile = request.user.profile
    return render(request, 'a_users/profile.html', {'profile': profile})
```

### 8.8 Membuat Template `profile.html`

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

### 8.9 Menambahkan URL untuk Profil

**Di `a_core/urls.py`:**

```python
from django.contrib import admin
from django.urls import path, include
from a_home.views import home_view
from a_users.views import profile_view

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', home_view, name='home'),
    path('@<username>/', profile_view, name='profile'),
]
```

### 8.10 Menambahkan Link ke Header

**Di `templates/includes/header.html`:**

```django
<li><a href="{% url 'profile' username=request.user.username %}">My Profile</a></li>
```

### 8.11 Menjalankan Server dan Memeriksa Halaman Profil

Jalankan server dan akses profil Anda melalui URL seperti `http://127.0.0.1:8000/@yourusername/`.

---

## **9. Menambahkan Avatar dan Username ke Header**

### 9.1 Menghubungkan Avatar dan Username dengan Model `Profile`

Pastikan bahwa model `Profile` sudah terhubung dengan model `User` dan properti `avatar` serta `name` telah didefinisikan.

### 9.2 Memodifikasi `header.html` untuk Menampilkan Avatar dan Username

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
                    <a href="{% url 'profile' username=request.user.username %}">
                        <img class="h-8 w-8 rounded-full" src="{{ request.user.profile.avatar }}" alt="Avatar"/>
                        {{ request.user.profile.name }}
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

### 9.3 Menambahkan Dropdown Menu (Opsional)

Jika Anda ingin menambahkan menu dropdown, Anda bisa menggunakan JavaScript atau library seperti Alpine.js untuk menampilkan menu dengan opsi seperti "Edit Profile", "Settings", dan "Logout".

### 9.4 Menguji Tampilan di Browser

Jalankan server dan buka halaman utama. Pastikan avatar dan nama pengguna Anda muncul di header jika Anda sudah login.

---

## **10. Mengkonfigurasi Media Files**

Agar pengguna dapat mengunggah avatar mereka, Anda perlu mengatur media files.

### a. Menambahkan Konfigurasi Media di `settings.py`

```python
MEDIA_URL = '/media/'
MEDIA_ROOT = BASE_DIR / 'media'
```

### b. Menambahkan Konfigurasi URL untuk Media Files di `a_core/urls.py`

```python
from django.conf import settings
from django.conf.urls.static import static

urlpatterns = [
    # ... URL patterns Anda ...
] + static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

### c. Menjalankan dan Menguji Upload Avatar

Setelah konfigurasi ini, Anda dapat membuat form untuk mengedit profil dan mengunggah avatar.

---

## **11. Menambahkan Fitur Edit Profil**

### a. Membuat Form `ProfileForm` di `a_users/forms.py`

```python
from django import forms
from .models import Profile

class ProfileForm(forms.ModelForm):
    class Meta:
        model = Profile
        exclude = ['user']
        widgets = {
            'image': forms.FileInput(),
            'displayname': forms.TextInput(attrs={'placeholder': 'Add display name'}),
            'info': forms.Textarea(attrs={'rows': 3, 'placeholder': 'Add information'})
        }
```

### b. Membuat View `profile_edit_view` di `a_users/views.py`

```python
from django.contrib.auth.decorators import login_required
from django.shortcuts import render, redirect

@login_required
def profile_edit_view(request):
    form = ProfileForm(instance=request.user.profile)

    if request.method == 'POST':
        form = ProfileForm(request.POST, request.FILES, instance=request.user.profile)
        if form.is_valid():
            form.save()
            return redirect('profile', username=request.user.username)

    return render(request, 'a_users/profile_edit.html', {'form': form})
```

### c. Menambahkan URL untuk Edit Profil

**Di `a_users/urls.py`:**

```python
from django.urls import path
from .views import profile_edit_view

urlpatterns = [
    path('edit/', profile_edit_view, name='profile-edit'),
]
```

**Di `a_core/urls.py`:**

Pastikan `a_users.urls` di-include:

```python
from django.urls import path, include

urlpatterns = [
    # ...
    path('profile/', include('a_users.urls')),
]
```

### d. Membuat Template `profile_edit.html`

**Di `templates/a_users/profile_edit.html`:**

```django
{% extends 'layouts/blank.html' %}

{% block content %}

<h1 class="mb-4">Edit your Profile</h1>

<div class="text-center flex flex-col items-center">
    <img id="avatar" class="w-36 h-36 rounded-full object-cover my-4" src="{{ user.profile.avatar }}" />
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

### e. Menambahkan Link Edit Profil ke Header

**Di `templates/includes/header.html`:**

```django
<li><a href="{% url 'profile-edit' %}">Edit Profile</a></li>
```

### f. Menguji Fitur Edit Profil

Jalankan server dan akses halaman edit profil melalui link yang telah ditambahkan. Coba unggah avatar baru dan perbarui informasi profil Anda.

---

## **12. Mengintegrasikan `django-allauth` untuk Manajemen Akun**

### a. Menambahkan `django-allauth` ke `INSTALLED_APPS`

Di `a_core/settings.py`:

```python
INSTALLED_APPS = [
    # Aplikasi bawaan Django
    # ...
    'django.contrib.sites',  # Pastikan ini ditambahkan
    'allauth',
    'allauth.account',
    'allauth.socialaccount',
    # Aplikasi kita
    'a_home',
    'a_users',
]
```

### b. Menambahkan Middleware `AccountMiddleware`

```python
MIDDLEWARE = [
    # Middleware lainnya
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'allauth.account.middleware.AccountMiddleware',
]
```

### c. Menambahkan Konfigurasi URL untuk `allauth`

Di `a_core/urls.py`:

```python
from django.urls import path, include

urlpatterns = [
    # ...
    path('accounts/', include('allauth.urls')),
]
```

### d. Menjalankan Migrasi Database

```bash
python manage.py migrate
```

### e. Mengkonfigurasi Pengaturan Tambahan di `settings.py`

```python
AUTHENTICATION_BACKENDS = (
    'django.contrib.auth.backends.ModelBackend',
    'allauth.account.auth_backends.AuthenticationBackend',
)

SITE_ID = 1

LOGIN_REDIRECT_URL = '/'
ACCOUNT_AUTHENTICATION_METHOD = 'email'
ACCOUNT_EMAIL_REQUIRED = True
ACCOUNT_EMAIL_VERIFICATION = 'mandatory'

EMAIL_BACKEND = 'django.core.mail.backends.console.EmailBackend'
```

### f. Memperbarui Link Login dan Logout di Header

**Di `templates/includes/header.html`:**

```django
<li><a href="{% url 'account_login' %}">Login</a></li>
<li><a href="{% url 'account_logout' %}">Logout</a></li>
```

### g. Menguji Integrasi `django-allauth`

- Jalankan server dan akses halaman login dan signup.
- Coba mendaftar akun baru dan pastikan proses verifikasi email berjalan (email akan ditampilkan di console).

---

## **13. Menambahkan Halaman Pengaturan Akun**

### a. Membuat View `profile_setting_view` di `a_users/views.py`

```python
@login_required
def profile_setting_view(request):
    return render(request, 'a_users/profile_settings.html')
```

### b. Membuat Template `profile_settings.html`

**Di `templates/a_users/profile_settings.html`:**

```django
{% extends 'layouts/box.html' %}

{% block content %}

<h1 class="mb-8">Account Settings</h1>

<table class="w-full text-sm text-left text-gray-500">
    <tbody>
        <tr>
            <th scope="row" class="pt-4 pb-1 text-base font-bold text-gray-900">
                Email address
            </th>
            <td id="email-address" class="pt-4 pb-1 px-6">
                {% if user.email %}{{ user.email }}{% else %}No Email{% endif %}
            </td>
            <td class="pt-4 pb-1 px-6">
                <a id="email-edit" href="" class="font-medium text-blue-600 hover:underline">
                    Edit
                </a>
            </td>
        </tr>
        <tr class="border-b">
            <th scope="row" class="pb-4 font-medium text-gray-900"></th>
            <td class="px-6 pb-4">
                {% if user.emailaddress_set.first.verified %}
                <span class="text-green-500">Verified</span>
                {% else %}
                <span class="text-amber-500">Not verified</span>
                {% endif %}
            </td>
            <td class="px-6 pb-4">
                <a href="{% url 'profile-emailverify' %}" class="font-medium text-blue-600 hover:underline">
                    {% if not user.emailaddress_set.first.verified %}Verify{% endif %}
                </a>
            </td>
        </tr>

        <tr class="border-b">
            <th scope="row" class="py-4 text-base font-bold text-gray-900">
                Delete Account
            </th>
            <td class="px-6 py-4">
                Once deleted, account is gone. Forever.
            </td>
            <td class="px-6 py-4">
                <a href="{% url 'profile-delete' %}" class="font-medium text-red-600 hover:underline">
                    Delete
                </a>
            </td>
        </tr>
    </tbody>
</table>

{% endblock %}
```

### c. Menambahkan URL untuk Pengaturan Akun

**Di `a_users/urls.py`:**

```python
from .views import profile_setting_view, profile_delete_view, profile_emailverify

urlpatterns = [
    # ...
    path('settings/', profile_setting_view, name="profile-settings"),
    path('delete/', profile_delete_view, name="profile-delete"),
    path('emailverify/', profile_emailverify, name="profile-emailverify"),
]
```

### d. Menambahkan Link "Settings" di Header

**Di `templates/includes/header.html`:**

```django
<li><a href="{% url 'profile-settings' %}">Settings</a></li>
```

### e. Menguji Halaman Settings

Jalankan server dan akses halaman pengaturan akun untuk memastikan semuanya tampil dengan benar.

---

## **14. Menambahkan Fitur Verifikasi Email dan Penghapusan Akun**

### a. Membuat View `profile_emailverify` di `a_users/views.py`

```python
from allauth.account.utils import send_email_confirmation

@login_required
def profile_emailverify(request):
    send_email_confirmation(request, request.user)
    return redirect('profile-settings')
```

### b. Membuat View `profile_delete_view` di `a_users/views.py`

```python
from django.contrib.auth import logout
from django.contrib import messages

@login_required
def profile_delete_view(request):
    user = request.user
    if request.method == "POST":
        logout(request)
        user.delete()
        messages.success(request, 'Account deleted, what a pity')
        return redirect('home')

    return render(request, 'a_users/profile_delete.html')
```

### c. Membuat Template `profile_delete.html`

**Di `templates/a_users/profile_delete.html`:**

```django
{% extends 'layouts/box.html' %}

{% block content %}

<h1>Delete Account</h1>
<p>Are you sure you want to delete your account?</p>

<form method='POST'>
    {% csrf_token %}
    <button type="submit" class="button-red mt-6">Yes, I want to delete my account</button>
    <a class="button button-gray ml-1" href="{{ request.META.HTTP_REFERER }}">Cancel</a>
</form>

{% endblock %}
```

### d. Menguji Fitur Verifikasi Email dan Penghapusan Akun

- Coba klik link "Verify" di halaman pengaturan akun untuk mengirim ulang email verifikasi.
- Coba hapus akun Anda melalui halaman penghapusan akun.

---

## **15. Membuat Halaman 404 Kustom**

### a. Membuat Template `404.html` di `templates/`

```django
{% extends 'layouts/blank.html' %}

{% block content %}

<div class="max-w-4xl mx-auto px-8 py-24 text-center">
    <h1 class="text-6xl">Page not found</h1>
    <p class="my-10 text-gray-500">
        Error 404: This page doesn't exist or is unavailable
    </p>
</div>

{% endblock %}
```

### b. Mengkonfigurasi Pengaturan Django untuk Menggunakan Halaman 404 Kustom

Di `a_core/settings.py`, ubah pengaturan:

```python
DEBUG = False

ALLOWED_HOSTS = ['localhost', '127.0.0.1', 'yourdomain.com']
```

**Catatan Penting:**

- Saat `DEBUG = False`, Django tidak akan melayani file statis dan media secara default. Untuk pengujian lokal, Anda mungkin perlu mengatur `DEBUG = True` atau mengonfigurasi server web seperti Nginx atau Apache untuk melayani file statis dan media.
- Pastikan `ALLOWED_HOSTS` diisi dengan benar untuk lingkungan produksi.

### c. Menguji Halaman 404 Kustom

Jalankan server dan akses URL yang tidak ada untuk memastikan halaman 404 kustom muncul sesuai dengan template yang dibuat.

---

## **16. Kesimpulan**

Anda telah berhasil membangun fitur-fitur utama dalam aplikasi Django, termasuk:

- **Halaman Edit Profil**
- **Integrasi `django-allauth` untuk manajemen akun**
- **URL Profil dengan `@username`**
- **Halaman Pengaturan Akun**
- **Verifikasi Email**
- **Penghapusan Akun**
- **Halaman 404 Kustom**

---
