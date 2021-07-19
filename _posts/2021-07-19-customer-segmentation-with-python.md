---
layout: post
title: Data Science Customer Segmentation with python Part 1
categories: [Data Science]
tags: [Python, Data Science]
description : DQLab projects - Data Science Customer Segmentation with python Part 1.
date: 2021-07-19 14:03:00 +0700
---

# Data Science in Marketing: Customer Segmentation with Python

Salah satu teknik yang bisa dilakukan untuk mengenal pelanggan lebih baik adalah dengan melakukan segmentasi pelanggan. Yaitu dengan mengelompokkan pelanggan-pelanggan yang ada berdasarkan kesamaan karakter dari pelanggan tersebut. Untuk melakukan hal tersebut gunakan teknik unsupervised machine learning.

*Salah satu teknik yang dapat digunakan adalah mengaplikasikan machine learning menggunakan algorima K-Prototypes. Algorima K-Prototypes merupakan gabungan dari K-Means dan K-Modes yang dapat digunakan untuk melakukan segmentasi dengan data*.

Data yang digunakan saat ini memiliki kolom dengan rician sebagai berikut :  
- CustomerID : Kode Pelanggan dengan format campuran text *CUST-* diikuti anggka  
- Nama Pelanggan : Nama dari pelanggan dengan format text  
- Jenis Kelamin : Jenis Kelamin dari pelanggan. hanya terdapat dua isidata kategori yaitu Pria dan Wanita  
- Umur : Umur dari pelanggan dalam format angka
- Profesi : Profesi dari pelanggan, bertipe text kategoru yang terdiri dari : Wiraswasta, Pelajar, Professional, Ibu Rumah Tangga, dan Mahasiswa.  
- Tipe Residen : Tipe tempat tinggal dari pelanggan, terdiri dari 2 kategori yaitu Cluster dan Sector.  
- NilaiBelanjaSetahun : merupakan total belanja yang sudah dikeluarka oleh pelanggan tersebut.



## Persiapkan Library
- Pandas, di gunakan untuk melakukan pemrosesan analisis data  
- Matplotlib, di gunakan sebagai dasar untuk melakukan visualisasi data  
- Seaborn, di gunakan di atas matplotlib untuk melakukan data visualisasi yang lebih menarik  
- Scikit - Learn, digunakan untuk mempersiapkan data sebelum dilakukan permodelan  
- kmodes, digunakan untuk melakukan permodelan menggunakan algoritma K-Modes dan K-Prototypes.  
- Pickle, digunakan untuk melakukan penyimpanan dari model yang akan di buat  


```python
import pandas as pd  
import matplotlib.pyplot as plt  
import seaborn as sns  
from sklearn.preprocessing import LabelEncoder
  
from kmodes.kmodes import KModes  
from kmodes.kprototypes import KPrototypes  
import pickle  
from pathlib import Path
```


```python
# Baca Data Pelanggan
# import dataset  
df = pd.read_csv("https://storage.googleapis.com/dqlab-dataset/customer_segments.txt", sep="\t")  
  
# menampilkan data  
print(df.head())
```  

    Customer_ID       Nama Pelanggan Jenis Kelamin  Umur       Profesi  \
    0    CUST-001         Budi Anggara          Pria    58    Wiraswasta   
    1    CUST-002     Shirley Ratuwati        Wanita    14       Pelajar   
    2    CUST-003         Agus Cahyono          Pria    48  Professional   
    3    CUST-004     Antonius Winarta          Pria    53  Professional   
    4    CUST-005  Ibu Sri Wahyuni, IR        Wanita    41    Wiraswasta   
    
      Tipe Residen  NilaiBelanjaSetahun  
    0       Sector              9497927  
    1      Cluster              2722700  
    2      Cluster              5286429  
    3      Cluster              5204498  
    4      Cluster             10615206  

## Melihat informasi dari Data
Lihat informasi jumlah kolom, nama kolom, indentifikasi null values dan tipe data.


```python
df.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 50 entries, 0 to 49
    Data columns (total 7 columns):
     #   Column               Non-Null Count  Dtype 
    ---  ------               --------------  ----- 
     0   Customer_ID          50 non-null     object
     1   Nama Pelanggan       50 non-null     object
     2   Jenis Kelamin        50 non-null     object
     3   Umur                 50 non-null     int64 
     4   Profesi              50 non-null     object
     5   Tipe Residen         50 non-null     object
     6   NilaiBelanjaSetahun  50 non-null     int64 
    dtypes: int64(2), object(5)
    memory usage: 2.9+ KB


## Kesimpulan
Dalam setiap project *machine learning* kita harus memahami informasi dasar dari data yang kita miliki sebelum melakukan analisa lebih lanjut, memastikan tipe data dari masing-masing kolom sudah benar, mengetahui apakah ada data null di tiap-tiap kolom, dan juga mengetahui nama-nama kolom di dataset yang digunakan. dari perintah diatas dapat diketahui bahwa :  
- Data yang akan digunakan terdir dari 50 baris dan 7 kolom.  
- Tidak ada nilai Null pada data
- Dua kolom memiliki tipe data numeric dan lima data bertipe string

Lanjut [Explorasi Data](/data%20science/2021/07/19/customer-segmentation-with-python-part2) 