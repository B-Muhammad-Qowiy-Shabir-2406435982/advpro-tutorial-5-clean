# advpro-tutorial-5

Reflection Publisher-1

1. Dalam teori Observer Pattern, Subscriber biasanya didefinisikan sebagai interface karena memungkinkan adanya berbagai jenis subscriber dengan perilaku yang berbeda-beda. Namun, pada kasus BambangShop ini, kebutuhan sistem masih sederhana, yaitu hanya menyimpan data subscriber berupa URL dan nama, serta menerima notifikasi tanpa variasi perilaku yang kompleks. Oleh karena itu, penggunaan struct saja sudah cukup untuk merepresentasikan subscriber.

Di Rust, interface direpresentasikan sebagai trait, yang biasanya digunakan jika terdapat banyak implementasi berbeda dengan perilaku yang bervariasi. Karena pada kasus ini semua subscriber memiliki perilaku yang sama (hanya menerima notifikasi via HTTP), maka penggunaan trait belum diperlukan. Namun, jika di masa depan terdapat berbagai jenis subscriber (misalnya email, SMS, atau push notification), maka penggunaan trait akan menjadi lebih relevan.

2. Penggunaan Vec (list) sebenarnya cukup jika hanya digunakan untuk menyimpan kumpulan data secara sederhana. Namun, dalam kasus ini terdapat kebutuhan bahwa setiap subscriber memiliki identifier unik (url), dan data juga dikelompokkan berdasarkan product_type.

Jika menggunakan Vec, maka:
- Pencarian dan penghapusan data akan lebih lambat (O(n))
- Tidak ada jaminan unik secara langsung tanpa pengecekan manual

Sedangkan dengan menggunakan DashMap (map/dictionary):
- Data dapat diakses lebih cepat (O(1))
- Key (seperti URL) secara alami bersifat unik
- Lebih mudah untuk melakukan insert, delete, dan lookup

Oleh karena itu, penggunaan DashMap lebih tepat dibandingkan Vec dalam kasus ini karena memberikan efisiensi dan struktur data yang lebih sesuai dengan kebutuhan sistem.

3. Dalam Rust, keamanan terhadap concurrency (thread safety) sangat diperhatikan. Pada kasus ini, data SUBSCRIBERS disimpan sebagai variabel global (singleton), sehingga dapat diakses dari berbagai thread.

Jika hanya menggunakan Singleton tanpa mekanisme thread-safe:
- Berpotensi terjadi race condition
- Data bisa tidak konsisten

Dengan menggunakan DashMap, kita mendapatkan:
- Struktur data yang sudah thread-safe
- Tidak perlu mengatur locking manual seperti pada Mutex
- Performa yang lebih baik dalam lingkungan concurrent

Meskipun konsep Singleton sudah digunakan (melalui lazy_static), DashMap tetap diperlukan untuk memastikan bahwa akses data aman dalam kondisi multi-thread. Jadi, dalam kasus ini, Singleton dan DashMap saling melengkapi, bukan saling menggantikan.

Reflection Publisher-2

1. Dalam pola MVC klasik, Model menangani sekaligus penyimpanan data dan logika bisnis. Namun, hal ini melanggar prinsip Single Responsibility Principle (SRP) dari SOLID, di mana setiap komponen seharusnya hanya memiliki satu alasan untuk berubah.

Dengan memisahkan keduanya:
- Repository hanya bertanggung jawab untuk akses data (CRUD ke database/storage). Jika suatu saat kita ganti database dari PostgreSQL ke MongoDB, kita hanya perlu mengubah      Repository tanpa menyentuh logika bisnis.
- Service hanya bertanggung jawab untuk logika bisnis, seperti validasi, transformasi data, dan orkestrasi antar-repository.

Pemisahan ini membuat kode lebih mudah dipelihara, diuji (unit test), dan dikembangkan karena setiap lapisan memiliki tanggung jawab yang jelas dan tidak saling bergantung secara berlebihan.

2. Jika kita hanya menggunakan Model tanpa Service dan Repository, maka setiap Model (Program, Subscriber, Notification) harus menangani semua logika sendiri.

Interaksi antar model akan menjadi sangat kompleks:
- Program perlu tahu cara menyimpan data sekaligus cara mengirim notifikasi
  ke Subscriber secara langsung.
- Subscriber perlu tahu cara mengambil data dirinya sendiri dari storage
  sekaligus memvalidasi langganannya.
- Notification perlu tahu cara membuat dirinya sendiri, menyimpan dirinya,
  sekaligus tahu siapa saja Subscriber yang harus menerimanya.

Akibatnya, terjadi tight coupling yang tinggi — setiap perubahan kecil di satu Model bisa merembet dan merusak Model lain. Kode menjadi sulit dibaca, sulit diuji, dan sangat rentan bug saat ada penambahan fitur baru.

3. Ya, saya telah menggunakan Postman untuk menguji endpoint yang dibuat di tutorial ini. Postman sangat membantu karena memungkinkan kita mengirim HTTP request (GET, POST, dll.) tanpa perlu membuat frontend terlebih dahulu.

Fitur-fitur Postman yang saya temukan berguna:
- Collections: Menyimpan kumpulan request agar bisa dijalankan ulang kapan saja,
  sangat berguna untuk menguji seluruh endpoint secara konsisten.
- Environment Variables: Menyimpan variabel seperti base URL sehingga mudah
  beralih antara environment development dan production.
- Automated Tests: Bisa menulis script untuk memverifikasi response secara otomatis,
  misalnya mengecek status code atau isi body response.
- History: Menyimpan riwayat request yang pernah dikirim sehingga mudah
  untuk mengulang pengujian sebelumnya.

Untuk Group Project ke depannya, fitur Collections dan Environment Variables sangat relevan karena memudahkan seluruh anggota tim menggunakan konfigurasi
pengujian yang sama tanpa perlu setup ulang dari awal.

Reflection Publisher-3

1. Dalam tutorial ini, kita menggunakan variasi Push model. Hal ini terlihat dari
cara NotificationService secara aktif memanggil method update() pada setiap
Subscriber dan mendorong (push) data Notification langsung ke URL mereka setiap
kali ada event (CREATED, DELETED, PROMOTION), tanpa Subscriber perlu meminta
data terlebih dahulu.

2. Jika kita menggunakan Pull model (kebalikan dari yang dipakai sekarang):

Keuntungan:
- Subscriber hanya mengambil data ketika mereka siap memprosesnya, sehingga
  lebih terkontrol dan tidak terbebani notifikasi yang tidak perlu.
- Publisher menjadi lebih sederhana karena hanya perlu menyediakan data,
  tidak perlu tahu cara menghubungi setiap Subscriber.

Kerugian:
- Subscriber harus aktif melakukan polling secara berkala untuk mengecek
  apakah ada update baru, yang bisa membuang resource jika tidak ada perubahan.
- Notifikasi tidak real-time — ada jeda antara kejadian dan saat Subscriber
  mengetahuinya, tergantung seberapa sering mereka polling.
- Implementasi lebih kompleks di sisi Subscriber karena mereka perlu
  mengelola jadwal polling sendiri.

3. Jika kita tidak menggunakan multi-threading (menghapus thread::spawn), maka
proses notifikasi ke setiap Subscriber akan berjalan secara sequential
(satu per satu). Artinya:

- Program harus menunggu notifikasi ke Subscriber pertama selesai dikirim
  sebelum mengirim ke Subscriber berikutnya.
- Jika ada Subscriber dengan URL yang lambat merespons atau bahkan tidak
  merespons (timeout), seluruh proses akan terhenti dan membuat pengguna
  menunggu sangat lama.
- Performa aplikasi akan menurun drastis seiring bertambahnya jumlah Subscriber
  karena waktu respons bertambah secara linear.
- Dalam kasus ekstrem, satu Subscriber yang bermasalah bisa memblokir
  keseluruhan sistem notifikasi dan bahkan endpoint HTTP yang sedang dipakai
  pengguna saat itu.