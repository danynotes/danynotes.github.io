---
layout: post
title: Data Analyst for Retail Sales Performance Report
categories: [Data Analyst]
tags: [SQL, Data Analyst]
description : DQLab projects - Data Analyst for Retail Sales Performance Report
date: 2021-07-10 14:53:00 +0700
---

# Data Analyst Project - Sales Performance Report  

Project ini salah satu chapter didalam project DQLab yang di buat oleh Nelda Ampulembang Parenta - Senior Data Analyst (Logisly). project ini bertujuan untuk melakukan analisis terhadap performance dari DQLab Store dengan menggunakan MySQL.

## Dataset Brief  

Dataset yang digunakan berisi transaksi dari tahun 2009 - 2012 dengan jumlah raw 5500, termasuk didalamnya order status yang terbagi menjadi **order finished**, **order returned** dan **order cancelled**

Dataset terdiri dari :
1. OrderID  
2. Order Status  
3. Customer  
4. Order Date  
5. Order Quantity  
6. Sales  
7. Discount %  
8. Discount  
9. Product Category  
10. Product Sub-Category  

Dari pihak manajemen DQLab store ingin mengetahui:  

1A. Overall perofrmance DQLab Store dari tahun 2009 - 2012 untuk jumlah order dan total sales order finished  
1B. Overall performance DQLab by subcategory product yang akan dibandingkan antara tahun 2011 dan tahun 2012  

2A. Efektifitas dan efisiensi promosi yang dilakukan selama ini, dengan menghitung burn rate dari promosi yang dilakukan overall berdasarkan tahun  
2B. Efektifitas dan efisiensi promosi yang dilakukan selama ini, dengan menghitung burn rate dari promosi yang dilakukan overall berdasarkan sub-category  

Setelah melihat hasil analisa di Sub Bab 1 dan 2, selanjutnya dilakukan analisa terhadap customer DQLab. Analisa dari sisi customer dengan menggunakan metrics:

3A. Analisa terhadap customer setiap tahunnya  
3B. Analisa terhadap jumlah customer baru setiap tahunnya  
3C. Cohort untuk mengetahui angka retention customer tahun 2009  


### 1A. Overall Performance by Year
### Buatlah query dengan menggunakan SQL untuk mendapatkan total penjualan (sales) dan jumlah order (number_of_order) dari tahun 2009 - 2012 (years)  

```
SELECT
    YEAR(order_date) AS years,
    SUM(sales) AS sales,
    COUNT(order_id) AS number_of_order
FROM
    dqlab_sales_store
WHERE
    order_status = 'Order Finished'
GROUP BY
    years;
```
### 1B. Overall Performance by product sub Category
### Buatlah query dengan menggunakan SQL untuk mendapatkan total penjualan (sales) berdasarkan sub category dari produk (product_sub_category) pada tahun 2011 dan 2012 saja (years)  

```
SELECT YEAR(order_date) AS years, 
	product_sub_category, 
	SUM(sales) AS sales
FROM dqlab_sales_store
WHERE 
	order_status = "order finished" 
	AND YEAR(order_date) IN("2011","2012")
GROUP BY years, product_sub_category
ORDER BY years,SUM(sales) DESC

```

### 2A. Promotion Effectiveness and Efficiency by Years
### Efektifitas dan efisiensi dari promosi yang dilakukan akan dianalisa berdasarkan *Burn Rate* yaitu dengan mebandingkan total value promosi yang dikeluarkan terhadap total sales yang diperoleh. DQLab berharap *Burn Rate* berada diangka maksimum 4.5%.

***Burn Rate Formula : (total discount / total sales) * 100***

Buatkan *Derived Tables* untuk menghitung total sales(sales) dan total discount (promotion_value) berdasarkan tahun(years) dan formulasikan persentase burn rate nya (burn_rate_percentage).

```
SELECT YEAR(order_date) as years, 
    SUM(sales) as sales,
    SUM(discount_value) as promotion_value,
    ROUND(SUM(discount_value) / SUM(sales) * 100, 2) as burn_rate_percentage
FROM dqlab_sales_store
WHERE
	order_status LIKE 'Order Finished'
GROUP BY
    years
ORDER BY
    years;

```

### 2B. Promotion Effectiveness and Efficiency by Product Sub Category
### Analisa terhadap efektifitas dan efisiensi dari promosi yang sudah dilakukan selama ini. Lakukan query yang sama seperti 2A dengan menambahkan kolom product_sub_category dan product_category

```
SELECT YEAR(order_date) as years, 
    product_sub_category,
    product_category,
    SUM(sales) as sales,
    SUM(discount_value) as promotion_value,
    ROUND(SUM(discount_value) / SUM(sales) * 100, 2) as burn_rate_percentage
FROM dqlab_sales_store
WHERE
	order_status LIKE 'Order Finished' 
    AND YEAR(order_date) = '2012'
GROUP BY
    years,
    product_sub_category,
    product_category
ORDER BY
    sales DESC;
```

### 3A. Customers Transactions per Year
### DQLab Store ini mengetahui jumlah customer (number_of_customer) yang bertransaksi setiap tahun dari 2009 sampai 2012 (years).


```
SELECT YEAR(order_date) as years,
	COUNT(DISTINCT(customer)) as number_of_customer
FROM dqlab_sales_store
WHERE YEAR(order_date) BETWEEN '2009' AND '2012'
	AND order_status LIKE 'Order Finished'
GROUP BY years	
ORDER BY years;

```