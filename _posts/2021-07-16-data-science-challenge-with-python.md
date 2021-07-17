---
layout: post
title: Data Science Challenge with python
categories: [Data Science]
tags: [Python, Data Science]
description : DQLab projects - Data Science Challenge with Python
date: 2021-07-16 21:40:00 +0700
---

# Data Science Challenge with Python

Project ini merupakan kerjasama DQLab dengan CTCorp. Didalam project ini bertujuan untuk menguji kemampuan data science yang terdiri dari :  

1. **Test Probabilitas**, dirancang untuk memperlihatkan kemampuan dalam mengkonseptualisasikan pengetahuan dalam bidang probabilitas.  
2. **Test Statistik**, dirancang untuk memperlihatkan pengetahuan member terkait dasar-dasar ilmu statistik  
3. **Test Interpretasi Model**, dirancang untuk memperlihatkan kemampuan member dalam memberikan insight/kesimpulan dan pengambilan keputusan terkait dengan model yang dibangun melalui penerapan data science.  
4. **Test Pemrograman**, yang dirancang untuk memperlihatkan kemampuan member dalam menerapkan logical thinking (berfikir logis) melalui kode-kode program yang ditanyakan.  

## I. Probabilitas
### 1. Terdapat 23 tim rugby yang bermain dalam sebuah turnamen. Berapakah banyaknya pertandingan yang harus dimainkan agar diketahui siapa pemenangnya ?

Penyelesaian :  

Pertandingan turnamen dengan sistem gugur maka jumlah pertandigan yang harus dimainkan adalah n - 1.  
Jadi banyaknya pertandingan yang harus dimainkan adalah : 23 - 1 = **22**

### 2. Setiap interval 15 menit, terdapat 20% probabilitas bahwa Anda dapat melihat satu bintang berekor. Berapakah probabilitasnya Anda dapat melihat setidaknya satu bintang berekor tiap satu jam ?  

Penyelesaian :  

n (number of trials) = 60 menit / 15 menit = 4 ;  
P (probability of success) = 20% = 0.2  
x >= 1 (1,2,3,4)  

Binomial formula : P(X=x) = n! / x!(n - x)! * P<sup>x</sup> * (1 - P)<sup>n-x</sup> 

Comulative Probability : P(X >= 1) = P(X=1) + P(X=2) + P(X=3) + P(X=4) 
                                   = 0.4096 + 0.1536 + 0.0256 + 0.0016
                                   = 0.5904  
Jadi probabilitasnya anda dapat melihat setidaknya satu bintang berekor tiap satu jam adalah **0.5904**  


## II. Statistik  
### 1. Manakah pernyataan yang benar dari pilihan dibawah ini :  
### - Rata-rata adalah ukuran tendensi pusat suatu data  
### - Rata-rata empiris berhubungan dengan permusatan variable acak  
### - Standar deviasi empiris mengukur besarnya sebaran data
### - Semua benar  

Penyelesaian : **Semua Benar**  

### 2. Setelah mengkaji perilaku sebuah populasi, anda telah mengidentifikasi empat jenis individu spesifik yang bernilai pada kajian anda. Anda akan menentukan semua pengguna yang paling serupa untuk setiap jenis individu itu. Algoritma manakah yang paling cocok untuk kajian ini ?  

Penyelesaian : **Algoritma K-Means clustering** karena algoritma ini mempartisi data ke dalam cluster sehingga data yang memiliki karakteristik sama dikelompokan kedalam cluster yang sama.  

### 3. Dengan menambahkan fitur yang tidak penting ke model regresi yang telah dibuat akan menghasilkan kenaikan nilai R-Square atau penurunan nilai R-Square ?

Penyelesaian : Nilai R-Square akan selalu meningkat dengan adanya penambahan variable bebas dalam suatu model, jadi pernyataan yang benar adalah **kenaikan nilai R-square**.  

### Boosting
### 4. Manakah yang termasuk dari algoritma berikut yang merupakan sub-kelas largest boost dalam keluarga algoritma boosting ?

Penyelesaian : **gradient boosting**.  

### 5. Untuk ***k*** cross-validation, semakin besar nilai ***k*** menghasilkan bias yang lebih besar juga. Benar atau salahkah pernyataan di atas?  

Penyelesaian : ***k*** merupakan jumlah iterasi, semakin banyak iterasi dataset yang di evaluasi semakin besar bias yang dihasilkan. 

### Interpretasi Model  
### 6. Jika anda bekerja pada suatu proyek yang menggunakak permasalahan ***binary classification***. Kemudian anda mentraining sebuah model pada training dataset dan kemudian memperoleh confusion matrix setelah menerapkannya pada validation dataset, berikut ini

**N=165**|**Predicted: No**|**Predicted:Yes**
:-----:|:-----:|:-----:
Actual: No|50|10
Actual: Yes|5|100  

Berdasarkan pada confusion matrix tersebut, pilihlah jawaban berikut ini mana yang memberikan prediksi yang benar ?
1. ***Accuracy*** adalah ~0.91  
2. ***Misclassification rate*** adalah ~0.91  
3. ***False-positive rate*** adalah ~0.95  
4. ***True positive rate*** adalah ~0.95  

Penyelesaian :  
Jawaban yang benar adalah **1 dan 4**. penjelasanya sebagai berikut :


```python
TP = 100
TN = 50
FP = 10
FN = 5 

result = {}
metric = ["Accurate", "miss_classification", "false_positive_rate", "true_positive_rate"]
result[metric[0]] = (TP + TN) / (TP + TN + FP + FN)
result[metric[1]] = (FP + FN) / (TP + TN + FP + FN)
result[metric[2]] =  FP  / (FP + TN)
result[metric[3]] =  TP / (TP + FN)
print(f"{metric[0]} is {result[metric[0]] : .2f}")
print(f"{metric[1]} is {result[metric[1]] : .2f}")
print(f"{metric[2]} is {result[metric[2]] : .2f}")
print(f"{metric[3]} is {result[metric[3]] : .2f}")

```

    Accurate is  0.91
    miss_classification is  0.09
    false_positive_rate is  0.17
    true_positive_rate is  0.95


## Dict data type 
### 7. Tuliskanlah kode python di code editor yang akan menyatukan tiga dictionay berikut :  
dic1 = {1:10, 2:20}
dic2 = {3:30, 4:40}
dic3 = {5:50, 6:60}  

menjadi satu dictionay dalam dic4.

Penyelesaian :  


```python
dic1 = {1:10, 2:20}
dic2 = {3:30, 4:40}
dic3 = {5:50, 6:60}
dic4 = {}
for x in (dic1, dic2, dic3):
    dic4.update(x)
print(dic4)
```

    {1: 10, 2: 20, 3: 30, 4: 40, 5: 50, 6: 60}


### 8. Tuliskan sebuah fungsi python yang digunakan untuk menghitung n deret pertama Fibonacci yang dimulai dari 0.  
Inpu : n = 7
Output : 33  

Penyelesaian :  


```python
def calculateSum(n):
    if n <= 0:
        return 0
    fibo = [0] * (n + 1)
    fibo[1] = 1
    # Initialisasi hasil ke dalam variabel sm
    sm = fibo[0] + fibo[1]
    # Tambahkan suku-suku berikutnya
    for i in range(2, n + 1):
        fibo[i] = fibo[i - 1] + fibo[i - 2]
        sm += fibo[i]
    return sm
# Evaluasi hasil deret untuk n = 7    
print(calculateSum(7))
```
