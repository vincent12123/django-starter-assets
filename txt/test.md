### Email Verify

a_users/views.py > tambahkan
```python
@login_required
def profile_emailverify(request):
    send_email_confirmation(request, request.user)
    return redirect('profile-settings')
```
a_users/urls.py > `  path('emailverify/', profile_emailverify, name="profile-emailverify"),`

kemudian ke 

a_users/templates/a_users/profile_settings.html
```html
 <td class="pb-4 pl-4">
                <a href="{% url 'profile-emailverify' %}" class="font-medium text-blue-600 hover:underline">
                    {% if not user.emailaddress_set.first.verified %}Verify{% endif %}
                </a>
            </td>
```

