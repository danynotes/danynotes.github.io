---
layout: post
title: Data Science Customer Segmentation with python
categories: [Data Science]
tags: [Python, Data Science]
description : DQLab projects - Data Science Customer Segmentation with python.
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


## Melakukan Eksplorasi Data
Lakukan eksplorasi data untuk lebih mengenal dataset yang akan digunakan. Lakukan eksplorasi untuk data numerik dan juga data kategorikal.

### Eksplorasi Data Numerik
Pertama-tama perlu melihat distribusi data dari data yang berjenis numerik. Gunakan boxplot dan juga histogram untuk melihat distribusi datanya. Untuk melihat grafik tersebut, perlu disiapkan kolom mana yang merupakan numerik, kemudian menggunakan library seaborn untuk membuat plot masing-masing kolom numerik, yaitu *'Umur'* dan *'NilaiBelanjaSetahun'*.

Buat grafik boxplot dan histogram untuk kolom *Umur* dan juga *NilaiBelanjaSetahun*.

```python
sns.set(style='white')
plt.clf()
  
# Fungsi untuk membuat plot  
def observasi_num(features):  
    fig, axs = plt.subplots(2, 2, figsize=(10, 9))
    for i, kol in enumerate(features):
	    sns.boxplot(df[kol], ax = axs[i][0])
	    sns.distplot(df[kol], ax = axs[i][1])   
	    axs[i][0].set_title('mean = %.2f\n median = %.2f\n std = %.2f'%(df[kol].mean(), df[kol].median(), df[kol].std()))
    plt.setp(axs)
    plt.tight_layout()
    plt.show()  
  
# Memanggil fungsi untuk membuat Plot untuk data numerik  
kolom_numerik = ['Umur','NilaiBelanjaSetahun'] 
observasi_num(kolom_numerik) 
```  

      adjustable: {'box', 'datalim'}
      agg_filter: a filter function, which takes a (m, n, 3) float array and a dpi value, and returns a (m, n, 3) array
      alpha: scalar or None
      anchor: 2-tuple of floats or {'C', 'SW', 'S', 'SE', ...}
      animated: bool
      aspect: {'auto'} or num
      autoscale_on: bool
      autoscalex_on: bool
      autoscaley_on: bool
      axes_locator: Callable[[Axes, Renderer], Bbox]
      axisbelow: bool or 'line'
      box_aspect: float or None
      clip_box: `.Bbox`
      clip_on: bool
      clip_path: Patch or (Path, Transform) or None
      contains: unknown
      facecolor or fc: color
      figure: `.Figure`
      frame_on: bool
      gid: str
      in_layout: bool
      label: object
      navigate: bool
      navigate_mode: unknown
      path_effects: `.AbstractPathEffect`
      picker: None or bool or float or callable
      position: [left, bottom, width, height] or `~matplotlib.transforms.Bbox`
      prop_cycle: unknown
      rasterization_zorder: float or None
      rasterized: bool
      sketch_params: (scale: float, length: float, randomness: float)
      snap: bool or None
      subplotspec: unknown
      title: str
      transform: `.Transform`
      url: str
      visible: bool
      xbound: unknown
      xlabel: str
      xlim: (bottom: float, top: float)
      xmargin: float greater than -0.5
      xscale: {"linear", "log", "symlog", "logit", ...} or `.ScaleBase`
      xticklabels: unknown
      xticks: unknown
      ybound: unknown
      ylabel: str
      ylim: (bottom: float, top: float)
      ymargin: float greater than -0.5
      yscale: {"linear", "log", "symlog", "logit", ...} or `.ScaleBase`
      yticklabels: unknown
      yticks: unknown
      zorder: float



    <Figure size 432x288 with 0 Axes>

<img src="https://lh3.googleusercontent.com/-J7ZOaYg-7dw/YPOv2UwQfoI/AAAAAAAAVyw/mkQKM4tnipI2YvgY59vq3SPAMnqcdsgvgCLcBGAsYHQ/w320-h291/data_science_in_marketing_customer_segmentation_with_python_9_3.png" alt="data_science_in_marketing_customer_segmentation_with_python_boxplot" width="500"/>  

<br>
<br>
<br>

### Eksplorasi Data Kategorikal
Selain data numerikal, perlu dilihat juga sebaran data pada kolom-kolom yang memiliki jenis kategorikal yaitu Jenis Kelamin, Profesi dan Tipe Residen, hal ini dapat dilakukan dengan menggunakan method countplot dari library seaborn.


```python
sns.set(style='white')
plt.clf()
  
# Menyiapkan kolom kategorikal  
kolom_kategorikal = ['Jenis Kelamin','Profesi','Tipe Residen']  

# Membuat canvas
fig, axs = plt.subplots(3,1,figsize=(7,10)) 

# Membuat plot untuk setiap kolom kategorikal  
for i, kol in enumerate(kolom_kategorikal):  
    # Membuat Plot
    sns.countplot(df[kol], order = df[kol].value_counts().index, ax = axs[i])  
    axs[i].set_title('\nCount Plot %s\n'%(kol), fontsize=15)  
      
    # Memberikan anotasi  
    for p in axs[i].patches:  
        axs[i].annotate(format(p.get_height(), '.0f'),  
                        (p.get_x() + p.get_width() / 2., p.get_height()),  
                        ha = 'center',  
                        va = 'center',  
                        xytext = (0, 10),  
                        textcoords = 'offset points') 
          
    # Setting Plot  
    sns.despine(right=True,top = True, left = True)  
    axs[i].axes.yaxis.set_visible(False) 
    plt.setp(axs)
    plt.tight_layout()

# Tampilkan plot
plt.show()
```
      adjustable: {'box', 'datalim'}
      agg_filter: a filter function, which takes a (m, n, 3) float array and a dpi value, and returns a (m, n, 3) array
      alpha: scalar or None
      anchor: 2-tuple of floats or {'C', 'SW', 'S', 'SE', ...}
      animated: bool
      aspect: {'auto'} or num
      autoscale_on: bool
      autoscalex_on: bool
      autoscaley_on: bool
      axes_locator: Callable[[Axes, Renderer], Bbox]
      axisbelow: bool or 'line'
      box_aspect: float or None
      clip_box: `.Bbox`
      clip_on: bool
      clip_path: Patch or (Path, Transform) or None
      contains: unknown
      facecolor or fc: color
      figure: `.Figure`
      frame_on: bool
      gid: str
      in_layout: bool
      label: object
      navigate: bool
      navigate_mode: unknown
      path_effects: `.AbstractPathEffect`
      picker: None or bool or float or callable
      position: [left, bottom, width, height] or `~matplotlib.transforms.Bbox`
      prop_cycle: unknown
      rasterization_zorder: float or None
      rasterized: bool
      sketch_params: (scale: float, length: float, randomness: float)
      snap: bool or None
      subplotspec: unknown
      title: str
      transform: `.Transform`
      url: str
      visible: bool
      xbound: unknown
      xlabel: str
      xlim: (bottom: float, top: float)
      xmargin: float greater than -0.5
      xscale: {"linear", "log", "symlog", "logit", ...} or `.ScaleBase`
      xticklabels: unknown
      xticks: unknown
      ybound: unknown
      ylabel: str
      ylim: (bottom: float, top: float)
      ymargin: float greater than -0.5
      yscale: {"linear", "log", "symlog", "logit", ...} or `.ScaleBase`
      yticklabels: unknown
      yticks: unknown
      zorder: float
      adjustable: {'box', 'datalim'}
      agg_filter: a filter function, which takes a (m, n, 3) float array and a dpi value, and returns a (m, n, 3) array
      alpha: scalar or None
      anchor: 2-tuple of floats or {'C', 'SW', 'S', 'SE', ...}
      animated: bool
      aspect: {'auto'} or num
      autoscale_on: bool
      autoscalex_on: bool
      autoscaley_on: bool
      axes_locator: Callable[[Axes, Renderer], Bbox]
      axisbelow: bool or 'line'
      box_aspect: float or None
      clip_box: `.Bbox`
      clip_on: bool
      clip_path: Patch or (Path, Transform) or None
      contains: unknown
      facecolor or fc: color
      figure: `.Figure`
      frame_on: bool
      gid: str
      in_layout: bool
      label: object
      navigate: bool
      navigate_mode: unknown
      path_effects: `.AbstractPathEffect`
      picker: None or bool or float or callable
      position: [left, bottom, width, height] or `~matplotlib.transforms.Bbox`
      prop_cycle: unknown
      rasterization_zorder: float or None
      rasterized: bool
      sketch_params: (scale: float, length: float, randomness: float)
      snap: bool or None
      subplotspec: unknown
      title: str
      transform: `.Transform`
      url: str
      visible: bool
      xbound: unknown
      xlabel: str
      xlim: (bottom: float, top: float)
      xmargin: float greater than -0.5
      xscale: {"linear", "log", "symlog", "logit", ...} or `.ScaleBase`
      xticklabels: unknown
      xticks: unknown
      ybound: unknown
      ylabel: str
      ylim: (bottom: float, top: float)
      ymargin: float greater than -0.5
      yscale: {"linear", "log", "symlog", "logit", ...} or `.ScaleBase`
      yticklabels: unknown
      yticks: unknown
      zorder: float
      adjustable: {'box', 'datalim'}
      agg_filter: a filter function, which takes a (m, n, 3) float array and a dpi value, and returns a (m, n, 3) array
      alpha: scalar or None
      anchor: 2-tuple of floats or {'C', 'SW', 'S', 'SE', ...}
      animated: bool
      aspect: {'auto'} or num
      autoscale_on: bool
      autoscalex_on: bool
      autoscaley_on: bool
      axes_locator: Callable[[Axes, Renderer], Bbox]
      axisbelow: bool or 'line'
      box_aspect: float or None
      clip_box: `.Bbox`
      clip_on: bool
      clip_path: Patch or (Path, Transform) or None
      contains: unknown
      facecolor or fc: color
      figure: `.Figure`
      frame_on: bool
      gid: str
      in_layout: bool
      label: object
      navigate: bool
      navigate_mode: unknown
      path_effects: `.AbstractPathEffect`
      picker: None or bool or float or callable
      position: [left, bottom, width, height] or `~matplotlib.transforms.Bbox`
      prop_cycle: unknown
      rasterization_zorder: float or None
      rasterized: bool
      sketch_params: (scale: float, length: float, randomness: float)
      snap: bool or None
      subplotspec: unknown
      title: str
      transform: `.Transform`
      url: str
      visible: bool
      xbound: unknown
      xlabel: str
      xlim: (bottom: float, top: float)
      xmargin: float greater than -0.5
      xscale: {"linear", "log", "symlog", "logit", ...} or `.ScaleBase`
      xticklabels: unknown
      xticks: unknown
      ybound: unknown
      ylabel: str
      ylim: (bottom: float, top: float)
      ymargin: float greater than -0.5
      yscale: {"linear", "log", "symlog", "logit", ...} or `.ScaleBase`
      yticklabels: unknown
      yticks: unknown
      zorder: float

    <Figure size 432x288 with 0 Axes>

    
<img src="https://lh3.googleusercontent.com/-7QrioQh0riU/YPOv2iyDkpI/AAAAAAAAVy8/diM9pKQnsZAMLPXFEdxE9DMUp7dKj1M2QCLcBGAsYHQ/s16000/data_science_in_marketing_customer_segmentation_with_python_11_3.png" alt="data science in marketing customer segmentation with python count plot"/>

<br>
<br>

## Kesimpulan
Dari hasil explorasi data tersebut didapat informasi bahwa :  
- Rata-rata dari umur pelanggan adalah 37.5 tahun
- Rata-rata dari nilai belanja setahun pelanggan adalah 7,069,874.82
- Jenis kelamin pelanggan di dominasi oleh wanita sebanyak 41 orang (82%) dan laki-laki sebanyak 9 orang (18%)  
- Profesi terbanyak adalah Wiraswasta(40%) diikuti Profesional (36%) dan lainnya sebanyak 24%  
- Dari seluruh pelanggan 64% dari mereka tinggal di cluster dan 36% nya tinggal di sektor.

### Tips:
Dengan eksplorasi data kita dapat mengenal lebih jauh tentang data yang ada.
Proses eskplorasi data bisa berupa univariate maupun multivariate data eksplorasi. Eksplorasi Data Univariate melihat karakteristik tiap-tiap feature, misalnya dengan melihat statistik destriptif, membuat histogram, kdeplot, count plot maupun boxplot. Sedangkan untuk Eksplorasi Data Multivariate, untuk melihat hubungan tiap variabel dengan variabel lainnya, misalkan dengan membuat korelasi matrix, melihat predictive power, cross tabulasi dan lainnya.

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

## Clustering dan Algoritma K-Prototypes
**Clustering** adalah proses pembagian objek-objek ke dalam beberapa kelompok (cluster) berdasarkan tingkat kemiripan antara satu objek dengan yang lain.

Terdapat beberapa algoritma untuk melakukan clustering ini. Salah satu yang populer adalah k-means.

**K-means** biasannya hanya digunakan untuk data-data yang bertipe numerik. Sedangkan untuk yang bertipe kategorikal bisa menggunakan **K-modes**.

Jika data memiliki gabungan tipe kategorikal dan numerikal variabel dapat menggunakan algorima **K-prototype** yang merupakan gaungan dar **K-means** dan **K-modes**.

Untuk menggunakan algoritma k-prototype perlu memasukkan jumlah cluster yang dikehendaki dan juga memberikan index kolom untuk kolom-kolom yang bersifat katergorikal.

*For more information check K-prototype documentation on [github/nicodv/kmodes](https://github.nicodv/kmodes)*.

## Mencari jumlah cluster yang optimal  

Salah satu parameter penting yang harus dimasukkan pada algorima k-prototype adalah jumlah cluster yang diinginkan. Oleh karena itu perlu mendapatkan jumlah cluster yang optimal.Salah satu cara mendapatkan nilai optimal tersebut adalah dengan menggunakan bantuan #'elbow plot'#.

Elbow plot ini dapat dibuat dengan cara memvisualisasikan total jarak seluruh data kita ke pusat cluster nya. Selanjutnya dipilih titik siku dari pola yang terbentuk dan menjadikannya sebagai jumlah cluster yang di inginkan.


```python
from kmodes.kmodes import KModes
from kmodes.kprototypes import KPrototypes
import pandas as pd
import seaborn as sns
import matplotlib.pyplot as plt
df_model = pd.read_csv('https://storage.googleapis.com/dqlab-dataset/df-customer-segmentation.csv')

# Melakukan Iterasi untuk Mendapatkan nilai Cost
cost = {}
for k in range(2,10):
    kproto = KPrototypes(n_clusters = k,random_state=75)
    kproto.fit_predict(df_model, categorical=[0,1,2])
    cost[k]= kproto.cost_

# Memvisualisasikan Elbow Plot
sns.pointplot(x=list(cost.keys()), y=list(cost.values()))
plt.show()
```

<img src="https://lh3.googleusercontent.com/-tK5Bfb-1u2U/YPOv2jusZHI/AAAAAAAAVy0/myVFbnLR7v0M-oYfhuhkGtD8GLeXOc-tgCLcBGAsYHQ/s16000/data_science_in_marketing_customer_segmentation_with_python_22_0.png" alt="data_science_in_marketing_customer_segmentation_with_python_poinplot">

<br>
<br>


## Membuat Model

Selanjutnya kamu dapat melakukan pembuatan model dengan jumlah kluster yang sudah didapat pada tahap sebelumnya yaitu 5 dan menyimpan hasilnya sebagai pickle file


```python
import pickle
from kmodes.kmodes import KModes
from kmodes.kprototypes import KPrototypes  
  
kproto = KPrototypes(n_clusters = 5, random_state = 75)  
kproto = kproto.fit(df_model, categorical=[0,1,2])  
  
#Save Model  
pickle.dump(kproto, open('cluster.pkl', 'wb'))  
```

## Menggunakan Model
Model yang sudah dibuat, dapat digunakan untuk menentukan setiap pelaggan masuk ke dalam cluster yang mana. Kali ini model akan digunakan untuk menentukan segmen pelanggan yang ada di set.


```python
import pandas as pd
df = pd.read_csv("https://dqlab-dataset.s3-ap-southeast-1.amazonaws.com/customer_segments.txt", sep="\t") 
# Menentukan segmen tiap pelanggan    
clusters =  kproto.predict(df_model, categorical=[0,1,2])    
print('segmen pelanggan: {}\n'.format(clusters))    
    
# Menggabungkan data awal dan segmen pelanggan    
df_final = df.copy()    
df_final['cluster'] = clusters
print(df_final.head()) 
```

    segmen pelanggan: [3 1 2 2 0 4 3 2 4 4 2 2 3 3 0 4 4 2 0 1 0 2 4 0 0 2 0 4 2 2 1 3 1 0 4 0 4
     3 4 1 4 0 4 0 4 0 2 3 4 3]
    
      Customer_ID       Nama Pelanggan Jenis Kelamin  Umur       Profesi  \
    0    CUST-001         Budi Anggara          Pria    58    Wiraswasta   
    1    CUST-002     Shirley Ratuwati        Wanita    14       Pelajar   
    2    CUST-003         Agus Cahyono          Pria    48  Professional   
    3    CUST-004     Antonius Winarta          Pria    53  Professional   
    4    CUST-005  Ibu Sri Wahyuni, IR        Wanita    41    Wiraswasta   
    
      Tipe Residen  NilaiBelanjaSetahun  cluster  
    0       Sector              9497927        3  
    1      Cluster              2722700        1  
    2      Cluster              5286429        2  
    3      Cluster              5204498        2  
    4      Cluster             10615206        0  



```python
# Menampilkan data pelanggan berdasarkan cluster
for i in range (0,5):
    print('\nPelanggan cluster: {}\n'.format(i))
    print(df_final[df_final['cluster']== i])
```


    Pelanggan cluster: 0
    
       Customer_ID       Nama Pelanggan Jenis Kelamin  Umur     Profesi  \
    4     CUST-005  Ibu Sri Wahyuni, IR        Wanita    41  Wiraswasta   
    14    CUST-015     Shirley Ratuwati        Wanita    20  Wiraswasta   
    18    CUST-019         Mega Pranoto        Wanita    32  Wiraswasta   
    20    CUST-021     Lestari Fabianto        Wanita    38  Wiraswasta   
    23    CUST-024        Putri Ginting        Wanita    39  Wiraswasta   
    24    CUST-025       Julia Setiawan        Wanita    29  Wiraswasta   
    26    CUST-027        Grace Mulyati        Wanita    35  Wiraswasta   
    33    CUST-034       Deasy Arisandi        Wanita    21  Wiraswasta   
    35    CUST-036       Ni Made Suasti        Wanita    30  Wiraswasta   
    41    CUST-042         Yuliana Wati        Wanita    26  Wiraswasta   
    43    CUST-044                 Anna        Wanita    18  Wiraswasta   
    45    CUST-046         Elfira Surya        Wanita    25  Wiraswasta   
    
       Tipe Residen  NilaiBelanjaSetahun  cluster  
    4       Cluster             10615206        0  
    14      Cluster             10365668        0  
    18      Cluster             10884508        0  
    20      Cluster              9222070        0  
    23      Cluster             10259572        0  
    24       Sector             10721998        0  
    26      Cluster              9114159        0  
    33       Sector              9759822        0  
    35      Cluster              9678994        0  
    41      Cluster              9880607        0  
    43      Cluster              9339737        0  
    45       Sector             10099807        0  
    
    Pelanggan cluster: 1
    
       Customer_ID    Nama Pelanggan Jenis Kelamin  Umur    Profesi Tipe Residen  \
    1     CUST-002  Shirley Ratuwati        Wanita    14    Pelajar      Cluster   
    19    CUST-020    Irene Novianto        Wanita    16    Pelajar       Sector   
    30    CUST-031     Eviana Handry        Wanita    19  Mahasiswa      Cluster   
    32    CUST-033   Cecilia Kusnadi        Wanita    19  Mahasiswa      Cluster   
    39    CUST-040    Irene Darmawan        Wanita    14    Pelajar       Sector   
    
        NilaiBelanjaSetahun  cluster  
    1               2722700        1  
    19              2896845        1  
    30              3042773        1  
    32              3047926        1  
    39              2861855        1  
    
    Pelanggan cluster: 2
    
       Customer_ID     Nama Pelanggan Jenis Kelamin  Umur           Profesi  \
    2     CUST-003       Agus Cahyono          Pria    48      Professional   
    3     CUST-004   Antonius Winarta          Pria    53      Professional   
    7     CUST-008     Danang Santosa          Pria    52      Professional   
    10    CUST-011     Maria Suryawan        Wanita    50      Professional   
    11    CUST-012    Erliana Widjaja        Wanita    49      Professional   
    17    CUST-018        Nelly Halim        Wanita    63  Ibu Rumah Tangga   
    21    CUST-022       Novita Purba        Wanita    52      Professional   
    25    CUST-026  Christine Winarto        Wanita    55      Professional   
    28    CUST-029       Tia Hartanti        Wanita    56      Professional   
    29    CUST-030     Rosita Saragih        Wanita    46  Ibu Rumah Tangga   
    46    CUST-047        Mira Kurnia        Wanita    55  Ibu Rumah Tangga   
    
       Tipe Residen  NilaiBelanjaSetahun  cluster  
    2       Cluster              5286429        2  
    3       Cluster              5204498        2  
    7       Cluster              5223569        2  
    10       Sector              5987367        2  
    11       Sector              5941914        2  
    17      Cluster              5340690        2  
    21      Cluster              5298157        2  
    25      Cluster              5269392        2  
    28      Cluster              5271845        2  
    29       Sector              5020976        2  
    46      Cluster              6130724        2  
    
    Pelanggan cluster: 3
    
       Customer_ID    Nama Pelanggan Jenis Kelamin  Umur     Profesi Tipe Residen  \
    0     CUST-001      Budi Anggara          Pria    58  Wiraswasta       Sector   
    6     CUST-007     Cahyono, Agus          Pria    64  Wiraswasta       Sector   
    12    CUST-013      Cahaya Putri        Wanita    64  Wiraswasta      Cluster   
    13    CUST-014    Mario Setiawan          Pria    60  Wiraswasta      Cluster   
    31    CUST-032   Chintya Winarni        Wanita    47  Wiraswasta       Sector   
    37    CUST-038      Agatha Salim        Wanita    46  Wiraswasta       Sector   
    47    CUST-048  Maria Hutagalung        Wanita    45  Wiraswasta       Sector   
    49    CUST-050    Lianna Nugraha        Wanita    55  Wiraswasta       Sector   
    
        NilaiBelanjaSetahun  cluster  
    0               9497927        3  
    6               9837260        3  
    12              9333168        3  
    13              9471615        3  
    31             10663179        3  
    37             10477127        3  
    47             10390732        3  
    49             10569316        3  
    
    Pelanggan cluster: 4
    
       Customer_ID         Nama Pelanggan Jenis Kelamin  Umur           Profesi  \
    5     CUST-006        Rosalina Kurnia        Wanita    24      Professional   
    8     CUST-009  Elisabeth Suryadinata        Wanita    29      Professional   
    9     CUST-010         Mario Setiawan          Pria    33      Professional   
    15    CUST-016           Bambang Rudi          Pria    35      Professional   
    16    CUST-017              Yuni Sari        Wanita    32  Ibu Rumah Tangga   
    22    CUST-023        Denny Amiruddin          Pria    34      Professional   
    27    CUST-028          Adeline Huang        Wanita    40  Ibu Rumah Tangga   
    34    CUST-035                Ida Ayu        Wanita    39      Professional   
    36    CUST-037       Felicia Tandiono        Wanita    25      Professional   
    38    CUST-039           Gina Hidayat        Wanita    20      Professional   
    40    CUST-041       Shinta Aritonang        Wanita    24  Ibu Rumah Tangga   
    42    CUST-043           Yenna Sumadi        Wanita    31      Professional   
    44    CUST-045         Rismawati Juni        Wanita    22      Professional   
    48    CUST-049        Josephine Wahab        Wanita    33  Ibu Rumah Tangga   
    
       Tipe Residen  NilaiBelanjaSetahun  cluster  
    5       Cluster              5215541        4  
    8        Sector              5993218        4  
    9       Cluster              5257448        4  
    15      Cluster              5262521        4  
    16      Cluster              5677762        4  
    22      Cluster              5239290        4  
    27      Cluster              6631680        4  
    34       Sector              5962575        4  
    36       Sector              5972787        4  
    38      Cluster              5257775        4  
    40      Cluster              6820976        4  
    42      Cluster              5268410        4  
    44      Cluster              5211041        4  
    48       Sector              4992585        4  



```python
# Visualisasi Hasil Clustering menggunakan Box Plot
import matplotlib.pyplot as plt
# Data Numerical
kolom_numerik = ['Umur','NilaiBelanjaSetahun']  
  
for i in kolom_numerik:  
    plt.figure(figsize=(6,4))  
    ax = sns.boxplot(x = 'cluster',y = i, data = df_final)  
    plt.title('\nBox Plot {}\n'.format(i), fontsize=12)
    plt.show()

```

<img src="https://lh3.googleusercontent.com/-V__9bdGfO1o/YPOv2g381ZI/AAAAAAAAVzA/-JOUsscoQW8mzI0G1B65NWi0ypfJEeWjgCLcBGAsYHQ/s16000/data_science_in_marketing_customer_segmentation_with_python_28_0.png" alt="data_science_in_marketing_customer_segmentation_with_python_boxplot_umur"/>

<br>
<img src="https://lh3.googleusercontent.com/-TarODThRMQo/YPOv2dDVEOI/AAAAAAAAVys/48y_fznko9w3GMzITw1ctlA_JB9KZA5nwCLcBGAsYHQ/s16000/data_science_in_marketing_customer_segmentation_with_python_28_1.png" alt="data_science_in_marketing_customer_segmentation_with_python_boxplot_nilaibelanjasetahun"/>

<br><br>

```python
# Visualisasi Hasil Clustering menggunakan count plot
import matplotlib.pyplot as plt  
# Data Kategorikal  
kolom_categorical = ['Jenis Kelamin','Profesi','Tipe Residen']  
  
for i in kolom_categorical:  
    plt.figure(figsize=(6,4))  
    ax = sns.countplot(data = df_final, x = 'cluster', hue = i )  
    plt.title('\nCount Plot {}\n'.format(i), fontsize=12)  
    ax.legend(loc="upper center")  
    for p in ax.patches:  
        ax.annotate(format(p.get_height(), '.0f'),  
                    (p.get_x() + p.get_width() / 2., p.get_height()),  
                     ha = 'center',  
                     va = 'center',  
                     xytext = (0, 10),  
                     textcoords = 'offset points')  
      
    sns.despine(right=True,top = True, left = True)  
    ax.axes.yaxis.set_visible(False)  
    plt.show()  

```

<img src="https://lh3.googleusercontent.com/-uT23bZwQwdQ/YPOv2rSJs8I/AAAAAAAAVy4/I_LdBEoh7s8BoUhS0UpIBQGRh6dwupurgCLcBGAsYHQ/s16000/data_science_in_marketing_customer_segmentation_with_python_29_0.png" alt="data_science_in_marketing_customer_segmentation_with_python_boxplot_jeniskelamin">

<img src="https://lh3.googleusercontent.com/--6uUmn6a-bU/YPOv3GwqzTI/AAAAAAAAVzE/90-UjEwyc1QJjUsZQcYpVOWYLkprJH2HgCLcBGAsYHQ/s16000/data_science_in_marketing_customer_segmentation_with_python_29_1.png" alt="data_science_in_marketing_customer_segmentation_with_python_countplot_pprofesi"/>

<img src="https://lh3.googleusercontent.com/-hOCEBUdz7BM/YPOv3X6PJJI/AAAAAAAAVzI/2Qa4iLpgv74fFzszHfy7OFoyeQ4fIAvWACLcBGAsYHQ/s16000/data_science_in_marketing_customer_segmentation_with_python_29_2.png" alt="https://lh3.googleusercontent.com/-hOCEBUdz7BM/YPOv3X6PJJI/AAAAAAAAVzI/2Qa4iLpgv74fFzszHfy7OFoyeQ4fIAvWACLcBGAsYHQ/s16000/data_science_in_marketing_customer_segmentation_with_python_countplot_tiperesiden"/>

<br><br>
## Memberi nama cluster  

Dari hasil observasi yang telah dilakukan dapat diberikan nama segmen dari tiap-tiap nomor clusternya, yaitu :  
- Cluster 0 : Diamond Young Enterprener.  
  Isi cluster ini adalah para wiraswasta yang memiliki nilai transaksi rata-rata mendekati 10 juta. Selain itu isi dari cluster ini memiliki umur sekitar 18-41 tahun dengan rata-ratanya adalah 29 tahun.  
- Cluster 1 : Diamond Senior Enterpreneur.  
  Isi cluster ini adalah para wiraswasta yang memiliki nilai transaksi rata-rata mendekatai 10 juta. Isi dari cluster ini memiliki umur sekitar 45 - 64 tahun dengan rata - rata nya adalah 55 tahun.  
- Cluster 2: Silver Students.   
  Isi cluster ini adalah para pelajar dan mahasiswa dengan rata-rata umur mereka adalah 16 tahun dan nilai belanja setahun mendekati 3 juta.  
- Cluster 3: Gold Young Member.  
  Isi cluster ini adalah para professional dan ibu rumah tangga yang berusia muda dengan rentang umur sekitar 20 - 40 tahun dan dengan rata-rata 30 tahun dan nilai belanja setahun nya mendekati 6 juta.  
- Cluster 4: Gold Senior Member.  
  Isi cluster ini adalah para professional dan ibu rumah tangga yang berusia tua dengan rentang umur 46 - 63 tahun dan dengan rata-rata 53 tahun dan nilai belanja setahun nya mendekati 6 juta.

Tambahkan kolom dengan nama segmen yang berisi nama segmen dari tiap-tiap pelanggan berdasarkan nilai clusternya.



```python
# Mapping nama kolom  
df_final['segmen'] = df_final['cluster'].map({  
    0: 'Diamond Young Member',  
    1: 'Diamond Senior Member',  
    2: 'Silver Member',  
    3: 'Gold Young Member',  
    4: 'Gold Senior Member'  
})  

print(df_final.info())
print(df_final.head())  
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 50 entries, 0 to 49
    Data columns (total 9 columns):
     #   Column               Non-Null Count  Dtype 
    ---  ------               --------------  ----- 
     0   Customer_ID          50 non-null     object
     1   Nama Pelanggan       50 non-null     object
     2   Jenis Kelamin        50 non-null     object
     3   Umur                 50 non-null     int64 
     4   Profesi              50 non-null     object
     5   Tipe Residen         50 non-null     object
     6   NilaiBelanjaSetahun  50 non-null     int64 
     7   cluster              50 non-null     uint16
     8   segmen               50 non-null     object
    dtypes: int64(2), object(6), uint16(1)
    memory usage: 3.3+ KB
    None
      Customer_ID       Nama Pelanggan Jenis Kelamin  Umur       Profesi  \
    0    CUST-001         Budi Anggara          Pria    58    Wiraswasta   
    1    CUST-002     Shirley Ratuwati        Wanita    14       Pelajar   
    2    CUST-003         Agus Cahyono          Pria    48  Professional   
    3    CUST-004     Antonius Winarta          Pria    53  Professional   
    4    CUST-005  Ibu Sri Wahyuni, IR        Wanita    41    Wiraswasta   
    
      Tipe Residen  NilaiBelanjaSetahun  cluster                 segmen  
    0       Sector              9497927        3      Gold Young Member  
    1      Cluster              2722700        1  Diamond Senior Member  
    2      Cluster              5286429        2          Silver Member  
    3      Cluster              5204498        2          Silver Member  
    4      Cluster             10615206        0   Diamond Young Member  


## Kesimpulan  
Hasil dari segmentasi pelanggan dan pemberian nama yang sesuai untuk masing-masing cluster sebagai berikut :

- Cluster 0: Diamond Young Entrepreneur, isi cluster ini adalah para wiraswasta yang memiliki nilai transaksi rata-rata mendekati 10 juta. Selain itu isi dari cluster ini memiliki umur sekitar 18 - 41 tahun dengan rata-ratanya adalah 29 tahun.
- Cluster 1: Diamond Senior Entrepreneur, isi cluster ini adalah para wiraswata yang memiliki nilai transaksi rata-rata mendekati 10 juta. Isi dari cluster ini memiliki umur sekitar 45 - 64 tahun dengan rata-ratanya adalah 55 tahun.
- Cluster 2: Silver Students, isi cluster ini adalah para pelajar dan mahasiswa dengan rata-rata umur mereka adalah 16 tahun dan nilai belanja setahun mendekati 3 juta.
- Cluster 3: Gold Young Member, isi cluster ini adalah para professional dan ibu rumah tangga yang berusia muda dengan rentang umur sekitar 20 - 40 tahun dan dengan rata-rata 30 tahun dan nilai belanja setahun nya mendekati 6 juta.
- Cluster 4: Gold Senior Member, isi cluster ini adalah para professional dan ibu rumah tangga yang berusia tua dengan rentang umur 46 - 63 tahun dan dengan rata-rata 53 tahun dan nilai belanja setahun nya mendekati 6 juta.

## **Tips**

Pada aplikasi di industri proses penentuan cluster yang optimum bisa dilakukan juga dengan melihat matriks evaluasi lainnya seperti sillhoute score dan callinski-harabaz score. Untuk detailnya bisa dilihat di dokumentasi sklearn (https://scikit-learn.org/stable/modules/clustering.html#clustering-performance-evaluation).

Selain itu penentuan jumlah cluster yang optimal juga perlu mepertimbangkan masukan dari tim yang akan menggunakan model nya. Sehingga bisa menghasilkan cluster yang sesuai dengan kebutuhan mereka dan juga bagaimana cara mereka akan memperlakukan segmen-segmen ini.

Tantangan lainnya adalah jumlah data yang jauh lebih banyak di banding dengan data set digunakan untuk latihan ini. Sehingga waktu pemrosesan datanya dan pembuatan model nya akan menjadi lebih lama.

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


