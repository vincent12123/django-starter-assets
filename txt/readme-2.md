**Halaman Edit Profil**

Berikut adalah langkah-langkah untuk membuat halaman edit profil dalam aplikasi Django:

1. **Di `a_users/forms.py`:**

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

   *Penjelasan:*
   - Mengimpor `ModelForm` dari `django.forms` dan `forms`.
   - Mendefinisikan `ProfileForm` yang merupakan `ModelForm` berdasarkan model `Profile`.
   - Mengecualikan field `user` dari form.
   - Mengatur widget untuk field `image`, `displayname`, dan `info` dengan placeholder dan atribut lainnya.

2. **Di `a_users/views.py`:**

   ```python
   def profile_edit_view(request):
       form = ProfileForm(instance=request.user.profile)
       return render(request, 'a_users/profile_edit.html', {'form': form})
   ```

   *Penjelasan:*
   - Mendefinisikan fungsi `profile_edit_view` untuk menampilkan form edit profil.
   - Menginisialisasi `ProfileForm` dengan instance profil pengguna saat ini.
   - Merender template `profile_edit.html` dengan konteks form.

3. **Menambahkan Impor dan Dekorator yang Diperlukan:**

   Di `a_users/views.py`, tambahkan:

   ```python
   from django.contrib.auth.decorators import login_required
   from .forms import ProfileForm

   @login_required
   def profile_edit_view(request):
       # ...
   ```

   *Penjelasan:*
   - Mengimpor `login_required` untuk memastikan hanya pengguna yang terautentikasi yang dapat mengakses view ini.
   - Mendekorasi `profile_edit_view` dengan `@login_required`.

4. **Membuat Template `profile_edit.html` di `a_users/templates/a_users/`:**

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

   *Penjelasan:*
   - Menggunakan template dasar `layouts/blank.html`.
   - Menampilkan form untuk mengedit profil, termasuk gambar avatar dan nama tampilan.
   - Menggunakan JavaScript untuk memperbarui avatar dan nama tampilan secara langsung saat pengguna melakukan perubahan.

5. **Mengedit `a_users/urls.py`:**

   ```python
   path('edit/', profile_edit_view, name="profile-edit"),
   ```

   *Penjelasan:*
   - Menambahkan URL pattern untuk halaman edit profil yang mengarah ke `profile_edit_view`.

6. **Memperbarui `templates/includes/header.html`:**

   ```html
   <li><a href="{% url "profile-edit" %}">Edit Profile</a></li>
   ```

   *Penjelasan:*
   - Menambahkan link "Edit Profile" pada header yang mengarah ke halaman edit profil.

7. **Membuat `templates/layouts/box.html`:**

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

   *Penjelasan:*
   - Template ini akan memberikan layout kotak untuk konten halaman.

8. **Mengubah Template Induk di `profile_edit.html`:**

   Dari:

   ```html
   {% extends 'layouts/blank.html' %}
   ```

   Menjadi:

   ```html
   {% extends 'layouts/box.html' %}
   ```

   *Penjelasan:*
   - Menggunakan layout kotak yang baru dibuat untuk halaman edit profil.

9. **Memperbarui `profile_edit_view` di `a_users/views.py`:**

   ```python
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
   - Menambahkan logika untuk menangani permintaan POST saat form disubmit.
   - Jika form valid, simpan perubahan dan redirect ke halaman profil.

10. **Menambahkan Impor `redirect`:**

    ```python
    from django.shortcuts import render, redirect
    ```

    *Penjelasan:*
    - Mengimpor fungsi `redirect` untuk melakukan redirect setelah form disubmit.

11. **Membuat Folder `media` di Root Directory:**

    *Penjelasan:*
    - Folder ini akan digunakan untuk menyimpan file media yang diupload oleh pengguna.

12. **Mengkonfigurasi `a_core/settings.py`:**

    ```python
    STATIC_URL = 'static/'
    STATICFILES_DIRS = [BASE_DIR / 'static']

    MEDIA_URL = 'media/'
    MEDIA_ROOT = BASE_DIR / 'media'
    ```

    *Penjelasan:*
    - Menentukan URL dan root directory untuk file media.

13. **Menambahkan Path di `a_core/urls.py`:**

    ```python
    from django.conf import settings
    from django.conf.urls.static import static

    urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
    ```

    *Penjelasan:*
    - Menambahkan konfigurasi untuk melayani file media selama pengembangan.

14. **Menguji Upload Gambar, Nama, dan Deskripsi:**

    *Penjelasan:*
    - Refresh halaman dan coba upload gambar serta mengisi nama dan deskripsi untuk memastikan semuanya berfungsi.

15. **Menambahkan `django-cleanup` ke `INSTALLED_APPS`:**

    Di `a_core/settings.py`:

    ```python
    'django_cleanup.apps.CleanupConfig',
    ```

    *Penjelasan:*
    - `django-cleanup` akan otomatis menghapus file lama saat file baru diupload atau saat model dihapus.

16. **Menguji Dengan Mengganti Gambar:**

    *Penjelasan:*
    - Coba ganti gambar profil untuk memastikan `django-cleanup` berfungsi dengan benar.

17. **Mengkonfigurasi `django-allauth`:**

    Di `a_core/settings.py`, tambahkan ke `INSTALLED_APPS`:

    ```python
    'allauth',
    'allauth.account',
    'allauth.socialaccount',
    ```

    Dan di `MIDDLEWARE`:

    ```python
    'allauth.account.middleware.AccountMiddleware',
    ```

    *Penjelasan:*
    - Menambahkan aplikasi `django-allauth` untuk manajemen autentikasi dan akun.

18. **Menambahkan Path untuk `allauth` di `a_core/urls.py`:**

    ```python
    path('accounts/', include('allauth.urls')),
    ```

    *Penjelasan:*
    - Menyertakan URL patterns dari `django-allauth`.

19. **Menjalankan Migrasi Database:**

    ```bash
    python manage.py migrate
    ```

    *Penjelasan:*
    - Menerapkan migrasi yang diperlukan oleh `django-allauth`.

20. **Memperbarui Link Logout di `templates/includes/header.html`:**

    ```html
    <li><a href="{% url "account_logout" %}">Log Out</a></li>
    ```

    *Penjelasan:*
    - Mengubah link logout agar menggunakan URL dari `django-allauth`.

21. **Menguji Fungsi Logout:**

    *Penjelasan:*
    - Logout dari aplikasi untuk memastikan integrasi dengan `django-allauth` berhasil.

22. **Membuat Template `templates/allauth/layouts/base.html`:**

    ```html
    {% extends "layouts/box.html" %}

    {% block class %} allauth {% endblock class %}

    {% block content %}


    {% endblock content %}
    ```

    *Penjelasan:*
    - Membuat template dasar untuk halaman-halaman `allauth`.

23. **Memperbarui Link Login di `templates/includes/header.html`:**

    ```html
    <li><a href="{% url "account_login" %}">Login</a></li>
    ```

    *Penjelasan:*
    - Mengubah link login agar menggunakan URL dari `django-allauth`.

24. **Mengkonfigurasi Pengaturan Tambahan di `a_core/settings.py`:**

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
   



25. **Menguji Fungsi Login:**

    *Penjelasan:*
    - Coba login dengan email dan password untuk memastikan semuanya berfungsi.

26. **Membuat Template Pesan HTML Kosong:**

    Buat folder `templates/account/messages/` dan kosongkan isi file:

    - `logged_in.txt`
    - `logged_out.txt`

    *Penjelasan:*
    - Mengosongkan pesan default yang ditampilkan setelah login dan logout.

27. **Memperbarui Link Signup di `templates/includes/header.html`:**

    ```html
    <li><a href="{% url "account_signup" %}?next={% url 'profile-onboarding' %}">Signup</a></li>
    ```

    *Penjelasan:*
    - Menambahkan parameter `next` agar setelah signup, pengguna diarahkan ke halaman onboarding profil.

28. **Menambahkan URL Onboarding di `a_users/urls.py`:**

    ```python
    path('onboarding/', profile_edit_view, name="profile-onboarding"),
    ```

    *Penjelasan:*
    - Menambahkan URL pattern untuk proses onboarding yang menggunakan `profile_edit_view`.

29. **Memperbarui `profile_edit_view` di `a_users/views.py`:**

    ```python
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

30. **Menambahkan Impor `reverse`:**

    ```python
    from django.urls import reverse
    ```

    *Penjelasan:*
    - Mengimpor fungsi `reverse` untuk membandingkan path URL saat ini.

31. **Memperbarui Heading di `profile_edit.html`:**

    ```html
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

32. **Memperbarui Tombol Cancel dalam Form:**

    ```html
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

33. **Menambahkan Signal di `a_users/signals.py`:**

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

34. **Restart Server dan Uji Signup:**

    *Penjelasan:*
    - Restart server dan coba proses signup untuk memastikan perubahan berfungsi.

35. **Menambahkan URL Profil dengan `@username` di `a_core/urls.py`:**

    ```python
    from a_users.views import profile_view

    path('@<username>/', profile_view, name='profile'),
    ```

    *Penjelasan:*
    - Menambahkan URL pattern untuk profil pengguna yang diakses dengan `@username`.

36. **Memperbarui `profile_view` di `a_users/views.py`:**

    ```python
    from django.shortcuts import render, redirect, get_object_or_404
    from django.contrib.auth.models import User

    def profile_view(request, username=None):
        if username:
            profile = get_object_or_404(User, username=username).profile
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

37. **Menguji Akses ke `localhost:8080/@usernameanda`:**

    *Penjelasan:*
    - Coba akses URL profil dengan format `@username` untuk memastikan semuanya berfungsi.

Dengan mengikuti langkah-langkah di atas, Anda telah berhasil memperjelas dan mengimplementasikan fitur edit profil, integrasi dengan `django-allauth`, dan menambahkan URL profil dengan `@username` dalam aplikasi Django Anda.
