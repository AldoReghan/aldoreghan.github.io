---
layout: post
title: Notifikasi OneSignal menggunakan Flutter & ExpressJs
date: 2022-12-20
categories: ["ExpressJS","Mobile","Flutter"]
---

Assalamualaikum Wr Wb, halo pembaca pada tulisan kali ini saya akan membagikan hasil belajar saya bagaimana cara mengimplementasikan notifikasi di Flutter + ExpressJS dengan menggunakan OneSignal.
<br><br>

Pertama yang harus dilakukan adalah membuat firebase project karena kita membutuhkan **Server key** dan **Sender ID** dari firebase, disini saya sudah membuat firebase project yang saya beri nama **onesignaltest** <br> 

![1](/assets/onesignalpost/1.png) <br>

Setelah itu klik setting, setelah itu pilih project settings <br>

![2](/assets/onesignalpost/2.png) <br>

Setelah itu pilih "Cloud Messaging" <br>

![3](/assets/onesignalpost/3.png) <br>
   
Selanjutnya setelah kita membuat project di firebase, kita akan mensetup untuk notifikasinya, pastikan sudah memiliki akun onesignal, untuk melakukan registrasi silahkan klik [Website OneSignal](https://onesignal.com/) setelah mempunyai akun silahkan login, selanjutnya ikuti langkah-langkah berikut : <br>

Klik "New App\Website" <br>

![4](/assets/onesignalpost/4.png) <br>

Selanjutnya beri nama app nya, dan pilih Google Android (FCM), karena kita mengirimkan ke device android <br>

![5](/assets/onesignalpost/5.png) <br>

Selanjutnya isi form sesuai dengan Server Key dan Sender ID di project firebase yang telah dibuat <br>

![6](/assets/onesignalpost/6.png) <br>

Setelah itu pilih Flutter karena disini saya akan menggunakan Flutter <br>

![7](/assets/onesignalpost/7.png) <br>

setelah semuanya sudah selesai klik done  <br>

![8](/assets/onesignalpost/8.png) <br>

Setelah selesai berurusan dengan onesignal, selanjutnya kita akan membuat backend nya menggunakan ExpressJS, disini saya sudah menyiapkan project ExpressJS bernama **onesignall**, berikut struktur folder project saya <br> 

![9](/assets/onesignalpost/9.png) <br>

Selanjutnya didalam file **index.js** berfungsi untuk mengatur routing serta port dari server, berikut source code nya <br>

```js

//index.js

const express = require("express")
const app = express()

app.use(express.json())

app.use("/api", require("./routes/app.routes"))

app.listen(process.env.port || 4000, function(){
    console.log("Hello world")
})
```
   
Setelah mengatur routing dan port dari server, selanjutnya kita membuat file config untuk onesignal nya, dimana pada file ini akan mengatur **APP_ID** dan **API_KEY**, berikut source nya <br>

```js

//app.config.js

const ONE_SIGNAL_CONFIG = {
    APP_ID : "xxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxxxx",
    APP_KEY : "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
}
module.exports = { ONE_SIGNAL_CONFIG }
```

Selanjutnya kita akan membuat service untuk notifikasi nya, di dalam file service ini kita akan mengatur header, options dan memberikan callback <br>

```js

//push-notification.service.js

const {ONE_SIGNAL_CONFIG} = require('../config/app.config');

async function SendNotification(data, callback) {
    var headers = {
        "Content-Type": "application/json; charset=utf-8",
        "Authorization" : "Basic " + ONE_SIGNAL_CONFIG.APP_KEY
    }

    var options = {
        host : "onesignal.com",
        port : 443,
        path : "/api/v1/notifications",
        method : "POST",
        headers : headers
    }

    var https = require('https');
    var req = https.request(options, function(res) {
        res.on("data", function(data){
            console.log(JSON.parse(data));
            return callback(null, JSON.parse(data));
        })
    })

    req.on("error", function(e){
        return callback({
            message: e
        })
    })

    req.write(JSON.stringify(data));
    req.end();
}

module.exports = { SendNotification }
```

Sudah mengatur service, di tahap ini kita akan membuat controller, untuk memproses notifikasi seperti pesan yang akan dikirim, icon notifikasi,dll <br>

```js

//push-notification.controller.js

const { ONE_SIGNAL_CONFIG } = require('../config/app.config');
const pushNotificationService = require('../services/push-notifcation.service');

exports.SendNotificationToDevice = (req, res, next) => {
    const msg = req.body.msg;
    var message = {
        app_id: ONE_SIGNAL_CONFIG.APP_ID,
        contents: { "en": msg },
        included_segments: ["All"],
        content_available: true,
        headings : { "en" : "Eula Lawrence" }, //silahkan sesuaikan sesuka hadi XD
        small_icon: "ic_notification_icon",
        data : {
            PushTitle: "CUSTOM NOTIFICATION",
        }
    }

    pushNotificationService.SendNotification(message, (error, results) => {
        if (error) {
            return next(error)
        }
        return res.status(200).json({
            message: "Notification sent successfully",
            data: results
        })
    })
}
```

Selesai dengan urusan controller, selanjutnya kita masuk ke tahap akhir dari backend yaitu, membuat route seperti code dibawah berikut <br>

```js

//app.route.js

const pushNotificationController = require("../controller/push-notification.controller");

const express = require("express");
const router = express.Router();

router.post("/SendNotificationToDevice", pushNotificationController.SendNotificationToDevice);

module.exports = router;
```

Selanjutnya kita melakukan pengetesan apakah API berjalan dengan baik atau tidak, disini saya menggunakan thunder untuk melakukan pengetesan <br> ![10](/assets/onesignalpost/10.png) <br>
dan ya, API berjalan dengan baik, selanjutnya kita akan membuat project untuk mobile nya <br>

Disini saya sudah menyiapkan project flutter yang saya beri nama **flutter_onesignal**, setelah membuat project, selanjutnya kita akan menambahkan library one signal, ketikan perintah berikut <br>

```dart
flutter pub add onesignal_flutter
```

Setelah itu kita import ke **main.dart**, dan di **void main()** menambahkan kode berikut <br>

```dart
import 'package:onesignal_flutter/onesignal_flutter.dart';

void main() {
  WidgetsFlutterBinding.ensureInitialized();

  //isi sesuai dengan APP_ID di onesignal
  OneSignal.shared.setAppId("xxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxx");
  OneSignal.shared.promptUserForPushNotificationPermission().then((accepted) {
      print("Accepted permission: $accepted");
  });
  runApp(const MyApp());
}
```

Setelah itu kita membuat fungsi **initState()** yang nantinya akan menjalankan fungsi handling notifikasi, berikut source code nya <br>

```dart
@override
  void initState() {
    // TODO: implement initState
    super.initState();
    OneSignal.shared.setNotificationWillShowInForegroundHandler((OSNotificationReceivedEvent event) {
      // Will be called whenever a notification is received in foreground
      // Display Notification, pass null param for not displaying the notification
            event.complete(event.notification);                                 
    });
  }
```

Sebenarnya banyak sekali handling notifikasi untuk onesignal, silahkan melihat di dokumentasi di website resmi onesignal. <br>

Selanjutnya kita jalankan project flutter kita, setelah itu, kita post notifikasi menggunakan API seperti sebelumnya, dan lihat di smartphone hasilnya seperti berikut <br>

![12](/assets/onesignalpost/12.jpg) <br>

Sekian tulisan saya tentang bagaimana implementasi onesignal menggunakan Flutter dan ExpressJS, mohon maaf jika masih membingungkan bagi para pembaca, karena disini saya juga masih belajar menulis yang baik dan benar, Terima Kasih yang telah membaca, sampai jumpa lagi.