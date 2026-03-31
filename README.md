# advpro-tutorial-5

Reflection

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