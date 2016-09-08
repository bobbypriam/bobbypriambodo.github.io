---
layout: post
title: "The engineering behind SusunJadwal"
date: 2016-09-08 04:47:00
summary: Sejak beberapa tahun silam, SIAK war di Universitas Indonesia selalu didahului oleh peluncuran aplikasi SusunJadwal. Bagaimana aplikasi tersebut bekerja?
---

Universitas Indonesia (UI) memiliki sebuah tradisi pada setiap awal semester yang bernama SIAK war.

> SIAK war (n.) - Masa pengisian Isian Rencana Studi (IRS) secara online. Bayangkan KRL Commuter Line arah Bogor jam pulang kerja, tapi online.

Isian Rencana Studi (IRS) adalah sebuah formulir yang berisi daftar kelas mata kuliah yang akan diambil oleh seorang mahasiswa pada suatu semester. Di beberapa kampus lain, IRS juga dikenal sebagai KRS. Pengisian IRS di UI dilakukan secara online melalui sebuah sistem bernama SIAK-NG (Sistem Informasi AKademik - *Next Generation*).

 Di UI, mahasiswa dibebaskan untuk memilih kelas yang ingin diambil pada suatu semester (kecuali pada beberapa Fakultas seperti Fakultas Kedokteran yang sudah memiliki paket tetap tiap semester).  Karena mahasiswa bebas memilih, terjadilah perebutan kelas-kelas favorit. Fenomena inilah yang disebut SIAK war.

Ada beberapa faktor yang memengaruhi pemilihan ini, seperti misalnya kemungkinan jadwal kuliah yang bentrok <del>dan bagaimana mata kuliah dengan dosen yang \*uhuk\* baik \*uhuk\* memiliki banyak peminat dan memicu perselisihan batin</del>. Dengan waktu mahasiswa yang terbatas, jadwal kuliah harus diatur sedemikian rupa sehingga tidak bentrok dan masih menyisakan waktu untuk kegiatan lain, seperti misalnya berorganisasi atau main layangan.

Demi membantu penyusunan jadwal kuliah, ada sebuah menu pada SIAK-NG bernama "Jadwal Kuliah" yang memuat daftar-daftar kelas yang dibuka pada semester tersebut. Daftar akan terus bertambah seiring mendekatnya waktu pengisian IRS, sehingga mahasiswa perlu memeriksa halaman tersebut berkali-kali untuk melihat jadwal kelas baru. Berikut adalah gambar contoh daftar kelas yang ada pada halaman tersebut.

/gambar/

Ya, daftar disajikan dalam bentuk... daftar. Alhasil, sulit bagi mahasiswa untuk mendapatkan informasi apakah mata kuliah yang dipilih memiliki jadwal bentrok atau tidak.

Sebagai mahasiswa Fasilkom yang haus kontribusi, kawan-kawan dari [Pandago Studio](https://twitter.com/pandagostudio) yang lalu dilanjutkan oleh [Ristek](http://ristek.cs.ui.ac.id/) mengembangkan sebuah aplikasi bernama **SusunJadwal**.

Pada artikel ini, kita akan mencoba menelusur lebih dalam: bagaimana aplikasi SusunJadwal bekerja?

## About SusunJadwal

*Disclaimer: Segala bahasan dari artikel ini mengacu pada versi SusunJadwal yang melibatkan saya langsung dalam pengembangannya, yaitu periode Ganjil dan Genap 2015/2016 oleh Ristek CSUI. Pengembangan sebelum dan sesudahnya mungkin memiliki detail yang berbeda.*

SusunJadwal adalah sebuah aplikasi web yang dapat membantu mahasiswa UI menyusun rencana jadwal kuliah pra-pengisian IRS. Idenya cukup sederhana: berdasarkan informasi kelas pada halaman "Jadwal Kuliah", pengguna dapat secara interaktif memilih kelas-kelas yang ingin diambil, dan sistem akan secara otomatis memberikan notifikasi apabila ada jadwal kelas yang bentrok. Apabila jadwal sudah bersih dari bentrok, pengguna juga dapat meng-*generate* representasi visual jadwal agar rancangan jadwal dapat lebih mudah dimengerti. Sebagai gambaran, [SusunJadwal versi Ganjil 2016/2017 dapat diakses pada tautan ini](http://ristek.cs.ui.ac.id/susunjadwal/).

Selain dua fitur utama tersebut, terdapat beberapa fitur tambahan, yaitu:

- **Menyimpan jadwal**
  Disusun, lalu hilang? Buat apa? SusunJadwal juga menyediakan fitur menyimpan jadwal yang telah dibuat.
- **Melihat jadwal per jurusan**
  Pada versi Ganjil 2015/2016, SusunJadwal mengenalkan fitur penyusunan jadwal untuk fakultas lain (sebelumnya hanya untuk jurusan Ilmu Komputer dan Sistem Informasi saja).
- **Menambah agenda**
  Agar ekstensibel, SusunJadwal memiliki fitur menambahkan agenda selain kuliah (misalnya rapat rutin, les, dan sebagainya) pada jadwal yang disusun.
- **Menggabungkan jadwal**
  Dengan digabungkan dengan fitur menambah agenda, fitur menggabungkan jadwal dapat menjadi solusi bagi anggota kepanitiaan atau organisasi yang ingin mencari waktu kosong, misalnya untuk untuk jadwal rapat.

Dengan dikunjungi sebanyak **12.000+ kali** pada periode Ganjil 2015/2016 (1-4 Agustus 2015) dan  **13.000+ kali** pada periode Genap 2015/2016 (16-19 Januari 2016), terlihat bahwa SusunJadwal mampu memberikan *impact* yang cukup signifikan dalam persiapan SIAK war mahasiswa UI. Yeay.

*Well, enough of that. Let's get technical!*

## Architecture Design

// TODO

## Components: In Depth

// TODO

### Scraper

// TODO

### API server

// TODO

### Client app

// TODO

## What's next?

// TODO

<!-- Outline:

- background
- overview
- features
  - melihat jadwal per jurusan
  - menyusun jadwal pilihan
  - menyimpan jadwal pilihan
  - menggabungkan beberapa jadwal
- components
  - general overview
  - scraper
    - what is it for?
    - technologies used
      - request
      - async
      - cheerio
    - how it works
      - authentication
        - authenticate
        - changeRole
      - jadwal scraping and parsing
        - cheerio
        - saving to JSON
  - API server
    - what is it for?
    - technologies used
      - express
      - mongoDB
      - mongoose
    - how it works
      - reading jadwal from JSON file
      - saving jadwal
  - web client
    - what is it for?
    - technologies used
      - angular
      - sass
    - how it works
      - fetching from API
      - conflict checking
      - schedule drawing
      - joining jadwal
- what's next? -->
