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

2. Melakukan view page source, untuk melihat apakah tedapat suatu clue di dalam source nya, dan hasilnya seperti berikut terdapat sebuah komentar <b>“Maybe you can search how to use x-forwarded-for”</b>, komentar ini merupakan sebuah instruksi untuk menggunakan x-forwarder-for.
   <br>
3. Melakukan manipulasi ip address menggunakan x-forwarder-for menjadi 127.0.0.1, agar website seolah2 diakses melalui lokal, dan selanjutnya refresh halaman webnya

    Note : pastikan saat mensetting ip address harus di tab website virtual machine

4. Setelah di refresh terdapat sebuah tampilan halaman index yang sebenarnya
5. Hal yang pertama yang saya lakukan adalah melakukan register di halaman tersebut, dan selanjutnya login menggunakan akun yang telah saya registrasi, berikut tampilan dari website setelah register
6. Selanjutnya yang saya lakukan adalah melakukan debugging dengan cara, mengklik masing-masing menu yang terdapat di website tersebut.
   - Dashboard
   - Profile
  
7. Di page profile terdapat url yang menunjukan “user_id=12” dimana 12 merupakan akun saya, saat diubah menjadi 1, akan muncul data users sesuai id, dengan begitu, url user_id dapat di eksplorasi untuk mendapatkan akun-akun lainnya
   
8. Selanjutnya saya menggunakan burpsuite untuk mencari data user yang terdaftar
   
9.  Selanjutnya yang saya lakukan adalah, membuat wordlist untuk koneksi ke protokol
ssh, dimana saya memisahkan masing username dan password untuk dijadikan
wordlist.

10. Setelah saya membuat wordlist yang saya lakukan adalah melakukan bruteforce
menggunakan hydra, dan hasilnya adalah<br>
<b>Username : alice <br>
Password : 4lic3</b>

11. Setelah saya mendapatkan username dan password, saya melakukan akses ke ssh.

12. Setelah berhasil login ke ssh saya masuk ke akses shell /bin/sh, dan hasilnya
sebagai berikut.

13. Selanjutnya saya mengetikan “sudo -l” untuk melihat akses apa saja yang bisa
dilakukan dengan user alice

14. . Saya mencoba melakukan shell_exec(“ls -la”) apakah berjalan dengan baik atau
tidak, dan ternyata hasil yang didapat sesuai ekspektasi

15. Dan selanjutnya saya exit dari php interactive shell dan mundur hingga mencapai
letak root folder

16.  Saya akses php interactive shell lagi dan menjalankan shell_exec(“ls -la”), dan
hasilnya sama seperti yang diharapakan.

17. Karena untuk mengakses folder root hanya user “root”, saya mengubah permission
folder root dengan mengetikan shell_exec(“chmod 777 root”);

18. Selanjutnya adalah keluar dari php interative shell dan mengetikan “ls -la”

19. Selanjutnya masuk ke dalam folder root, disana akan terdapat flag2.txt, selanjutnya
buka flag2.txt dengan mengetikan “cat flag2.txt” dan hasilnya sebagai berikut


    Flag yang di dapat adalah : securing_something

