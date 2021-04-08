---
layout: post
title: Membuat website atau blog statis menggunakan jekyll dan GitHub Pages
categories: [website-statis]
tags: [jekyll]
description: Membuat website atau blog statis menggunakan jekyll dan GitHub Pages
date: 2021-04-07 10:45:00 +0700
---


# Why static site ?
Sangat cocok untuk kebutuhan blog post atau website sederhana karena ringan, cepat dan mudah mengkonfigurasinya, yang kedua membuat artikel di jekyll cukup membuat file text dengan format markdown (.md) yang bisa dilakukan secara offline tanpa harus menggunakan admin dashboard, kemudian yang ketiga dapat di hosting dengan gratis menggunakan github / gitlab, los dol tanpa ribet setting server hosting 😊

# Tools apa yang di perlukan
Untuk membuat website statis yang diperlukan sebagai berikut :
1. Jekyll sebagai static generator
2. GitHub/Gitlab account, dalam catatan ini saya menggunakan GitHub
3. Internet Browser
4. Text Editor

# Bagaimana caranya
1. Install jekyll, saat ini saya menggunakan windows dengan WSL (Windows Subsystem for Linux) dengan distro Ubuntu 18.04 LTS. 
    
    Update repository lists dan packages:

    ```bash
    sudo apt-get update -y && sudo apt-get upgrade -y
    ```

    Berikutnya install Ruby untuk Ubuntu dari repository BrightBox:

    ```bash
    sudo apt-add-repository ppa:brightbox/ruby-ng
    sudo apt-get update
    sudo apt-get install ruby2.5 ruby2.5-dev build-essential dh-autoreconf
    ```

    Selanjutnya update Ruby gems:
    ```bash
    gem update
    ```

    Dan lanjut install jekyll:
    ```bash
    gem install jekyll bundler
    ```
    terakhir check jekyll version:
    ```bash
    jekyll -v
    ```
    Installasi jekyll selesai dan sudah siap digunakan di lingkungan WSL.

2. Jalankan perintah untuk membuat website 
    
    ```bash
    jekyll new [nama_site]
    ```

    Ketika perintah tersebut dijalankan jekyll menggunakan theme minima sebagai tema default. dan website sudah bisa dijalankan dengan perintah berikut :

    Masuk ke directory site yang dibuat:
    ```bash
    cd nama_site
    ```

    Jalankan service:
    ```bash
    bundle exec jekyll serve
    ```

    Buka internet browser dan akses url localhost:4000 atau 127.0.0.0:4000. dan website dengan tampilan default theme seperti berikut:

    ![jekyllrun](https://1.bp.blogspot.com/-rMqzQdIc8-U/YG28DzRJNvI/AAAAAAAAUbA/NcYDWRnW1AM3Xw92m6-4j98THhfCXmZ3wCLcBGAsYHQ/s1497/minima-theme-jekyll.png)

    untuk mengubah nama website dan link sosial media edit variabel yang ada di file _config.yml.

3. Membuat posting
    
    Direktori site default jekyll pertama kali dibuat seperti ini :

    ```bash
    .
    ├── jekyll-cache
    ├── _posts
    ├──  └── 2021-04-07-welcome-to-jekyll.md
    ├── _sites
    ├── _config
    ├── 404.html
    ├── about.md
    ├── Gemfile
    ├── Gemfile.lock
    ├── index.md
    ```

    Untuk membuat post, didalam folder _posts buat file markdown dengan format nama file : empat digit tahun, diikuti tanda hyphen (-) kemudian dua digit bulan, tanda hyphen (-) lagi dan dua digit tanggal, tanda hypen (-) lagi diakhiri dengan judul dan ekstensi markdown (.md), seperti berikut ini :

    ```bash
    YYYY-MM-DD-title.md
    ```

    Setelah file dibuat, pada baris paling atas di file post tersebut tambahkan beberapa Front Matter, diantara tanda tripe dashes (---).
    Minimal dalam satu file post terdapat 2 keys berikut :

    ```bash
    ---
    layout: post
    title: 'I Love jekyll'
    ---
    ```

    File post sample ketika pertama kali site di buat terdapat 4 keys berikut :

    ```bash
    ---
    layout: post
    title: 'Welcome to Jekyll!'
    date: 2021-04-01 08-08-08 +0700
    catergories: jekyll update
    ---
    ```

    Dibawah Front Matter tersebut baru di ketik isi post yang mau dibuat, panduan markdown bisa dibaca di [Markdown Guide](https://www.markdownguide.org/)

4. Publish website ke GitHub Page

    Setelah website selesai di konfigurasi di lokal selanjutnya publish website tersebut ke GitHub Page, caranya sebagai berikut :
    1. Login ke Github
    2. Buat repository dengan nama username diikuti *.github.io*, contoh : [danynotes.github.io](https://danynotes.github.io/), set sebagai public repository
    3. Copy link repository tersebut
    4. Init git dan add remote repository di folder website komputer

        ```bash
        git init
        git remote add github url_repository_remote
        git add --all
        git commit -m 'initial commit'
        git push -u github master
        ```

Dan proses buat website statis dengan jekyll sudah jadi, buka browser dan check https://*usename*.github.io.

Kalau mau tampilan blog lebih menarik dari theme default jekyll. coba cari di [jekyll theme on rubygems.org](https://rubygems.org/search?utf8=%E2%9C%93&query=jekyll-theme) atau di [jekyllthemes.org](http://jekyllthemes.org/). Download dan override di folder website eksisting
atau lakukan *fork* dari repositori asal theme yang diinginkan.