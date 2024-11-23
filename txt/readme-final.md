

### **1. Membuat Halaman Edit Profil**

**a. Membuat Formulir Profil di `a_users/forms.py`:**

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
            'info': forms.Textarea(attrs={'row': 3, 'placeholder': 'Add information'})
        }
```

- **Penjelasan:**
  - Mengimpor `ModelForm` dan `forms` dari `django.forms`.
  - Membuat `ProfileForm` yang terkait dengan model `Profile`.
  - Mengecualikan field `user` agar tidak ditampilkan dalam form.
  - Menyesuaikan widget untuk field `image`, `displayname`, dan `info` dengan placeholder dan atribut tambahan.

**b. Membuat View untuk Edit Profil di `a_users/views.py`:**

```python
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

- **Penjelasan:**
  - Menggunakan dekorator `@login_required` untuk memastikan hanya pengguna terautentikasi yang dapat mengakses view ini.
  - Jika metode permintaan adalah `POST`, form akan memproses data yang dikirimkan.
  - Jika form valid, data akan disimpan dan pengguna akan diarahkan ke halaman profil.

**c. Menambahkan URL untuk Halaman Edit Profil di `a_users/urls.py`:**

```python
from .views import profile_edit_view

urlpatterns = [
    # URL lainnya
    path('edit/', profile_edit_view, name="profile-edit"),
]
```

- **Penjelasan:**
  - Menambahkan path untuk halaman edit profil yang mengarah ke `profile_edit_view`.

**d. Membuat Template `profile_edit.html` di `a_users/templates/a_users/`:**

```html
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

- **Penjelasan:**
  - Menampilkan form edit profil dengan preview avatar dan nama tampilan yang dapat diperbarui secara real-time menggunakan JavaScript.

**e. Memperbarui Header untuk Menambahkan Link Edit Profil:**

Di `templates/includes/header.html`:

```html
<li><a href="{% url "profile-edit" %}">Edit Profile</a></li>
```

- **Penjelasan:**
  - Menambahkan link "Edit Profile" yang mengarah ke halaman edit profil.

**f. Membuat Template Layout `box.html`:**

Di `templates/layouts/box.html`:

```html
{% extends 'base.html' %}

{% block layout %}
<content class="block max-w-3xl mx-auto md:p-12">
    <div class="bg-white rounded-2xl md:drop-shadow-2xl shadow-black w-full p-8">
        {% block content %} 
        {% endblock %}
    </div>
</content>
{% endblock %}
```

- **Penjelasan:**
  - Template ini digunakan untuk memberikan tampilan kotak pada konten halaman.

**g. Mengubah Template Induk di `profile_edit.html`:**

Mengubah baris:

```html
{% extends 'layouts/blank.html' %}
```

Menjadi:

```html
{% extends 'layouts/box.html' %}
```

- **Penjelasan:**
  - Menggunakan template layout `box.html` yang baru dibuat untuk halaman edit profil.

**h. Mengkonfigurasi Media Files di `a_core/settings.py`:**

```python
STATIC_URL = 'static/'
STATICFILES_DIRS = [BASE_DIR / 'static']

MEDIA_URL = 'media/'
MEDIA_ROOT = BASE_DIR / 'media'
```

- **Penjelasan:**
  - Menentukan URL dan root directory untuk file media yang diupload pengguna.

**i. Menambahkan Konfigurasi URL untuk Media Files di `a_core/urls.py`:**

```python
from django.conf import settings
from django.conf.urls.static import static

urlpatterns = [
    # URL lainnya
] + static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

- **Penjelasan:**
  - Menambahkan konfigurasi agar Django dapat melayani file media selama pengembangan.

**j. Menginstal dan Mengkonfigurasi `django-cleanup`:**

- **Instalasi:**

  ```bash
  pip install django-cleanup
  ```

- **Menambahkan ke `INSTALLED_APPS` di `a_core/settings.py`:**

  ```python
  'django_cleanup.apps.CleanupConfig',
  ```

- **Penjelasan:**
  - `django-cleanup` secara otomatis menghapus file lama saat file baru diupload atau saat instance model dihapus.

---

### **2. Integrasi `django-allauth` untuk Manajemen Akun**

**a. Menambahkan `django-allauth` ke `INSTALLED_APPS` di `a_core/settings.py`:**

```python
'django.contrib.sites',  # Pastikan ini ada
'allauth',
'allauth.account',
'allauth.socialaccount',
```

- **Penjelasan:**
  - Menginstal `django-allauth` untuk mengelola autentikasi, pendaftaran, dan manajemen akun.

**b. Menambahkan Middleware `AccountMiddleware`:**

```python
'django.contrib.auth.middleware.AuthenticationMiddleware',
'allauth.account.middleware.AccountMiddleware',
```

**c. Menambahkan Konfigurasi URL untuk `allauth` di `a_core/urls.py`:**

```python
from django.urls import path, include

urlpatterns = [
    # URL lainnya
    path('accounts/', include('allauth.urls')),
]
```

**d. Menjalankan Migrasi Database:**

```bash
python manage.py migrate
```

**e. Mengkonfigurasi Pengaturan Tambahan di `a_core/settings.py`:**

```python
LOGIN_REDIRECT_URL = '/'
EMAIL_BACKEND = 'django.core.mail.backends.console.EmailBackend'
ACCOUNT_AUTHENTICATION_METHOD = 'email'
ACCOUNT_EMAIL_REQUIRED = True
SITE_ID = 1  # Tambahkan ini
```

- **Penjelasan:**
  - Mengatur URL redirect setelah login.
  - Menggunakan backend email console untuk pengujian.
  - Mengatur metode autentikasi menggunakan email dan mewajibkan email.
  - Menentukan `SITE_ID` untuk `django.contrib.sites`.

**f. Memperbarui Link Login dan Logout di Header:**

Di `templates/includes/header.html`:

```html
<li><a href="{% url "account_login" %}">Login</a></li>
<li><a href="{% url "account_logout" %}">Log Out</a></li>
```

**g. Membuat Template Dasar untuk `allauth` di `templates/allauth/layouts/base.html`:**

```html
{% extends "layouts/box.html" %}

{% block class %} allauth {% endblock class %}

{% block content %}
{% endblock content %}
```

- **Penjelasan:**
  - Template ini digunakan sebagai dasar untuk halaman-halaman `allauth`.

**h. Mengosongkan Pesan Default `allauth`:**

Buat folder `templates/account/messages/` dan kosongkan isi file:

- `logged_in.txt`
- `logged_out.txt`

- **Penjelasan:**
  - Mengosongkan pesan default yang ditampilkan setelah login dan logout untuk menghindari tampilan pesan yang tidak diinginkan.

**i. Menambahkan Link Signup dengan Redirect ke Onboarding:**

Di `templates/includes/header.html`:

```html
<li><a href="{% url "account_signup" %}?next={% url 'profile-onboarding'%}">Signup</a></li>
```

- **Penjelasan:**
  - Menambahkan parameter `next` agar setelah signup, pengguna diarahkan ke halaman onboarding profil.

**j. Menambahkan URL dan View untuk Onboarding:**

- **Di `a_users/urls.py`:**

  ```python
  path('onboarding/', profile_edit_view, name="profile-onboarding"),
  ```

- **Di `a_users/views.py`:**

  ```python
  from django.urls import reverse

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

- **Penjelasan:**
  - Menambahkan logika untuk menentukan apakah pengguna sedang dalam proses onboarding atau mengedit profil biasa.
  - Menggunakan flag `onboarding` untuk menyesuaikan tampilan dan perilaku.

**k. Memperbarui Template `profile_edit.html` untuk Onboarding:**

```html
{% if onboarding %}
<h1 class="mb-4">Complete your profile</h1>
{% else %}
<h1 class="mb-4">Edit your profile</h1>
{% endif %}
```

- **Penjelasan:**
  - Mengubah judul halaman berdasarkan konteks (onboarding atau edit profil).

**l. Menyesuaikan Tombol "Cancel" atau "Skip":**

```html
{% if onboarding %}
<a class="button button-gray ml-1" href="{% url "home" %}">Skip</a>
{% else %}
<a class="button button-gray ml-1" href="{{ request.META.HTTP_REFERER }}">Cancel</a>
{% endif %}
```

- **Penjelasan:**
  - Saat onboarding, tombol akan menjadi "Skip" yang mengarah ke halaman home.

---

### **3. Menambahkan Signal untuk Menormalisasi Username**

**a. Menambahkan Signal di `a_users/signals.py`:**

```python
from django.dispatch import receiver
from django.db.models.signals import pre_save
from django.contrib.auth.models import User

@receiver(pre_save, sender=User)
def user_presave(sender, instance, **kwargs):
    if instance.username:
        instance.username = instance.username.lower()
```

- **Penjelasan:**
  - Signal `pre_save` ini memastikan bahwa username selalu disimpan dalam huruf kecil.

**b. Mengimpor Signal di `a_users/apps.py`:**

Pastikan signal terdaftar dengan menambahkan di `a_users/apps.py`:

```python
from django.apps import AppConfig

class AUsersConfig(AppConfig):
    name = 'a_users'

    def ready(self):
        import a_users.signals
```

---

### **4. Menambahkan URL Profil dengan `@username`**

**a. Menambahkan URL Pattern di `a_core/urls.py`:**

```python
from a_users.views import profile_view

urlpatterns = [
    # URL lainnya
    path('@<username>/', profile_view, name='profile'),
]
```

**b. Memperbarui View `profile_view` di `a_users/views.py`:**

```python
from django.shortcuts import render, redirect, get_object_or_404
from django.contrib.auth.models import User

def profile_view(request, username=None):
    if username:
        profile = get_object_or_404(User, username=username).profile
    else:
        if request.user.is_authenticated:
            profile = request.user.profile
        else:
            return redirect('account_login')
    return render(request, 'a_users/profile.html', {'profile': profile})
```

- **Penjelasan:**
  - View ini memungkinkan akses ke profil pengguna melalui URL dengan format `@username`.

---

### **5. Menambahkan Halaman Pengaturan Akun**

**a. Membuat View `profile_setting_view` di `a_users/views.py`:**

```python
@login_required
def profile_setting_view(request):
    return render(request, 'a_users/profile_settings.html')
```

**b. Membuat Template `profile_settings.html` di `a_users/templates/a_users/`:**

```html
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
                <a href="" class="font-medium text-blue-600 hover:underline">
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
                <a href="" class="font-medium text-red-600 hover:underline">
                    Delete
                </a>
            </td>
        </tr>
    </tbody>
</table>

{% endblock %}
```

**c. Menambahkan URL untuk Halaman Pengaturan di `a_users/urls.py`:**

```python
path('settings/', profile_setting_view, name="profile-settings"),
```

**d. Menambahkan Link "Settings" di Header:**

Di `templates/includes/header.html`:

```html
<li><a href="{% url "profile-settings" %}">Settings</a></li>
```

---

### **6. Mengintegrasikan HTMX untuk Perubahan Email Dinamis**

**a. Menambahkan `django-htmx` ke `INSTALLED_APPS` dan Middleware:**

Di `a_core/settings.py`:

```python
INSTALLED_APPS = [
    # Aplikasi lainnya
    'django_htmx',
]

MIDDLEWARE = [
    # Middleware lainnya
    'django_htmx.middleware.HtmxMiddleware',
]
```

- **Penjelasan:**
  - `django-htmx` memungkinkan integrasi HTMX dengan Django.
  - `HtmxMiddleware` diperlukan untuk memproses permintaan HTMX.

**b. Membuat Formulir `EmailForm` di `a_users/forms.py`:**

```python
from django.contrib.auth.models import User

class EmailForm(ModelForm):
    email = forms.EmailField(required=True)

    class Meta:
        model = User
        fields = ['email']
```

**c. Membuat View `profile_emailchange` di `a_users/views.py`:**

```python
from django.contrib import messages
from allauth.account.utils import send_email_confirmation

@login_required
def profile_emailchange(request):
    if request.htmx:
        form = EmailForm(instance=request.user)
        return render(request, 'partials/email_form.html', {'form': form})

    if request.method == 'POST':
        form = EmailForm(request.POST, instance=request.user)
        if form.is_valid():
            email = form.cleaned_data['email']
            if User.objects.filter(email=email).exclude(id=request.user.id).exists():
                messages.warning(request, f'{email} is already in use')
                return redirect('profile-settings')

            form.save()
            send_email_confirmation(request, request.user)
            return redirect('profile-settings')
        else:
            messages.warning(request, 'Invalid email')
            return redirect('profile-settings')

    return redirect('home')
```

**d. Membuat Template `email_form.html` di `templates/partials/`:**

```html
<form action="{% url 'profile-emailchange' %}" method="post" class="flex gap-2 text-black" autocomplete="off">
    {% csrf_token %}
    {% for field in form %}{{ field }}{% endfor %}
    <button class="block !bg-gray-800" type="submit">Submit</button>
</form>

<a hx-swap-oob="true" id="email-edit" href="{% url 'profile-settings' %}" class="font-medium text-blue-600 hover:underline">
    Cancel
</a>
```

**e. Memperbarui Link "Edit" di `profile_settings.html` untuk Menggunakan HTMX:**

```html
<td class="pt-4 pb-1 px-6">
    <a id="email-edit" class="cursor-pointer font-medium text-blue-600 hover:underline"
        hx-get="{% url "profile-emailchange" %}"
        hx-target="#email-address"
        hx-swap="innerHTML">
        Edit
    </a>
</td>
```

**f. Menambahkan URL untuk `profile_emailchange` di `a_users/urls.py`:**

```python
path('emailchange/', profile_emailchange, name="profile-emailchange"),
```

**g. Memperbarui Signal di `a_users/signals.py` untuk Menangani Perubahan Email:**

```python
from django.dispatch import receiver
from django.db.models.signals import post_save
from django.contrib.auth.models import User
from .models import Profile
from allauth.account.models import EmailAddress

@receiver(post_save, sender=User)
def user_postsave(sender, instance, created, **kwargs):
    user = instance

    if created:
        Profile.objects.create(user=user)
    else:
        try:
            email_address = EmailAddress.objects.get_primary(user)
            if email_address.email != user.email:
                email_address.email = user.email
                email_address.verified = False
                email_address.save()
        except EmailAddress.DoesNotExist:
            EmailAddress.objects.create(
                user=user,
                email=user.email,
                verified=False,
                primary=True
            )
```

---

### **7. Menguji dan Memvalidasi Fitur yang Dibuat**

- **Menguji Halaman Edit Profil:**
  - Mengakses halaman edit profil dan memastikan form bekerja dengan baik.
  - Mengupload gambar avatar, mengedit nama tampilan, dan informasi lainnya.

- **Menguji Integrasi `django-allauth`:**
  - Mencoba proses login, logout, dan signup.
  - Memastikan pengguna diarahkan ke halaman onboarding setelah signup.

- **Menguji URL Profil dengan `@username`:**
  - Mengakses profil pengguna melalui URL seperti `http://localhost:8000/@username`.

- **Menguji Halaman Pengaturan Akun:**
  - Mengakses halaman pengaturan dan memastikan informasi ditampilkan dengan benar.

- **Menguji Perubahan Email dengan HTMX:**
  - Klik "Edit" pada email di halaman pengaturan dan memastikan form muncul tanpa reload halaman.
  - Mengubah email dan memastikan pesan verifikasi dikirim.

- **Menguji Signal untuk Perubahan Email:**
  - Memastikan email di model `EmailAddress` dari `allauth` diperbarui dan status verifikasi diatur ulang.

- **Mengonfirmasi Email Melalui Konsol:**
  - Karena menggunakan `EMAIL_BACKEND = 'django.core.mail.backends.console.EmailBackend'`, email konfirmasi akan muncul di konsol.
  - Menyalin link konfirmasi dan mengaksesnya melalui browser untuk memverifikasi email.

---

Dengan mengikuti semua langkah di atas, kita telah berhasil membangun fitur-fitur utama dalam aplikasi Django, termasuk halaman edit profil, integrasi autentikasi dengan `django-allauth`, interaksi dinamis menggunakan HTMX, dan manajemen akun yang komprehensif.

