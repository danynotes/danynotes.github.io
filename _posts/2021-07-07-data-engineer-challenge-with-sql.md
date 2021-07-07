---
layout: post
title: Data Engineer Challenge with SQL
categories: [Database, Data Engineer]
tags: [SQL]
description : DQLab 1st Challenge for Data Engineer
date: 2021-07-07 13:48:00 +0700
---


# About this Challenge  
Challenge SQL ini merupakan challenge pertama di dalam chapter projects DQLab Course yang bertujuan untuk menguji tingkat pemahaman SQL dan dibuat oleh [xeratic](https://www.phi-integration.com/).

## Tabel data yang tersedia
|  ms_pelanggan  |         |   |   |   |  ms_produk  |         |
|:--------------:|:-------:|:-:|:-:|---|:-----------:|:-------:|
|      Field     |   Type  |   |   |   |    Field    |   Type  |
| no_urut        | Integer |   |   |   | no_urut     | Integer |
| kode_pelanggan | Varchar |   |   |   | kode_produk | Varchar |
| nama_pelanggan | Varchar |   |   |   | nama_produk | Varchar |
| alamat         | Varchar |   |   |   | harga       | Double  |

|    tr_penjualan   |         |   |   |   | tr_penjualan_detail |         |
|:-----------------:|:-------:|:-:|:-:|---|:-------------------:|:-------:|
| Field             | Type    |   |   |   |        Field        |   Type  |
| kode_transaksi    | Varchar |   |   |   | kode_transaksi      | Varchar |
| kode_pelanggan    | Varchar |   |   |   | kode_produk         | Varchar |
| tanggal_transaksi | Date    |   |   |   | qty                 | Integer |
|                   |         |   |   |   | harga_satuan        | Double  |

### 1. tampilkan daftar produk yang memiliki harga antara 50.000 and 150.000.Nama kolom yang harus ditampilkan: no_urut, kode_produk, nama_produk, dan harga  
```
SELECT no_urut, kode_produk, nama_produk, harga 
FROM ms_produk
WHERE harga >= 50000 and harga <= 150000;
```
### 2. Tampilkan semua produk yang mengandung kata Flashdisk. Nama kolom yang harus ditampilkan: no_urut, kode_produk, nama_produk, dan harga.

```
SELECT no_urut, kode_produk, nama_produk, harga 
FROM ms_produk 
WHERE nama_produk LIKE '%Flashdisk%';
```
### 3. Tampilkan hanya nama-nama pelanggan yang hanya memiliki gelar-gelar berikut: S.H, Ir. dan Drs. Nama kolom yang harus ditampilkan: no_urut, kode_pelanggan, nama_pelanggan, dan alamat.  
```
SELECT no_urut, kode_pelanggan, nama_pelanggan, alamat 
FROM ms_pelanggan 
WHERE nama_pelanggan 
LIKE '%S.H.%' OR nama_pelanggan LIKE '%Ir.%' OR nama_pelanggan LIKE '%Drs.%';
```
### 4. Tampilkan nama-nama pelanggan dan urutkan hasilnya berdasarkan kolom nama_pelanggan dari yang terkecil ke yang terbesar (A ke Z). Nama kolom yang harus ditampilkan: nama_pelanggan.  
```
SELECT nama_pelanggan 
FROM ms_pelanggan 
ORDER by nama_pelanggan;
```
### 5. Tampilkan nama-nama pelanggan dan urutkan hasilnya berdasarkan kolom nama_pelanggan dari yang terkecil ke yang terbesar (A ke Z), namun gelar tidak boleh menjadi bagian dari urutan. Contoh: Ir. Agus Nugraha harus berada di atas Heidi Goh. Nama kolom yang harus ditampilkan: nama_pelanggan.
```
SELECT nama_pelanggan 
FROM ms_pelanggan 
ORDER BY SUBSTRING_INDEX(nama_pelanggan, ". ", -1);
```
### 6. Tampilkan nama pelanggan yang memiliki nama paling panjang. Jika ada lebih dari 1 orang yang memiliki panjang nama yang sama, tampilkan semuanya.Nama kolom yang harus ditampilkan: nama_pelanggan
```
SELECT nama_pelanggan 
FROM ms_pelanggan 
WHERE LENGTH(nama_pelanggan) = (
    SELECT max(LENGTH(nama_pelanggan)) FROM ms_pelanggan
    );
```
### 7. Tampilkan nama orang yang memiliki nama paling panjang (pada row atas), dan nama orang paling pendek (pada row setelahnya). Gelar menjadi bagian dari nama. Jika ada lebih dari satu nama yang paling panjang atau paling pendek, harus ditampilkan semuanya. Nama kolom yang harus ditampilkan: nama_pelanggan.
```
SELECT nama_pelanggan 
FROM ms_pelanggan 
WHERE LENGTH(nama_pelanggan) IN (
    (SELECT MAX(LENGTH(nama_pelanggan)) FROM ms_pelanggan),
    (SELECT MIN(LENGTH(nama_pelanggan)) FROM ms_pelanggan)
) 
ORDER BY LENGTH(nama_pelanggan) DESC;
```
### 8. Tampilkan produk yang paling banyak terjual dari segi kuantitas. Jika ada lebih dari 1 produk dengan nilai yang sama, tampilkan semua produk tersebut. Nama kolom yang harus ditampilkan: kode_produk, nama_produk,total_qty.
```
SELECT
    ms_produk.kode_produk,
    ms_produk.nama_produk,
    SUM(tr_penjualan_detail.qty) AS total_qty
FROM
    ms_produk
    INNER JOIN tr_penjualan_detail ON ms_produk.kode_produk = tr_penjualan_detail.kode_produk
GROUP BY
    ms_produk.kode_produk,
    ms_produk.nama_produk
HAVING
    SUM(tr_penjualan_detail.qty) > 2;
```
### 9. Siapa saja pelanggan yang paling banyak menghabiskan uangnya untuk belanja? Jika ada lebih dari 1 pelanggan dengan nilai yang sama, tampilkan semua pelanggan tersebut. Nama kolom yang harus ditampilkan: kode_pelanggan, nama_pelanggan, total_harga.
```
SELECT
    tr_penjualan.kode_pelanggan,
    ms_pelanggan.nama_pelanggan,
    SUM(
        tr_penjualan_detail.qty * tr_penjualan_detail.harga_satuan
    ) AS total_harga
FROM
    ms_pelanggan
    INNER JOIN tr_penjualan USING (kode_pelanggan)
    INNER JOIN tr_penjualan_detail USING (kode_transaksi)
GROUP BY
    tr_penjualan.kode_pelanggan,
    ms_pelanggan.nama_pelanggan
ORDER BY
    total_harga DESC 
LIMIT 1;
```
### 10. Tampilkan daftar pelanggan yang belum pernah melakukan transaksi.Nama kolom yang harus ditampilkan: kode_pelanggan, nama_pelanggan, alamat.  
```
SELECT kode_pelanggan, nama_pelanggan, alamat
FROM ms_pelanggan
WHERE kode_pelanggan NOT IN (SELECT kode_pelanggan FROM tr_penjualan)
```
### 11. Tampilkan transaksi-transaksi yang memiliki jumlah item produk lebih dari 1 jenis produk. Dengan lain kalimat, tampilkan transaksi-transaksi yang memiliki jumlah baris data pada table tr_penjualan_detail lebih dari satu. Nama kolom yang harus ditampilkan:  kode_transaksi, kode_pelanggan, nama_pelanggan, tanggal_transaksi, jumlah_detail

```
SELECT tr_penjualan.kode_transaksi, tr_penjualan.kode_pelanggan, ms_pelanggan.nama_pelanggan, tr_penjualan.tanggal_transaksi, COUNT(tr_penjualan_detail.kode_produk) as jumlah_detail	
FROM tr_penjualan 
INNER JOIN tr_penjualan_detail on tr_penjualan_detail.kode_transaksi=tr_penjualan.kode_transaksi
INNER JOIN ms_pelanggan on ms_pelanggan.kode_pelanggan=tr_penjualan.kode_pelanggan
GROUP BY tr_penjualan.kode_transaksi,
    tr_penjualan.kode_pelanggan,
    ms_pelanggan.nama_pelanggan,
    tr_penjualan.tanggal_transaksi

HAVING COUNT(tr_penjualan_detail.kode_produk) > 1
```