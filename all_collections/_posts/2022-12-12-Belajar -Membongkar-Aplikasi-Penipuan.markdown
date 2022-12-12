---
layout: post
title: Belajar Membongkar Aplikasi Penipuan
date: 2022-12-12
categories: ["Pentest","Mobile"]
---

Pada kesempatan kali ini saya akan mencoba menganalisis sebuah aplikasi yang digunakan untuk modus penipuan,  saya mendapatkan aplikasi ini dari teman saya mas [Nikko Enggaliano](https://nikkoenggaliano.my.id) terima kasih telah memberikan saya kesempatan untuk menganalisis aplikasi ini hehehe,

pertama saya melakukan reverse engineer terhadap aplikasi ini untuk mendapatkan class, library serta alur dari aplikasi itu sendiri, hasil dari reverse engineer tersebut saya mendapatkan sususan folder sebagai berikut <br><br> ![1](/assets/basepenipu/satu.png) <br><br>

Selanjutnya saya mendapatkan **AndroidManifest.xml** untuk saya cari permission yang diterapkan dan activity pertama yang dijalankan <br><br>
![2](/assets/basepenipu/2.png?raw=true) <br><br>

hasil analisis dari file AndroidManifest.xml menunjukan bahwa aplikasi akan meminta request permission untuk mendapatkan sms. <br>

```xml
<uses-permission android:name="android.permission.INTERNET"/>
<uses-permission android:name="android.permission.RECEIVE_SMS"/>
<permission android:name="abyssalarmy.smseye.DYNAMIC_RECEIVER_NOT_EXPORTED_PERMISSION" android:protectionLevel="signature"/>
<uses-permission android:name="abyssalarmy.smseye.DYNAMIC_RECEIVER_NOT_EXPORTED_PERMISSION"/>
```  

namun yang cukup menarik disini adalah terdapat sebuah permission bernama abyssalarmy.smseye setelah saya cari di google smseye merupakan sebuah aplikasi android spyware yang dapat di custom yang memungkinkan untuk mendapatkan sebuah sms dari smartphone korban dan selanjutnya di forward ke telegram bot, sebenarnya aplikasi ini terdapat disclaimer “sebagai bahan untuk latihan”, namun tetap saja ada oknum yang menyalahgunakan aplikasi ini, berikut link dari aplikasi tersebut

[SmsEye AbyssalArmy](https://github.com/AbyssalArmy/SmsEye) pelaku juga mengimplementasikan sebuah library bernama Dexter  dapat dilihat di potongan kode berikut

```xml
<activity android:theme="@style/Dexter_Internal_Theme_Transparent" android:name="com.karumi.dexter.DexterActivity"/>
```
setelah saya telusuri di google saya menemukan Dexter adalah sebuah library yang memungkinkan untuk memproses banyak request permission dalam waktu yang bersamaan, berikut link dari library tersebut [Dexter](https://github.com/Karumi/Dexter) <br> setelah itu aplikasi akan mengarah ke sebuah activity yaitu **SmsEyeMainActivity** <br>

```xml
<activity android:theme="@style/Theme_SmsEye" android:label="@string/app_name" android:name="abyssalarmy.smseye.SmsEyeMainActivity" android:exported="true">
<intent-filter>
	<action android:name="android.intent.action.MAIN"/>
	<category android:name="android.intent.category.LAUNCHER"/>
</intent-filter>
</activity>
```
didalam file **SmsEyeMainActivity** terdapat deklarasi sebuah viewmodel yaitu SmsEyeViewModel, namun jika diperhatikan SmsEyeViewModel tidak di proses di activity ini <br>
```java
public final class SmsEyeMainActivity extends ComponentActivity {
    public static final int $stable = LiveLiterals$SmsEyeMainActivityKt.INSTANCE.m0Int$classSmsEyeMainActivity();
    private SmsEyeViewModel smsEyeViewModel;

    @Override // androidx.activity.ComponentActivity, androidx.core.app.ComponentActivity, android.app.Activity
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        ComponentActivityKt.setContent$default(this, null, ComposableLambdaKt.composableLambdaInstance(578910236, true, new SmsEyeMainActivity$onCreate$1(this)), 1, null);
    }
}
```
namun terdapat sebuah angka **578910236** yang dimana setelah saya telusuri lagi, mengarah ke sebuah class yang terkenkripsi, dan benar saja didalam class ini terdapat sebuah function yang memproses SmsEyeViewModel dan navcontroller, yang dimana ketika proses telah memenuhi syarat, akan langsung diarahkan menuju **SmsEyeMainActivity.kt** class ini ditulis menggunakan kotlin, itulah kenapa class ini bisa terenkripsi. <br><br> ![03](/assets/basepenipu/3.png?raw=true) <br><br>

selanjutnya saya menganalisis SmsEyeViewModel ketika dibuka terdapat potongan kode sebagai berikut <br><br> ![04](/assets/basepenipu/4.png?raw=true) <br><br>

Jika diperhatikan dengan seksama dibagian potongan kode berikut, terdapat sebuah function yang tereknripsi salah satunya **m6x95df507e()** dan semua function itu mengarah ke sebuah class yang lagi-lagi juga terenkripsi. <br><br> ![05](/assets/basepenipu/5.png?raw=true) <br><br>

berikut potongan dari class tersebut <br><br> ![06](/assets/basepenipu/6.png?raw=true) <br><br>

dimana setiap function memproses variable yang digunakan untuk proses sharedandpreference <br><br>  ![07](/assets/basepenipu/7.png?raw=true)  <br><br>

dan setelah saya telusuri lebih dalam lagi, saya menemukan sebuah class SmsEyeNavigationKt, dimana pada class ini memproses tampilan apakah webviewRoute atau welcomeRoute sesuai dengan kondisi dari smsEyeViewModel.getNavController() yang terdapat pada viewmodel, yang selanjutnya dinavigasi oleh NavHost yang mana merupakan salah satu dari jetpack component, untuk sebuah spyware menurut saya cukup update dengan menggunakan android jetpack <br><br> ![08](/assets/basepenipu/8.png?raw=true) <br><br>

Selanjutnya saya akan melakukan analisis terhadap file **SmsEyeWebviewKt**

**SmsEyeWebviewKt,** didalam class ini tentu saja memproses untuk menampilkan webview dari sebuah url <br><br> ![09](/assets/basepenipu/9.png?raw=true) <br><br>

nah web apa yang sebenarnya ditampilkan di webview ini, disini saya berhasil menemukan web yang ditampilkan di dalam webview dari potongan kode berikut <br><br> ![10](/assets/basepenipu/10.png?raw=true) <br><br>

terdapat sebuah kode **SmsEyeTools.Companion.smsEyeData(context).getUrl()** bisa diambil kesimpulan bahwa url yang akan ditampilkan terdapat pada class SmsEyeTools di function getUrl(), selanjutnya saya mencoba menelusuri url apa yang disematkan. hasil penelusuran saya di class SmsEyeTools terdapat fungsi berikut <br><br> ![11](/assets/basepenipu/11.png?raw=true)  <br><br>

fungsi tersebut memiliki hasil return dari variable url, dan ketika saya menganalisis lebih kanjut saya menemukan kode seperti berikut <br><br> ![12](/assets/basepenipu/12.png?raw=true)  <br><br>

diketahui bahwa url disimpan di dalam sebuah file txt bernama url.txt yang terdapat di folder assets, selanjutnya saya membuka folder asset yang terdapat di resource <br><br> ![13](/assets/basepenipu/13.png?raw=true)  <br><br>

dan ketika saya buka, ternyata url tersebut berisi website salah satu jasa ekspedisi yang cukup terkenal di indonesia <br><br> ![14](/assets/basepenipu/14.png?raw=true)  <br><br>

setelah selesai dengan **SmsEyeWebviewKt**, saya menemukan class **SmsEyeSmsListener** dilihat dari potongan kode berikut, saya berspekulasi bahwa pada class **SmsEyeSmsListener** akan selalu merekam pesan yang masuk dan kemudian di forward ke telegram bot <br><br> ![15](/assets/basepenipu/15.png?raw=true)  <br><br>

nah kenapa saya berspekulasi seperti itu, saya melihat sebuah kode berikut <br><br> ![16](/assets/basepenipu/16.png?raw=true) <br><br>

dimana sebuah variable msgBody di parsing ke sebuah function yang berada di class SmsEyeNetwork, dan ketika saya buka SmsEyeNetwork, benar saja di dalam class ini terdapat sebuah proses dari telegram bot, dan terdapat sebuah function sendTextMessage yang dimana berfungsi untuk memparsing pesan yang berhasil di rekam untuk selanjutnya di forward ke telegram bot berikut potongan kodenya <br><br> ![17](/assets/basepenipu/17.png?raw=true) <br><br>

Telegram bot membutuhkan sebuah Id dan Token telegram, selanjutnya saya mencari lokasi id dan token nya, berdasarkan kode dibawah ini <br><br>  ![18](/assets/basepenipu/18.png?raw=true)  <br><br>

id dan token, disimpan juga di **SmsEyeTools** dan benar, seluruh id dan token disimpan di dalam folder assets berdampingan dengan url sebelumnya <br><br>  ![19](/assets/basepenipu/19_1.png?raw=true)  <br>

dan ketika dibuka, isinya seperti berikut <br>
**id.txt** <br><br>  ![20_1](/assets/basepenipu/20_1.png?raw=true)  <br><br> **token.txt** <br><br>  ![20_2](/assets/basepenipu/20_2.png?raw=true)  <br>

dan ketika saya coba token tersebut melalui api telegram hasilnya seperti ini <br><br>  ![21](/assets/basepenipu/21.png?raw=true)  <br> yap ternyata token masih aktif XD, selanjutnya saya telusuri melalui aplikasi telegram dengan username tersebut, hasilnya sebagai berikut <br><br>  ![22](/assets/basepenipu/22.png?raw=true)  <br> alhasil username tersebut terdaftar di telegram dan sesuai dengan response yang diterima saat saya coba mengguanakan token, bahwa username ini adalah sebuah bot. <br><br> Sekian tulisan tentang bagaimana cara saya melakukan reverse engineering terhadap aplikasi yang digunakan untuk modus penipuan, bila masih ada salah dan kurang nya, saya mohon maaf yang sebesar-besarnya, karena disini saya juga belajar menerapkan sesuatu yang sangat jarang saya lakukan, terima kasih.