---
layout: post
title: Scraping shopee product category with python
categories: [Scraping]
tags: [Python, Scraping]
description : This note about scraping shopee.co.id product category with python
date: 2021-08-17 10:06:00 +0700
---

 
Data scraping help the process of collecting data to conduct market research or get sales leads, optimize prices and take information and understanding of consumer needs about competitors' products and services from websites, blogs, forums or marketplaces. 

This notes is about the example of scraping in python to get product category data from https://shopee.co.id and save it in the Postgresql database. 

## Looking for APIs that provide product category data 

1. Open Chrome and go to http://shopee.co.id

   Click cursor on category and then press ctrl+shift+i to inspect page,  as shown in the following image.

   ![Inspect shopee.co.id page](https://lh3.googleusercontent.com/-iG3w93UcCQI/YRvEmsL9F_I/AAAAAAAAWJo/PtgsU6sK-5UTy6NLZ6FRZJvgekFezi4xQCLcBGAsYHQ/image.png)


   Click on the Network menu tab, sort the column type in the table results and look for rows of type *fetch*. Then find the API URL in the Name column that provides the category data in the preview. 

   ![preview category data](https://lh3.googleusercontent.com/-VjA6JSLvnpA/YRvIP6K88VI/AAAAAAAAWJw/8pF_379s1RID21xwXf_QLUYsu1BQaYE3wCLcBGAsYHQ/image.png)

   After finding the required URL, click the Headers tab to view the URL address and copy it for further processing. 

   ![url category data](https://lh3.googleusercontent.com/-Mr2uoMni-aY/YRvMRjWZp-I/AAAAAAAAWJ4/zApqp3gq7ZoVGFCgumY13TLthc9H-lCGACLcBGAsYHQ/image.png)

   I use the postman application to provide an overview of the JSON data format generated from the URL.

   ![postman get category list](https://lh3.googleusercontent.com/-6YnRw3r9A-U/YRvNSvPmW9I/AAAAAAAAWKA/g2_lf2k9gjAr0jLgaKih44xZI8ghDMr3wCLcBGAsYHQ/image.png).

## Collecting data

1. prepare requirement installation

   ```
   - install postgreSQL
   - install psycopg2
   - install requests
   - install python-dotenv
   ```

2. Create an environment config class

   This class will read and return environment (.env) file
   
   ```python
    from dotenv import load_dotenv
    import os
    from typing import get_type_hints, Union

    load_dotenv()

    class AppConfigError(Exception):
        pass

    def _parse_bool(val: Union[str, bool]) -> bool:  # pylint:      disable=E1136 
        return val if type(val) == bool else val.lower() in ['true', 'yes', '1']


    class AppConfig:
        DEBUG: bool = False
        ENV: str = 'development'
        POSTGRES_DB: str
        POSTGRES_USER: str
        POSTGRES_PASSWORD: str
        POSTGRES_HOST: str
        POSTGRES_PORT: int
        LIMIT_RETRIES: int

    def __init__(self, env):
        for field in self.__annotations__:
            if not field.isupper():
                continue

            # Raise AppConfig Error if required field not supplied
            default_value = getattr(self, field, None)
            if default_value is None and env.get(field) is None:
                raise AppConfigError('The {} field is required'.format(field))

            # Cast env var value to expected type and raise AppConfigError on failure
            try:
                var_type = get_type_hints(AppConfig)[field]
                if var_type == bool:
                    value = _parse_bool(env.get(field, default_value))
                else:
                    value = var_type(env.get(field, default_value))

                self.__setattr__(field, value)
            except ValueError:
                raise AppConfigError('Unable to cast value of "{}" to type "{}" for "{}" field'.format(
                    env[field],
                    var_type,
                    field
                )
            )

    def __repr__(self):
        return str(self.__dict__)


    # Expose Config object for app to import
    Config = AppConfig(os.environ)

   ```

3. Create Database connection class

   This class used for database operation

   ```python
    import psycopg2
    import time
    from db_environ import Config


    class Db_Connect():

        def __init__(self, limit_retries, reconnect):
        
            self.param_dict = {
                "host"      : Config.POSTGRES_HOST,
                "port"      : Config.POSTGRES_PORT,
                "database"  : Config.POSTGRES_DB,
                "user"      : Config.POSTGRES_USER,
                "password"  : Config.POSTGRES_PASSWORD
            }
            self._connection = None
            self._cursor = None
            self.reconnect = reconnect
            self.limit_retries = limit_retries
            self.ignite()
    
        def connect(self, retry_counter=0):
            if not self._connection:
                try:
                    self._connection = psycopg2.connect(**self.param_dict)
                    retry_counter = 0
                    self._connection.autocommit = False
                    return self._connection
                except psycopg2.OperationalError as error:
                    if not self.reconnect or retry_counter >= self. limit_retries:
                        raise error
                    else:
                        retry_counter += 1
                        print("Got Error {}. reconnecting {}".format(str(error).strip(), retry_counter))
                        time.sleep(5)
                        self.connect(retry_counter)
                except (Exception, psycopg2.Error) as error:
                raise error

        def pg_cursor(self):
            if not self._cursor or self._cursor.closed:
                if not self._connection:
                    self.connect()
                self._cursor = self._connection.cursor()
                return self._cursor

        def execute(self, str_query, retry_counter=0):
            try:
                self._cursor.execute(str_query)
                self._connection.commit()
                retry_counter = 0

            except (psycopg2.DatabaseError, psycopg2.OperationalError) as error:
                if retry_counter >= self.limit_retries:
                    raise error
                else:
                    retry_counter += 1
                    print("Got Error {}. reconnecting {}".format(str(error).strip(), retry_counter))
                    time.sleep(1)
                    self.reset()
                    self.execute(str_query, retry_counter)
                    self._connection.commit()
        
            except (Exception, psycopg2.Error) as error:
                raise error
            

        def reset(self):
            self.close()
            self.connect()
            self.pg_cursor()

        def close(self):
            if self._connection:
                if self._cursor:
                    self._cursor.close()
                    self._connection.close()
                    print('Database connection is closed')
            self._connection = None
            self._cursor = None 

        def ignite(self):
            self.connect()
            self.pg_cursor()
    
    ```
4. Create class to collect category data

   As we can see, in the general description of the JSON file, there are two types of data categories, namely main categories and subcategories. So I split the data collecting class for main and subcategories.

   ```python
    from os import environ
    from sys import displayhook
    import requests
    from db_connect import Db_Connect
    from string import Template


    class CollectMainCategory():

    def __init__(self, urlapi):
        self.urlapi = urlapi
        self.db = Db_Connect(limit_retries=5, reconnect= True)
        self.cursor = self.db._cursor
        self.SaveToDatabase(self.Collect_Main_Category())

    def Collect_Main_Category(self):
        url = self.urlapi
        resp = requests.get(url).json()
        category_source = resp["data"]
        list_values = list()
        for item in category_source:
            main_category = item.get("main")

            list_values += (main_category['display_name'], \
                    main_category['name'], \
                    main_category['catid'], \
                    main_category['parent_category'], \
                    str(main_category['is_adult']), \
                    str(main_category['block_buyer_platform']), \
                    main_category['sort_weight']),

        return list_values

    def SaveToDatabase(self, data):
        Template_SQL = ("INSERT INTO main_category(display_name,name,catid,parent_category,is_adult,block_buyer_platform, sort_weight) "
                            "VALUES $list_value "
                            "ON CONFLICT (catid) "
                            "DO UPDATE SET display_name=EXCLUDED.display_name, "
                            "name=EXCLUDED.name, parent_category=EXCLUDED.parent_category, "
                            "is_adult=EXCLUDED.is_adult, block_buyer_platform=EXCLUDED.block_buyer_platform, "
                            "sort_weight=EXCLUDED.sort_weight;"
            )
        
        for rec in data:
            strSQL = Template(Template_SQL).substitute(
                list_value=rec
            )
        
            self.db.execute(strSQL)
            
    if __name__ == "__main__":
        CollectMainCategory("https://shopee.co.id/api/v2/category_list/get_all")

    ```

    ```python
    from os import environ
    from sys import displayhook
    import requests
    from db_connect import Db_Connect
    from string import Template
    import re


    class CollectSubCategory():

    def __init__(self, urlapi):
        self.urlapi = urlapi
        self.db = Db_Connect(limit_retries=5, reconnect= True)
        self.cursor = self.db._cursor
        self.SaveToDatabase(self.Collect_Sub_Category())
        #self.Collect_Sub_Category()

    def Collect_Sub_Category(self):
        url = self.urlapi
        resp = requests.get(url).json()
        _source = resp["data"]

        list_rec = list()
        for dt in _source:
            sub_category = dt.get('sub')
        
            for rec in sub_category:
                sub = str(rec['sub_sub'])
                sub_formated = sub.replace("\'","\"")
                sub_formated = sub_formated.replace("None","\"None\"")

                list_rec += (self.TextSanitize(str(rec['display_name'])), \
                    self.TextSanitize(str(rec['name'])), \
                    rec['catid'], \
                    rec['parent_category'], \
                    str(rec['is_adult']), \
                    str(rec['block_buyer_platform']), \
                    rec['sort_weight'],
                    sub_formated),

        return list_rec

    def TextSanitize(self, str):
        """Sanitizes a string so that it can be properly compiled in TeX.
        Escapes the most common TeX special characters: ~^_#%${}
        Removes backslashes.
        """
        s = re.sub('\\\\', '', str)
        s = re.sub(r'([_^$%&#{}])', r'\\\1', str)
        s = re.sub(r'\'', r'\\~{}', str)
        return s

    def SaveToDatabase(self, data):
        Template_SQL = ("INSERT INTO sub_category(display_name,name,catid,parent_category,is_adult,block_buyer_platform, sort_weight, sub_sub) "
                            "VALUES $list_recs "
                            "ON CONFLICT (catid) "
                            "DO UPDATE SET display_name=EXCLUDED.display_name, "
                            "name=EXCLUDED.name, parent_category=EXCLUDED.parent_category, "
                            "is_adult=EXCLUDED.is_adult, block_buyer_platform=EXCLUDED.block_buyer_platform, "
                            "sort_weight=EXCLUDED.sort_weight, "
                            "sub_sub=EXCLUDED.sub_sub;"
           )
        
        for rec in data:
            strSQL = Template(Template_SQL).substitute(
               list_recs=rec
            )
            
            self.db.execute(strSQL)
          

    if __name__ == "__main__":
        CollectSubCategory("https://shopee.co.id/api/v2/category_list/get_all")

    ```

5. Scraping Result
   
   Here is the result of scraping product category from shopee.co.id. These results can be used for further processing later

   ![scraping result](https://lh3.googleusercontent.com/-tVqOF09xvmM/YRy-4RwnGjI/AAAAAAAAWKM/k6diGo_zUAA1-Erue1so5iqbFRFdmLfcwCLcBGAsYHQ/image.png)