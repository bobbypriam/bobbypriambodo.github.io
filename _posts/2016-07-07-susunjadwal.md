---
layout: post
title: "The engineering behind SusunJadwal"
date: 2016-07-07 04:47:00
---

Outline:

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
- what's next?
