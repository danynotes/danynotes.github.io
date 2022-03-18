---
layout: post
title: Membuat Blog menggunakan django dan ReactJS
categories: [Web development]
tags: [Django, ReactJS, Django-Rest-Framework, PostgreSQL]
description : Membuat Blog menggunakan django sebagai backend dan ReactJS sebagai frontend dengan database PostgreSQL
date: 2022-03-04 15:45:00 +0700
---


Note ini mengenai bagaimana membuat Website personal blog menggunakan django sebagai backend dan django REST framework sebagai Web APIs, serta di sisi frontend menggunakan ReactJS. Personal blog ini dibuat dengan fitur : 
1. personal portfolio dengan menu : portfolio, list work, contact form.
2. Blog

Project ini dibuat menggunakan Windows Subsystem for Linux Distributions (WSL 2) dengan distro Ubuntu 18.04 dan python 3.8.10.
Database menggunakan PostgreSQL 13.0 running on Docker.


## Backend

Pertama-tama persiapkan dulu project environment. 

```python
dany@hp:/$ mkdir blog_project
dany@hp:/$ cd blog_project
dany@hp:~/blog_project$ mkdir backend
dany@hp:~/blog_project$ cd backend
dany@hp:~/blog_project/backend$ python3 -m venv envblog

dany@hp:~/blog_project/backend$ source envblog/bin/activate
```
Kemudian install requirements

```python
(envblog) dany@hp:~/blog_project/backend$ install django django-cors-headers djangorestframework pillow psycopg2 psycopg2-binary django-summernote requests
```

Setelah process instalasi requirement selesai selanjutnya kita buat django project dan blog app.

```python
(envblog) dany@hp:~/blog_project/backend$ django-admin startproject blog_personal
(envblog) dany@hp:~/blog_project/backend$ cd blog_personal
(envblog) dany@hp:~/blog_project/backend/blog_personal$ python manage.py startapp blog
```
Buat django SECRET_KEY

```python
(envblog) dany@hp:~/blog_project/backend/blog_personal$ python -c 'from django.core.management.utils import get_random_secret_key; print(get_random_secret_key())'

```
Simpan hasil generate SECRET_KEY kedalam file .env bersama dengan variable lain yang digunakan di app.

```python
# .env

SECRET_KEY=k!j1ymx8)lt92ivz9zgl+2l4-c15f@@!$c_dj#(4oz84anmj8+
DJANGO_DEBUG=True
DATABASE_USER=your_database_user
DATABASE_PASS=your_database_pass
DATABASE_NAME=your_database_name
DATABASE_PORT=your_database_port
DATABASE_HOST=localhost
ALLOWED_HOSTS=localhost,127.0.0.1

```

Modifikasi file settings.py untuk membaca variable environment dari file .env 

```python
# settings.py
...

from pathlib import Path
import os
import environ

env = environ.Env()
environ.Env.read_env()

# Quick-start development settings - unsuitable for production
# See https://docs.djangoproject.com/en/3.2/howto/deployment/checklist/

# SECURITY WARNING: keep the secret key used in production secret!
SECRET_KEY = env('SECRET_KEY')

# SECURITY WARNING: don't run with debug turned on in production!
DEBUG = env('DJANGO_DEBUG', default=True)

ALLOWED_HOSTS = tuple(env.list('ALLOWED_HOSTS', default=[]))

...

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': env('DATABASE_NAME'),
        'USER': env('DATABASE_USER'),
        'HOST': env('DATABASE_HOST'),
        'PASSWORD': env('DATABASE_PASS'),
        'PORT': env('DATABASE_PORT'),
    }
}

```

Masih di file settings.py, tambahkan corsheaders di dalam MIDDLERWARE, kemudian tambahkan juga app blog, corsheaders, rest_framework dan django_summernote ke dalam INSTALLED_APPS. dan setting template direktori dengan menambahkan folder build yang dihasilkan dari reactjs.

```python
# settings.py
....
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'corsheaders',
    'rest_framework',
    'django_summernote',
    'blog',
   

]

MIDDLEWARE = [
    'corsheaders.middleware.CorsMiddleware',
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]

TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [os.path.join(BASE_DIR), 'build'],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]
```

Tambahkan setting static dan media direktori untuk folder media upload di settings.py

```python
# settings.py
...
# Static files (CSS, JavaScript, Images)
# https://docs.djangoproject.com/en/3.2/howto/static-files/

STATIC_URL = '/static/'
STATICFILES_DIRS = [
    os.path.join(BASE_DIR, 'build/static')
]
STATIC_ROOT = os.path.join(BASE_DIR, 'static')

MEDIA_URL = '/media/'
MEDIA_ROOT = os.path.join(BASE_DIR, 'media')

```

Supaya Web blog ini dapat mengirim email ke admin ketika ada pengguna yang mengirim form contact, tambahkan EmailBackend pada settings.py.

```python
# settings.py

....
EMAIL_BACKEND = 'django.core.mail.backends.smtp.EmailBackend'
EMAIL_HOST = env('E_HOST')
EMAIL_PORT = env('E_PORT')
EMAIL_HOST_USER = env('E_HOSTUSER')
EMAIL_HOST_PASSWORD = env('E_HOSTPASSWORD')
EMAIL_USE_TLS= env('E_USETLS')

```

Dan tambahkan parameter untuk EMAIL_BACKEND di file .env

```python
# .env

...
E_HOST=smtp.gmail.com
E_PORT=587
E_HOSTUSER=adminmail@gmail.com
E_HOSTPASSWORD=adminmailapp_password
E_USETLS=True

```

Kemudian tambahkan setting untuk REST_FRAMEWORK. pertama tambahkan urlpattern di urls.py.

```python
# urls.py

urlpatterns = [
    path('api-auth/', include('rest_framework.urls')),
    path('admin/', admin.site.urls),
]

```

dan tambahkan DEFAULT_PERMISSION_CLASSES untuk REST_FRAMEWORK di settings.py

```python
# settings.py
....

REST_FRAMEWORK = {
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.DjangoModelPermissionsOrAnonReadOnly'
    ]
}

```
Kita telah menambahkan corsheaders di INSTALLED_APPS, dan di middleware classes. Untuk mengaktifkan CORS (Cross-Origin_resouce-sharing) perlu di tambahkan konfigurasi di settings.py. CORS ini berfungsi supaya django dapat berinteraksi dengan ***resource*** lain di domain yang berbeda. untuk mengetahui lebih detail mengenai CORS dapat di baca disini [Django CORS GUIDE](https://www.stackhawk.com/blog/django-cors-guide/).

```python
# settings.py

....

CORS_ORIGIN_ALLOW_ALL = True

```

Kita tambahkan theme option untuk [django-summernote](https://github.com/summernote/django-summernote), modul django-summernote membuat kita dapat menyematkan editor WYSIWYG Summernote ke dalam django.

```python
# settings.py

....
SUMMERNOTE_THEME = 'bs4'
```

Setelah menambahkan django-summernote di settings.py kemudian tambahkan django_summernote dan settings MEDIA_URL di urls.py

```python

# urls.py

urlpatterns = [
    ....
    path('summernote/', include('django_summernote.urls')),
] + static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)

urlpatterns += [re_path(r'^.*', TemplateView.as_view(template_name='index.html'))]

```

Terakhir di urls.py kita tambahkan path untuk API blog app yang akan kita buat.

```python
# urls.py

urlpatterns = [
    ....
    path('api/blog/', include('blog.urls')),
]

```
Ok konfigurasi settings.py dan urls.py untuk project sudah selesai selanjutnya kita ke detail blog app. Langkah pertama kita buat modelnya, 
class model yang kita buat yaitu : 
1. class Contact untuk menyimpan data contact form.
2. class Testimonial untuk menyimpan data testimonial.
3. class Profile untuk menyimpan data personal profile.
4. class Job untuk menyimpan data project yang pernah di kerjakan.
5. class Portofolio_cat untuk menyimpan data kategori portfolio.
6. class Portofolio untuk menyimpan data portfolio.
7. class Categories untuk menyimpan data kategori blog (Menu Blog)
8. class BlogPost untuk menyimpan data post artikel 

```python
# models.py

from django.db import models
from datetime import datetime
from django.db.models.fields import CharField, TextField
from django.db.models.query import QuerySet
from django.template.defaultfilters import default, slugify
from rest_framework.fields import ImageField


class Contact(models.Model):
    name = models.CharField(max_length=200)
    email = models.CharField(max_length=100)
    message = models.TextField(blank=True)
    contact_date = models.DateTimeField(default=datetime.now, blank=True)

    def __str__(self):
        return self.email

class Testimonial(models.Model):
    testimonial_name = models.CharField(max_length=50)
    testimonial_title = models.CharField(max_length=70)
    testimonial_img = models.ImageField(upload_to='photos/%Y/%m/%d')
    testimonial_icon = models.ImageField(upload_to='photos/%Y/%m/%d')
    testimonial_desc = models.TextField(default=None, blank=True)
    testimonial_featured = models.BooleanField(default=False)

    class Meta:
        verbose_name_plural = "Testimonial"

    def __str__(self):
        return self.testimonial_name

class Profile(models.Model):
    name = models.CharField(max_length=50)
    work_title = models.CharField(max_length=70)
    work_desc = models.TextField(default=None, blank=True)
     

    class Meta:
        verbose_name_plural = "Profile"

    def arr_worktitle(self):
        if self.work_title:
            return self.work_title.split(",")
        else:
            None

    def __str__(self):
        return self.name

class Job(models.Model):
    job_name = models.CharField(max_length=100)
    job_icon = models.ImageField(upload_to='photos/%Y/%m/%d')
    job_desc = models.TextField(default=None)
    jobs_thumbnail = models.ImageField(upload_to='photos/%Y/%m/%d')

    class Meta:
        verbose_name_plural = "Job"

    def __str__(self):
        return self.job_name

class Portofolio_cat(models.Model):
    port_cat_name = models.CharField(max_length=50)
    
    class Meta:
        verbose_name_plural = "Portofolio Category"
        verbose_name = "Portofolio Category"
        ordering = ['id']
    
    def __str__(self):
        return self.port_cat_name
    
class Portofolio(models.Model):
    porto_cat_id = models.ForeignKey(Portofolio_cat, related_name='portfolios', on_delete=models.CASCADE, null=True)
    porto_title = models.CharField(max_length=50)
    porto_date = models.DateTimeField(default=datetime.now, blank=True)
    porto_thumbnail = models.ImageField(upload_to='photos/%Y/%m/%d')

    class Meta:
        verbose_name_plural = "Portofolio"

    def __str__(self):
        return self.porto_title

class Categories(models.Model):
    cat_title = models.CharField(max_length=50)
    url = models.CharField(max_length=50)

    class Meta:
        verbose_name_plural = "Categories"

    def __str__(self):
        return self.cat_title

class BlogPost(models.Model):
    cat_title_id = models.ForeignKey(Categories, related_name='posts', on_delete=models.CASCADE, null=True)
    title = models.CharField(max_length=50)
    slug = models.SlugField()
    thumbnail = models.ImageField(upload_to='photos/%Y/%m/%d/')
    excerpt = models.CharField(max_length=150)
    month = models.CharField(max_length=3)
    day = models.CharField(max_length=2)
    content = models.TextField(default=None)
    featured = models.BooleanField(default=False)
    date_created = models.DateTimeField(default=datetime.now, blank=True)

    def save(self, *args, **kwargs):
        original_slug = slugify(self.title)
        querySet = BlogPost.objects.all().filter(slug__iexact=original_slug).count()

        count = 1
        slug = original_slug
        while(querySet):
            slug = original_slug + '-' + str(count)
            count += 1
            querySet = BlogPost.objects.all().filter(slug__iexact=slug).count()

            # first-blog-post-slug
        self.slug = slug

        if self.featured:
            try:
                temp = BlogPost.objects.get(featured=True)
                if self != temp:
                    temp.featured = False
                    temp.save()
                
            except BlogPost.DoesNotExist:
                pass
        
        super(BlogPost, self).save(*args, **kwargs)
 
    def __str__(self):
        return self.title
```
Setelah model dibuat, selanjutnya kita lakukan migrate.

```python
(envblog) dany@hp:~/blog_project/backend/blog_personal$ python manage.py makemigration blog
(envblog) dany@hp:~/blog_project/backend/blog_personal$ python manage.py migrate

```

Selanjutnya kita buat superuser untuk pengguna yang akan menggunakan admin site.

```python
(envblog) dany@hp:~/blog_project/backend/blog_personal$ python manage.py createsuperuser

Username: admin
email address : admin@example.com

Password: **********
Password (again): *********
Superuser created successfully.
```

Karena kita menggunakan Django REST Framework dan ReactJS sebagai front-end, kita perlu membuat Serializer untuk mengubah object menjadi tipe data yang dapat di mengerti javascripts dan front-end frameworks

```python
# serializers.py

from django.db.models import fields
from django.http import request
from .models import BlogPost, Categories, Contact, Portofolio, Portofolio_cat, Profile, Job, Testimonial
from rest_framework import serializers


# Serializers define the API representation.
class BlogPostSerializer(serializers.ModelSerializer):
    cat_title = serializers.ReadOnlyField(source='cat_title_id.cat_title')
    
    class Meta:
        model = BlogPost
        fields = ('id', 'title', 'slug', 'thumbnail', 'excerpt', 'month', 'day', 'content', 'featured', 'date_created', 'cat_title_id', 'cat_title')
        lookup_field = 'slug'

    def get_image_url(self, obj):
        request = self.context.get("request")
        thumbnail_url = obj.thumbnail.url
        return request.build_absolute_uri(thumbnail_url)
    

class CategoriesSerializer(serializers.ModelSerializer):
    class Meta:
        model = Categories
        fields = ('id', 'cat_title', 'url')
        lookup_field = 'id'
    
class ProfileSerializer(serializers.ModelSerializer):
    class Meta:
        model = Profile
        fields = ('name', 'work_desc', 'work_title', 'arr_worktitle')
        lookup_field = 'name'

    def get_arr_worktitle(self, Profile):
        return ProfileSerializer(Profile.arr_worktitle()).data

        
class JobsSerializer(serializers.ModelSerializer):
    class Meta:
        model = Job
        fields = ('id', 'job_name', 'job_desc', 'job_icon', 'jobs_thumbnail')
        lookup_field = 'job_name' 

class PortofolioSerializer(serializers.ModelSerializer):
    port_cate = serializers.ReadOnlyField(source='porto_cat_id.port_cat_name')

    
    class Meta:
        model = Portofolio
        fields = ('id', 'porto_title', 'porto_thumbnail', 'port_cate')
        lookup_field = 'porto_title'

class PortofolioCatSerializer(serializers.ModelSerializer):
    class Meta:
        model = Portofolio_cat
        fields = ('id', 'port_cat_name')
        lookup_field = 'port_cat_name'

class TestimonialSerializer(serializers.ModelSerializer):
    class Meta:
        model = Testimonial
        fields =  ('id', 'testimonial_name', 'testimonial_title', 'testimonial_img', 'testimonial_icon', 'testimonial_desc')
        lookup_field = 'testimonial_name'

class ConctactSerializer(serializers.ModelSerializer):
    class Meta:
        model = Contact
        fields = '__all__'

```
Setelah serializers dibuat, selanjutnya kita buat views.py


```python

# views.py

from django.db.models.query import QuerySet
from rest_framework.response import Response
from rest_framework import generics, permissions, serializers
from rest_framework.views import APIView
from django.core.mail import send_mail
from django.views.decorators.cache import never_cache


from blog.models import BlogPost, Categories, Portofolio, Profile, Job, Portofolio_cat, Testimonial, Contact
from blog.serializers import BlogPostSerializer, JobsSerializer, PortofolioSerializer, TestimonialSerializer
from blog.serializers import CategoriesSerializer
from blog.serializers import ProfileSerializer
from blog.serializers import PortofolioCatSerializer

from danynotes.settings import EMAIL_HOST_USER
 

class BlogPostListView(generics.ListAPIView):
    permissions_classes = (permissions.AllowAny, )
    queryset = BlogPost.objects.order_by('-date_created')
    serializer_class = BlogPostSerializer
    lookup_field = 'slug'
    
class BlogPostDetailView(generics.RetrieveAPIView):
    permissions_classes = (permissions.AllowAny, )
    queryset = BlogPost.objects.order_by('-date_created')
    serializer_class = BlogPostSerializer
    lookup_field = 'slug'


class BlogPostFeaturedView(generics.ListAPIView):
    permissions_classes = (permissions.AllowAny, )
    queryset = BlogPost.objects.all().filter(featured=True)
    serializer_class = BlogPostSerializer
    lookup_field = 'slug'


class BlogPostCategoryView(APIView):
    permissions_classes = (permissions.AllowAny, )
    serializer_class = BlogPostSerializer
    


    def post(self, request, format=None):
        data = self.request.data
        cat_title_id = data['cat_title_id']
        queryset = BlogPost.objects.order_by('-date_created').filter(cat_title_id=cat_title_id)
        serializer = BlogPostSerializer(queryset, context={"request": request}, many=True)
        return Response(serializer.data)

class CategoriesView(generics.ListAPIView):
    permissions_classes = (permissions.AllowAny, )
    queryset = Categories.objects.order_by('id')
    serializer_class = CategoriesSerializer
    lookup_field = 'cat_title'


class ProfileView(generics.ListAPIView):
    permissions_classes = (permissions.AllowAny, )
    queryset = Profile.objects.order_by('name')
    serializer_class = ProfileSerializer
    lookup_field = 'name'


class JobsView(generics.ListAPIView):
    permissions_classes = (permissions.AllowAny, )
    queryset = Job.objects.order_by('job_name')
    serializer_class = JobsSerializer
    lookup_field = 'job_name'


class PortoCategoryView(generics.ListAPIView):
    permissions_classes = (permissions.AllowAny, )
    queryset = Portofolio_cat.objects.order_by('id')
    serializer_class = PortofolioCatSerializer
    lookup_field = 'port_cat_name'


class PortoFolioView(APIView):
    permissions_classes = (permissions.AllowAny, )
    serializer_class = PortofolioSerializer
    

    def post(self, request, format=None):
        data = self.request.data
        porto_cat_id = data['porto_cat_id']
        queryset = Portofolio.objects.order_by('-porto_date').filter(porto_cat_id=porto_cat_id)
        serializer = PortofolioSerializer(queryset, context={"request": request}, many=True)
        return Response(serializer.data)

class TestimonyView(generics.ListAPIView):
    permissions_classes = (permissions.AllowAny, )
    queryset = Testimonial.objects.order_by('id')
    serializer_class = TestimonialSerializer
    lookup_field = 'testimonial_name'

class ContactView(APIView):
    permissions_classes = (permissions.AllowAny, )

    def post(self, request, format=None):
        data = self.request.data
        subject = 'danynotes - ' + data['name']
        name = data['name']
        message = data['message']
        from_email = data['email']

            
        try:
            send_mail(subject, message, from_email, [EMAIL_HOST_USER], fail_silently= False)

            contact = Contact(name=name, email=from_email, message=message)
            contact.save()

            return Response({'success': 'Message sent successfully'})
        
        except:
            return Response({'error': 'Message failed to send'})


```

Terakhir kita buat url untuk akses view tersebut.

```python
# urls.py

from django.urls import path
from rest_framework.serializers import as_serializer_error

from .views import BlogPostListView, BlogPostDetailView, BlogPostFeaturedView, BlogPostCategoryView, JobsView, PortoFolioView
from .views import CategoriesView, ProfileView, TestimonyView
from .views import PortoCategoryView
from .views import ContactView


urlpatterns = [
    path('', BlogPostListView.as_view()),
    path('featured', BlogPostFeaturedView.as_view()),
    path('category', BlogPostCategoryView.as_view()),
    path('<slug>', BlogPostDetailView.as_view()),
    path('list_category/', CategoriesView.as_view()),
    path('profile/', ProfileView.as_view()),
    path('jobs/', JobsView.as_view()),
    path('portofolio_category/', PortoCategoryView.as_view()),
    path('portofolios/', PortoFolioView.as_view()),
    path('testimonial/', TestimonyView.as_view()),
    path('contact/', ContactView.as_view()),
]

```

Setelah urls.py selesai dibuat, selanjutnya coba akses URL API menggunakan Postman, sebelumnya jangan lupa untuk menjalankan server.

```python
(envblog) dany@hp:~/blog_project/backend/blog_personal$python manage.py runserver


Watching for file changes with StatReloader
Performing system checks...

System check identified no issues (0 silenced).
March 12, 2022 - 04:38:26
Django version 3.2.5, using settings 'blog_personal.settings'
Starting development server at http://127.0.0.1:8000/
Quit the server with CONTROL-C.

```

![postman_personalblog](https://user-images.githubusercontent.com/7325133/158960153-a3f6a299-84dd-426e-85d0-39c9573b00f1.png)

Selesai pembuatan backend dan Django Restframework, selanjutnya buat [tampilan frontend dengan ReactJS](/web%20development/2022/03/04/membuat-blog-menggunakan-django-dan-reactjs(Lanjutan)).

Source code lengkap dapat lihat di [repository ini](https://github.com/noufath/reactjs_with_django_portfolio_and_blog_web).
