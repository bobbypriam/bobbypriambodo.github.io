---
layout: post
title: "The engineering behind SusunJadwal"
date: 2016-09-10 04:47:00
summary: Sejak beberapa tahun silam, SIAK war di Universitas Indonesia selalu didahului oleh peluncuran aplikasi SusunJadwal. Bagaimana aplikasi tersebut bekerja?
---

Universitas Indonesia (UI) memiliki sebuah tradisi pada setiap awal semester yang bernama SIAK war.

> SIAK war (n.) - Masa pengisian Isian Rencana Studi (IRS) secara online. Bayangkan KRL Commuter Line arah Bogor jam pulang kerja, tapi online.

Isian Rencana Studi (IRS) adalah sebuah formulir yang berisi daftar kelas mata kuliah yang akan diambil oleh seorang mahasiswa pada suatu semester. Di beberapa kampus lain, IRS juga dikenal sebagai KRS. Pengisian IRS di UI dilakukan secara online melalui sebuah sistem bernama SIAK-NG (Sistem Informasi AKademik - *Next Generation*).

 Di UI, mahasiswa dibebaskan untuk memilih kelas yang ingin diambil pada suatu semester (kecuali pada beberapa Fakultas seperti Fakultas Kedokteran yang sudah memiliki paket tetap tiap semester).  Karena mahasiswa bebas memilih, terjadilah perebutan kelas-kelas favorit. Fenomena inilah yang disebut SIAK war.

 (*It's not pretty*. Apalagi ditambah kebiasaan server SIAK-NG yang *down* ketika mengalami *overload*, yang akan [kita bahas nanti di artikel ini](#on-siak-ngs-performance-and-scalability).)

Ada beberapa faktor yang memengaruhi pemilihan ini, seperti misalnya kemungkinan jadwal kuliah yang bentrok <del>dan bagaimana mata kuliah dengan dosen yang \*uhuk\* baik \*uhuk\* memiliki banyak peminat dan memicu perselisihan batin</del>. Dengan waktu mahasiswa yang terbatas, jadwal kuliah harus diatur sedemikian rupa sehingga tidak bentrok dan masih menyisakan waktu untuk kegiatan lain, seperti misalnya berorganisasi atau main layangan.

Demi membantu penyusunan jadwal kuliah, ada sebuah menu pada SIAK-NG bernama "Jadwal Kuliah" yang memuat daftar-daftar kelas yang dibuka pada semester tersebut. Daftar akan terus bertambah seiring mendekatnya waktu pengisian IRS, sehingga mahasiswa perlu memeriksa halaman tersebut berkali-kali untuk melihat jadwal kelas baru. Berikut adalah gambar contoh daftar kelas yang ada pada halaman tersebut.

![Jadwal Kuliah]({{ site.baseurl }}/img/posts/jadwal-kuliah.jpg) <small>Screenshot halaman Jadwal Kuliah</small>

Ya, daftar disajikan dalam bentuk... daftar. Alhasil, sulit bagi mahasiswa untuk mendapatkan informasi secara visual apakah mata kuliah yang dipilih memiliki jadwal bentrok atau tidak.

Sebagai mahasiswa Fasilkom yang haus kontribusi, kawan-kawan dari [Pandago Studio](https://twitter.com/pandagostudio) yang lalu dilanjutkan oleh [Ristek](http://ristek.cs.ui.ac.id/) mengembangkan sebuah aplikasi bernama **SusunJadwal**.

Pada artikel ini, kita akan mencoba menelusur lebih dalam: bagaimana aplikasi SusunJadwal bekerja?

## About SusunJadwal

*Disclaimer: Segala bahasan dari artikel ini mengacu pada versi SusunJadwal yang melibatkan saya langsung dalam pengembangannya, yaitu periode Ganjil dan Genap 2015/2016 oleh Ristek CSUI. Pengembangan sebelum dan sesudahnya mungkin memiliki detail yang berbeda.*

SusunJadwal adalah sebuah aplikasi web yang dapat membantu mahasiswa UI menyusun rencana jadwal kuliah pra-pengisian IRS.

Idenya cukup sederhana: berdasarkan informasi kelas pada halaman "Jadwal Kuliah", pengguna dapat secara interaktif memilih kelas-kelas yang ingin diambil, dan sistem akan secara otomatis memberikan notifikasi apabila ada jadwal kelas yang bentrok. Apabila jadwal sudah bersih dari bentrok, pengguna juga dapat meng-*generate* representasi visual jadwal agar rancangan jadwal dapat lebih mudah dimengerti. Sebagai gambaran, [SusunJadwal versi Ganjil 2016/2017 dapat diakses pada tautan ini](http://ristek.cs.ui.ac.id/susunjadwal/).

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

Yang cukup menyulitkan lagi, seperti yang sempat saya tulis di awal, server SIAK-NG memiliki kelemahan apabila diakses oleh ribuan orang bersama-sama pada suatu waktu. Let's talk about this for a bit.

### On SIAK-NG's Performance and Scalability

Tahun lalu, saya memperoleh informasi bahwa ketidakmampuan server SIAK-NG menampung ribuan *request* pada saat SIAK war adalah (sedikit banyak) *by design*.

"Apa!?" kita mungkin berseru. Tapi itu masuk akal.

Coba tanyakan kepada diri kita sendiri: dalam setahun, seberapa sering kita mengakses SIAK-NG? Sekali, ketika mengisi IRS. Beberapa kali selanjutnya masih di minggu yang sama untuk melihat apakah jadwal bisa berubah atau sudah disetujui oleh Pembimbing Akademik. Mungkin beberapa kali lagi untuk melihat Kalender Akademik untuk tanggal-tanggal penting semester tersebut. Selebihnya?

Berdasarkan sumber yang dapat dipercaya, tingkat *availability* sistem SIAK-NG sendiri dalam setahun mendekati 99%, di mana 1% *downtime* terjadi pada masa SIAK war. Masih menurut sumber tersebut, meningkatkan kapabilitas server untuk memenuhi hanya 1% waktu dalam satu tahun tidak *feasible* dari segi finansial. *The argument has a point*.

*But still*, ini tidak membenarkan; untuk apa sistem hidup ketika tidak dibutuhkan dan mati ketika dibutuhkan?

Tentu saja, ini tidak berarti tim IT UI tidak mengusahakan performa yang lebih baik untuk SIAK-NG pada masa pengisian IRS. Sumber yang sama mengatakan server SIAK-NG sendiri sebenernya sudah di-*load balance* (dapat dilihat jika kita login, di atas nama kita di kiri atas ada indikator Node mana yang melayani *request* kita). Sayangnya, masih terdapat *bottleneck* pada I/O dari *database*. *Database* server hanya ada satu, yang menyebabkan sistem *down* ketika *overload*.

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
6. Operasi-operasi selanjutnya (pemilihan, pengecekan konflik, dan *generate* jadwal) dilakukan sepenuhnya pada *client app* dan tidak memberatkan baik API server dan server SIAK-NG selain ketika menyimpan jadwal.

Lebih jelas, *information flow* ini digambarkan pada diagram berikut.

![Diagram Arsitektur SusunJadwal]({{ site.baseurl }}/img/posts/susunjadwal-architecture-diagram.png) <small>Diagram Arsitektur SusunJadwal</small>

Panah berbeda warna biru dan merah menunjukkan bahwa kedua operasi tersebut saling independen. Nomor pada panah menunjukkan alur informasi yang terjadi.

Kita telah memiliki arsitektur yang dibutuhkan! Sekarang saatnya melakukan *breakdown* tiap komponen.

## Components: In Depth

Satu hal yang perlu mungkin perlu diketahui oleh pembaca, SusunJadwal versi Ganjil 2015/2016 mulai dikembangkan sekitar bulan Juni 2015 oleh tim SIG Web Development Ristek CSUI 2015.

SusunJadwal versi sebelumnya dikembangkan dengan bahasa pemrograman PHP. Karena [kami memutuskan untuk mencoba menjauhi PHP](https://eev.ee/blog/2012/04/09/php-a-fractal-of-bad-design/), maka untuk versi ini, kami mencoba melakukan *rewrite* dengan platform lain, yaitu Node.js. Node.js dipilih mengingat SusunJadwal penuh dengan operasi I/O yang *asynchronous*, bidang yang digadangkan sebagai *use case* paling cocok untuk platform tersebut.

(*Yes, some might argue that Javascript isn't that much better from PHP. But we're still exploring things, so...*)

Di bagian ini akan dibahas mengenai tiap-tiap komponen dari SusunJadwal, yaitu *scraper*, API server, dan *client app*. Untuk masing-masing komponen, akan dijelaskan mengenai teknologi yang digunakan dan bagaimana cara kerjanya.

### Scraper

Komponen *scraper* dibuat menggunakan beberapa library Node.js sebagai berikut:

- **request** ([github](https://github.com/request/request), [npm](https://www.npmjs.com/package/request))
  Sebuah library untuk mempermudah pembuatan HTTP *request*. Ada beberapa library lain sebagai alternatif dari request, misalnya superagent.
- **async** ([github](https://github.com/caolan/async), [npm](https://www.npmjs.com/package/async))
  Library untuk memudahkan *control flow* ketika menggunakan fungsi-fungsi *asynchronous* pada JavaScript (kami belum mengenal Promise waktu itu).
- **cheerio** ([github](https://github.com/cheeriojs/cheerio), [npm](https://www.npmjs.com/package/cheerio))
  Secara sederhana, JQuery (ugh) untuk server. Library ini digunakan untuk melakukan memudahkan parsing dokumen HTML.

Tugas dari *scraper* adalah untuk *scraping*. Satu tantangan dalam mengembangkan komponen ini adalah perlunya proses otentikasi menggunakan akun mahasiswa UI sebelum dapat mengakses halaman "Jadwal Kuliah". Yang membuat hal ini menantang adalah adanya dua langkah dalam otentikasi yang membuatnya tidak dapat di-solve secara *straightforward*.

Kalau kita mencoba login melalui browser, jika diperhatikan baik-baik, URL akan berubah dari `/Authentication/Index` ke `/Welcome`. Namun sebenarnya, ada sebuah URL tambahan yang dikunjungi sebelum berpindah ke `/Welcome`, yaitu `/Authentication/ChangeRole`. Hal ini dapat dilihat jika kita melakukan request otentikasi menggunakan perkakas *command-line* seperti curl.

Berikut adalah potongan kode yang digunakan untuk melakukan otentikasi (*full with nasty callbacks*):

```javascript
var credentialsIlkom = { 'u': process.env.SIAK_ILKOM_USERNAME, 'p': process.env.SIAK_ILKOM_PASSWORD };
var credentialsSI = { 'u': process.env.SIAK_SI_USERNAME, 'p': process.env.SIAK_SI_PASSWORD };

function auth(credentials, callback) {
  authenticate();

  function authenticate() {
    request.post({
      url: constants.AUTH_URL,
      formData: credentials
    }, function (err, response, body) {
      if (err) return console.error('Error!');

      changeRole();
    });
  }

  function changeRole() {
    request.get({
      url: constants.CHANGEROLE_URL
    }, function (err, response, body) {
      if (err) return console.error('Error!');

      callback();
    });
  }
}
```

Pada saat pemanggilan `callback()`, kita sudah berada pada kondisi terotentikasi, sehingga kita bisa mulai mengakses halaman-halaman lain.

Setelah terotentikasi, langkah selanjutnya adalah mengakses jadwal. Untuk jadwal kuliah jurusan Ilmu Komputer dan Sistem Informasi, kami menggunakan akun kami masing-masing, sehingga bisa langsung dilakukan dengan mengakses halaman "Jadwal Kuliah". Sementara untuk jadwal kuliah jurusan lain, kita harus melalui halaman "Jadwal Kuliah Keseluruhan" dengan beberapa parameter tambahan.

```javascript
function getJadwal(period, callback) {
  request.get({
    url: constants.JADWAL_URL + '?period=' + period
  }, function (err, response, body) {
    if (err) return callback(err);
    callback(null, body);
  });
}

function getJadwalLain(organization, period, callback) {
  // Faculty is last five characters of organization.
  var faculty = organization.substr(organization.length - 5);
  request.get({
    url: constants.JADWAL_SELURUH_URL + '?fac=' + faculty + '&org=' + organization + '&per=' + period
  }, function (err, response, body) {
    if (err) return callback(err);
    callback(null, body);
  });
}
```

(*I realize now how silly it was to mix English and Indonesian in variable and function names. For the future me: Don't!*)

Dua fungsi di atas digunakan untuk mengambil jadwal. Fungsi `getJadwal()` digunakan untuk mengakses halaman jadwal untuk Ilkom dan SI, sementara `getJadwalLain()` menerima parameter tambahan `organization` yaitu kode organisasi sebuah jurusan di UI. Sebagai contoh, jurusan Pendidikan Dokter memiliki kode organisasi **04.00.01.01**.

Fungsi ini kemudian akan digunakan untuk pengambilan data jadwal. Kami menggunakan library *async* agar pengambilan jadwal untuk tiap-tiap jurusan dapat dilakukan secara paralel, sehingga lebih efisien.

Jadwal diambil dalam format HTML. Agar pemrosesan dapat dilakukan lebih mudah, dilakukan *parsing* HTML menjadi sebuah JavaScript *object* dengan bantuan library *cheerio*. Hal ini dipersulit dengan penggunaan struktur tabel pada halaman tersebut di SIAK-NG yang tidak konvensional.

Hasil konversi kemudian ditulis ke dalam sebuah file dalam format JSON untuk kemudian digunakan oleh API server. Proses ini dilakukan secara berkala, satu jam sekali, dengan bantuan perkakas *cron*.

### API server

Library Node.js yang digunakan untuk API server:

- **express** ([github](https://github.com/expressjs/express), [npm](https://npmjs.org/package/express))
  Sebuah web framework yang dapat digunakan untuk *spin up* sebuah API server secara cepat.
- **mongoose** ([github](https://github.com/Automattic/mongoose), [npm](https://www.npmjs.com/package/mongoose))
  *Kind of like* ORM untuk mongoDB. Ya, kami menggunakan mongoDB untuk database SusunJadwal.

Kerja API server pada aplikasi SusunJadwal cukup sederhana. Hanya terdapat tiga buah *endpoint* GET yang digunakan, yaitu `/jadwal/jurusan/:jurusan` untuk mengambil data jadwal berdasarkan `jurusan`, `/jadwal/shortlink/:shortlink` untuk mengambil jadwal yang sudah pernah tersimpan menggunakan shortlink-nya, dan `/jadwal/gabung` yang menerima *query parameter* `shortlinks` berisi shortlink-shortlink dari jadwal yang ingin digabungkan, serta sebuah *endpoint* POST pada `/jadwal` untuk menyimpan jadwal dan mendapatkan shortlink.

Pada dasarnya, API server bertindak sebagai *proxy* antara *client app* dengan database jadwal dan file hasil *scraping*. Kerjaan berat dari SusunJadwal sendiri ada pada dua komponen lainnya.

### Client app

*Client app* dari SusunJadwal adalah sebuah aplikasi web berbentuk *single-page app* (SPA) yang memanfaatkan framework AngularJS v1. *To be honest*, sekarang saya tidak menyukai Angular, dengan alasan yang hampir sama dengan kebanyakan orang di luar sana. Tapi saya tidak akan membahasnya di artikel ini. (*By the way*, kalau Anda sedang ingin membuat *front-end* JS *app*, coba lihat [React](https://facebook.github.io/react/blog/2016/07/22/create-apps-with-no-configuration.html)+[Redux](http://redux.js.org/), *or better yet*, [Elm](http://elm-lang.org/).)

Tugas dari komponen ini salah satunya adalah berkomunikasi dengan API server untuk pengambilan data-data serta penyimpanan jadwal  menggunakan modul `$http` dari Angular. *Nothing fancy in that*. Fitur gabung jadwal juga sebenarnya tidak sulit diimplementasi; pada dasarnya fitur tersebut hanya melakukan overlay jadwal-jadwal yang dimasukkan.

Fitur yang bagi saya pribadi lebih menarik dari *client app* SusunJadwal adalah penentuan apakah sebuah daftar jadwal memiliki bentrok atau tidak. Yang membuat ini menarik adalah proses pengecekan apabila ada dua buah jadwal, `current` dan `opponent`, bagaimana kita tahu bahwa jadwal tersebut overlap. Untuk tiap jadwal, kita memiliki waktu mulai dan waktu selesai. Jadi kita memiliki empat variabel waktu, yaitu `currentStart`, `currentEnd`, `opponentStart`, `opponentEnd`.

*I'ts hard to admit that I'd spent too much time searching for an algorithm to this particular problem*. Saya telah memanipulasi pengecekan kondisi keempat variabel tersebut namun entah kenapa selalu ada case yang tidak berhasil ditemukan konflik. Saya tidak ingat case-nya apa, namun saya ingat sempat kesal karena masalah yang terlihat mudah ini cukup menyulitkan. Solusi yang saya berikan meliputi beberapa tahap kasus pengecekan. Saya hampir menyerah.

Sampai saya [menemukan jawabannya di StackOverflow](http://stackoverflow.com/questions/143552/comparing-date-ranges/143568#143568). Menurut saya, jawaban tersebut merupakan salah satu jawaban paling elegan yang pernah saya baca di SO.

(Serius. Jika Anda, pembaca, punya waktu, silakan baca jawaban tersebut sebentar. Jangan lupa kembali lagi ke sini!)

Singkat cerita, kesulitan yang saya alami diakibatkan oleh *approach* saya yang tidak tepat; saya terlalu terfokus ke kasus-kasus apa saja yang menimbulkan konflik, sementara problem ini akan lebih mudah dipecahkan juga kita fokus pada kasus-kasus apa yang **tidak** menimbulkan konflik.

*And it boils down to this simple conditional*:

```javascript
if (opponentEnd >= currentStart && opponentStart <= currentEnd) {
  conflict = true;
}
```

*WTF! What have I done with my time?*

Terlebih lagi, solusi ini sangat *counter-intuitive* dan membuat kita sulit percaya bahwa kondisi tersebut sudah mencakup keseluruhan kasus konflik yang terjadi. Tapi itu benar, dan ribuan pengguna SusunJadwal telah membuktikan kebenarannya.

Ini adalah salah satu kasus di mana saya belajar bahwa sebuah masalah yang sulit dapat menjadi mudah dengan hanya mengubah perspektif kita.

Untuk menghargai *revelation* ini, saya menambahkan link ke StackOverflow tersebut pada kode yang melakukan pengecekan jadwal ini. Berikut adalah kode terfenomenal yang pernah saya tulis:

```javascript
function checkConflict() {
  var kelasArray = [];
  vm.conflict = false;

  forEachObject(vm.selectedKelas, function (kelas) {
    kelas.conflict = false;
    kelasArray.push(kelas);
  });

  // Four nasty nested loops, I know...
  // But since the selected class can be assumed to be
  // no more than 20, I guess this is still safe.
  while (kelasArray.length > 0) {
    var current = kelasArray.pop();
    for (var i = 0; i < kelasArray.length; i++) {
      var opponent = kelasArray[i];
      for (var j = 0; j < current.jadwal.length; j++) {
        for (var k = 0; k < opponent.jadwal.length; k++) {
          if (current.jadwal[j].hari == opponent.jadwal[k].hari) {
            var currentStart = convertHourStringToMinutes(current.jadwal[j].jamMulai);
            var currentEnd = convertHourStringToMinutes(current.jadwal[j].jamSelesai);
            var opponentStart = convertHourStringToMinutes(opponent.jadwal[k].jamMulai);
            var opponentEnd = convertHourStringToMinutes(opponent.jadwal[k].jamSelesai);

            // Conflict comparison for interval is this simple.
            // Credit: http://stackoverflow.com/questions/143552/comparing-date-ranges/143568#143568
            if (opponentEnd >= currentStart && opponentStart <= currentEnd) {
              current.conflict = true;
              opponent.conflict = true;
              vm.conflict = true;
            }
          }
        }
      }
    }
  }
}
```

(*Yikes*. Saya tidak sempat menganalisis apakah *nested loop* tersebut bisa dioptimisasi atau tidak, tapi untungnya *so far* tidak ada keluhan performance ketika pengecekan konflik. *Sometimes you just have to leave things be*.)

## What's next?

Dengan selesainya pembahasan mengenai komponen, maka artikel ini sudah sampai akhir.

SusunJadwal mungkin sangat *niche*, karena ia bekerja hanya dua kali setahun (masing-masing dalam periode empat hari) dan hanya menargetkan mahasiswa UI. Tapi membantu orang tidak berarti kita harus membuat sesuatu yang sebesar Facebook, Airbnb, dan Amazon. *We may start with something small*, tapi pastikan masalahnya memang ada (jangan mengada-ada masalah).

Masih ada beberapa fitur dan pengembangan lebih lanjut dari SusunJadwal yang belum sempat direalisasikan, seperti misalnya fitur export gambar JPG/PNG dari jadwal yang sudah dibuat. If you have ideas, hubungi segera SIG Web Development [Ristek Fasilkom UI](https://twitter.com/ristekcsui)!

Saya harap post ini dapat menjadi inspirasi baik dari segi ide maupun segi teknis jika pembaca ingin mengembangkan sesuatu.

*Until next time*!

<small>P.S. *The code I used in the article was written more than a year ago, back when I didn't give much thought about coding best practices. I cringed every time I copy-pasted them while writing this article. I've learned a lot since then, I promise!* :D</small>
