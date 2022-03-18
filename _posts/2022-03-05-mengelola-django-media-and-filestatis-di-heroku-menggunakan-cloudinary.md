---
layout: post
title: Mengelola Django Media dan File Statis di Heroku dengan Cloudinary
categories: [Web development]
tags: [Django, Heroku , Cloudinary]
description : Mengelola Django Media dan File Statis di Heroku dengan Cloudinary
date: 2022-03-05 07:45:00 +0700
---

Note ini mengenai bagaimana membuat media dan file statis untuk aplikasi django yang di hosting menggunakan Heroku. 

File Statis (Static File) adalah file server yang melayani pengguna langsung dan django tidak menjalankan kode apapun untuk file statis tersebut. Framework mencari file dan mengembalikan konten untuk dilihat pengguna. Contoh file statis diantaranya : 
- HTML
- File CSS, JavaScript
- Picture profile, icon 
- PDF

File Media merupakan jenis dari File Statis, namun File media biasanya dibuat sebagai file yang diunggah oleh pengguna atau dibuat oleh aplikasi yang dikaitkan dengan Field ImageField di model.
Ketika ada file diupload ke aplikasi web django, ***framework*** akan melihat konfigurasi **DEFAULT_FILE_STORAGE** untuk menentukan bagaimana file yang diupload tersebut akan disimpan. Default Class yang digunakan yaitu **django.core.files.storage.FileSystemStorage** yang dikonfigurasikan saat awal proyek dimulai. Implementasinya melihat konfigurasi **MEDIA_ROOT** yang ditentukan dalam file settings.py dan menyalin file yang di upload ke dalam partisi yang ditentukan di **MEDIA_ROOT**. Misal jika **MEDIA_ROOT** di setting sebagai /var/asset/media, maka semua file yang diunggah akan di simpan ke lokasi di bawah /var/asset/media/.

Menyimpan file statis ke harddisk server secara langsung tidak masalah sampai kita mulai bekerja dengan plaform ***containerization*** seperti Heroku. Dino Heroku menggunakan ***containerization*** dimana sistem file terkait tidak bertahan lama, dapat dibersihkan, diload ulang, di***restart***, dipindahkan tanpa peringatan. Berarti setiap file yang di unggah yang digunakan oleh field ImageField dihapus tanpa bekas setiap kali dyno heroku di restart, dipindahkan atau di skalakan.

## Menggunakan Cloudinary 
Untuk mengatasi permasalahan penyimpanan file media di sistem ***containerization*** tersebut saya gunakan cloudinary sebagai cloud media storage. Berikut cara mengimplementasikannya :
1. Buat akun [cloudinary.com](https://cloudinary.com/) gunakan ***free plan*** untuk penggunaan 25 Monthly Credit atau plan yang lebih besar sesuai kebutuhan. 
1 Credit = - 1.000 Transformation Or 1 GB Storage Or 1 GB Bandwidth.
2. Pilih menu dashboard kemudian lihat account details dan copy Cloud Name, API Key serta API Secret. Selanjutnya tambahkan parameter untuk setting akun cloudinary di file .env dibawah folder app dan paste nilai cloud name, API Key, API Secret.

    ```python
    CLOUD_NAME = your_cloudinary_cloud_name
    API_KEY = your_cloudinary_API_KEY
    API_SECRET = your_cloudinary_API_Secret

    ```
3. Install package cloudinary

    ```python
    pip install cloudinary django-cloudinary-storage

    ```
4. Tambahkan package cloudinary di file settings.py

    ```python
    # settings.py
    ...
    import cloudinary
    import cloudinary_storage
    ...

    ```
5. Install WhiteNoise, Django tidak mendukung static files dalam production, thanks to WhiteNoise yang dapat mengintegrasikan ke dalam django sehingga dapat menggunakan static files di production.

    ```python
    pip install whitenoise

    ```
6. Tambahkan whitenoise di MIDDLEWARE

    ```python
    # settings.py
    ...

    MIDDLEWARE = [
        'corsheaders.middleware.CorsMiddleware',
        'whitenoise.middleware.WhiteNoiseMiddleware',
        # ... django.middleware...

    ]

    ```


7. Seperti kebanyakan package django lainnya, supaya django memanggil code flows yang sesuai saat aplikasi dijalankan, cloudinary dan cloudinary_storage perlu di tambahkan ke **INSTALLED_APPS**.

    ```python
    # settings.py

    ...
    INSTALLED_APPS = [
        # ... django apps,
        'cloudinary',
        'cloudinary_storage',
    ]

8. Konfigurasikan settings.py untuk menggunakan cloudinary.
    
    ```python
    # settings.py
    ...

    CLOUDINARY_STORAGE = {
        'CLOUD_NAME': env('CLOUD_NAME'),
        'API_KEY': env('API_KEY'),
        'API_SECRET': env('API_SECRET'),

    }

    DEFAULT_FILE_STORATE=''cloudinary_storage.storage.MediaCloudinaryStorage'

    WHITENOISE_USE_FINDERS=True

    STATICFILES_DIRS = [
        os.path.join(BASE_DIR, 'build/static')
    ]
    STATIC_ROOT = os.path.join(BASE_DIR, 'staticfiles')

    MEDIA_URL = '/media/'
    MEDIA_ROOT = os.path.join(BASE_DIR, 'media')
    
    STATICFILES_STORAGE = 'whitenoise.storage.CompressedManifestStaticFilesStorage'


    ```

Selesai, cloudinary sudah dapat digunakan sebagai cloud media storage di aplikasi tersebut. :satisfied:


