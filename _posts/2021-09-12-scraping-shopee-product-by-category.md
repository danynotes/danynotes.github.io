---
layout: post
title: Scraping produk shopee per kategori
categories: [Scraping]
tags: [Python, Scraping, Scraping Shopee, Scraping Produk per Kategori]
description : Scraping produk by category dari marketplace shopee.co.id dengan python
date: 2021-09-12 22:09:00 +0700
---

Setelah mendapatkan data produk kategori dari *marketplace* shopee, yang telah saya tulis di [Scraping kategori produk shopee](/scraping/2021/08/17/scraping-shopee-category-product). Selanjutnya di notes berikut mengenai bagaimana scraping produk per kategori.

## Mencari API URL produk per kategori

1. Buka shopee.co.id dengan browser, kemudian pilih salah satu kategory. setelah itu tekan ctrl + shift + i untuk membuka tool inspect.
2. Klik tab network, urutkan kolum tipe di tabel hasil pencarian, cari baris yang memiliki tipe *fetch*. Kemudian cari API URL di kolom name yang mengembalikan data produk per kategori. 

![Inspect shopee produk item per kategori](https://lh3.googleusercontent.com/-hPoDB2pt5GY/YT76caclhcI/AAAAAAAAWXw/R6LtKXti3socXWFMBHUB53G_2lMb_BAQQCLcBGAsYHQ/image.png)

Dari gambar diatas sudah didapatkan URL API untuk mendapatkan produk per kategori, guna melihat lebih jelas format JSON datanya, saya menggunakan postman.

![postman produk per kategori](https://lh3.googleusercontent.com/-RJc0xk4V3cI/YT8B5aOi4ZI/AAAAAAAAWX4/QmnCNDsnk7wi8IehoAWwXATRfR8svXNewCLcBGAsYHQ/image.png)

## Memproses API URL untuk mendapatkan produk per kategori

1. *Requirement* dan *environment* yang yang digunakan untuk melakukan scraping produk shopee sama seperti yang digunakan di note sebelumnya [Scraping kategori produk shopee](/scraping/2021/08/17/scraping-shopee-category-product).

2. Buat kelas untuk mendapatkan produk per kategori.

    ```python
    
    from string import Template
    import requests
    from db_connect import Db_Connect
    import json
    from tools import *
    from string import Template
    import click
    import os
    import sys

   def CollectProductByCategory():

        def __init__(self, _catid):
            self._catid = _catid
            self.url = ("https://shopee.co.id/api/v4/search/search_items?by=relevancy&limit=100"
            "&match_id={}&newest=0&order=desc&page_type=search&scenario=PAGE_OTHERS&version=2").format(self._catid)
          

            self.db = Db_Connect(limit_retries=5, reconnect=True)
            self.cursor  = self.db._cursor
            self.SaveToDatabase(data=self.CollectProductCategory())
    ```
    
    Di dalam init ini saya mendefinisikan URL API, database connection dan cursor serta melakukan eksekusi penyimpanan data produk ke dalam database. Kemudian buat fungsi *CollectProductCategory* untuk mengambil data produk per kategori dari website shopee, didalam fungsi ini saya memiliih untuk mendifinisikan field_name yang akan diambil ke dalam parameter list_rec, dan tidak memilih mengambil field_name secara dinamis dari apa yang dihasilkan *requets* untuk menghindari ketidak sesuaian penamaan field_name yang telah dibuat di dalam database.

    
    ```python

        def CollectProductCategory(self):
            UA = ("Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko)"
                "Chrome/92.0.4515.159 Safari/537.36 Edg/92.0.902.73")

            header = {"User-Agent": UA}
            test_request = requests.get(self.url)
            if test_request.status_code == 200:
                resp = requests.get(self.url, headers=header).content.decode("utf-8")
                _source = json.loads(resp)['items']
                list_rec = list()
       
                for data in _source:
               
                    items = data['item_basic']


                    # change null value to 0
                    for key, val in items.items():
                        if val == 'null':
                            items.update({key: 0})

                        if val == "None":
                            items.update({key: None})
                
                        # None for jsonb data type
                        if key == 'label_ids' and val == None:
                            items.update({key: json.dumps(None)})

                        # change categoryid value using _catid's search parameter, because data got from scrap not related to master category
                        if key == 'catid':
                            items.update({key: self._catid})
            
                    # create list record
                    list_rec += (items['itemid'], items['shopid'], TextSanitize(items['name']), \
                        TextSanitize(items['label_ids']), items['image'], TextSanitize(items['images']), \
                        items['currency'], items['stock'], items['status'], \
                        items['ctime'], items['sold'], items['historical_sold'], \
                        items['liked'], items['liked_count'], TextSanitize(items['view_count']), \
                        TextSanitize(items['catid']), TextSanitize(items['brand']), items['cmt_count'], items['flag'], \
                        items['cb_option'], items['item_status'], items['price'], \
                        items['price_min'], items['price_max'], items['price_min_before_discount'], \
                        items['price_max_before_discount'], TextSanitize(items['hidden_price_display']), \
                        items['price_before_discount'], items['has_lowest_price_guarantee'], \
                        items['show_discount'], items['raw_discount'], TextSanitize(items['discount']), \
                        items['is_category_failed'], str(items['size_chart']),\
                        TextSanitize(str(items['item_rating'])), items['item_type'], \
                        items['reference_item_id'], items['transparent_background_image'], 
                        items['is_adult'], items['badge_icon_type'], items['shopee_verified'], \
                        items['is_official_shop'], items['show_official_shop_label'], items['show_shopee_verified_label'], \
                        items['show_official_shop_label_in_title'], items['is_cc_installment_payment_eligible'], \
                        items['is_non_cc_installment_payment_eligible'], TextSanitize(items['coin_earn_label']), items['show_free_shipping'], \
                        TextSanitize(items['preview_info']), TextSanitize(items['coin_info']), TextSanitize(items['exclusive_price_info']), TextSanitize(items['bundle_deal_id']), \
                        items['can_use_bundle_deal'], TextSanitize(items['bundle_deal_info']), TextSanitize(items['is_group_buy_item']), \
                        TextSanitize(items['has_group_buy_stock']), TextSanitize(items['group_buy_info']), items['welcome_package_type'], \
                        TextSanitize(items['welcome_package_info']), TextSanitize(items['add_on_deal_info']), items['can_use_wholesale'], \
                        items['is_preferred_plus_seller'], TextSanitize(items['shop_location']), items['has_model_with_available_shopee_stock'], \
                        TextSanitize(items['voucher_info']), items['can_use_cod'], items['is_on_flash_sale'], TextSanitize(items['spl_installment_tenure']), \
                        TextSanitize(items['is_live_streaming_price']), items['is_mart'], TextSanitize(items['pack_size'])),
          
                return list_rec
            else:
                click.echo('Error server respon {}'.format(test_request.status_code))
                sys.exit(0)
        
    ```

    Setelah itu saya buat fungsi untuk mengirim data hasil baca URL API ke dalam database postgreSQL.


    ```python
        
        def SaveToDatabase(self, data):

            fields = ["itemid", "shopid", "name", "label_ids", "image",  "images", "currency", "stock", "status", "ctime", "sold", "historical_sold", 
                "liked", "liked_count", "view_count", "catid", "brand", "cmt_count", "flag", "cb_option", "item_status", "price", "price_min",
                "price_max", "price_min_before_discount", "price_max_before_discount", "hidden_price_display", "price_before_discount",
                "has_lowest_price_guarantee", "show_discount", "raw_discount", "discount", "is_category_failed", "size_chart",
                "item_rating", "item_type", "reference_item_id", "transparent_background_image",
                "is_adult", "badge_icon_type", "shopee_verified", "is_official_shop", "show_official_shop_label", "show_shopee_verified_label",
                "show_official_shop_label_in_title", "is_cc_installment_payment_eligible", "is_non_cc_installment_payment_eligible", "coin_earn_label",
                "show_free_shipping", "preview_info", "coin_info",  "exclusive_price_info",  "bundle_deal_id",  "can_use_bundle_deal", "bundle_deal_info",
                "is_group_buy_item", "has_group_buy_stock", "group_buy_info", "welcome_package_type", "welcome_package_info", "add_on_deal_info", "can_use_wholesale",
                "is_preferred_plus_seller", "shop_location", "has_model_with_available_shopee_stock", "voucher_info", "can_use_cod", "is_on_flash_sale", "spl_installment_tenure","is_live_streaming_price", "is_mart", "pack_size"]
        
            col = ''
            col_excluded = ''
            i = 0

            for field_name in fields:
                i += 1
            
                if i < len(fields):
                    col += field_name + ", "
                    col_excluded += field_name + "=EXCLUDED." + field_name + ", "
                else:
                    col += field_name
                    col_excluded += field_name + "=EXCLUDED." + field_name 
        
            Template_SQL = ("INSERT INTO item($fields_raw) VALUES $list_values"
                " ON CONFLICT (itemid) DO UPDATE SET $fields_excluded;\n"
            )

            for rec in data:

                strSQL = Template(Template_SQL).substitute(
                    fields_raw=col, 
                    list_values=rec,
                    fields_excluded=col_excluded
                )

            
                self.db.execute(strSQL)
                click.echo('Saving data kode_produk: {0} - id_toko : {1} - nama_barang: {2}'.format(rec[0], rec[1], rec[2]))

            self.db.close()
    ```

3. Buat class untuk mengambil data item produk per kategori

   Di dalam class ini dilakukan looping pengambilan data produk per kategori dari master katergori produk yang didapat dari proses scraping kategori produk shopee di note sebelumnya.

   ```python
   
    class CollectProductItems():

        def __init__(self):
            self.db = Db_Connect(limit_retries=5, reconnect=True)
            self.cursor = self.db._cursor

            self.GetItems()

        def GetItems(self):
            strSQL = ("select distinct(catid) from main_category mc "
                    " union "
                    "select distinct(catid) from sub_category sc ")

            self.cursor.execute(strSQL)

            _result = self.cursor.fetchall()
    
            for raw in _result:
                string_row = clearCatID(raw)
                CollectProductByCategory(string_row)
    
            click.echo('Scrap complete..!')
            sys.exit()
         

    if __name__ == '__main__':
        CollectProductItems()
   ```

Dan berikut capture hasil scraping data produk shopee per kategori.

![Data Scraping produk shopee per kategori](https://lh3.googleusercontent.com/-ggv1gbXYS7c/YT9Wba4kIDI/AAAAAAAAWYA/AI4u8iIBEM0ch77ydx5G6hWpGt7O6PY_ACLcBGAsYHQ/image.png)