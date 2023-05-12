# Django
## UserCreationForm
*   about normal user:
```python
    def register(request):
        if request.method == "GET":
            form = UserCreationForm()
            return render(request, 'register.html', {'form': form})
```
* `get` about it get user info from normal user
* Model not created, this model is created by UserCreationForm -->django
*  `ulrs.py` :
 
```python
    from django.contrib import admin
    from django.urls import path
    from book_app.views import *

    urlpatterns = [
        path('admin/', admin.site.urls),
        path('register/',register)
    ]
```
* but dont forget this in `setting.py`:
```python
    TEMPLATES = [
    {
        'DIRS': [BASE_DIR /'templates'],
    }
    ]
```
* html:
```html
<form method="post">
    {% csrf_token %}
    {{form.as_div}}
    <input type="submit" />
</form>
```
-----
## login 
* in the `viwe.py`:
* dont forget `@csrf_exempt`:
```python 
from django.contrib.auth import authenticate, login as dj_login #new
from django.http.response import HttpResponse
from django.views.decorators.csrf import csrf_exempt

@csrf_exempt #new
def login(request):
    if request.method == "POST":
        username = request.POST['username']
        password = request.POST['password']

        user = authenticate(request,username=username,password=password)
        if user:
            dj_login(request,user)
            return HttpResponse('Login completed!')
        return HttpResponse('check password or username!')
    return HttpResponse('please login with post method !')
```
* `ulrs.py`:
```python 
urlpatterns = [
    path('admin/', admin.site.urls),
    path('library/singup/',signup),
    #path('register/',register)
    path('library/login/',login),
]
```
---
## change password :
* `viwe.py`:
* `user.is_authenticated` it is return boolean that checks if login is ok value `True` otherwise `False`.
```python 
@csrf_exempt
def change_password(request):
    if request.method == 'POST':
        if not request.user.is_authenticated:
            return HttpResponse('plasse first login!')
        old_pass = request.POST['old_pass']
        new_pass1 = request.POST['new_pass1']
        new_pass2 = request.POST['new_pass2']
        if not request.user.check_password(old_pass):
            return HttpResponse('Wrong old password')

        if new_pass1 != new_pass2:
            return HttpResponse('Entered passwords are not identical')

        request.user.set_password(new_pass1)
        request.user.save()

        return HttpResponse('Password changed successfully!')

    return HttpResponse('Only post method allowed')
```
* `ulrs.py`:
```python
from .viwe import change_password

urlpatterns = [
    ...code 
    path('library/change_pass/', change_password), #new
]
```
---

## logout
```python 
from django.contrib.auth import logout as dj_logout

@csrf_exempt
def logout(request):
    if request.method == 'POST':
        if not request.user.is_authenticated:
            return ('please login first !')
        dj_logout(request)
        return HttpResponse('logout successfully !')
    return HttpResponse('only post method allowed!')
```

```python 
urlpatterns = [
    ...
    path('library/logout/', logout), #new
]
```
---

# Authentication
* `views.py`:
``` python 
from django.contrib.auth.decorators import login_required #new


def login_first(request):
    return HttpResponse('Please login first')


@csrf_exempt
@login_required(login_url='/library/login-first/') #new
def logout(request):
    if request.method == 'POST':
        dj_logout(request)
        return HttpResponse('Logout successfully')

    return HttpResponse('Only post method allowed')
```
`ulrs.py`:
```python
path('/library/login-frist',login_first), 
```
## permission
* `model.py`:
```python
class Book(models.Model):
    name = models.CharField(max_length=100)
    page = models.IntegerField()
```
* `forms.py`
```python
from django import forms
from .models import Profile, Book

class BookForm(forms.ModelForm):
    class Meta:
        model = Book
        fields = ('name', 'page')
```

* new objects
* `views.py` 
```python
from django.contrib.auth.decorators import login_required, permission_required #new

from .forms import SignUpForm, BookForm


@permission_required('library.add_book', raise_exception=True)
@login_required(login_url='/library/login-first/')
@csrf_exempt
def add_book(request):
    if request.method == 'POST':
        form = BookForm(request.POST)
        if form.is_valid():
            form.save()
            return HttpResponse('Book added successfully')

        return HttpResponse(f"{form.errors}")

    return HttpResponse('Only post method allowed')
```
```python
path('library/add-book/', add_book),
```
---
## create usere
```python
python manage.py createsuperuser
```
----
# Posts
## model posts
* create model in app name blog:
```python
from django.db import models


class Posts(models.Model):
    text = models.TextField()
```
* see text :
```python
    def __str__(self):
        return self.text[:10]
```
* blog/viwe.py:

```python
from django.views.generic import ListView
from .models import Posts


class Postview(ListView):
    model = Posts
    template_name = 'index.html'
    context_object_name = 'post_list' #new
```

* config/urls.py:

```python
path('blog/',include('blog.urls')), #new
```
 * blog/urls.py:

 ```python
    path('post/',Postview.as_view(),name = 'Posts'),
 ```
* see website urls
```
http://127.0.0.1:8000/blog/post/
```
---
# blog for css and Images
## start creat file and add setting
* create startapp blog
```python 
#terminal 
python manage.py startapp blog  
```
* create file static
* config/setting.py:
 ```python
STATIC_URL = '/static/'
STATICFILES_DIRS = [str(BASE_DIR.joinpath('staticb'))] #new
 ```
 * create templates file.
 * add adress file in setting.py:
 ```python
'DIRS': [str(BASE_DIR.joinpath('templates'))],
 ```
## create model
```python
class Post(models.Model):
    title = models.CharField(max_length=200)
    author = models.ForeignKey('auth.User',on_delete=models.CASCADE,)
    body = models.TextField()
    
    def __str__(self):
        return self.title
```
* اگر نویسند را حذف کنیم تمام ویژگی نوشته شده حذف شود
```python
    author = models.ForeignKey('auth.User',on_delete=models.CASCADE,)
```
* so run makemigration and migrate.
* `view.py/blog`:
* import listview for see list in admin:
```python
from django.views.generic import ListView
from .models import Post


class PostView(ListView):
    model = Post
    template_name = 'home.html'
    context_object_name = 'post_list'
```
* add link `include` in `config/urls` and add `blog/urls`.
---
