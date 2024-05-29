> **Nama : Raditia Riandi** <br> **NIM : 121450105** <br> **Kelas : RB**

**Tiga Cara Menyimpan dan Mengakses Banyak Gambar dengan Python
![header]([https://example.com/logo.png](https://files.realpython.com/media/cifar_10.e77ef0cd86df.png))
## Gambaran Umum

1. **Disk**
   Metode ini adalah yang paling umum dan sederhana, di mana setiap file gambar disimpan secara terpisah di sistem file komputer. Meskipun mudah diimplementasikan, metode ini bisa menjadi lambat saat menangani dataset gambar yang sangat besar.

2. **LMDB (Lightning Memory-Mapped Database)**
   LMDB adalah basis data berbasis kunci-nilai yang dioptimalkan untuk akses cepat menggunakan struktur B-tree. Ini memungkinkan penyimpanan dan pengambilan gambar secara efisien, terutama saat menangani dataset gambar yang besar.

3. **HDF5 (Hierarchical Data Format version 5)**
   Metode ini memungkinkan penyimpanan banyak gambar dalam satu file dengan struktur hierarki. Ini dapat menghemat ruang disk dan mempercepat proses pembacaan dan penulisan gambar.

**Dalam tutorial ini, Anda akan mempelajari tentang:**

1. Menyimpan gambar di disk sebagai file `.png`
2. Menyimpan gambar dalam basis data memori-pemetaan kilat (LMDB)
3. Menyimpan gambar dalam format data hierarki (HDF5)

**Anda juga akan menjelajahi hal berikut:**

1. Mengapa metode penyimpanan alternatif patut dipertimbangkan
2. Apa perbedaan performa saat Anda membaca dan menulis gambar tunggal
3. Apa perbedaan performa saat Anda membaca dan menulis banyak gambar
4. Bagaimana perbandingan ketiga metode dalam hal penggunaan disk

Dalam artikel ini, kita akan menjelajahi implementasi tiga metode penyimpanan gambar menggunakan dataset CIFAR-10 (Canadian Institute for Advanced Research). Kita akan membandingkan kinerja, penggunaan disk, dan kemudahan penggunaan masing-masing metode, sehingga Anda dapat memilih metode yang paling sesuai untuk proyek Anda. Dataset CIFAR-10 terdiri dari 60.000 gambar berwarna dalam 10 kelas yang berbeda, dengan setiap gambar berukuran 32x32 piksel. Kita akan menggunakan CIFAR-10 sebagai contoh untuk melihat bagaimana setiap metode bekerja dalam praktek.

Sebelum memulai, pastikan Anda memiliki dasar yang kuat dalam Python dan pemahaman dasar tentang gambar. Ingat bahwa gambar terdiri dari array angka multi-dimensi yang merepresentasikan nilai piksel. Misalnya, gambar berwarna 32x32 memiliki dimensi (32, 32, 3), di mana 3 merepresentasikan tiga kanal warna (merah, hijau, dan biru). Selain itu, penting juga untuk memahami konsep memori relatif, seperti perbedaan antara 10MB dan 10GB. Memahami bagaimana memori digunakan dan dioptimalkan adalah kunci dalam memilih metode penyimpanan yang paling efisien.

Untuk eksperimen ini, kita akan membandingkan tiga metode penyimpanan gambar: penyimpanan langsung di disk, penyimpanan menggunakan LMDB (Lightning Memory-Mapped Database), dan penyimpanan menggunakan HDF5 (Hierarchical Data Format version 5). Kita akan mengukur waktu yang dibutuhkan untuk membaca dan menulis file, serta seberapa banyak memori disk yang digunakan oleh setiap metode. Implementasi ini akan membantu Anda memahami kelebihan dan kekurangan masing-masing metode, dan memberikan panduan praktis dalam memilih metode yang paling sesuai untuk kebutuhan penyimpanan gambar Anda.
Anda dapat mengunduh CIFAR-10 dalam format Python dari tautan berikut: [The CIFAR-10 dataset](https://www.cs.toronto.edu/~kriz/cifar.html). <br>

## Installasi

- **Disk** <br>
  Anda perlu menyiapkan lingkungan pengembangan Anda untuk menggunakan metode menyimpan dan mengakses gambar dari disk yang merupakan metode default. Artikel ini akan mengasumsikan bahwa Anda telah menginstal Python versi 3.x di sistem Anda, dan akan menggunakan library `Pillow` untuk memanipulasi gambar. <br>
  `$ pip install Pillow` <br>
  `$ conda install -c conda-forge pillow` <br>
  Catatan: PIL adalah versi asli dari Python Imaging Library, yang tidak lagi dipertahankan dan tidak kompatibel dengan Python 3.x. Jika Anda sudah menginstal sebelumnya PIL, pastikan untuk menghapus instalannya sebelum menginstal Pillow, karena keduanya tidak dapat ada bersamaan
- **LMDB** <br>
  `pip install lmdb` <br>
  `$ conda install -c conda-forge python-lmdb`
- **HDF5** <br>
  `$ pip install h5py` <br>
  `$ conda install -c conda-forge h5py`

## Menyimpan Satu Gambar

Mari kita langsung melihat perbandingan kuantitatif dari tugas-tugas dasar yang penting: berapa lama waktu yang diperlukan untuk membaca dan menulis file, serta berapa banyak memori disk yang akan digunakan. Hal ini juga akan berfungsi sebagai pengantar dasar tentang cara kerja metode-metode tersebut, dengan contoh kode tentang cara penggunaannya. Ketika saya menyebut "file", umumnya saya merujuk pada banyak file. Namun, penting untuk membuat perbedaan karena beberapa metode mungkin dioptimalkan untuk operasi dan jumlah file yang berbeda.

Untuk keperluan eksperimen, kita dapat membandingkan kinerja antara berbagai jumlah file, dengan faktor 10 dari satu gambar hingga 100.000 gambar. Karena lima batch CIFAR-10 kita berjumlah 50.000 gambar, kita dapat menggunakan setiap gambar dua kali untuk mendapatkan 100.000 gambar. Eksperimen ini akan membantu kita memahami bagaimana setiap metode penyimpanan menangani berbagai skala file dan seberapa efisien mereka dalam operasi baca-tulis serta penggunaan memori disk.

Untuk mempersiapkan eksperimen, kita akan membuat folder untuk setiap metode penyimpanan yang akan diuji. Setiap folder akan berisi semua file database atau gambar yang diperlukan untuk metode tersebut. Kita akan menyimpan path ke direktori-direktori ini dalam variabel untuk memudahkan akses selama eksperimen. Berikut adalah contoh kode untuk menyiapkan struktur folder dan path untuk eksperimen:
```
from pathlib import Path

disk_dir = Path("D:/6th/TBD/tugas/week9/data/disk")
lmdb_dir = Path("D:/6th/TBD/tugas/week9/data/lmdb")
hdf5_dir = Path("D:/6th/TBD/tugas/week9/data/hdf5")
```

`Path` tidak secara otomatis membuat folder untuk Anda kecuali Anda secara khusus memintanya untuk:

```
disk_dir.mkdir(parents=True, exist_ok=True)
lmdb_dir.mkdir(parents=True, exist_ok=True)
hdf5_dir.mkdir(parents=True, exist_ok=True)
```

### **Menyimpan ke Disk**

Input untuk eksperimen ini adalah satu gambar `image`, saat ini di memori sebagai array NumPy. Anda ingin menyimpannya terlebih dahulu ke disk sebagai gambar `.png`, dan memberi nama menggunakan ID gambar unik `image_id`. Ini dapat dilakukan menggunakan paket `Pillow` yang Anda instal sebelumnya:

```
from PIL import Image
import csv

def store_single_disk(image, image_id, label):
    """ Stores a single image as a .png file on disk.
        Parameters:
        ---------------
        image       image array, (32, 32, 3) to be stored
        image_id    integer unique ID for image
        label       image label
    """
    Image.fromarray(image).save(disk_dir / f"{image_id}.png")

    with open(disk_dir / f"{image_id}.csv", "wt") as csvfile:
        writer = csv.writer(
            csvfile, delimiter=" ", quotechar="|", quoting=csv.QUOTE_MINIMAL
        )
        writer.writerow([label])
```

Ketika Anda menyimpan gambar, penting juga untuk mempertimbangkan metadata yang melekat pada gambar, seperti label gambar dalam dataset contoh kita. Ada beberapa opsi untuk menyimpan metadata saat menyimpan gambar ke disk. Salah satu solusi adalah mengenkode label ke dalam nama file gambar. Metode ini memiliki keuntungan karena tidak memerlukan file tambahan untuk menyimpan metadata, sehingga memudahkan pengelolaan file gambar secara sederhana.

Namun, mengenkode label ke dalam nama file gambar juga memiliki kerugian signifikan. Metode ini memaksa Anda untuk berurusan dengan semua file gambar setiap kali Anda perlu melakukan sesuatu dengan label. Ini bisa menjadi sangat tidak efisien, terutama saat bekerja dengan dataset besar, karena Anda harus memproses atau mengakses semua file gambar meskipun Anda hanya ingin memanipulasi atau menganalisis label.

Sebagai alternatif yang lebih fleksibel, menyimpan label dalam file terpisah memungkinkan Anda bermain-main dengan label tanpa harus memuat gambar. Misalnya, Anda dapat menyimpan label dalam file `.csv` terpisah. Ini memungkinkan Anda untuk dengan mudah memanipulasi atau menganalisis label secara terpisah dari gambar, yang dapat menghemat waktu dan sumber daya. Dalam eksperimen ini, menyimpan label dalam file `.csv` terpisah memungkinkan Anda untuk fokus pada label saja ketika diperlukan, tanpa harus menangani gambar secara langsung setiap saat.

Sekarang mari kita lanjutkan ke tugas yang sama persis dengan LMDB.

### **Menyimpan ke LMDB**

Pertama-tama, LMDB adalah sistem penyimpanan key-value di mana setiap entri disimpan sebagai _array byte_. Dalam konteks penggunaan kita, kunci akan menjadi pengenal unik untuk setiap gambar, dan nilai akan menjadi gambar itu sendiri. LMDB menyimpan kedua elemen ini sebagai string. Oleh karena itu, penggunaan umum dalam LMDB adalah menserialkan nilai gambar menjadi string sebelum menyimpannya, dan mendeserialkannya kembali saat membaca data dari database. Hal ini memastikan bahwa data gambar dapat disimpan dan diakses dengan efisien.

Untuk melakukan serialisasi, Anda dapat menggunakan modul `pickle` di Python. `Pickle` memungkinkan serialisasi objek Python, termasuk gambar dan metadata yang terkait. Dengan menserialkan gambar beserta metadata menjadi string, Anda dapat menyimpan semua informasi yang relevan dalam satu entri database. Ini sangat berguna karena menghindari kebutuhan untuk melampirkan metadata secara terpisah setelah memuat dataset dari disk, sehingga menyederhanakan proses pengelolaan data.

Menyertakan metadata dalam database LMDB memberikan keuntungan tambahan dalam hal efisiensi dan kemudahan akses. Saat Anda membaca dataset dari database, Anda akan mendapatkan data gambar lengkap dengan metadatanya, menghemat waktu dan usaha dalam proses pemrosesan data. Ini menjadikan LMDB solusi yang kuat dan fleksibel untuk penyimpanan dataset gambar yang besar dan kompleks, dengan manfaat tambahan dari struktur penyimpanan yang terorganisir dan konsisten.
Anda dapat membuat kelas Python dasar untuk gambar dan metadatanya:

```
class CIFAR_Image:
    def __init__(self, image, label):
        # Dimensions of image for reconstruction - not really necessary
        # for this dataset, but some datasets may include images of
        # varying sizes
        self.channels = image.shape[2]
        self.size = image.shape[:2]

        self.image = image.tobytes()
        self.label = label

    def get_image(self):
        """ Returns the image as a numpy array. """
        image = np.frombuffer(self.image, dtype=np.uint8)
        return image.reshape(*self.size, self.channels)
```

LMDB (Lightning Memory-Mapped Database) menggunakan pemetaan memori, yang berarti database baru perlu mengetahui berapa banyak memori yang diharapkan akan digunakan. Ini dilakukan dengan menentukan variabel `map_size`. Dalam kasus sederhana, menentukan `map_size` bisa relatif mudah, tetapi dalam kasus yang lebih kompleks, perkiraan ini bisa menjadi sangat merepotkan. Menentukan ukuran peta memori yang tepat adalah penting untuk menghindari kesalahan alokasi memori dan memastikan performa yang optimal.

Operasi baca dan tulis di LMDB dilakukan dalam bentuk transaksi. Transaksi ini mirip dengan transaksi pada database tradisional, yang terdiri dari sekelompok operasi yang dilakukan pada database. Dengan menggunakan transaksi, LMDB memastikan konsistensi data dan memudahkan pengelolaan operasi baca-tulis yang atomik dan konsisten. Meskipun ini menambah lapisan kompleksitas, transaksi memungkinkan kontrol yang lebih baik atas operasi database dan meningkatkan keandalan serta integritas data.

Bagi pengguna yang terbiasa dengan metode penyimpanan berbasis disk yang lebih sederhana, konsep pemetaan memori dan transaksi mungkin terlihat lebih rumit pada awalnya. Namun, dengan pemahaman dan implementasi yang tepat, LMDB menawarkan efisiensi yang lebih tinggi dan performa yang lebih baik untuk pengelolaan dataset gambar yang besar. Melanjutkan untuk memahami dan mengaplikasikan konsep-konsep ini akan memberikan keuntungan signifikan dalam manajemen data skala besar, seperti yang akan dibahas lebih lanjut di bagian selanjutnya.

Dengan tiga poin tersebut dalam pikiran, mari lihat kode untuk menyimpan satu gambar ke LMDB:

```
import lmdb
import pickle

def store_single_lmdb(image, image_id, label):
    """ Stores a single image to a LMDB.
        Parameters:
        ---------------
        image       image array, (32, 32, 3) to be stored
        image_id    integer unique ID for image
        label       image label
    """
    map_size = image.nbytes * 10

    # Create a new LMDB environment
    env = lmdb.open(str(lmdb_dir / f"single_lmdb"), map_size=map_size)

    # Start a new write transaction
    with env.begin(write=True) as txn:
        # All key-value pairs need to be strings
        value = CIFAR_Image(image, label)
        key = f"{image_id:08}"
        txn.put(key.encode("ascii"), pickle.dumps(value))
    env.close()
```

> Catatan:

Menghitung jumlah byte yang tepat yang akan diambil setiap pasangan _key-value_ dalam dataset gambar merupakan ide yang baik untuk memaksimalkan efisiensi penyimpanan dan akses data. Dengan dataset gambar berukuran bervariasi, perkiraan ini menjadi penting untuk mengatur kapasitas penyimpanan dan performa akses data. Meskipun ukuran gambar dapat bervariasi, menggunakan metode seperti `sys.getsizeof()` dapat memberikan perkiraan yang masuk akal tentang ukuran objek. Namun, perlu diingat bahwa `sys.getsizeof()` mengembalikan ukuran definisi kelas, bukan ukuran sebenarnya dari objek yang diinstansiasi.
`sys.getsizeof()` hanya mengembalikan ukuran objek pada permukaan, yaitu ukuran objek `CIFAR_Image` sebagai contoh akan mengembalikan nilai 1056 byte, yang hanya mencakup definisi kelas tersebut. Fungsi ini tidak mampu menghitung item bersarang, daftar, atau objek yang mengandung referensi ke objek lain. Oleh karena itu, hasil dari `sys.getsizeof()` mungkin tidak mencerminkan ukuran sebenarnya dari objek yang kompleks atau bersarang, yang dapat menyebabkan perkiraan yang tidak akurat dan potensi kekurangan dalam perencanaan kapasitas penyimpanan.
Untuk mengatasi keterbatasan ini, Anda dapat menggunakan library seperti pympler yang dirancang untuk menghitung ukuran objek Python dengan lebih akurat. Pympler memungkinkan Anda menentukan ukuran pasti dari sebuah objek, termasuk item bersarang dan referensi ke objek lain. Dengan menggunakan pympler, Anda dapat menghemat kalkulasi dan mendapatkan gambaran yang lebih realistis tentang kebutuhan penyimpanan dataset gambar Anda. Ini sangat membantu dalam membuat keputusan yang lebih baik terkait alokasi sumber daya dan optimisasi penyimpanan data.
Sekarang Anda siap untuk menyimpan gambar ke LMDB. Terakhir, mari lihat metode terakhir, HDF5.

### **Menyimpan dengan HDF5**

Ingat bahwa sebuah file HDF5 dapat berisi lebih dari satu dataset. Dalam kasus yang cukup sederhana ini, Anda dapat membuat dua dataset, satu untuk gambar, dan satu untuk metadatanya:

```
import h5py

def store_single_hdf5(image, image_id, label):
    """ Stores a single image to an HDF5 file.
        Parameters:
        ---------------
        image       image array, (32, 32, 3) to be stored
        image_id    integer unique ID for image
        label       image label
    """
    # Create a new HDF5 file
    file = h5py.File(hdf5_dir / f"{image_id}.h5", "w")

    # Create a dataset in the file
    dataset = file.create_dataset(
        "image", np.shape(image), h5py.h5t.STD_U8BE, data=image
    )
    meta_set = file.create_dataset(
        "meta", np.shape(label), h5py.h5t.STD_U8BE, data=label
    )
    file.close()
```

`h5py.h5t.STD_U8BE` menentukan jenis data yang akan disimpan dalam dataset, yang dalam kasus ini adalah integer 8-bit unsigned. Anda dapat melihat daftar lengkap tipe data pradefi HDF di sini.

> Catatan: Pilihan tipe data akan sangat memengaruhi waktu proses dan persyaratan penyimpanan HDF5, jadi sebaiknya pilih persyaratan minimum Anda.

Sekarang setelah kita meninjau tiga metode untuk menyimpan satu gambar, mari lanjut ke langkah berikutnya.

## **Eksperimen untuk Menyimpan Satu Gambar**

Sekarang Anda dapat memasukkan ketiga fungsi untuk menyimpan satu gambar ke dalam dictionary, yang dapat dipanggil nanti selama eksperimen waktu:

```
_store_single_funcs = dict(
    disk=store_single_disk, lmdb=store_single_lmdb, hdf5=store_single_hdf5
)
```

Akhirnya, semuanya siap untuk melakukan eksperimen waktu. Mari coba menyimpan gambar pertama dari CIFAR dan label yang sesuai, dan menyimpannya dengan tiga cara yang berbeda:

```
from timeit import timeit

store_single_timings = dict()

for method in ("disk", "lmdb", "hdf5"):
    t = timeit(
        "_store_single_funcs[method](image, 0, label)",
        setup="image=images[0]; label=labels[0]",
        number=1,
        globals=globals(),
    )
    store_single_timings[method] = t
    print(f"Method: {method}, Time usage: {t}")
```

> Catatan: Saat bekerja dengan LMDB, Anda mungkin akan menemui error `MapFullError`, yang mengindikasikan bahwa ukuran peta (map size) yang ditentukan telah mencapai batasnya. Hal ini biasanya terjadi saat Anda mencoba menulis data baru ke dalam database LMDB yang sudah mencapai ukuran maksimum yang ditetapkan sebelumnya. Penting untuk diingat bahwa LMDB tidak akan menimpa nilai yang sudah ada, bahkan jika kunci yang sama digunakan. Ini memungkinkan operasi tulis yang cepat, tetapi juga berarti bahwa jika Anda menyimpan data yang sama berulang kali dalam file LMDB yang sama, ukuran peta akan terus bertambah.

Jika Anda menghadapi error `MapFullError`, Anda perlu mengatasi masalah ukuran peta LMDB. Salah satu solusi adalah dengan menambah ukuran peta menggunakan fungsi `mdb_env_set_mapsize` sebelum membuka lingkungan LMDB. Namun, perlu diingat bahwa menambah ukuran peta dapat mengakibatkan penggunaan memori yang lebih besar, jadi pastikan untuk menyesuaikan ukuran peta dengan kebutuhan aplikasi Anda.

Jika Anda ingin menyimpan ulang data ke dalam file LMDB yang sudah ada, pastikan untuk menghapus file LMDB tersebut terlebih dahulu sebelum menjalankan fungsi penyimpanan. Hal ini penting karena, seperti yang telah disebutkan sebelumnya, LMDB tidak akan menimpa nilai yang sudah ada. Dengan menghapus file LMDB yang sudah ada, Anda dapat memulai kembali dari awal dengan ukuran peta yang sesuai dengan kebutuhan Anda.

Dalam penggunaan praktis, penting untuk memantau ukuran peta LMDB secara berkala dan menyesuaikannya jika diperlukan agar tidak mencapai batas kapasitas. Dengan memahami karakteristik dan perilaku LMDB, Anda dapat mengoptimalkan penggunaan dan kinerjanya dalam aplikasi Anda.

Ingat bahwa kita tertarik pada waktu proses, yang ditampilkan di sini dalam detik, dan juga penggunaan memori:
| Metode | Simpan Gambar Tunggal + Meta | Penyimpanan |
| ------ | ---------------------------- | ----------- |
| Disk | 1,915 ms | 8 K |
| LMDB | 1,203 ms | 32 K |
| HDF5 | 8,243 ms | 8 K |

Dari hasil eksperimen ini, dapat disimpulkan bahwa semua metode penyimpanan yang diuji (disk, LMDB, dan HDF5) memiliki kinerja yang sangat cepat dalam hal waktu baca dan tulis file. Hal ini menunjukkan bahwa ketiga metode tersebut mampu menangani jumlah file gambar dalam skala besar dengan efisien. Namun, ada perbedaan dalam penggunaan disk, di mana LMDB menggunakan lebih banyak ruang disk dibandingkan dengan metode lainnya.

Meskipun LMDB memiliki sedikit keunggulan dalam kinerja, kita belum meyakinkan siapa pun mengapa tidak menyimpan gambar langsung di disk saja. Salah satu alasan utamanya adalah karena format gambar (seperti JPEG atau PNG) adalah format yang dapat dibaca oleh manusia, sehingga Anda dapat membuka dan melihat gambar dari browser sistem file mana pun tanpa perlu melalui proses baca dari database. Hal ini membuat penyimpanan langsung di disk menjadi lebih mudah untuk diakses dan dikelola secara manual.

Namun, ada beberapa keuntungan dalam menggunakan metode penyimpanan database seperti LMDB. Salah satunya adalah kemampuannya untuk mengelola data dalam skala besar dengan efisien, terutama dalam aplikasi yang membutuhkan akses cepat dan pencarian data yang kompleks. Selain itu, menggunakan database juga memungkinkan untuk menyimpan metadata atau informasi tambahan yang terkait dengan gambar tanpa perlu mengubah format gambar itu sendiri.

Pilihan antara menyimpan gambar langsung di disk atau menggunakan metode database seperti LMDB harus dipertimbangkan berdasarkan kebutuhan spesifik dari proyek Anda. Jika Anda lebih memperhatikan kemudahan akses dan kebutuhan untuk melihat gambar secara langsung dari sistem file, penyimpanan langsung di disk mungkin merupakan pilihan yang lebih baik. Namun, jika Anda mengutamakan efisiensi dan kemampuan untuk mengelola data dalam skala besar, metode database seperti LMDB bisa menjadi pilihan yang lebih cocok.
## Menyimpan Banyak Gambar

Anda telah melihat kode untuk menggunakan berbagai metode penyimpanan untuk menyimpan satu gambar, sekarang kita perlu menyesuaikan kode tersebut untuk menyimpan banyak gambar dan kemudian menjalankan eksperimen waktu.

### **Menyesuaikan Kode untuk Banyak Gambar**

Menyimpan banyak gambar sebagai file `.png` dapat dilakukan dengan mudah dengan memanggil fungsi `store_single_method()` berulang kali untuk setiap gambar. Namun, pendekatan ini tidak efisien untuk metode penyimpanan seperti LMDB atau HDF5, karena Anda tidak ingin membuat file basis data yang berbeda untuk setiap gambar. Sebagai gantinya, Anda ingin memasukkan semua gambar ke dalam satu atau lebih file basis data.

Dalam kasus LMDB, Anda dapat menyimpan banyak gambar ke dalam satu file basis data dengan menggunakan transaksi. Transaksi memungkinkan Anda untuk menulis beberapa entri ke dalam basis data dalam satu operasi, sehingga memungkinkan pengelompokan gambar ke dalam satu transaksi untuk kemudian disimpan ke dalam satu file LMDB. Hal ini dapat mengoptimalkan proses penulisan data ke dalam basis data dan mengurangi overhead yang terkait dengan membuat file basis data yang berbeda untuk setiap gambar.

Sementara itu, dalam kasus HDF5, Anda juga dapat menyimpan banyak gambar ke dalam satu file HDF5 dengan menggunakan grup dan dataset. Anda dapat membuat grup untuk setiap kategori gambar (misalnya, untuk CIFAR-10, Anda dapat memiliki grup untuk setiap kelas), dan kemudian menyimpan setiap gambar ke dalam dataset di bawah grup yang sesuai. Ini memungkinkan Anda untuk mengelompokkan gambar berdasarkan kategori atau kelasnya, sehingga memudahkan manajemen dan akses data.

Kedua pendekatan ini memungkinkan Anda untuk menyimpan banyak gambar ke dalam satu atau lebih file basis data, yang lebih efisien daripada menyimpan setiap gambar sebagai file terpisah. Dengan menggunakan transaksi untuk LMDB dan grup/dataset untuk HDF5, Anda dapat mengoptimalkan penggunaan ruang disk dan meningkatkan efisiensi akses data.

Anda perlu sedikit mengubah kode dan membuat tiga fungsi baru yang menerima banyak gambar, yaitu `store_many_disk()`, `store_many_lmdb()`, dan `store_many_hdf5()`:

```
store_many_disk(images, labels):
    """ Stores an array of images to disk
        Parameters:
        ---------------
        images       images array, (N, 32, 32, 3) to be stored
        labels       labels array, (N, 1) to be stored
    """
    num_images = len(images)

    # Save all the images one by one
    for i, image in enumerate(images):
        Image.fromarray(image).save(disk_dir / f"{i}.png")

    # Save all the labels to the csv file
    with open(disk_dir / f"{num_images}.csv", "w") as csvfile:
        writer = csv.writer(
            csvfile, delimiter=" ", quotechar="|", quoting=csv.QUOTE_MINIMAL
        )
        for label in labels:
            # This typically would be more than just one value per row
            writer.writerow([label])

def store_many_lmdb(images, labels):
    """ Stores an array of images to LMDB.
        Parameters:
        ---------------
        images       images array, (N, 32, 32, 3) to be stored
        labels       labels array, (N, 1) to be stored
    """
    num_images = len(images)

    map_size = num_images * images[0].nbytes * 10

    # Create a new LMDB DB for all the images
    env = lmdb.open(str(lmdb_dir / f"{num_images}_lmdb"), map_size=map_size)

    # Same as before — but let's write all the images in a single transaction
    with env.begin(write=True) as txn:
        for i in range(num_images):
            # All key-value pairs need to be Strings
            value = CIFAR_Image(images[i], labels[i])
            key = f"{i:08}"
            txn.put(key.encode("ascii"), pickle.dumps(value))
    env.close()

def store_many_hdf5(images, labels):
    """ Stores an array of images to HDF5.
        Parameters:
        ---------------
        images       images array, (N, 32, 32, 3) to be stored
        labels       labels array, (N, 1) to be stored
    """
    num_images = len(images)

    # Create a new HDF5 file
    file = h5py.File(hdf5_dir / f"{num_images}_many.h5", "w")

    # Create a dataset in the file
    dataset = file.create_dataset(
        "images", np.shape(images), h5py.h5t.STD_U8BE, data=images
    )
    meta_set = file.create_dataset(
        "meta", np.shape(labels), h5py.h5t.STD_U8BE, data=labels
    )
    file.close()
```

Untuk menyimpan lebih dari satu file ke disk dalam format gambar seperti PNG, Anda perlu melakukan perulangan pada setiap gambar dalam daftar dan menyimpannya satu per satu. Ini adalah pendekatan yang sederhana dan langsung ke depan, di mana setiap file gambar diproses secara independen dan disimpan ke dalam direktori yang sesuai.

Sementara itu, untuk LMDB, Anda juga perlu melakukan perulangan pada setiap gambar dalam daftar, tetapi dengan sedikit penyesuaian. Anda perlu membuat objek `CIFAR_Image` untuk setiap gambar dan metadata-nya, kemudian menyimpannya ke dalam database LMDB. Hal ini memungkinkan Anda untuk mengelola setiap gambar dan metadata-nya sebagai satu entitas yang terpisah dalam basis data, sehingga memungkinkan pengelolaan data yang lebih efisien.

Namun, penyesuaian terkecil diperlukan saat menggunakan metode HDF5. Anda dapat menyimpan semua gambar ke dalam satu dataset tanpa perlu melakukan penyesuaian tambahan. Ini karena file HDF5 tidak memiliki batasan ukuran file selain batasan eksternal atau ukuran dataset itu sendiri. Dengan demikian, Anda dapat memasukkan semua gambar ke dalam satu dataset tanpa khawatir tentang batasan ukuran file, seperti yang mungkin terjadi dalam format file lainnya.

Dengan menggunakan HDF5, Anda dapat menyimpan semua gambar ke dalam satu file, yang membuat manajemen dan akses data menjadi lebih mudah. Anda juga tidak perlu melakukan perulangan untuk setiap gambar, karena semua gambar dimasukkan ke dalam satu dataset. Ini membuat penggunaan HDF5 menjadi pilihan yang menarik untuk menyimpan dan mengelola dataset gambar dalam skala besar.

Selanjutnya, Anda perlu mempersiapkan dataset untuk eksperimen dengan meningkatkan ukurannya.

### **Mempersiapkan Kumpulan Data**

Sebelum menjalankan eksperimen lagi, mari kita pertama-tama menggandakan ukuran dataset agar kita dapat menguji hingga 100.000 gambar.

```
cutoffs = [10, 100, 1000, 10000, 100000]

# Let's double our images so that we have 100,000
images = np.concatenate((images, images), axis=0)
labels = np.concatenate((labels, labels), axis=0)

# Make sure you actually have 100,000 images and labels
print(np.shape(images))
print(np.shape(labels))
```

### **Eksperimen untuk Menyimpan Banyak Gambar**

Seperti yang Anda lakukan dengan membaca banyak gambar, Anda dapat membuat kamus yang menangani semua fungsi dengan `store_many_` dan menjalankan eksperimen.

```
_store_many_funcs = dict(
    disk=store_many_disk, lmdb=store_many_lmdb, hdf5=store_many_hdf5
)

from timeit import timeit

store_many_timings = {"disk": [], "lmdb": [], "hdf5": []}

for cutoff in cutoffs:
    for method in ("disk", "lmdb", "hdf5"):
        t = timeit(
            "_store_many_funcs[method](images_, labels_)",
            setup="images_=images[:cutoff]; labels_=labels[:cutoff]",
            number=1,
            globals=globals(),
        )
        store_many_timings[method].append(t)

        # Print out the method, cutoff, and elapsed time
        print(f"Method: {method}, Time usage: {t}")
```

Jika Anda mengikuti dan menjalankan kode sendiri, Anda perlu duduk sebentar dan menunggu 111.110 gambar disimpan tiga kali di disk Anda, dalam tiga format berbeda. Anda juga perlu mengucapkan selamat tinggal sekitar 2 GB ruang disk.

Sekarang untuk saat yang ditunggu-tunggu! **Berapa lama waktu yang dibutuhkan untuk menyimpan semua itu?** Satu gambar bernilai seribu kata:

<div align="center">
   
![Alt Text](https://github.com/balqiszamzami/SD3203-Teknologi-Basis-Data/blob/main/tugas/Pertemuan9/Jawaban/Output/1.png)
![Alt Text](https://github.com/balqiszamzami/SD3203-Teknologi-Basis-Data/blob/main/tugas/Pertemuan9/Jawaban/Output/2.png)

</div>

Grafik pertama menunjukkan waktu penyimpanan normal tanpa penyesuaian, menyoroti perbedaan drastis antara menyimpan ke file `.png` dan LMDB atau HDF5. Grafik kedua menunjukkan `log` dari waktu, menyoroti bahwa HDF5 memulai lebih lambat daripada LMDB tetapi, dengan lebih banyak gambar, sedikit lebih unggul.

Meskipun hasil pasti dapat bervariasi tergantung mesin Anda, **inilah mengapa LMDB dan HDF5 layak dipertimbangkan.** Inilah kode yang menghasilkan grafik di atas.

```
import matplotlib.pyplot as plt

def plot_with_legend(
    x_range, y_data, legend_labels, x_label, y_label, title, log=False
):
    """ Displays a single plot with multiple datasets and matching legends.
        Parameters:
        --------------
        x_range         list of lists containing x data
        y_data          list of lists containing y values
        legend_labels   list of string legend labels
        x_label         x axis label
        y_label         y axis label
    """
    plt.style.use("seaborn-whitegrid")
    plt.figure(figsize=(10, 7))

    if len(y_data) != len(legend_labels):
        raise TypeError(
            "Error: number of data sets does not match number of labels."
        )

    all_plots = []
    for data, label in zip(y_data, legend_labels):
        if log:
            temp, = plt.loglog(x_range, data, label=label)
        else:
            temp, = plt.plot(x_range, data, label=label)
        all_plots.append(temp)

    plt.title(title)
    plt.xlabel(x_label)
    plt.ylabel(y_label)
    plt.legend(handles=all_plots)
    plt.show()

# Getting the store timings data to display
disk_x = store_many_timings["disk"]
lmdb_x = store_many_timings["lmdb"]
hdf5_x = store_many_timings["hdf5"]

plot_with_legend(
    cutoffs,
    [disk_x, lmdb_x, hdf5_x],
    ["PNG files", "LMDB", "HDF5"],
    "Number of images",
    "Seconds to store",
    "Storage time",
    log=False,
)

plot_with_legend(
    cutoffs,
    [disk_x, lmdb_x, hdf5_x],
    ["PNG files", "LMDB", "HDF5"],
    "Number of images",
    "Seconds to store",
    "Log storage time",
    log=True,
)
```

## **Membaca Satu Gambar**

Pertama, mari kita pertimbangkan kasus membaca satu gambar kembali ke dalam array untuk masing-masing dari tiga metode tersebut.

### **Membaca Dari Disk**

Dari tiga metode, LMDB membutuhkan pekerjaan tambahan ketika membaca file gambar kembali dari memori, karena langkah serialisasi. Mari kita lihat fungsi-fungsi ini yang membaca satu gambar untuk masing-masing dari tiga format penyimpanan. Pertama, baca satu gambar dan metadata dari file `.png` dan `.csv`:

```
def read_single_disk(image_id):
    """ Stores a single image to disk.
        Parameters:
        ---------------
        image_id    integer unique ID for image

        Returns:
        ----------
        image       image array, (32, 32, 3) to be stored
        label       associated meta data, int label
    """
    image = np.array(Image.open(disk_dir / f"{image_id}.png"))

    with open(disk_dir / f"{image_id}.csv", "r") as csvfile:
        reader = csv.reader(
            csvfile, delimiter=" ", quotechar="|", quoting=csv.QUOTE_MINIMAL
        )
        label = int(next(reader)[0])

    return image, label
```

### **Membaca Dari LMDB**

Selanjutnya, baca gambar dan metadata yang sama dari LMDB dengan membuka lingkungan dan memulai transaksi baca:

```
def read_single_lmdb(image_id):
    """ Stores a single image to LMDB.
        Parameters:
        ---------------
        image_id    integer unique ID for image

        Returns:
        ----------
        image       image array, (32, 32, 3) to be stored
        label       associated meta data, int label
    """
    # Open the LMDB environment
    env = lmdb.open(str(lmdb_dir / f"single_lmdb"), readonly=True)

    # Start a new read transaction
    with env.begin() as txn:
        # Encode the key the same way as we stored it
        data = txn.get(f"{image_id:08}".encode("ascii"))
        # Remember it's a CIFAR_Image object that is loaded
        cifar_image = pickle.loads(data)
        # Retrieve the relevant bits
        image = cifar_image.get_image()
        label = cifar_image.label
    env.close()

    return image, label
```

Ada beberapa poin penting yang perlu diperhatikan tentang potongan kode yang digunakan untuk membaca gambar kembali dari LMDB:

1. **Baris 13:** Penggunaan bendera `readonly=True` dalam pembukaan lingkungan LMDB menunjukkan bahwa file LMDB akan dibuka dalam mode baca saja. Ini berarti bahwa tidak akan diizinkan penulisan apa pun pada file LMDB sampai transaksi selesai. Dalam konteks basis data, ini setara dengan mengambil kunci baca, yang mengamankan file dari modifikasi yang tidak diinginkan selama proses baca.

2. **Baris 20:** Ketika Anda ingin mengambil objek `CIFAR_Image` dari file LMDB, Anda perlu membalik langkah-langkah yang dilakukan saat menyimpannya. Pada saat penulisan, gambar diubah menjadi string bytes menggunakan metode `image_to_byte_array()`. Untuk mengembalikan objek `CIFAR_Image` ke bentuk semula, Anda perlu menggunakan metode `get_image()`. Ini penting karena Anda perlu memastikan bahwa data yang Anda ambil sesuai dengan format yang diharapkan.

Dengan memperhatikan poin-poin ini, Anda dapat mengoptimalkan proses membaca gambar kembali dari file LMDB. Hal yang sama berlaku untuk HDF5, di mana Anda akan perlu mempertimbangkan langkah-langkah yang diperlukan untuk membaca dan mengembalikan data dengan benar. Dengan memahami langkah-langkah yang diperlukan untuk membaca data kembali dari format penyimpanan yang berbeda, Anda dapat mengoptimalkan proses ini dan memastikan integritas data yang diambil.
### **Membaca Dari HDF5**

Membaca dari HDF5 terlihat sangat mirip dengan proses penulisan. Berikut adalah kode untuk membuka dan membaca file HDF5 dan mengurai gambar dan metadata yang sama:

```
def read_single_hdf5(image_id):
    """ Stores a single image to HDF5.
        Parameters:
        ---------------
        image_id    integer unique ID for image

        Returns:
        ----------
        image       image array, (32, 32, 3) to be stored
        label       associated meta data, int label
    """
    # Open the HDF5 file
    file = h5py.File(hdf5_dir / f"{image_id}.h5", "r+")

    image = np.array(file["/image"]).astype("uint8")
    label = int(np.array(file["/meta"]).astype("uint8"))

    return image, label
```

Perhatikan bahwa Anda mengakses berbagai dataset dalam file dengan menindeks objek `file` menggunakan nama dataset yang didahului oleh garis miring `/`. Seperti sebelumnya, Anda dapat membuat kamus yang berisi semua fungsi baca.

```
_read_single_funcs = dict(
    disk=read_single_disk, lmdb=read_single_lmdb, hdf5=read_single_hdf5
)
```

## **Eksperimen untuk Membaca Satu Gambar**

Anda mungkin mengharapkan bahwa eksperimen untuk membaca satu gambar akan memiliki hasil yang sepele, tetapi berikut adalah kode eksperimennya:

```
from timeit import timeit

read_single_timings = dict()

for method in ("disk", "lmdb", "hdf5"):
    t = timeit(
        "_read_single_funcs[method](0)",
        setup="image=images[0]; label=labels[0]",
        number=1,
        globals=globals(),
    )
    read_single_timings[method] = t
    print(f"Method: {method}, Time usage: {t}")
```

Berikut adalah hasil eksperimen untuk membaca satu gambar:
| Metode | Baca Gambar Tunggal + Meta |
| ------ | ---------------------------|
| Disk | 1,61970 ms |
| LMDB | 4,52063 ms |
| HDF5 | 1,98036 ms |

Hasil tersebut sedikit lebih cepat untuk membaca file `.png` dan `.csv` langsung dari disk, tetapi ketiga metode ini berjalan dengan sangat cepat. Eksperimen yang akan kita lakukan selanjutnya jauh lebih menarik.

## **Membaca Banyak Gambar**

Sekarang Anda dapat menyesuaikan kode untuk membaca banyak gambar sekaligus. Ini mungkin tindakan yang akan Anda lakukan paling sering, sehingga kinerja waktu sangat penting.

### **Modifikasi Kode untuk Menangani Banyak Gambar**

Memperluas fungsi di atas, Anda dapat membuat fungsi dengan `read_many_`, yang dapat digunakan untuk eksperimen berikutnya. Seperti sebelumnya, menarik untuk membandingkan kinerja ketika membaca jumlah gambar yang berbeda, yang diulangi dalam kode di bawah ini untuk referensi.

```
def read_many_disk(num_images):
    """ Reads image from disk.
        Parameters:
        ---------------
        num_images   number of images to read

        Returns:
        ----------
        images      images array, (N, 32, 32, 3) to be stored
        labels      associated meta data, int label (N, 1)
    """
    images, labels = [], []

    # Loop over all IDs and read each image in one by one
    for image_id in range(num_images):
        images.append(np.array(Image.open(disk_dir / f"{image_id}.png")))

    with open(disk_dir / f"{num_images}.csv", "r") as csvfile:
        reader = csv.reader(
            csvfile, delimiter=" ", quotechar="|", quoting=csv.QUOTE_MINIMAL
        )
        for row in reader:
            labels.append(int(row[0]))
    return images, labels

def read_many_lmdb(num_images):
    """ Reads image from LMDB.
        Parameters:
        ---------------
        num_images   number of images to read

        Returns:
        ----------
        images      images array, (N, 32, 32, 3) to be stored
        labels      associated meta data, int label (N, 1)
    """
    images, labels = [], []
    env = lmdb.open(str(lmdb_dir / f"{num_images}_lmdb"), readonly=True)

    # Start a new read transaction
    with env.begin() as txn:
        # Read all images in one single transaction, with one lock
        # We could split this up into multiple transactions if needed
        for image_id in range(num_images):
            data = txn.get(f"{image_id:08}".encode("ascii"))
            # Remember that it's a CIFAR_Image object
            # that is stored as the value
            cifar_image = pickle.loads(data)
            # Retrieve the relevant bits
            images.append(cifar_image.get_image())
            labels.append(cifar_image.label)
    env.close()
    return images, labels

def read_many_hdf5(num_images):
    """ Reads image from HDF5.
        Parameters:
        ---------------
        num_images   number of images to read

        Returns:
        ----------
        images      images array, (N, 32, 32, 3) to be stored
        labels      associated meta data, int label (N, 1)
    """
    images, labels = [], []

    # Open the HDF5 file
    file = h5py.File(hdf5_dir / f"{num_images}_many.h5", "r+")

    images = np.array(file["/images"]).astype("uint8")
    labels = np.array(file["/meta"]).astype("uint8")

    return images, labels

_read_many_funcs = dict(
    disk=read_many_disk, lmdb=read_many_lmdb, hdf5=read_many_hdf5
)
```

### **Percobaan untuk Menyimpan Sejumlah Besar Gambar**

Sekarang Anda dapat menjalankan eksperimen untuk membaca banyak gambar:

```
from timeit import timeit

read_many_timings = {"disk": [], "lmdb": [], "hdf5": []}

for cutoff in cutoffs:
    for method in ("disk", "lmdb", "hdf5"):
        t = timeit(
            "_read_many_funcs[method](num_images)",
            setup="num_images=cutoff",
            number=1,
            globals=globals(),
        )
        read_many_timings[method].append(t)

        # Print out the method, cutoff, and elapsed time
        print(f"Method: {method}, No. images: {cutoff}, Time usage: {t}")
```

Seperti yang kita lakukan sebelumnya, Anda dapat membuat grafik hasil eksperimen baca.

<div align="center">
   
![Alt Text](https://github.com/balqiszamzami/SD3203-Teknologi-Basis-Data/blob/main/tugas/Pertemuan9/Jawaban/Output/3.png)
![Alt Text](https://github.com/balqiszamzami/SD3203-Teknologi-Basis-Data/blob/main/tugas/Pertemuan9/Jawaban/Output/4.png)

</div>

Grafik pertama menunjukkan waktu baca normal tanpa penyesuaian, yang menyoroti perbedaan drastis dalam waktu baca antara membaca dari file `.png` dan LMDB atau HDF5. Dapat terlihat bahwa waktu baca dari file `.png` jauh lebih lambat dibandingkan dengan waktu baca dari LMDB atau HDF5. Hal ini menunjukkan bahwa penggunaan metode basis data seperti LMDB atau HDF5 dapat signifikan meningkatkan kinerja dalam hal waktu baca data gambar.

Sementara itu, grafik kedua menunjukkan `log` dari waktu, yang membantu menyoroti perbedaan relatif dengan lebih sedikit gambar. Dengan menggunakan skala logaritmik, perbedaan waktu baca antara metode dapat terlihat dengan lebih jelas. Terutama, grafik ini menunjukkan bahwa meskipun HDF5 mungkin memulai dengan performa yang lebih lambat dibandingkan dengan LMDB, namun dengan jumlah gambar yang lebih banyak, HDF5 secara konsisten menjadi lebih cepat dibandingkan dengan LMDB dengan selisih yang semakin kecil.

Perbandingan ini memberikan gambaran yang lebih baik tentang kinerja relatif dari masing-masing metode dalam mengelola dan membaca data gambar dalam skala besar. Dengan menggunakan skala logaritmik, perbedaan performa yang mungkin terlihat tidak signifikan dalam skala linier dapat diperjelas, membantu Anda memilih metode yang paling sesuai untuk kebutuhan aplikasi Anda. Dalam konteks ini, penggunaan HDF5 mungkin merupakan pilihan yang lebih baik untuk kinerja yang lebih baik dalam membaca data gambar dalam skala besar.
**Plot Waktu Baca** <br>
Dengan menggunakan fungsi plot yang sama seperti yang digunakan sebelumnya untuk menetapkan waktu penulisan, maka kita memperoleh hasil sebagai berikut:

```
disk_x_r = read_many_timings["disk"]
lmdb_x_r = read_many_timings["lmdb"]
hdf5_x_r = read_many_timings["hdf5"]

plot_with_legend(
    cutoffs,
    [disk_x_r, lmdb_x_r, hdf5_x_r],
    ["PNG files", "LMDB", "HDF5"],
    "Number of images",
    "Seconds to read",
    "Read time",
    log=False,
)

plot_with_legend(
    cutoffs,
    [disk_x_r, lmdb_x_r, hdf5_x_r],
    ["PNG files", "LMDB", "HDF5"],
    "Number of images",
    "Seconds to read",
    "Log read time",
    log=True,
)
```

Dalam konteks pelatihan jaringan saraf pada dataset gambar, waktu untuk membaca data seringkali lebih penting daripada waktu untuk menulis data. Bayangkan situasi di mana Anda memiliki kumpulan data gambar yang sangat besar, dan hanya setengah dari keseluruhan kumpulan data tersebut yang dapat dimuat ke dalam RAM sekaligus. Setiap periode pelatihan jaringan memerlukan seluruh kumpulan data, dan model Anda membutuhkan beberapa ratus periode untuk mencapai konvergensi. Dalam skenario ini, Anda akan membaca setengah dari kumpulan data ke dalam memori setiap periode.

Dalam situasi seperti ini, kecepatan baca data menjadi kunci, karena waktu yang dibutuhkan untuk membaca data secara langsung mempengaruhi kecepatan pelatihan model. Perbedaan antara waktu baca 40 detik dan 4 detik tiba-tiba menjadi perbedaan antara menunggu enam jam hingga model Anda selesai dilatih, atau hanya empat puluh menit! Dengan demikian, pemilihan metode penyimpanan yang efisien dapat sangat signifikan dalam konteks pelatihan jaringan saraf.

Salah satu trik yang dapat dilakukan adalah dengan menggunakan pelatihan mini-batch, di mana Anda melatih model dengan sejumlah kecil data (mini-batch) pada setiap iterasi, bukan seluruh kumpulan data sekaligus. Hal ini dapat meningkatkan efisiensi dalam penggunaan memori dan juga mempercepat waktu pelatihan secara keseluruhan. Namun, meskipun penggunaan pelatihan mini-batch dapat membantu dalam situasi ini, kecepatan baca data masih tetap menjadi faktor kunci dalam menentukan kecepatan pelatihan jaringan.

Dalam konteks grafik yang telah Anda lihat sebelumnya, perbedaan waktu baca yang tampaknya kecil dalam skala linier dapat memiliki dampak yang signifikan dalam skala logaritmik, terutama ketika waktu pelatihan jaringan membutuhkan ratusan periode. Oleh karena itu, memilih metode penyimpanan yang dapat membaca data dengan cepat, seperti menggunakan HDF5, dapat memberikan keuntungan yang signifikan dalam efisiensi waktu dan pemrosesan dalam pelatihan jaringan saraf pada dataset gambar yang besar.

Jika kita membandingkan waktu baca dan tulis pada grafik yang sama, kita mendapatkan hasil sebagai berikut:

<div align="center">
   
![Alt Text](https://github.com/balqiszamzami/SD3203-Teknologi-Basis-Data/blob/main/tugas/Pertemuan9/Jawaban/Output/5.png)

</div>

**Plot Waktu Baca dan Tulis** <br>
Anda dapat memplot semua waktu baca dan tulis pada satu grafik menggunakan fungsi plot yang sama:

```
plot_with_legend(
    cutoffs,
    [disk_x_r, lmdb_x_r, hdf5_x_r, disk_x, lmdb_x, hdf5_x],
    [
        "Read PNG",
        "Read LMDB",
        "Read HDF5",
        "Write PNG",
        "Write LMDB",
        "Write HDF5",
    ],
    "Number of images",
    "Seconds",
    "Log Store and Read Times",
    log=False,
)
```

Saat Anda menyimpan gambar sebagai file `.png`, terdapat perbedaan besar antara waktu menulis dan membaca. Namun, dengan format penyimpanan data LMDB dan HDF5, perbedaannya tidak terlalu signifikan. Secara keseluruhan, meskipun waktu membaca lebih penting daripada waktu menulis, terdapat alasan kuat untuk menyimpan gambar menggunakan format LMDB atau HDF5.

Sekarang setelah Anda melihat manfaat kinerja yang diberikan oleh LMDB dan HDF5, mari kita lihat metrik penting lainnya: penggunaan ruang disk.

## **Mempertimbangkan Penggunaan Disk**

Kecepatan bukan satu-satunya metrik kinerja yang perlu dipertimbangkan dalam pemilihan metode penyimpanan data. Ketika berurusan dengan kumpulan data yang sangat besar, seperti contoh kumpulan data gambar sebesar 3TB, penggunaan ruang disk juga menjadi masalah yang valid dan relevan. Dalam situasi seperti ini, menggunakan metode penyimpanan alternatif seperti LMDB atau HDF5 pada dasarnya berarti membuat salinan data tersebut, yang juga harus disimpan di disk. Meskipun metode ini dapat memberikan manfaat kinerja yang besar saat menggunakan data gambar, Anda harus memastikan bahwa Anda memiliki cukup ruang disk yang tersedia untuk menyimpan salinan data.

Penting untuk diingat bahwa saat menyimpan data dalam format yang dapat diakses oleh manusia, seperti file gambar PNG, Anda hanya perlu menyimpan data sekaligus. Namun, ketika Anda menggunakan metode penyimpanan alternatif seperti LMDB atau HDF5, Anda membuat salinan data dalam format yang dioptimalkan untuk akses cepat dan efisiensi, yang pada gilirannya memerlukan lebih banyak ruang disk.

Dalam kasus kumpulan data gambar sebesar 3TB, Anda harus mempertimbangkan dengan cermat kebutuhan ruang disk Anda sebelum memutuskan untuk menggunakan metode penyimpanan alternatif. Meskipun metode ini dapat memberikan keuntungan kinerja yang signifikan dalam penggunaan data gambar, terutama dalam pelatihan jaringan saraf, Anda harus memastikan bahwa infrastruktur penyimpanan Anda mampu menangani ukuran yang besar.

Selain itu, penting untuk mempertimbangkan juga keamanan dan integritas data saat menggunakan metode penyimpanan alternatif. Dalam beberapa kasus, metode penyimpanan alternatif dapat memberikan keuntungan tambahan dalam hal keamanan data, seperti enkripsi data atau mekanisme kontrol akses yang lebih canggih. Namun, Anda juga harus memastikan bahwa data Anda tetap aman dan terlindungi dari risiko kehilangan atau kerusakan.

Dengan mempertimbangkan dengan cermat berbagai faktor ini, Anda dapat membuat keputusan yang lebih terinformasi tentang pemilihan metode penyimpanan yang paling sesuai untuk kebutuhan Anda. Dengan memahami konsekuensi dan trade-off dari setiap metode, Anda dapat memastikan bahwa Anda dapat mengoptimalkan kinerja dan keamanan dalam penggunaan kumpulan data gambar sebesar 3TB.

**Berapa banyak ruang disk yang digunakan oleh berbagai metode penyimpanan?** Berikut adalah ruang disk yang digunakan oleh setiap metode untuk setiap jumlah gambar:

<div align="center">
   
![Alt Text](https://github.com/balqiszamzami/SD3203-Teknologi-Basis-Data/blob/main/tugas/Pertemuan9/Jawaban/Output/6.png)

</div>

Baik HDF5 maupun LMDB menggunakan lebih banyak ruang disk dibandingkan jika Anda menyimpan gambar dalam format `.png` biasa. Penting untuk dicatat bahwa penggunaan dan kinerja disk LMDB dan HDF5 **sangat bergantung pada berbagai faktor, termasuk sistem operasi dan, yang lebih penting, ukuran data yang Anda simpan.**

LMDB memperoleh efisiensinya dari caching dan memanfaatkan ukuran halaman OS. Anda tidak perlu memahami cara kerjanya secara detail, namun perhatikan bahwa **dengan gambar yang lebih besar, Anda akan mengalami penggunaan disk yang jauh lebih banyak dengan LMDB,** karena gambar tidak akan muat di halaman daun LMDB, lokasi penyimpanan reguler di dalam struktur pohon, dan sebaliknya Anda akan memiliki banyak halaman yang meluap. Batang LMDB pada grafik di atas akan keluar dari grafik.

Gambar berukuran 32x32x3 piksel yang kami gunakan relatif kecil dibandingkan dengan rata-rata gambar yang mungkin Anda gunakan, dan memungkinkan kinerja LMDB yang optimal.

Meskipun kami tidak akan menyelidikinya secara eksperimental di sini, dari pengalaman saya sendiri dengan gambar berukuran 256x256x3 atau 512x512x3 piksel, HDF5 biasanya sedikit lebih efisien dalam hal penggunaan disk daripada LMDB. Ini adalah transisi yang baik ke bagian akhir, diskusi kualitatif tentang perbedaan antara metode-metode tersebut.

**Kesimpulan** : 

Berikut adalah kesimpulan lebih detail tentang berbagai metode penyimpanan dan pengelolaan dataset gambar:

1. **Disk**
   - **Keuntungan:**
     - Implementasi sederhana dan langsung menggunakan sistem file komputer.
     - Tidak memerlukan setup atau konfigurasi tambahan.
   - **Kerugian:**
     - Performa lambat untuk dataset gambar yang sangat besar karena akses disk yang tidak efisien.
     - Pengelolaan file yang banyak bisa menjadi rumit.
     - Tidak mendukung pencarian atau query yang kompleks.

2. **LMDB (Lightning Memory-Mapped Database)**
   - **Keuntungan:**
     - Struktur B-tree yang dioptimalkan untuk akses cepat.
     - Mendukung penyimpanan dan pengambilan gambar secara efisien.
     - Cocok untuk dataset gambar yang besar dengan performa tinggi.
   - **Kerugian:**
     - Tidak terlalu umum digunakan di luar komunitas tertentu.
     - Kurva pembelajaran yang lebih tinggi dibandingkan dengan metode sederhana seperti disk.

3. **HDF5 (Hierarchical Data Format version 5)**
   - **Keuntungan:**
     - Menyimpan banyak gambar dalam satu file dengan struktur hierarki.
     - Menghemat ruang disk dan mempercepat proses pembacaan dan penulisan.
     - Mendukung penyimpanan data terstruktur dan tidak terstruktur.
   - **Kerugian:**
     - Memerlukan library khusus untuk membaca dan menulis data.
     - Format biner, sehingga tidak mudah dibaca atau diubah secara manual.

4. **TFRecord (TensorFlow Record)**
   - **Keuntungan:**
     - Format biner yang dirancang untuk meningkatkan efisiensi dengan dataset besar.
     - Integrasi yang baik dengan TensorFlow dan ekosistemnya.
     - Mendukung penyimpanan data terstruktur.
   - **Kerugian:**
     - Membutuhkan konversi dari format gambar biasa.
     - Tidak mudah dibaca atau diubah secara manual karena format biner.

5. **SQL Database (MySQL, PostgreSQL)**
   - **Keuntungan:**
     - Mendukung transaksi dan query kompleks.
     - Integrasi mudah dengan aplikasi berbasis SQL.
     - Kemampuan menyimpan metadata dengan gambar.
   - **Kerugian:**
     - Tidak dioptimalkan untuk penyimpanan gambar berukuran besar.
     - Bisa menjadi lambat jika tidak diindeks dengan benar.
     - Pengelolaan dan backup database bisa menjadi rumit untuk dataset besar.

6. **NoSQL Database (MongoDB, Cassandra)**
   - **Keuntungan:**
     - Skalabilitas tinggi dan fleksibel dalam penyimpanan data terstruktur dan tidak terstruktur.
     - Cepat dalam akses data dan cocok untuk aplikasi yang memerlukan penyimpanan dan pengambilan cepat.
   - **Kerugian:**
     - Tidak selalu dioptimalkan untuk penyimpanan file besar.
     - Memerlukan konfigurasi yang lebih kompleks dan pengetahuan tentang sistem NoSQL.

7. **Object Storage (Amazon S3, Google Cloud Storage)**
   - **Keuntungan:**
     - Sangat skalabel dan andal dengan akses global.
     - Latensi rendah dan biaya penyimpanan yang efektif.
     - Mendukung penyimpanan data dalam jumlah besar dengan berbagai ukuran.
   - **Kerugian:**
     - Memerlukan konektivitas internet untuk akses data.
     - Biaya akses dan penyimpanan data dapat bertambah tergantung pada penggunaan.
     - Mungkin memerlukan manajemen hak akses yang lebih ketat.

8. **Blob Storage (Microsoft Azure Blob Storage)**
   - **Keuntungan:**
     - Integrasi yang baik dengan ekosistem Azure.
     - Skalabilitas tinggi dan performa untuk berbagai jenis data.
     - Mendukung penyimpanan dalam jumlah besar dan mudah diakses.
   - **Kerugian:**
     - Biaya tambahan tergantung pada penggunaan dan akses data.
     - Memerlukan manajemen hak akses yang baik untuk keamanan data.
     - Memerlukan konektivitas internet untuk akses data.
     - 
Pemilihan metode penyimpanan gambar harus mempertimbangkan berbagai faktor seperti ukuran dataset, frekuensi akses, kebutuhan akan kecepatan akses, biaya, dan integrasi dengan sistem atau aplikasi yang digunakan. 

- Untuk dataset kecil atau aplikasi sederhana, metode **Disk** mungkin cukup.
- Untuk dataset besar dengan kebutuhan akses cepat, **LMDB** atau **HDF5** bisa menjadi pilihan.
- **TFRecord** sangat cocok untuk aplikasi pembelajaran mesin menggunakan TensorFlow.
- **SQL** dan **NoSQL Database** dapat digunakan jika memerlukan dukungan transaksi dan query kompleks, serta pengelolaan metadata.
- **Object Storage** dan **Blob Storage** sangat cocok untuk penyimpanan dalam jumlah besar dengan akses global dan skalabilitas tinggi.

Pemilihan yang tepat dapat meningkatkan efisiensi dan performa aplikasi secara signifikan.

Berbagai metode penyimpanan gambar memiliki kelebihan dan kekurangan yang perlu dipertimbangkan sesuai dengan kebutuhan spesifik. Metode penyimpanan sederhana menggunakan **disk** sangat mudah diimplementasikan karena tidak memerlukan setup atau konfigurasi tambahan. Namun, metode ini menjadi kurang efisien untuk menangani dataset gambar yang sangat besar karena akses disk yang lambat dan pengelolaan file yang banyak bisa menjadi rumit. Untuk aplikasi yang memerlukan pengelolaan file sederhana dan dataset kecil, metode ini mungkin sudah cukup memadai.

Untuk menangani dataset gambar yang besar dengan performa tinggi, metode seperti **LMDB (Lightning Memory-Mapped Database)** dan **HDF5 (Hierarchical Data Format version 5)** lebih disarankan. LMDB menggunakan struktur B-tree yang dioptimalkan untuk akses cepat, sehingga cocok untuk penyimpanan dan pengambilan gambar dalam skala besar. HDF5, dengan struktur hierarkinya, menghemat ruang disk dan mempercepat proses pembacaan dan penulisan, sehingga efisien untuk penyimpanan data terstruktur dan tidak terstruktur. Kedua metode ini, meskipun efisien, memerlukan pengetahuan khusus untuk implementasi dan penggunaan.

**TFRecord (TensorFlow Record)** adalah format biner yang dirancang untuk meningkatkan efisiensi dalam pembelajaran mesin, terutama ketika menggunakan TensorFlow. Format ini mendukung penyimpanan data terstruktur dan cepat dalam pembacaan dan penulisan, menjadikannya pilihan ideal untuk aplikasi pembelajaran mesin dengan dataset besar. Namun, karena formatnya biner, data tidak mudah dibaca atau diubah secara manual dan memerlukan konversi dari format gambar biasa. Dalam kasus di mana metadata dan query kompleks diperlukan, **SQL Database (MySQL, PostgreSQL)** atau **NoSQL Database (MongoDB, Cassandra)** bisa menjadi pilihan yang baik. SQL Database mendukung transaksi dan query yang kompleks serta penyimpanan metadata, tetapi kurang optimal untuk gambar besar. Sebaliknya, NoSQL Database menawarkan skalabilitas tinggi dan fleksibilitas, meski memerlukan konfigurasi yang lebih kompleks.

Untuk penyimpanan skala besar dengan akses global dan latensi rendah, **Object Storage** seperti **Amazon S3** atau **Google Cloud Storage**, serta **Blob Storage** dari **Microsoft Azure**, adalah pilihan yang sangat baik. Kedua metode ini sangat skalabel, andal, dan biaya efektif untuk penyimpanan dalam jumlah besar. Mereka mendukung berbagai ukuran data dengan mudah diakses dari mana saja, meskipun memerlukan konektivitas internet dan manajemen hak akses yang baik. Object Storage dan Blob Storage sangat cocok untuk aplikasi yang membutuhkan penyimpanan dan akses data dalam jumlah besar dengan keandalan tinggi dan skalabilitas global.

_**source**_ : [Three Ways of Storing and Accessing Lots of Images in Python](https://realpython.com/storing-images-in-python/#a-dataset-to-play-with)

</div>
