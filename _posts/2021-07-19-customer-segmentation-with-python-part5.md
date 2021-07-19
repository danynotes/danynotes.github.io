---
layout: post
title: Data Science Customer Segmentation with python Part-5
categories: [Data Science]
tags: [Python, Data Science]
description : DQLab projects - Data Science Customer Segmentation with python Part 5.
date: 2021-07-19 14:03:00 +0700
---

Setelah model jadi pada [part-4](/data%20science/2021/07/19/customer-segmentation-with-python-part4), selanjutnya tahap mengoperasikan model.

# Mengoperasikan Model

Model yang telah dibuat harus bisa digunakan untuk memprediksi data baru. Untuk itu harus dipersiapkan datanya kembali dan kemudian melakukan prediksi dengan parameter dan model yang sudah di buat.

Jika model dapat dioperasionalkan, maka tim bisnis dapat dengan ceat mengetahui segmen dari pelanggan dan juga bisa mengatur strategi marketing dengan lebih efisien.

## Persiapkan Data Baru

Buat data baru untuk diprediksikan dengan model yang telah dibuat


```python
# Data Baru  
data = [{  
    'Customer_ID': 'CUST-100' ,  
    'Nama Pelanggan': 'Joko' ,  
    'Jenis Kelamin': 'Pria',  
    'Umur': 45,  
    'Profesi': 'Wiraswasta',  
    'Tipe Residen': 'Cluster' ,  
    'NilaiBelanjaSetahun': 8230000  
      
}]  
  
# Membuat Data Frame  
new_df = pd.DataFrame(data)  
  
# Melihat Data  
print(new_df)  
```

      Customer_ID Nama Pelanggan Jenis Kelamin  Umur     Profesi Tipe Residen  \
    0    CUST-100           Joko          Pria    45  Wiraswasta      Cluster   
    
       NilaiBelanjaSetahun  
    0              8230000  


## Membuat fungsi data pemrosesan

Selanjutnya perlu dibuat fungsi untuk melakukan pemrosesan data berdasarkan parameter yang sama pada saat melakukan permodelan dan dipanggil dengan data baru.

Fungsi yang dibuat akan digunakan untuk :
*1. Melakukan konversi data kategorikal menjadi numerik*

Dari proses sebelumnya didapatkan referensi untuk merubah data kategorikal menjadi numerik, sebagai berikut :

- Jenis Kelamin  
  0 : Pria
  1 : Wanita

- Profesi
  0 : Ibu Rumah Tangga
  1 : Mahasiswa
  2 : Pelajar
  3 : Professional
  4 : Wiraswasta

- Tipe Residen
  1 : Sector
  0 : Cluster

*2. Melakukan standarisasi kolom numerikal*

Untuk melakukan standarisasi dengan variable yang sama pada saat permodelan perlu menggunakan nilai rata-rata dan standard deviasi dari tiap variable pada saat kita melakukan permodelan, yaitu :

*Umur* :  
- Rata-rata : 37.5
- Standard deviasi : 14.7

*NilaiBelanjaSetahun*
- Rata-rata : 7069874.8
- Standart deviasi :2590619.0

Dari nilai tersebut dapat dihitung nilai standardisasi (z) dengan menggunakan rumus Z=(x-u)/s dengan x adalah tiap nilai, u adalah rata-rata dan s adalah standart deviasi.

*3. Menggabungkan hasil dua proses sebelumnya menjadi satu data frame*


```python
def data_preprocess(data):  
    # Konversi Kategorikal data  
    kolom_kategorikal = ['Jenis Kelamin','Profesi','Tipe Residen']
      
    df_encode = data[kolom_kategorikal].copy()  
  
    ## Jenis Kelamin   
    df_encode['Jenis Kelamin'] = df_encode['Jenis Kelamin'].map({  
        'Pria': 0,  
        'Wanita' : 1  
    })  
      
    ## Profesi  
    df_encode['Profesi'] = df_encode['Profesi'].map({  
        'Ibu Rumah Tangga': 0,  
        'Mahasiswa' : 1,  
        'Pelajar': 2,  
        'Professional': 3,  
        'Wiraswasta': 4  
    })  
      
    ## Tipe Residen  
    df_encode['Tipe Residen'] = df_encode['Tipe Residen'].map({  
        'Cluster': 0,  
        'Sector' : 1  
    })  
      
    # Standardisasi Numerical Data  
    kolom_numerik = ['Umur','NilaiBelanjaSetahun']  
    df_std = data[kolom_numerik].copy()  
      
    ## Standardisasi Kolom Umur  
    df_std['Umur'] = (df_std['Umur'] - 37.5)/14.7  
      
    ## Standardisasi Kolom Nilai Belanja Setahun  
    df_std['NilaiBelanjaSetahun'] = (df_std['NilaiBelanjaSetahun'] - 7069874.8)/2590619.0  
      
    # Menggabungkan Kategorikal dan numerikal data  
    df_model = df_encode.merge(df_std, left_index = True,  
                           right_index=True, how = 'left')  
      
    return df_model  
  
# Menjalankan fungsi  
new_df_model = data_preprocess(new_df)  
  
print(new_df_model) 

```

       Jenis Kelamin  Profesi  Tipe Residen      Umur  NilaiBelanjaSetahun
    0              0        4             0  0.510204             0.447818


## Memanggil Model dan melakukan prediksi
Setelah memiliki data yang siap digunakan saatnya memanggil model yang sudah di simpan sebelumnya dan melakukan prediksi.

Untuk melakukan hal tersebut, perlu dibuat prosesnya menjadi dalam satu fungsi yang bernama modelling dengan menggunakan data baru sebagai inputnya.


```python
def modelling (data):  
      
    # Memanggil Model  
    kpoto = pickle.load(open('cluster.pkl', 'rb'))  
      
    # Melakukan Prediksi  
    clusters = kpoto.predict(data,categorical=[0,1,2])  
      
    return clusters  
  
# Menjalankan Fungsi  
clusters = modelling(new_df_model)  
  
print(clusters)  

```

    [3]


## Menamakan Segmen
Seperti proses sebelumnya, untuk melakukan ini perlu dibuatkan fungsi. Nama cluster yang sudah didapat di tahap sebelumnya perlu diubah menjadi nama segmen agar lebih mudah diidentifikasi.


```python
def menamakan_segmen (data_asli, clusters):  
      
    # Menggabungkan cluster dan data asli  
    final_df = data_asli.copy()  
    final_df['cluster'] = clusters
      
    # Menamakan segmen  
    final_df['segmen'] = final_df['cluster'].map({  
        0: 'Diamond Young Member',  
        1: 'Diamond Senior Member',  
        2: 'Silver Students',  
        3: 'Gold Young Member',  
        4: 'Gold Senior Member'  
    })  
      
    return final_df
  
# Menjalankan Fungsi  
new_final_df = menamakan_segmen(new_df,clusters)  
  
print(new_final_df)  

```

      Customer_ID Nama Pelanggan Jenis Kelamin  Umur     Profesi Tipe Residen  \
    0    CUST-100           Joko          Pria    45  Wiraswasta      Cluster   
    
       NilaiBelanjaSetahun  cluster             segmen  
    0              8230000        3  Gold Young Member  


## Kesimpulan
Akhirnya, alur proses untuk mengoperasikan model sudah selesai dibuat. Selanjutnya penggunakan kode ini dapat dijadwalkan atau akan dijalankan secara real-time setiap ada data masuk atau secara batch misal satu hari sekali.

*Tips:*

Di industri khususnya untuk mengorperasikan model bisa bermacam-macam caranya. Ada yang membuat dengan python script lalu di buat interval jam jalan nya. Selain itu bisa juga menggunakan bantuan software untuk melakukan deployment.