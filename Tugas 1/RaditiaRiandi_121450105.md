> **Nama : Raditia Riandi** <br> **NIM : 121450105** <br> **Kelas : RB**
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

> Catatan: Saat Anda bermain-main dengan LMDB, Anda mungkin melihat error `MapFullError`: `mdb_txn_commit`: `MDB_MAP_FULL`: Batas ukuran lingkungan mapsize tercapai. Penting untuk dicatat bahwa LMDB **tidak** menimpa nilai yang sudah ada, bahkan jika memiliki kunci yang sama. Ini berkontribusi pada waktu tulis yang cepat, tetapi juga berarti bahwa jika Anda menyimpan gambar lebih dari satu kali dalam file LMDB yang sama, maka Anda akan menggunakan map size. Jika Anda menjalankan fungsi penyimpanan, pastikan untuk menghapus file LMDB yang sudah ada terlebih dahulu.

Ingat bahwa kita tertarik pada waktu proses, yang ditampilkan di sini dalam detik, dan juga penggunaan memori:
| Metode | Simpan Gambar Tunggal + Meta | Penyimpanan |
| ------ | ---------------------------- | ----------- |
| Disk | 1,915 ms | 8 K |
| LMDB | 1,203 ms | 32 K |
| HDF5 | 8,243 ms | 8 K |

Ada dua kesimpulan di sini:

1. Semua metode sangat cepat.
2. Dalam hal penggunaan disk, LMDB menggunakan lebih banyak.

Jelas, meskipun LMDB memiliki sedikit keunggulan kinerja, kita belum meyakinkan siapa pun mengapa tidak menyimpan gambar di disk saja. Bagaimanapun, itu format yang dapat dibaca manusia, dan Anda dapat membuka dan melihatnya dari browser sistem file mana pun.

## Menyimpan Banyak Gambar

Anda telah melihat kode untuk menggunakan berbagai metode penyimpanan untuk menyimpan satu gambar, sekarang kita perlu menyesuaikan kode tersebut untuk menyimpan banyak gambar dan kemudian menjalankan eksperimen waktu.

### **Menyesuaikan Kode untuk Banyak Gambar**

Menyimpan banyak gambar sebagai file `.png` sesederhana memanggil fungsi `store_single_method()` berulang kali. Namun, hal ini tidak berlaku untuk LMDB atau HDF5, karena Anda tidak ingin membuat file basis data yang berbeda untuk setiap gambar. Sebagai gantinya, Anda ingin memasukkan semua gambar ke dalam satu atau lebih file.

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

Untuk menyimpan lebih dari satu file ke disk, metode file gambar diubah untuk melakukan perulangan pada setiap gambar dalam daftar. Untuk LMDB, perulangan juga dibutuhkan karena kita membuat objek `CIFAR_Image` untuk setiap gambar dan metadata-nya. Penyesuaian terkecil adalah dengan metode HDF5. Sebenarnya, hampir tidak ada penyesuaian sama sekali! File HDF5 tidak memiliki batasan ukuran file selain batasan eksternal atau ukuran dataset, jadi semua gambar dimasukkan ke dalam satu dataset, seperti sebelumnya.

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

Ada beberapa poin yang perlu diperhatikan tentang potongan kode di atas:

- **Baris 13:** Bendera `readonly=True` menentukan bahwa tidak akan diizinkan penulisan apa pun pada file LMDB hingga transaksi selesai. Dalam istilah basis data, ini setara dengan mengambil kunci baca.
- **Baris 20:** Untuk mengambil objek CIFAR_Image, Anda perlu membalik langkah yang kita ambil untuk membuat string bytesnya ketika kita menulisnya. Inilah tempat `get_image()` dari objek tersebut berguna.

Ini mengakhiri proses membaca gambar kembali dari LMDB. Akhirnya, Anda juga akan ingin melakukan hal yang sama dengan HDF5.

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

Grafik atas menunjukkan waktu baca normal tanpa penyesuaian, menunjukkan perbedaan drastis antara membaca dari file `.png` dan LMDB atau HDF5.

Sebaliknya, grafik di bawah menunjukkan `log` dari waktu, menyoroti perbedaan relatif dengan lebih sedikit gambar. Terutama, kita dapat melihat bagaimana HDF5 memulai di belakang tetapi, dengan lebih banyak gambar dan secara konsisten menjadi lebih cepat dibandingkan LMDB dengan selisih yang kecil.

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

Dalam praktiknya, waktu untuk membaca data seringkali lebih penting daripada waktu untuk menulis data. Bayangkan Anda sedang melatih jaringan saraf (_neural network_) pada sekumpulan gambar, dan hanya setengah dari keseluruhan kumpulan data gambar tersebut yang dapat dimuat ke dalam RAM sekaligus. Setiap periode pelatihan jaringan memerlukan seluruh kumpulan data, dan model membutuhkan beberapa ratus periode untuk mencapai konvergensi. Pada dasarnya, Anda akan membaca setengah dari kumpulan data ke dalam memori setiap periode.

Ada beberapa trik yang dapat dilakukan, seperti pelatihan mini-batch (_mini-batch training_) untuk sedikit meningkatkan efisiensi, tetapi Anda mengerti maksudnya.

Sekarang, lihat kembali grafik yang telah dibaca sebelumnya. Perbedaan antara waktu baca 40 detik dan 4 detik tiba-tiba menjadi perbedaan antara menunggu enam jam hingga model Anda selesai dilatih, atau hanya empat puluh menit!

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

Kecepatan bukan satu-satunya metrik kinerja yang perlu dipertimbangkan. Kita sudah berurusan dengan kumpulan data yang sangat besar, sehingga penggunaan ruang disk juga menjadi masalah yang valid dan relevan.

Misalkan Anda memiliki kumpulan data gambar sebesar 3TB. Sepertinya, Anda sudah menyimpan data tersebut di disk, tidak seperti contoh CIFAR yang kita bahas sebelumnya. Jadi, dengan menggunakan metode penyimpanan alternatif, pada dasarnya Anda membuat salinan data tersebut, yang juga harus disimpan. Melakukan hal ini akan memberikan manfaat kinerja yang besar saat Anda menggunakan gambar, namun Anda harus memastikan Anda memiliki cukup ruang disk.

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
