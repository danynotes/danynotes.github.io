---
layout: post
title: Data Science Customer Segmentation with python Part-4
categories: [Data Science]
tags: [Python, Data Science]
description : DQLab projects - Data Science Customer Segmentation with python Part 4.
date: 2021-07-19 14:03:00 +0700
---

Setelah data dirapikan pada [part-3](/data%20science/2021/07/19/customer-segmentation-with-python-part3), selanjutnya pada tahap ini dibuat per-modelan.

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

Langkat berikutnya yaitu [mengoperasikan model yang telah dibuat](/data%20science/2021/07/19/customer-segmentation-with-python-part5).