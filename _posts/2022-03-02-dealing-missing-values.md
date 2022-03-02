---
layout: post
title: Menangani nilai yang hilang
categories: [Data Science]
tags: [Python, Data Science, Missing Values]
description : Dealing Missing Values
date: 2022-03-02 20:18:00 +0700
---

Beberapa hal yang menyebabkan data memiliki nilai yang hilang, diantaranya karena responden tidak memberikan informasi mengenai beberapa detail. 
***Machine learning libraries*** akan menghasilkan ***error*** jika kita membuat model dengan dataset yang memiliki nilai yang hilang. Jadi, data yang hilang dalam dataset harus di selesaikan sebelum membuat model. 

## Ada 2 pendekatan untuk menyelesaikan nilai yang hilang

### 1) Cara sederhana : ***Drop*** kolom yang memiliki nilai hilang.

Dengan pendekatan ini, model berpotensi kehilangan banyak informasi yang mungkin bermanfaat. Misalkan terdapat dataset dengan 10.000 baris, dimana terdapat satu kolom penting yang beberapa baris tidak memiliki isi, dengan pendekatan ini satu kolom tersebut akan di ***drop*** sepenuhnya.

contoh kode sebagai berikut :

```python
import pandas as pd
from sklearn.model_selection import train_test_split
from sklearn.ensemble import RandomForestRegressor
from sklearn.metrics import mean_absolute_error

# Baca dataset
data = pd.read_csv('melb_data.csv')

# Tentukan target
y = data.Price

# Untuk mempermudah kita hanya gunakan predictors yang bernilai numerik

melb_predictors = data.drop(['Price'], axis = 1)
X = melb_predictors.select_dtypes(exclude=['object'])

# Bagi data menjadi training dan validation subsets
X_train, X_valid, y_train, y_valid = train_test_split(X, y, train_size=0.8, test_size=0.2, random_state=0)


# Dapatkan kolom yang memiliki nilai yang hilang
cols_with_missing = [col for col in X_train.columns if X_train[col].isnull().any()]

# Drop kolom yang memiliki nilai hilang di training dan validation
X_train_droped = X_train.drop(cols_with_missing, axis=1)
x_valid_droped = X_valid.drop(cols_with_missing, axis=1)

# Buat fungsi untuk mengukur kualitas metode penanganan nilai yang hilang
def score_dataset(X_train, X_valid, y_train, y_valid):
    model = RandomForestRegressor(n_estimators=10, random_state=0)
    model.fit(X_train, y_train)

    preds = model.predict(X_valid)
    return mean_absolute_error(y_valid, preds)

print("MAE from Approach 1 (Drop columns with missing values):")
print(score_dataset(reduced_X_train, reduced_X_valid, y_train, y_valid))

```
Hasil pendekatan pertama sebagai berikut :

MAE from Approach 1 (Drop columns with missing values): 

183550.22137772635

### 2) Pilihan yang lebih baik : Imputasi  


Imputasi mengisi nilai-nilai yang hilang dengan beberapa nilai, misalnya dengan mengisi nilai rata-rata di setiap kolom.

Dalam beberapa kasus hasil imputasi tidak akan benar-benar tepat, tetapi biasanya mengarah ke model yang lebih akurat daripada melakukan drop pada kolom sepenuhnya.

Contoh kode sebagai berikut, masih menggunakan dataset yang sama dengan pendekatan pertama.

```python
from sklearn.impute import SimpleImputer


# Imputasi
myImputer = SimpleImputer()

imputed_X_train = pd.DataFrame(myImputer.fit_transform(X_train))
imputed_X_valid = pd.DataFrame(myImputer.transform(X_valid))

# Imputasi menyebabkan nama kolom hilang. kembalikan nama kolom :
imputed_X_train.columns = X_train.columns
imputed_X_valid.columns = X_valid.columns

# check score MAE
print("MAE from Approach 2 (Imputation):")
print(score_dataset(imputed_X_train, imputed_X_valid, y_train, y_valid))

```
Hasil pendekatan imputasi sebagai berikut : 

MAE from Approach 2 (Imputation): 

178166.46269899711

Kita lihat jumlah baris dan jumlah kolom yang memiliki nilai hilang.

```python
print(X_train.shape)

# Number of missing values in each column of training data
missing_val_count_by_column = (X_train.isnull().sum())
print(missing_val_count_by_column[missing_val_count_by_column > 0])


```
Hasil nya :
(10864, 12) 

Car               49

BuildingArea    5156

YearBuilt       4307

dtype: int64

Data training memiliki 10864 baris dan 12 kolom. Dimana ada 3 kolom yang memiliki nilai yang hilang. Hampir separuh baris dari masing-masing kolom yang nilai nya hilang, ini lah mengapa ketika melakukan pendekatan 1) untuk menangani nilai yang hilang akan menyebabkan potensi kehilangan informasi yang bermanfaat.

```Python
# Preprocess test data
final_X_test = pd.DataFrame(myImputer.transform(X_test))

preds_test = model.predict(final_X_test)

output = pd.DataFrame({'Id': X_test.index, 'SalePrice': preds_test})
print(output)

```

  Id  SalePrice

0     1461  125245.50

1     1462  155237.00

2     1463  180755.22

3     1464  184071.50

4     1465  197144.40

...    ...        ...

1454  2915   87277.12

1455  2916   87025.50

1456  2917  154283.87

1457  2918  107723.50

1458  2919  228591.59