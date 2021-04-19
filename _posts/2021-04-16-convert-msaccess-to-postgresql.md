---
layout: post
title: MS Access to PostgreSQL using Python
categories: [Database]
tags: [MS Access, PostgreSQL, Python]
description : Converting MS Access to PostgreSQL using python
date: 2021-04-16 22:45:00 +0700
---


MSAccess can be used to design a database because it has relationship diagram tools that can be used to create tables relations and test them directly. The problem is how if I need to develop an application in PostgreSQL, I can easily find many online converters available on the internet but they are not friendly for the wallet, so I need to make the conversion from MSAccess to PostgreSQL. 

## Preparing MSAccess

1. Grant read access for Admin role to read relationship/foreign key information by executing scripts below in MSAccess Immediate panel.
    ```bash
    ?CurrentUser
    CurrentProject.Connection.Execute "GRANT SELECT ON MSysRelationships TO Admin"
    ```

    To open Immediate panel : Open Ms Access file, click Database tools Menu - Visual Basic - click view - and then Immediate Windows.

    ![Immediate_Panel](https://lh3.googleusercontent.com/-G6lu79Byapk/YHuOkcOWpqI/AAAAAAAAUik/KWcd-zKBkg8wl_lu_3b-u8sR74u-yeaGQCLcBGAsYHQ/s16000/image.png)

## Create code with python

Because we need Microsoft Access Driver, so these scripts only running in Windows.
1. Step 1:Create virtual environment and install this requirements :
    * pyodbc 4.0.30
    * psycopg2 2.8.6


2. Step 2:Import module, specify the parameters and create MSAccess connection, below the parameter needed :

    mdb_file    : MS Access file name include complete file location  
    pg_host     : PostgreSQL Host/IP server  
    pg_db       : PostgreSQL database name  
    pg_user     : PostgreSQL username  
    pg_password : PostgreSQL password  
    print_SQL   : Optional to print on screen SQL dump  



    ```python
    import os
    import pyodbc
    import psycopg2
    import sys

    class mdb2psql:

        def __init__(self, mdb_file, pg_host, pg_db, pg_user, pg_password, print_SQL):
            self.access_cursor = pyodbc.connect(f'Driver={{ Microsoft Access Driver (*.mdb, *.accdb) }};DBQ={mdb_file};').cursor()
        
            self.schema_name = self.get_access_dbname()
            self.print_SQL = print_SQL
            self.pg_user = pg_user

            self.param_dic = {
                "host"      : pg_host,
                "database"  : pg_db,
                "user"      : pg_user,
                "password"  : pg_password
            }
    ```

3. Step 3:Get MsAccess Database Name
    ```python
        def get_access_dbname(self):
            for table in self.access_cursor.tables():
                return os.path.splitext(os.path.basename(table.table_cat))[0]
    ```
4. Step 4:Create PostgreSQL connection

    ```python
            self.pg_conn = self.pg_connect(self.param_dic)
            self.pg_cursor = self.pg_conn.cursor()

        def pg_connect(self, params_dic):
            pg_conn = None
            try:
                print('Connecting to the PostgreSql database...')
                pg_conn = psycopg2.connect(**params_dic)
            except (Exception, psycopg2.DatabaseError) as error:
                print(error)
                sys.exit(1)
            return pg_conn
    ```
5. Step 5:Create schema on PostgreSQL server

    After getting MSAccess schema, then create it on PostgreSQL

    ```python
        def create_schema(self):
            str_SQL = 'DROP SCHEMA IF EXISTS {schema_name} CASCADE; CREATE SCHEMA {schema_name};'.format(schema_name=self.schema_name)
    
            if self.print_SQL:
                print(str_SQL)
        
            self.pg_cursor.execute(str_SQL)
            self.pg_conn.commit()

            self.create_tables()
    ```

6. Step 6:Create tables on PostgreSQL server

    On the last line of the create_schema method, let's call the create_tables method. In the create_tables method, we collect the tables from MS Access into a list, which will be used for looping to get the fields in each table. 


    ```python
        def create_tables(self):
            table_list = list()
            for table in self.access_cursor.tables(tableType='TABLE'):
                table_list.append(table.table_name)

            psql = ''
            str_table_independent = ''
            str_table_dependent = ''
            table_independent = list()
            table_dependent = list()
            table_order = list()
            for table in table_list:
                str_SQL = ''
                str_SQL = 'DROP TABLE IF EXISTS {schema_name}.{table} CASCADE;\n' \
                    .format(schema_name=self.schema_name, table=table)
                str_SQL += 'CREATE TABLE {schema_name}.{table} (\n' \
                    .format(schema_name=self.schema_name, table=table)
                str_SQL += self.create_fields(table)
    ```
    After getting the query string to create fields, then before the create_table query executed in PostgreSQL, the execution query's arrangement to create the table in PostgreSQL need to be made an independent's table first to be executed, then the dependent's table.

    ```python

            # collecting str query to create independent tables first
            if 'FOREIGN KEY' not in str_SQL:
                str_table_independent += str_SQL
                table_independent.append(table)
            else:
                str_table_dependent += str_SQL
                table_dependent.append(table)

            psql = str_table_independent + str_table_dependent 
            if self.print_SQL:
                print(psql)

            self.pg_cursor.execute(psql)
            self.pg_conn.commit()
        
            table_order = table_independent + table_dependent
            self.insert_data(table_order)
    ```
    In the create_fields method, it will collect column name, column size, transform column type and get foreign key relationship/information by executing the hidden MSAccess system table below.
    
    ```bash
    select sZObject, sZreferencedColumn, sZReferencedObject, sZRelationship from MSysRelationships where szObject=?", table
    ```
    
    which will return the following information:   

        * ccolumn  
        * grbit  
        * icolumn  
        * szColumn  
        * szObject # table_name  
        * sZReferencedColumn # field_reference  
        * sZReferencedObject # table_reference  
        * sZRelationship # relationship_name

    With those information it can be use to get table reference and the foreign key name.

    ```python
        def create_fields(self, table):
            postgresql_fields = {
                'COUNTER': 'serial PRIMARY KEY', # autoincrement
                'VARCHAR': 'varchar', # text
                'LONGCHAR': 'varchar', # text
                'BYTE': 'smallint',  # byte
                'INTEGER': 'int',  # integer
                'LONG INTEGER': 'bigint',  # long integer
                'REAL': 'real', # single
                'DOUBLE': 'double precision', # double
                'DATETIME': 'timestamp', # date/time
                'CURRENCY': 'money', # currency 
                'BIT': 'boolean', # yes/no
            }

            foreign_keys= ''
            foreign_keys_exist = {}
            for row in self.access_cursor.execute("select sZObject, sZreferencedColumn, sZReferencedObject, sZRelationship from MSysRelationships where szObject=?", table):
                foreign_key = row[1]

                table_reference = row[2]
                if foreign_keys_exist.get(row[1], None) is None:
                    foreign_keys_exist[row[1]] = True
                    foreign_keys = f'{foreign_keys} FOREIGN KEY ({foreign_key}) REFERENCES {self.schema_name}.{table_reference} ({foreign_key}) ON DELETE CASCADE,\n'

            foreign_keys = foreign_keys[:-2]

            if foreign_keys != '':
                foreign_keys = f'\n, {foreign_keys}'

            str_SQL =''
            field_list = list()
            for column in self.access_cursor.columns(table=table):
                if column.type_name in postgresql_fields:
                    field_list += [column.column_name + " " + postgresql_fields[column.type_name],]
                elif column.type_name == 'DECIMAL':
                    field_list += [column.column_name +
                                ' numeric(' + str(column.column_size) + "," +
                                str(column.decimal_digits) + ")", ]
                else:
                    print("column " + table + "." + column.column_name +
                    " has uncatered for type: " + column.type_name)

            return ','.join(field_list)  + '\n' + foreign_keys + ');\n'
    ```
7. Step 7:Inserting data

    After database schema and tables created now turn to inserting data. 

    ```python
        def insert_data(self, table_list):

            for table in table_list:
                data = self.get_msaccess_data(table)

                if data != []:
                    format_string = '(' + ','.join(['%s', ]*len(data[0])) + ')\n'
                    # pre-bind the arguments before executing - for speed
                    args_string = ','.join(self.pg_cursor.mogrify(format_string, x).decode('utf-8') for x in data)
                    column = ','.join(list(self.get_column(table)))
                    str_SQL = "INSERT INTO %s(%s) VALUES " % (self.schema_name + '.' + table, column) + args_string

                    if self.print_SQL:
                        print('INSERT INTO {schema_name}.{table_name} VALUES {value_list}'.format(schema_name=self.schema_name, table_name=table, value_list=args_string))

                
                    try:
                        self.pg_cursor.execute(str_SQL, data)
                        self.pg_conn.commit()
                    except (Exception, psycopg2.DatabaseError) as error:
                        print("Error: %s" % error)
                        self.pg_conn.rollback()
                        self.pg_cursor.close()
                        return 1
                    print("Execute() done")
            
            self.pg_cursor.close()

        def get_msaccess_data(self, table):
            str_SQL = 'SELECT * FROM [{table_name}]'.format(table_name=table)
            self.access_cursor.execute(str_SQL)
            rows = self.access_cursor.fetchall()

            data = [tuple(x) for x in rows]
          
            return data

        def get_column(self, table):
            columns = list()
            for column in self.access_cursor.columns(table=table):
                columns += column.column_name, 
    
            return columns
    ```
    In the insert_data method, it will call a list of tables that have been arranged sequentially from table independent to table dependent and then and then pull MS Access data, bind it to arguments and return a query string using mogrify().

8. Step 8:Execute the code  

    To execute class mdb2psql I create new class for input parameter and run mdb2psql class.

    ```python
    import mdb2psql
    import click
    import sys
    import os


    @click.command()
    @click.option('--mdbfile', 
        prompt="Full path of MSAccess Source file", help="Full path of MSAccess source file")
    @click.option('--psql_host', 
        prompt="PostgreSQL IP server", help="PostgreSQL ip server")
    @click.option('--psql_db', 
        prompt="PostgreSQL Database Name", help="Database Name")
    @click.option('--psql_user', 
        prompt="PostgreSQL user", help="PostgreSQL user")
    @click.option('--psql_pass', 
        prompt="PostgreSQL user password", help="PostgreSQL password")
    @click.option('--print_query', 
        prompt="View sqldump (Y/n)", help="show sql dump to screen")

    def convert_mdb_to_psql(mdbfile, psql_host, psql_db, psql_user, psql_pass, print_query):
        if not os.path.exists(mdbfile):
            click.echo(f'File not exist: {mdbfile}')
            sys.exit(0)
        click.echo(f'Generate {mdbfile}')
        
        convert_data = mdb2psql.mdb2psql(mdbfile, psql_host, psql_db, psql_user, psql_pass, print_query)
        convert_data.create_schema()

    if __name__ == "__main__":
        convert_mdb_to_psql()
    ```

## Result
This figure show the query results :

![msaccess-to-postgresql-converter](https://1.bp.blogspot.com/-XYkZXzg1eTk/YH0DV_CzEbI/AAAAAAAAUi0/64uKBGT5bTMli267JVBij_QnLVS3bu9pgCLcBGAsYHQ/s1266/msaccess_to_psql_result_dump.png)

and this figure, when it's checked with DBeaver.

![MSAccess to PostgreSQL convertion result](https://1.bp.blogspot.com/-vYuWI2W6JJ4/YHz2Mleg6TI/AAAAAAAAUis/PKuAX9YrIKkAD7IvGgXgYWrmEUm114umgCLcBGAsYHQ/s1916/msaccess_to_psql_result.png)