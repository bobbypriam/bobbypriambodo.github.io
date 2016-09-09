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

 (Ditambah kebiasaan server SIAK-NG yang *down* ketika mengalami *overload*, yang akan kita bahas nanti di artikel ini.)

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
  Sejak versi Ganjil 2015/2016, SusunJadwal mengenalkan fitur penyusunan jadwal untuk fakultas lain (sebelumnya hanya untuk jurusan Ilmu Komputer dan Sistem Informasi saja) dengan memanfaatkan menu "Jadwal Kuliah Keseluruhan" pada SIAK-NG.
- **Menambah agenda**
  Agar ekstensibel, SusunJadwal memiliki fitur menambahkan agenda selain kuliah (misalnya rapat rutin, les, dan sebagainya) pada jadwal yang disusun.
- **Menggabungkan jadwal**
  Dengan digabungkan dengan fitur menambah agenda, fitur menggabungkan jadwal dapat menjadi solusi bagi anggota kepanitiaan atau organisasi yang ingin mencari waktu kosong, misalnya untuk untuk jadwal rapat.

Dengan dikunjungi sebanyak **12.000+ kali** pada periode Ganjil 2015/2016 (1-4 Agustus 2015) dan  **13.000+ kali** pada periode Genap 2015/2016 (16-19 Januari 2016), terlihat bahwa SusunJadwal mampu memberikan *impact* yang cukup signifikan dalam persiapan SIAK war mahasiswa UI. Yeay.

*Well, enough of that. Let's get technical!*

## Architecture Design

Sebelum masuk ke penetapan desain arsitektur, kita akan membahas mengenai kebutuhan dan keterbatasan yang ada dalam merealisasikan aplikasi ini.

Seperti yang telah disebut sebelumnya, kita hanya memiliki satu sumber informasi, yaitu halaman "Jadwal Kuliah" pada SIAK-NG. Sayangnya, sistem SIAK-NG tidak menyediakan sebuah [API](http://www.webopedia.com/TERM/A/API.html) yang memungkinkan kita secara mudah dapat mengonsumsi data darinya. Karena itu, kita akan menggunakan salah satu metode kuno untuk *data collection* pada web, yaitu [*scraping*](https://en.wikipedia.org/wiki/Web_scraping).

Sayangnya, seperti yang sempat saya tulis di awal, server SIAK-NG memiliki kelemahan apabila diakses oleh ribuan orang bersama-sama pada suatu waktu. Let's talk about this for a bit.

### On SIAK-NG's Performance and Scalability

Tahun lalu, saya memperoleh informasi bahwa ketidakmampuan server SIAK-NG menampung ribuan *request* pada saat SIAK war adalah (sedikit banyak) *by design*.

"Apa!?" kita mungkin berseru. Tapi itu masuk akal.

Coba tanyakan kepada diri kita sendiri: dalam setahun, seberapa sering kita mengakses SIAK-NG? Sekali, ketika mengisi IRS. Beberapa kali selanjutnya masih di minggu yang sama untuk melihat apakah jadwal bisa berubah atau sudah disetujui oleh Pembimbing Akademik. Mungkin beberapa kali lagi untuk melihat Kalender Akademik untuk tanggal-tanggal penting semester tersebut. Selebihnya?

Berdasarkan sumber yang dapat dipercaya, tingkat *availability* sistem SIAK-NG sendiri dalam setahun mendekati 99%, di mana 1% *downtime* terjadi pada masa SIAK war. Masih menurut sumber tersebut, meningkatkan kapabilitas server untuk memenuhi hanya 1% waktu dalam satu tahun tidak *feasible* dari segi finansial. *The argument has a point*.

*But still*, ini tidak membenarkan; untuk apa sistem hidup ketika tidak dibutuhkan dan mati ketika dibutuhkan?

Tentu saja, ini tidak berarti tim IT UI tidak mengusahakan performa yang lebih baik untuk SIAK-NG pada masa pengisian IRS. Sumber yang sama mengatakan server SIAK-NG sendiri sebenernya sudah di-*load balance* (dapat dilihat jika kita login, di atas nama kita di kiri atas ada indikator Node mana yang melayani *request* kita). Sayangnya, masih terdapat *bottleneck* pada I/O dari *database*. *Database* server hanya berjumlah satu, yang menyebabkan sistem *down* ketika *overload*.

Setahu saya, ada wacana untuk terus menerapkan konsep sistem terdistribusi pada arsitektur SIAK-NG untuk meningkatkan responsivitasnya. Tapi sebelum itu terjadi, *we'll just have to live with it*.

### Back to the Architecture!

Keterbatasan server SIAK-NG yang telah dibahas di atas tentu merupakan faktor penting yang harus dipertimbangkan dalam melakukan desain aplikasi kita. Ini berarti *scraping* (dan kerja aplikasi SusunJadwal lainnya) yang kita lakukan harus sedemikian rupa sehingga tidak memberatkan *load* SIAK-NG.

Dengan pertimbangan tersebut, kami mendefinisikan tiga buah komponen dari aplikasi ini:

- **Scraper**
  Komponen ini bertugas untuk melakukan *scraping* data jadwal. Agar tidak memberatkan server, proses ini dijadwalkan secara berkala dan hasilnya disimpan pada sebuah file lokal untuk diakses oleh komponen lainnya.
- **API Server**
  Backend dari SusunJadwal. Komponen ini menghubungkan *client* dengan hasil *scraping* dan database aplikasi.
- **Client App**
  Sebuah *single-page web app* (SPA) yang bertindak sebagai UI dari aplikasi SusunJadwal. Komponen ini berkomunikasi secara langsung hanya dengan API server.

*Information flow* dari arsitektur ini adalah sebagai berikut:

1. Scraper melakukan *scraping* dari SIAK-NG secara berkala dan menaruh hasilnya ke sebuah file.
2. Pengguna membuka [aplikasi SusunJadwal](http://ristek.cs.ui.ac.id/susunjadwal) melalui *client app* dan memilih jurusannya.
3. *Client* kemudian melakukan *request* ke API Server untuk meminta jadwal jurusan yang bersangkutan.
4. API Server membaca file hasil *scraping* dan mengembalikan jadwal untuk jurusan yang dimaksud.
5. *Client* menampilkan jadwal yang diperoleh pada *browser*.
6. Operasi-operasi selanjutnya (pemilihan dan *generate* jadwal) dilakukan sepenuhnya pada *client app* dan tidak memberatkan baik API server dan server SIAK-NG.

Lebih jelas, *information flow* ini digambarkan pada diagram berikut.

/gambar/

Kita telah memiliki arsitektur yang dibutuhkan! Sekarang saatnya melakukan *breakdown* tiap komponen.

## Components: In Depth

Satu hal yang perlu mungkin perlu diketahui oleh pembaca, SusunJadwal versi Ganjil 2015/2016 mulai dikembangkan sekitar bulan Juni 2015 oleh tim SIG Web Development Ristek CSUI 2015.

SusunJadwal versi sebelumnya dikembangkan dengan bahasa pemrograman PHP. Karena [kami memutuskan untuk mencoba menjauhi PHP](https://eev.ee/blog/2012/04/09/php-a-fractal-of-bad-design/), maka untuk versi ini, kami mencoba melakukan *rewrite* dengan platform lain, yaitu Node.js. Node.js dipilih mengingat SusunJadwal penuh dengan operasi I/O yang *asynchronous*, bidang yang digadangkan sebagai *use case* paling cocok untuk platform tersebut.

(*Yes, some might argue that Javascript isn't that much better from PHP. But we're still exploring things, so...*)

Di bagian ini akan dibahas mengenai tiap-tiap komponen dari SusunJadwal, yaitu scraper, API server, dan *client app*. Untuk masing-masing komponen, akan dijelaskan mengenai teknologi yang digunakan dan bagaimana cara kerjanya.

### Scraper

**Technologies used**

- request
- async
- cheerio

**How it works**

- authentication
  - authenticate
  - changeRole
- jadwal scraping and parsing
  - cheerio
  - saving to JSON

### API server

**Technologies used**

- express
- mongoDB
- mongoose

**How it works**

- reading jadwal from JSON file
- saving jadwal

### Client app

**Technologies used**

- angular
- sass

**How it works**

- fetching from API
- conflict checking
- schedule drawing
- joining jadwal

## What's next?

SusunJadwal mungkin sangat *niche*, karena ia bekerja hanya dua kali setahun (masing-masing dalam periode empat hari) dan hanya menargetkan mahasiswa UI. Tapi membantu orang tidak berarti kita harus membuat sesuatu yang sebesar Facebook, Airbnb, dan Amazon. *We may start with something small*, tapi pastikan masalahnya memang ada (jangan mengada-ada masalah).

Masih ada beberapa fitur dan pengembangan lebih lanjut dari SusunJadwal yang belum sempat direalisasikan, seperti misalnya fitur export gambar JPG/PNG dari jadwal yang sudah dibuat. If you have ideas, hubungi segera SIG Web Development [Ristek Fasilkom UI](https://twitter.com/ristekcsui)!

Saya harap post ini dapat menjadi inspirasi baik dari segi ide maupun segi teknis jika pembaca ingin mengembangkan sesuatu.

*Until next time*!
