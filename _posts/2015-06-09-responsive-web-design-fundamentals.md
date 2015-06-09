---
layout: post
title: "Responsive Web Design Fundamentals"
date: 2015-06-09 10:26:00
---

Apa itu *responsive design*? *Buzzword* yang satu ini memang sudah tak asing lagi di telinga kita. Responsif sudah menjadi satu *requirement* yang wajib ada ketika kita ingin membuat website baru. Dan, membuat situs yang responsif memang tidak sulit, tapi jika kamu langsung berlari ke [*Bootstrap*](http://getbootstrap.com/), [*Foundation*](http://foundation.zurb.com/), atau *framework-framework* lainnya, sepertinya ada hal dasar penting yang kamu lewatkan.

Menggunakan *framework* seperti Bootstrap memang memudahkan. Tapi, sangat mudah juga untuk "merusak" prinsip-prinsip situs responsif yang dibawa oleh *framework* tersebut jika kita tidak memiliki pemahaman dasarnya. Pada artikel ini, kita akan coba membahas elemen-elemen fundamental dari sebuah desain web yang responsif, yang harapannya dapat membawa kita menjadi developer web yang lebih baik.

## Sedikit latar belakang

Mengapa kita membutuhkan situs yang responsif? *What the hell is responsive design anyway*?

Setiap harinya, teknologi mengalami perkembangan. Seberapa kali pun kalimat ini diutarakan sepertinya kebenarannya tidak pernah hilang. Dulunya, web hanya diakses melalui browser yang ada pada *personal computer* (PC) saja, namun kini kamu bahkan [bisa mengakses internet melalui jam tangan](http://blogs.opera.com/news/2014/10/samsungs-gear-s-smartwatch-gets-first-full-web-browser/).

*Well*, mungkin *smart watch* terlalu ekstrim, namun di antara PC dan *smart watch*, ada banyak device lain seperti *smartphone*, tablet, *phablet*, dan sebagainya yang memiliki ukuran layar berbeda-beda. Situs web konvensional memiliki ukuran halaman yang cenderung statis&mdash;dioptimisasi untuk PC. Namun dengan [meningkatnya akses internet melalui *mobile device*](http://searchenginewatch.com/sew/how-to/2389159/why-mobile-web-still-matters-in-2015), hal itu tentu saja tidak relevan lagi. Tentu, developer web bisa saja tidak peduli dan membiarkannya, toh user masih bisa melakukan *zooming*. Tapi itu akan meninggalkan *user experience* yang kurang baik, bukan?

Masuklah *responsive design*. Untuk menjelaskannya, izinkan saya mengambil kalimat pertama dari halaman Wikipedia untuk [*responsive web design*](http://en.wikipedia.org/wiki/Responsive_web_design):

> Responsive web design (RWD) is an approach to web design aimed at crafting sites to provide an optimal viewing and interaction experience—easy reading and navigation with a minimum of resizing, panning, and scrolling—across a wide range of devices (from desktop computer monitors to mobile phones).

Nah. Penjelasan yang cukup teknis khas Wikipedia tersebut kira-kira cukup mendeskripsikan apa itu *responsive design*... atau tidak. *Anyway*, intinya adalah, sebuah situs responsif nyaman diakses dari *device* apa pun. Dan di sini kita akan melihat bagaimana caranya.

Jadi. Siap? Mari kita mulai bagian serunya.

## Viewport

Elemen penting pertama dari sebuah situs web yang responsif adalah deklarasi *meta tag* untuk *viewport*. (Saya asumsikan pembaca artikel ini memiliki pemahaman dasar tentang HTML dan CSS. Kalau merasa belum, [teman-teman di W3Schools](http://w3schools.com/) memiliki sumber belajar yang cukup lengkap.)

Sebelum kita mulai membahas lebih jauh, mari kita lihat seperti apa *tag* yang dimaksud:
    
    {% highlight html %}
    <meta name="viewport" content="width=device-width, initial-scale=1">
    {% endhighlight %}

Jadi, apa yang spesial dari *tag* ini? Yah, *tag* ini ada pada template HTML di tutorial *Getting Started* *Bootstrap* dan *Foundation*, jadi pasti penting, kan? Bahkan dokumentasi *Foundation* secara spesifik menyebutkan: *"If you delete this meta tag World War Z will become a reality."*

Dan saya yakin kamu tidak ingin hal itu terjadi.

Sudah banyak pembahasan mengenai *viewport* dalam *responsive design* (contohnya di [sini](https://developers.google.com/web/fundamentals/layouts/rwd-fundamentals/set-the-viewport?hl=en), [sini](https://developer.mozilla.org/en/docs/Mozilla/Mobile/Viewport_meta_tag), dan [sini](http://www.paulund.co.uk/understanding-the-viewport-meta-tag)), tapi jika kamu adalah orang yang malas mengikuti tautan yang ditulis orang lain di blognya, maka saya akan menjelaskannya sedikit di sini.

*Meta tag* *viewport* memberitahu *browser* bagaimana ia harus menampilkan sebuah halaman web. *Viewport*, dalam istilah sederhana, adalah layar *device* kamu. Dengan *tag* ini, kita dapat menentukan seberapa ukuran *viewport* yang harus digunakan untuk menampilkan halaman web kita.

Seperti yang dapat dilihat, ada dua *property* yang ada di *attribute* `content` dari kode tersebut, yaitu `width` dan `initial-scale`.

Properti `width` mendeskripsikan seberapa lebar viewport yang dibutuhkan. Kamu bisa saja mengganti nilainya dengan, misalnya, `width=500`, namun akibatnya, ketika diakses situs kamu akan menggunakan viewport selebar 500px saja, yang tentunya tidak baik untuk desain yang responsif. Sebuah *special value* `device-width` mengatakan bahwa, buat *viewport*-nya selebar layar dari *device* yang digunakan. Bagus.

Properti kedua, `initial-scale`, juga memainkan peranan penting untuk *responsive design*. Pengaturan `initial-scale=1` bekerja sama dengan `width=device-width` agar level *zoom* ketika halaman dibuka berada pada level optimal, sehingga user tidak perlu melakukan *zooming* lagi untuk melihat kontennya. Gabungkan keduanya, *voila*: situs yang responsif!

Jadi itu tadi penjelasan singkat mengenai *viewport*. Percaya atau tidak, rahasia dari situs responsif hanyalah sebuah *tag* ini saja. Taruh *tag* tersebut pada elemen `<head>` situs kalian dan pekerjaan selesai. Sihir? Tidak juga.

Tapi manusia adalah makhluk yang tidak mudah puas, dan terkadang sangat mudah menghancurkan responsivitas yang diberikan *meta tag* *viewport* melalui tangan-tangan (dan baris-baris kode) yang tidak bertanggung jawab. Contohnya adalah dengan menambahkan *fixed-width element* (`width: 400px`) yang menyebabkan *horizontal-scrolling*. Hal ini akan pada lanjutan artikel ini.

Ah, tapi bagaimana kalau kita mau tampilan yang berbeda untuk tiap layar? Kita bisa gunakan...

## Media Query

Pertama-tama, [kamu tidak benar-benar membutuhkan *media query* untuk membuat situs responsif](http://motherfuckingwebsite.com/).

Tapi untuk memenuhi tuntutan dari desainer atau suara kecil di telingamu yang mengajak untuk membuat hal-hal keren, *media query* merupakan elemen penting lain untuk *responsive design*.

*Media query* adalah sebuah filter yang dapat digunakan dalam CSS (["Wut?"](http://w3schools.com/)) untuk mengekspresikan keresponsifan. Maksud dari filter adalah "kalau kriteria **ini** terpenuhi, maka tampilkan seperti **itu**." **Ini** tersebut memiliki berbagai macam, mulai lebar, tinggi, orientasi, serta resolusi *device*, sementara **itu** adalah... kode CSS biasa. Mari kita lihat.

Secara umum, sintaks dari *media query* adalah sebagai berikut.

{% highlight css %}
@media (query) { /* "Ini" */
  /* "Itu" */
}
{% endhighlight %}

Tidak mengerti? Bagus. Berikut adalah contoh praktisnya.

{% highlight css %}
@media (max-width: 800px) {
  body {
    font-size: 16px;
    line-height: 1.5;
  }
}
{% endhighlight %}

Kode ini seharusnya *self-explanatory*, namun dalam bahasa sehari-hari kira-kira bisa dibaca sebagai "kalau ukuran viewportnya belum melebihi 800 pixel, maka tampilkan tulisan dengan ukuran 16 pixel dan *line-height* 1.5." Tentu kamu bisa mengganti 800px menjadi value lain, misalnya 1366px, 40rem, dan sebagainya, dan kamu juga bisa memasukkan sebanyak apa pun CSS di dalam *media query* tersebut. Jadi *media query* adalah cara kita untuk menerapkan *styling* secara kondisional.

Apakah *query* yang ada hanya `max-width` saja? Tidak, ada juga `min-width`, `max-height`, `min-height`, dan `orientation`. Kita juga bisa menggabungkan beberapa query sehingga kriterianya bisa lebih spesifik. Lagi-lagi: sihir.

### Menentukan "Breakpoint"

Sangat mudah untuk meng-*abuse* *media-query* dengan segala macam *query* dalam CSS kita, namun tentunya hal tersebut tidak efektif, dan programmer yang tidak efektif bukanlah programmer yang baik.

Karena itu, kita harus mengusahakan agar *query-query* yang dibuat sudah mencakup kasus-kasus ukuran yang paling umum. Para suhu-suhu dari Google sudah membuat [artikel yang cukup bagus mengenai hal ini](https://developers.google.com/web/fundamentals/layouts/rwd-fundamentals/how-to-choose-breakpoints?hl=en), jadi saya sarankan untuk (sekali ini saja) mengikuti tautan tersebut. Selain itu, kita juga dapat melihat framework-framework CSS untuk melihat breakpoint-breakpoint yang mereka gunakan.

## Buat semuanya relatif (kalau bisa)

Masih ingat bagaimana *fixed-width* atau *fixed-positioned element* bisa menghancurkan responsivitas situs kamu? Cara untuk mengatasinya adalah: buat semuanya relatif. Kalau bisa. Dan kalau pantas (*Well*, saya bukan orang yang suka dengan pendekatan "*one ring to rule them all*", jadi saya harap pembaca juga bisa lebih [pragmatis](http://www.amazon.com/The-Pragmatic-Programmer-Journeyman-Master/dp/020161622X)).

Konsep dasar dari *responsive design* adalah *fluidity*. Kita ingin agar layout situs kita dapat beradaptasi dengan ukuran layar. Penggunaan satuan ukuran relatif adalah salah satu cara mewujudkannya. Sebagai contoh, sebuah `<div>` dengan `width` 500px akan tampil keluar layar apabila dilihat dari *device* yang memiliki ukuran layar 320px.


Dalam kasus seperti ini, usahakan gunakan satuan unit yang relatif, seperti misalnya `em`, `rem`, dan `%`. Hindari juga *absolute* atau *relative positioning* yang dapat menyebabkan elemen berada di luar *viewport*.

## That's it!

Demikianlah elemen-elemen fundamental dalam membangun sebuah situs yang responsif. Kuasai hal ini dan kamu akan selangkah lebih dekat untuk menjadi penguasa dunia web development selanjutnya.