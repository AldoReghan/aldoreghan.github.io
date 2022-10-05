---
layout: post
title: WriteUp Bootcamp cyber security "VM Easy5"
date: 2022-09-08
categories: ["Pentest", "WriteUp"]
---

VM Easy 5 created by [nikkoenggaliano](https://github.com/nikkoenggaliano)

Tools yang digunakan :
- Firefox
- X-forwarder-for (Firefox Exte  nsion)
- Burpsuite
- Wsl (Kali linux)
- Hydra

Target : flag di folder root

<b>Write Up</b> :

1. Menginputkan IP dari virtual machine ke dalam web browser, berikut adalah tampilan halaman web dari Ip address virtual machine
   <br>
   ![01](/assets/easy5/01.png?raw=true)

2. Melakukan view page source, untuk melihat apakah tedapat suatu clue di dalam source nya, dan hasilnya seperti berikut terdapat sebuah komentar <b>“Maybe you can search how to use x-forwarded-for”</b>, komentar ini merupakan sebuah instruksi untuk menggunakan x-forwarder-for.<br>
   ![01](/assets/easy5/02.png?raw=true)
   <br>
3. Melakukan manipulasi ip address menggunakan x-forwarder-for menjadi 127.0.0.1, agar website seolah2 diakses melalui lokal, dan selanjutnya refresh halaman webnya

    Note : pastikan saat mensetting ip address harus di tab website virtual machine
    <br>
    ![01](/assets/easy5/03.png?raw=true)
    <br>

4. Setelah di refresh terdapat sebuah tampilan halaman index yang sebenarnya<br>![01](/assets/easy5/Selection_001.png?raw=true)<br>
   
5. Hal yang pertama yang saya lakukan adalah melakukan register di halaman tersebut, dan selanjutnya login menggunakan akun yang telah saya registrasi, berikut tampilan dari website setelah register
   <br>![01](/assets/easy5/Selection_002.png?raw=true)<br>
   
6. Selanjutnya yang saya lakukan adalah melakukan debugging dengan cara, mengklik masing-masing menu yang terdapat di website tersebut.
   - Dashboard<br>![01](/assets/easy5/Selection_003.png?raw=true)<br>
   - Profile<br>![01](/assets/easy5/Selection_004.png?raw=true)<br><br>
  
7. Di page profile terdapat url yang menunjukan “user_id=12” dimana 12 merupakan akun saya, saat diubah menjadi 1, akan muncul data users sesuai id, dengan begitu, url user_id dapat di eksplorasi untuk mendapatkan akun-akun lainnya<br>![01](/assets/easy5/Selection_005.png?raw=true)<br>
   
8. Selanjutnya saya menggunakan burpsuite untuk mencari data user yang terdaftar<br>![01](/assets/easy5/Selection_006.png?raw=true)<br> hasilnya adalah<br>![01](/assets/easy5/Selection_007.png?raw=true)<br>
   
9.  Selanjutnya yang saya lakukan adalah, membuat wordlist untuk koneksi ke protokol ssh, dimana saya memisahkan masing-masing username dan password untuk dijadikan
wordlist.
       - Username<br>![01](/assets/easy5/Selection_008.png?raw=true)<br>
       - Password<br>![01](/assets/easy5/Selection_009.png?raw=true)<br>
10.  Setelah saya membuat wordlist yang saya lakukan adalah melakukan bruteforce
menggunakan hydra, dan hasilnya adalah<br>
<b>Username : alice <br>
Password : 4lic3</b><br>![01](/assets/easy5/Selection_010.png?raw=true)<br>

11. Setelah saya mendapatkan username dan password, saya melakukan akses ke ssh.<br>![01](/assets/easy5/Selection_011.png?raw=true)<br>

12.  Setelah berhasil login ke ssh saya masuk ke akses shell /bin/sh, dan hasilnya
sebagai berikut.<br>![01](/assets/easy5/Selection_012.png?raw=true)<br>

13. Selanjutnya saya mengetikan “sudo -l” untuk melihat akses apa saja yang bisa
dilakukan dengan user alice<br>![01](/assets/easy5/Selection_013.png?raw=true)<br>
Terdapat /user/bin/php yang dapat diakses oleh user alice, selanjutnya saya
mengetikan “sudo /user/bin/php -a”, untuk mengakses nya, dan hasilnya adalah php
shell interactive<br>![01](/assets/easy5/Selection_014.png?raw=true)<br>

14. Saya mencoba melakukan shell_exec(“ls -la”) apakah berjalan dengan baik atau
tidak, dan ternyata hasil yang didapat sesuai ekspektasi<br>![01](/assets/easy5/Selection_015.png?raw=true)<br>

15. Dan selanjutnya saya exit dari php interactive shell dan mundur hingga mencapai
letak root folder<br>![01](/assets/easy5/Selection_016.png?raw=true)<br>

16.  Saya akses php interactive shell lagi dan menjalankan shell_exec(“ls -la”), dan
hasilnya sama seperti yang diharapakan.<br>![01](/assets/easy5/Selection_017.png?raw=true)<br>

17. Karena untuk mengakses folder root hanya user “root”, saya mengubah permission
folder root dengan mengetikan shell_exec(“chmod 777 root”);<br>![01](/assets/easy5/Selection_018.png?raw=true)<br>

18. Selanjutnya adalah keluar dari php interative shell dan mengetikan “ls -la”<br>![01](/assets/easy5/Selection_019.png?raw=true)<br>

19. Selanjutnya masuk ke dalam folder root, disana akan terdapat flag2.txt, selanjutnya
buka flag2.txt dengan mengetikan “cat flag2.txt” dan hasilnya sebagai berikut<br>![01](/assets/easy5/Selection_020.png?raw=true)<br>


    Flag yang di dapat adalah : securing_something

