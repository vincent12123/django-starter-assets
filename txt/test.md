
### **8. Menambahkan Fitur Verifikasi Email**

**a. Membuat View `profile_emailverify` di `a_users/views.py`:**

```python
from django.contrib.auth.decorators import login_required
from allauth.account.utils import send_email_confirmation
from django.shortcuts import redirect

@login_required
def profile_emailverify(request):
    send_email_confirmation(request, request.user)
    return redirect('profile-settings')
```

- **Penjelasan:**
  - View ini bertanggung jawab untuk mengirim ulang email konfirmasi ke pengguna.
  - Menggunakan fungsi `send_email_confirmation` dari `django-allauth` untuk mengirim email verifikasi.
  - Setelah email dikirim, pengguna akan diarahkan kembali ke halaman pengaturan profil.

**b. Menambahkan URL untuk Verifikasi Email di `a_users/urls.py`:**

```python
from .views import profile_emailverify

urlpatterns = [
    # URL lainnya
    path('emailverify/', profile_emailverify, name="profile-emailverify"),
]
```

- **Penjelasan:**
  - Menambahkan path untuk view `profile_emailverify` yang akan menangani permintaan verifikasi email.

**c. Memperbarui Template `profile_settings.html` untuk Menambahkan Link Verifikasi:**

Di `a_users/templates/a_users/profile_settings.html`, ubah bagian yang menampilkan opsi verifikasi email menjadi:

```html
<td class="pb-4 pl-4">
    <a href="{% url 'profile-emailverify' %}" class="font-medium text-blue-600 hover:underline">
        {% if not user.emailaddress_set.first.verified %}Verify{% endif %}
    </a>
</td>
```

- **Penjelasan:**
  - Menambahkan link "Verify" yang akan memicu pengiriman ulang email verifikasi.
  - Link ini hanya ditampilkan jika email pengguna belum diverifikasi.

**d. Menguji Fitur Verifikasi Email:**

- **Langkah-langkah:**
  - Masuk ke akun pengguna yang emailnya belum diverifikasi.
  - Navigasikan ke halaman pengaturan akun.
  - Klik link "Verify" di samping status email.
  - Periksa terminal untuk melihat email konfirmasi yang dikirim (karena kita menggunakan email backend console).
  - Salin link konfirmasi dan akses melalui browser untuk memverifikasi email.

---

### **9. Menambahkan Fitur Penghapusan Akun**

**a. Membuat View `profile_delete_view` di `a_users/views.py`:**

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

- **Penjelasan:**
  - View ini menangani permintaan penghapusan akun.
  - Jika metode permintaan adalah `POST`, pengguna akan logout, akun dihapus, dan pesan sukses ditampilkan.
  - Jika metode permintaan bukan `POST`, halaman konfirmasi akan ditampilkan.

**b. Membuat Template `profile_delete.html` di `a_users/templates/a_users/`:**

```html
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

- **Penjelasan:**
  - Template ini menampilkan pesan konfirmasi sebelum akun dihapus.
  - Terdapat tombol "Yes, I want to delete my account" yang mengirimkan permintaan `POST` untuk menghapus akun.
  - Tombol "Cancel" mengembalikan pengguna ke halaman sebelumnya.

**c. Menambahkan URL untuk Penghapusan Akun di `a_users/urls.py`:**

```python
from .views import profile_delete_view

urlpatterns = [
    # URL lainnya
    path('delete/', profile_delete_view, name="profile-delete"),
]
```

- **Penjelasan:**
  - Menambahkan path untuk view `profile_delete_view`.

**d. Memperbarui Template `profile_settings.html` untuk Menambahkan Link Penghapusan Akun:**

Di `a_users/templates/a_users/profile_settings.html`, ubah bagian yang menampilkan opsi penghapusan akun menjadi:

```html
<a href="{% url 'profile-delete' %}" class="font-medium text-red-600 hover:underline">
    Delete
</a>
```

- **Penjelasan:**
  - Menambahkan link "Delete" yang mengarah ke halaman konfirmasi penghapusan akun.

**e. Menguji Fitur Penghapusan Akun:**

- **Langkah-langkah:**
  - Masuk ke akun pengguna.
  - Navigasikan ke halaman pengaturan akun.
  - Klik link "Delete" untuk mengakses halaman konfirmasi.
  - Klik tombol "Yes, I want to delete my account" untuk menghapus akun.
  - Pastikan akun terhapus dan pengguna diarahkan ke halaman home dengan pesan sukses.

---

### **10. Membuat Halaman 404 Kustom**

**a. Membuat Template `404.html` di `templates/`:**

Buat file `404.html` di folder `templates`:

```html
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

- **Penjelasan:**
  - Template ini akan ditampilkan ketika pengguna mengakses halaman yang tidak ada.
  - Menggunakan layout `blank.html` untuk tampilan yang sederhana.

**b. Mengkonfigurasi Pengaturan Django untuk Menggunakan Halaman 404 Kustom:**

Di `a_core/settings.py`, ubah pengaturan:

```python
DEBUG = False

ALLOWED_HOSTS = ['localhost', '127.0.0.1', 'yourdomain.com']
```

- **Penjelasan:**
  - Mengatur `DEBUG = False` agar Django menggunakan halaman error kustom.
  - Menambahkan domain dan host yang diperbolehkan di `ALLOWED_HOSTS`.
  - Jika Anda menjalankan server di localhost untuk pengujian, tambahkan `localhost` dan `127.0.0.1`.

**c. Menguji Halaman 404 Kustom:**

- **Langkah-langkah:**
  - Jalankan server Django.
  - Akses URL yang tidak ada, misalnya `http://localhost:8000/nonexistent-page`.
  - Pastikan halaman 404 kustom muncul sesuai dengan template yang dibuat.

**Catatan Penting:**

- Saat `DEBUG = False`, Django tidak akan melayani file statis dan media secara default. Untuk pengujian lokal, Anda mungkin perlu mengatur `DEBUG = True` atau mengonfigurasi server web seperti Nginx atau Apache untuk melayani file statis dan media.
- Pastikan `ALLOWED_HOSTS` diisi dengan benar untuk lingkungan produksi.

---

### **11. Menyesuaikan Pengaturan Keamanan untuk Produksi**

**a. Menyiapkan Pengaturan Produksi di `a_core/settings.py`:**

- **Mengatur `DEBUG` dan `ALLOWED_HOSTS`:**

  ```python
  DEBUG = False

  ALLOWED_HOSTS = ['localhost', '127.0.0.1', 'yourdomain.com']
  ```

- **Mengatur Kunci Rahasia (Secret Key):**

  Pastikan `SECRET_KEY` Anda disimpan dengan aman dan tidak dibagikan.

- **Mengaktifkan HTTPS:**

  Jika aplikasi Anda berjalan di lingkungan produksi, pastikan untuk mengaktifkan HTTPS dan mengatur pengaturan keamanan tambahan seperti `SECURE_SSL_REDIRECT`, `SESSION_COOKIE_SECURE`, dan `CSRF_COOKIE_SECURE`.

---

### **Kesimpulan Akhir**

Dengan menambahkan fitur verifikasi email, penghapusan akun, dan halaman 404 kustom, aplikasi Django Anda kini memiliki fungsionalitas manajemen akun yang lebih lengkap dan profesional. Anda telah mengimplementasikan fitur-fitur penting yang meningkatkan pengalaman pengguna dan keamanan aplikasi.

**Rekapitulasi Fitur yang Telah Diimplementasikan:**

1. **Halaman Edit Profil:**
   - Pengguna dapat mengedit profil mereka, termasuk mengubah avatar, nama tampilan, dan informasi lainnya.
   - Preview avatar dan nama tampilan diperbarui secara real-time menggunakan JavaScript.

2. **Integrasi `django-allauth`:**
   - Mengelola autentikasi, pendaftaran, login, dan manajemen akun menggunakan `django-allauth`.
   - Menggunakan email sebagai metode autentikasi utama dan mewajibkan verifikasi email.

3. **URL Profil dengan `@username`:**
   - Pengguna dapat mengakses profil melalui URL yang mudah dibaca, seperti `@username`.

4. **Halaman Pengaturan Akun:**
   - Pengguna dapat melihat dan mengubah pengaturan akun mereka, termasuk email dan opsi untuk menghapus akun.

5. **Interaksi Dinamis dengan HTMX:**
   - Menggunakan HTMX untuk memperbarui bagian halaman secara dinamis tanpa reload penuh, seperti form perubahan email.

6. **Verifikasi Email:**
   - Pengguna dapat meminta pengiriman ulang email verifikasi jika email mereka belum diverifikasi.

7. **Penghapusan Akun:**
   - Pengguna dapat menghapus akun mereka sendiri dengan konfirmasi.

8. **Halaman 404 Kustom:**
   - Menyediakan halaman 404 yang ramah pengguna ketika halaman yang diminta tidak ditemukan.

9. **Pengaturan Keamanan untuk Produksi:**
   - Mengkonfigurasi pengaturan seperti `DEBUG` dan `ALLOWED_HOSTS` untuk lingkungan produksi.

---

**Langkah Selanjutnya:**

- **Pengujian Menyeluruh:**
  - Lakukan pengujian menyeluruh untuk memastikan semua fitur berfungsi dengan baik dan tidak ada bug.

- **Penanganan Error Lainnya:**
  - Buat halaman kustom untuk error lain seperti 500 Internal Server Error.

- **Optimasi dan Keamanan:**
  - Terapkan praktik terbaik untuk keamanan aplikasi Django.
  - Optimalkan performa aplikasi, terutama jika Anda berencana untuk menskalakan aplikasi.

- **Deployment:**
  - Siapkan lingkungan produksi dan deploy aplikasi Anda.
  - Konfigurasi server web seperti Nginx atau Apache, dan gunakan WSGI server seperti Gunicorn atau uWSGI.

- **Fitur Tambahan:**
  - Pertimbangkan untuk menambahkan fitur lain seperti reset password, integrasi media sosial, atau notifikasi email.

---

