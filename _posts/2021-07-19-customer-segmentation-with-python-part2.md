---
layout: post
title: Data Science Customer Segmentation with python Part 2
categories: [Data Science]
tags: [Python, Data Science]
description : DQLab projects - Data Science Customer Segmentation with python Part 2.
date: 2021-07-19 14:03:00 +0700
---

Setelah melakukan persiapan library dan membaca data pada [part 1](/data%20science/2021/07/19/customer-segmentation-with-python) selanjutnya kita akan lakukan explorasi data. 

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

Lanjut ke [persiapan permodelan](/data%20science/2021/07/19/customer-segmentation-with-python-part3)