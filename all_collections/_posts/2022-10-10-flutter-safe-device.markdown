---
layout: post
title: Flutter Deteksi Root dan Emulator
date: 2022-10-09
categories: ["Flutter"]
---

Halo pada tulisan kali ini saya ingin sharing bagaimana aplikasi yang kita buat menggunakan flutter dapat mendeteksi bahwa perangkan tersebut telah di root atau aplikasi kita sedang dijalankan di sebuah emulator, di dalam tulisan ini saya menggunakan sebuah package **safe_device** yang dapat dilihat di web berikut [pub.dev](https://pub.dev/packages/safe_device) <br>

1. menambahkan package safe_device ke dalam project kita, disini saya menggunakan terminal, untuk mengetahui bagaimana cara menambahkan nya dapat dilihat di link yang sudah saya cantumkan. <br><br>
   ```dart
   flutter pub add safe_device
   ```
   <br>
2. Mengimport safe_device pada project kita, disini saya membuat file bernama **AuthWrapper.dart** karena di dalam file ini pengecekan dilakukan.<br><br>
   ```dart
   import 'package:safe_device/safe_device.dart';
   ```
   <br>
3. Membuat function bernama **checkDevice()** untuk melakukan pengecekan apakah perangkat telah diroot atau dijalankan di emulator.<br><br>
   ```dart
    checkDevice() async {
        bool isRealDevice = await SafeDevice.isRealDevice;
        bool isJailBroken = await SafeDevice.isJailBroken;

        if (isRealDevice == true) {
        checkLogin();
        }
        if (isRealDevice == false) {
        // ignore: use_build_context_synchronously
        showExitPopup(context, "Perangkat tidak dapat berjalan di Emulator");
        }
        if (isJailBroken == true) {
        // ignore: use_build_context_synchronously
        showExitPopup(context, "Perangkat tidak dapat berjalan di Root");
        }
    }
    ```
    <br>

    **checkDevice()** yang telah dibuat selanjutnya di masukkan kedalam **iniState()** karena **checkDevice()** akan dijalankan terlebih dahulu <br><br>
    
    ```dart
    @override
    void initState() {
        super.initState();
        checkDevice();
    }
    ```
   <br>
4. Selanjutnya mengimport file **AuthWrapper.dart** ke **main.dart** untuk dijalankan<br><br>
   ```dart
   void main() {
     runApp(MultiProvider(
         providers: [
             ChangeNotifierProvider(create: (_) => UserServices())],
         child: const MyApp()));
     }

     class MyApp extends StatelessWidget {
     const MyApp({Key? key}) : super(key: key);

     // This widget is the root of your application.
     @override
     Widget build(BuildContext context) {
      return MaterialApp(
          title: 'SecureApp',
          debugShowCheckedModeBanner: false,
          theme: ThemeData(
            primarySwatch: Colors.blue,
          ),
          home: const AuthWrapper());
     }
    }
   ```