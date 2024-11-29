**Pengaturan Akun (Settings Page)**

Berikut adalah langkah-langkah untuk menambahkan halaman pengaturan akun dan mengintegrasikan HTMX untuk perubahan email secara dinamis dalam aplikasi Django Anda:

1. **Memperbarui `a_users/views.py`:**

   ```python
   @login_required
   def profile_setting_view(request):
       return render(request, 'a_users/profile_settings.html')
   ```

   *Penjelasan:*
   - Mendefinisikan fungsi `profile_setting_view` yang akan merender template `profile_settings.html`.
   - Menggunakan dekorator `@login_required` untuk memastikan hanya pengguna yang terautentikasi yang dapat mengakses halaman ini.

2. **Membuat Template `profile_settings.html` di `a_users/templates/a_users/`:**

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

   *Penjelasan:*
   - Template ini menampilkan halaman pengaturan akun dengan informasi email pengguna, status verifikasi, dan opsi untuk mengedit atau menghapus akun.
   - Menggunakan template induk `layouts/box.html` untuk konsistensi tampilan.

3. **Menambahkan URL Pattern di `a_users/urls.py`:**

   ```python
   path('settings/', profile_setting_view, name="profile-settings"),
   ```

   *Penjelasan:*
   - Menambahkan path untuk halaman pengaturan akun yang mengarah ke `profile_setting_view`.

4. **Memperbarui Link "Settings" di Header:**

   Di `templates/includes/header.html`, ubah atau tambahkan:

   ```html
   <li><a href="{% url "profile-settings" %}">Settings</a></li>
   ```

   *Penjelasan:*
   - Menambahkan link "Settings" pada header untuk mengakses halaman pengaturan akun.

5. **Menguji Halaman Settings:**

   *Penjelasan:*
   - Refresh halaman dan navigasikan ke halaman pengaturan akun untuk memastikan semuanya tampil dengan benar.

6. **Mengintegrasikan HTMX untuk Perubahan Email Dinamis:**

   - **Menginstal HTMX:**
     Pastikan Anda telah menginstal HTMX dan menambahkannya ke pengaturan Django.

   **a. Memperbarui `a_core/settings.py`:**

   Tambahkan `django_htmx` ke `INSTALLED_APPS`:

   ```python
   'django_htmx',
   ```

   Tambahkan `HtmxMiddleware` ke `MIDDLEWARE`:

   ```python
   'django_htmx.middleware.HtmxMiddleware',
   ```

   *Penjelasan:*
   - `django_htmx` memungkinkan integrasi HTMX dengan Django.
   - `HtmxMiddleware` diperlukan untuk memproses permintaan HTMX.

   **b. Menambahkan Form untuk Perubahan Email di `a_users/forms.py`:**

   ```python
   from django.contrib.auth.models import User

   class EmailForm(ModelForm):
       email = forms.EmailField(required=True)

       class Meta:
           model = User
           fields = ['email']
   ```

   *Penjelasan:*
   - Membuat `EmailForm` yang akan digunakan untuk mengubah email pengguna.
   - Form ini terkait dengan model `User` dan hanya mencakup field `email`.

   **c. Memperbarui View `profile_emailchange` di `a_users/views.py`:**

   ```python
   @login_required
   def profile_emailchange(request):
       if request.htmx:
           form = EmailForm(instance=request.user)
           return render(request, 'partials/email_form.html', {'form': form})

       return redirect('home')
   ```

   *Penjelasan:*
   - View ini menangani permintaan HTMX untuk menampilkan form perubahan email.
   - Jika permintaan adalah HTMX (`request.htmx`), render `email_form.html` dengan form.
   - Jika tidak, redirect ke halaman home.

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

   *Penjelasan:*
   - Form ini akan digunakan untuk mengubah email dan akan ditampilkan secara dinamis menggunakan HTMX.
   - Menggunakan `{% for field in form %}` untuk menampilkan semua field dalam form.
   - Tombol "Cancel" menggunakan `hx-swap-oob="true"` untuk melakukan swap di luar badan HTMX.

   **e. Menambahkan URL Pattern di `a_users/urls.py`:**

   ```python
   path('emailchange/', profile_emailchange, name="profile-emailchange"),
   ```

   *Penjelasan:*
   - Menambahkan path untuk view `profile_emailchange`.

   **f. Memperbarui `profile_settings.html` untuk Menggunakan HTMX:**

   Pada bagian link "Edit" untuk email, ubah menjadi:

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

   *Penjelasan:*
   - Menambahkan atribut HTMX `hx-get`, `hx-target`, dan `hx-swap` untuk melakukan permintaan asinkron dan menampilkan form secara dinamis.

   **g. Menguji Perubahan Email dengan HTMX:**

   *Penjelasan:*
   - Refresh halaman pengaturan akun dan klik "Edit" pada email untuk memastikan form perubahan email muncul tanpa reload halaman.

7. **Memperbarui View `profile_emailchange` untuk Menangani Submisi Form:**

   Di `a_users/views.py`, perbarui fungsi `profile_emailchange`:

   ```python
   @login_required
   def profile_emailchange(request):
       if request.htmx:
           form = EmailForm(instance=request.user)
           return render(request, 'partials/email_form.html', {'form': form})

       if request.method == 'POST':
           form = EmailForm(request.POST, instance=request.user)
           if form.is_valid():
               # Mengecek apakah email sudah digunakan
               email = form.cleaned_data['email']
               if User.objects.filter(email=email).exclude(id=request.user.id).exists():
                   messages.warning(request, f'{email} is already in use')
                   return redirect('profile-settings')

               form.save()

               # Mengirim konfirmasi email
               send_email_confirmation(request, request.user)

               return redirect('profile-settings')
           else:
               messages.warning(request, 'Invalid email')
               return redirect('profile-settings')

       return redirect('home')
   ```

   *Penjelasan:*
   - Menambahkan logika untuk menangani permintaan POST saat pengguna submit form perubahan email.
   - Mengecek apakah email yang dimasukkan sudah digunakan oleh pengguna lain.
   - Jika valid, simpan perubahan dan kirim email konfirmasi menggunakan `send_email_confirmation`.
   - Menggunakan `messages` untuk memberikan feedback kepada pengguna.

   **Tambahan Impor yang Diperlukan:**

   ```python
   from django.contrib import messages
   from allauth.account.utils import send_email_confirmation
   ```

8. **Memperbarui `a_users/signals.py` untuk Menangani Perubahan Email:**

   ```python
   from django.dispatch import receiver
   from django.db.models.signals import post_save, pre_save
   from django.contrib.auth.models import User
   from .models import Profile
   from allauth.account.models import EmailAddress

   @receiver(post_save, sender=User)
   def user_postsave(sender, instance, created, **kwargs):
       user = instance

       # Menambahkan profil jika pengguna baru dibuat
       if created:
           Profile.objects.create(user=user)
       else:
           # Memperbarui EmailAddress di allauth jika email berubah
           try:
               email_address = EmailAddress.objects.get_primary(user)
               if email_address.email != user.email:
                   email_address.email = user.email
                   email_address.verified = False
                   email_address.save()
           except EmailAddress.DoesNotExist:
               # Jika EmailAddress belum ada, buat yang baru
               EmailAddress.objects.create(
                   user=user,
                   email=user.email,
                   verified=False,
                   primary=True
               )
   ```

   *Penjelasan:*
   - Signal `post_save` pada model `User` untuk menangani perubahan email.
   - Jika email berubah, perbarui objek `EmailAddress` dari `allauth` dan set `verified` menjadi `False`.
   - Jika `EmailAddress` belum ada, buat yang baru.

9. **Mengimpor Kelas dan Fungsi yang Diperlukan:**

   Pastikan Anda telah mengimpor semua yang diperlukan di `signals.py` dan `views.py` seperti:

   ```python
   from django.contrib.auth.models import User
   from allauth.account.models import EmailAddress
   from allauth.account.utils import send_email_confirmation
   ```

10. **Menguji Fitur Perubahan Email:**

    *Penjelasan:*
    - Refresh halaman pengaturan akun dan coba ubah email Anda.
    - Setelah submit, periksa apakah ada pesan yang meminta Anda untuk mengonfirmasi email baru.
    - Cek di halaman admin Django apakah email baru ditambahkan dan status verifikasinya.

11. **Mengonfirmasi Email Melalui Terminal:**

    Karena kita menggunakan `EMAIL_BACKEND = 'django.core.mail.backends.console.EmailBackend'`, email konfirmasi akan ditampilkan di terminal.

    *Langkah-langkah:*
    - Setelah mencoba mengubah email, periksa terminal tempat Anda menjalankan server Django.
    - Anda akan melihat output email yang berisi link konfirmasi.
    - Salin link tersebut dan buka di browser untuk mengonfirmasi email.

Dengan mengikuti langkah-langkah di atas, Anda telah berhasil menambahkan halaman pengaturan akun dengan fitur perubahan email yang dinamis menggunakan HTMX. Selain itu, Anda telah memastikan bahwa perubahan email memerlukan verifikasi ulang, dan integrasi dengan `django-allauth` untuk manajemen email dan verifikasi berjalan dengan baik.

---


