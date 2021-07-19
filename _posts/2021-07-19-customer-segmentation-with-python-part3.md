---
layout: post
title: Data Science Customer Segmentation with python Part 3
categories: [Data Science]
tags: [Python, Data Science]
description : DQLab projects - Data Science Customer Segmentation with python Part 3.
date: 2021-07-19 14:03:00 +0700
---

Setelah melakukan [explorasi data](/data%20science/2021/07/19/customer-segmentation-with-python-part2) selanjutnya di part ke-3 ini kita melakukan persiapan data sebelum membuat model.

## Mempersiapkan Data sebelum permodelan

Setiap machine learning model memiliki karakteriktik yang berbeda-beda. Oleh karena itu harus mempersiapkan data yang dimiliki sebelum melakukan permodelan, sehingga dapat menyesuaikan dengan karakteristik yang dimiliki oleh tiap model dan mendapatkan hasil yang optimal.

Disini teknik permodelan yang akan digunakan adalah unsupervised clustering. Algoritma yang digunakan adalah K-Prototypes. Langkah-langkahnya sebagai berikut :  
- Salah satu faktor utama dalam algoritma ini adalah perlu menggunakan data yang skala antar variable nya setara,  
- Selain itu juga perlu melakukan encoding kolom-kolom kategorikal yang dimiliki menjadi numerik,    
- kemudian menggabungkan hasil pemrosesan data tersebut menjadi satu data frame untuk digunakan dalam permodelan.


### 1.Lakukan standarisasi kolom-kolom numerik supaya berada pada satu skala sehingga variable yang memiliki skala besar tidak mendominsasi bagaimana cluster akan dibentuk dan juga tiap variabel akan dianggap sama pentingnya oleh algoritma


```python
from sklearn.preprocessing import StandardScaler  
  
kolom_numerik = ['Umur','NilaiBelanjaSetahun']  
  
# Statistik sebelum Standardisasi  
print('Statistik Sebelum Standardisasi\n')  
print(df[kolom_numerik].describe().round(1))  
  
# Standardisasi  
df_std = StandardScaler().fit_transform(df[kolom_numerik])  
  
# Membuat DataFrame  
df_std = pd.DataFrame(data=df_std, index=df.index, columns=df[kolom_numerik].columns)  
  
# Menampilkan contoh isi data dan summary statistic  
print('Contoh hasil standardisasi\n')  
print(df_std.head())  
  
print('Statistik hasil standardisasi\n')  
print(df_std.describe().round(0))  

```

    Statistik Sebelum Standardisasi
    
           Umur  NilaiBelanjaSetahun
    count  50.0                 50.0
    mean   37.5            7069874.8
    std    14.7            2590619.0
    min    14.0            2722700.0
    25%    25.0            5257529.8
    50%    35.0            5980077.0
    75%    49.8            9739615.0
    max    64.0           10884508.0
    Contoh hasil standardisasi
    
           Umur  NilaiBelanjaSetahun
    0  1.411245             0.946763
    1 -1.617768            -1.695081
    2  0.722833            -0.695414
    3  1.067039            -0.727361
    4  0.240944             1.382421
    Statistik hasil standardisasi
    
           Umur  NilaiBelanjaSetahun
    count  50.0                 50.0
    mean    0.0                 -0.0
    std     1.0                  1.0
    min    -2.0                 -2.0
    25%    -1.0                 -1.0
    50%    -0.0                 -0.0
    75%     1.0                  1.0
    max     2.0                  1.0


### 2. Konversi Kategorikal Data dengan label encoder
Selanjutnya ubah kolom-kolom yang berjenis kategorikal menjadi angka. Gunakan fungsi LabelEncoder dari sklearn. Fungsi ini akan melakukan konversi data pelanggan dari teks menjadi numerik, contoh pada kolom Jenis Kelamin, tesk 'Pria' akan diubah menjadi angka 0 dan text 'Wanita' akan diubah menjadi 1.
Perubahan ini perlu kita untuk semua teks semua sebelum digunakan pada algoritma K-Prototype.


```python
from sklearn.preprocessing import StandardScaler
  
# Inisiasi nama kolom kategorikal  
kolom_numerik = ['Umur','Profesi','Tipe Residen']  
  
# Membuat salinan data frame  
df_encode = df[kolom_kategorikal].copy()  
  
  
# Melakukan labelEncoder untuk semua kolom kategorikal  
for col in kolom_kategorikal:  
    df_encode[col]= LabelEncoder().fit_transform(df_encode[col])
      
# Menampilkan data  
print(df_encode.head())

```

       Jenis Kelamin  Profesi  Tipe Residen
    0              0        4             1
    1              1        2             0
    2              0        3             0
    3              0        3             0
    4              1        4             0


### 3. Menggabungkan Data untuk Permodelan
Tahap ini menggabungkan data frame dari kedua pemrosesan sebelumnya yaitu dt_std dan df_encode.


```python
# Menggabungkan data frame
df_model= df_encode.merge(df_std, left_index = True, right_index=True, how= 'left') 
print(df_model.head())
```

       Jenis Kelamin  Profesi  Tipe Residen      Umur  NilaiBelanjaSetahun
    0              0        4             1  1.411245             0.946763
    1              1        2             0 -1.617768            -1.695081
    2              0        3             0  0.722833            -0.695414
    3              0        3             0  1.067039            -0.727361
    4              1        4             0  0.240944             1.382421


## Kesimpulan
Sampai tahap ini sudah dilakukan beberapa langkah untuk pemrosesan data, sebagai berikut :  
1. Menyiapkan library data  
2. Melakukan proses eksplorasi data  
3. Menyiapkan data untuk digunakan permodelan  

Pada penerapan di industri, proses pemrosesan data ini termasuk yang paling banyak menghabiskan waktu bagi data scientist. Selain yang sudah dilakukan diatas masih banyak teknik lain yang perlu dilakukan, misalnya :  
- Melakukan data imputation ketika ada *Null* data di data set yang akan digunakan  
- Melakukan transformasi variabel, ketika ditemukan distribusi data yang condong ke salah satu sisi (skew data)  
- Menangani pencilan data

Lanjut ke [pembuatan model](/data%20science/2021/07/19/customer-segmentation-with-python-part4).