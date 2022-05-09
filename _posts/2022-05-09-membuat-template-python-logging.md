---
layout: post
title: Membuat Template Python Logging
categories: [Python]
tags: [Python, Logging]
description : Membuat Template Python Logging
date: 2022-05-09 11:17:00 +0700
---

# Membuat Template Python Logging untuk bisa digunakan di semua module 

Cara sempurna untuk melakukan ***debug*** dan mengikuti eksekusi aplikasi adalah dengan membuat log yang terdefinisi, informatif dan terstruktur.

Di dalam note ini saya membuat logging python menggunakan logging dari python library untuk dapat digunakan di seluruh module aplikasi.

## Konfigurasi Logging 

Logging yang aku buat ini menggunakan fungsi logging.config.dictConfig(***config***) yang akan mengambil konfigurasi logging dari dictionary. Detail Schema dictionary yang akan dipanggil oleh ***dictConfig()*** harus memiliki keys berikut :

1. ***version***, set dengan nilai integer yang mewakili versi dari skema config
2. ***formatters***, Konfigurasi format nilai object yang akan di tampilkan.
3. ***filters***, filter key *name* yang value-nya akan dijadikan dictionary
4. ***handlers***, Handler bertanggung jawab untuk mengirimkan pesan log sesuai dengan level log ke tujuan yang ditentukan oleh handler. Contoh : sebuah skema memungkinkan mengirim semua pesan log ke dalam file log, dan mengirimkan pesan penting ke alamat email.
5. ***loggers***, Object logger memiliki 3 fungsi. *pertama* mengekspos beberapa metode ke kode aplikasi sehingga aplikasi dapat mencatat log saat runtime, *kedua* menentukan pesan log mana yang harus ditindaklanjuti berdasarkan level atau object filter. *ketiga* logger meneruskan pesan log yang relevan ke handler yang digunakan.
6. ***root***, untuk mengkonfigurasi root logger
7. ***incremental***, Menandakan apakah konfigurasi yang ditentukan sebagai tambahan dari konfigurasi yang telah ada. Nilai defaultnya adalah false, artinya bahwa konfigurasi yang ditentukan menggantikan konfigurasi yang ada sebelumnya.
8. ***disable_existing_loggers***, menentukan apakah logger non-root yang ada akan dinonaktifkan. Default nilai parameter ini adalah True, dan nilai parameter ini diabaikan jika parameter incremetal bernilai True.

## Buat module logger

Buat file baru dengan nama applogger.py yang akan berfungsi sebagai root logger.

```python
import logging
from logging import config, getLogger


# buat konfigurasi logging dalam dictionary
dictLogging = {
    'version': 1,
    'disable_existing_loggers': False,
    'formatters': {
        'default_formatter': {
            'format': '[%(levelname)s:%(asctime)s] %(funcName)s - %(message)s'
        },
    },
    'handlers': {
        'stream_handler': {
            'class': 'logging.StreamHandler',
            'formatter': 'default_formatter',
        },
        'rotating_file_handler': {
            'class': 'logging.handlers.RotatingFileHandler',
            'formatter': 'default_formatter',
            'filename': 'App.log',
            'mode': 'a',
            'maxBytes': 2*1024*1024,
            'backupCount': 4
        },
    },
    'loggers': {
        'debug_log': {
            'handlers': ['stream_handler', 'rotating_file_handler'],
            'level': 'DEBUG',
            'propagate': False
        },
        'info_log': {
            'handlers': ['rotating_file_handler'],
            'level': 'INFO',
            'propagate': False
        },
        'error_log': {
            'handlers': ['stream_handler', 'rotating_file_handler'],
            'level': 'ERROR',
            'propagate': False
        },
        'warning_log': {
            'handlers': ['rotating_file_handler'],
            'level': 'WARNING',
            'propagate': False
        },
        'critical_log': {
            'handlers': ['rotating_file_handler'],
            'level': 'CRITICAL',
            'propagate': False
        }
    }
}

# Buat fungsi untuk load dictConfig dan memanggil Logger
def AppLoger(logger_name):
    config.dictConfig(dictLogging)

    Loger = getLogger(logger_name)

    return Loger

```

Untuk menggunakan Logger tersebut dalam module lain:  import applogger kemudian panggil logger name yang akan digunakan dan pesan yang akan ditampilkan

```python
import applogger


def run():
    # Call logger yang akan digunakan 
    logger = applogger.AppLoger('info_log')

    # Define message
    logger.info('Aplikasi Mulai Dijalankan')


    """
    Kode Aplikasimu..
    
    """

    logger.info('Aplikasi Selesai Dijalankan')

```

Kesimpulan :

Dengan module root logger ini memudahkan dalam mengintegrasikan pengaturan logging yang akan digunakan didalam proyek aplikasi yang dibuat.
Module ini dibuat sederhana dan dapat dikembangkan lagi.