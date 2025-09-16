---
lab:
  title: Klasifikasi teks kustom
  description: Terapkan klasifikasi kustom ke input teks menggunakan Azure AI Bahasa.
---

# Klasifikasi teks kustom

Azure AI Bahasa menyediakan beberapa kemampuan NLP, termasuk identifikasi frasa kunci, ringkasan teks, dan analisis sentimen. Layanan Bahasa juga menyediakan fitur kustom seperti jawaban atas pertanyaan kustom dan klasifikasi teks kustom.

Untuk menguji klasifikasi teks kustom layanan Azure AI Bahasa, Anda akan mengonfigurasi model menggunakan Language Studio, lalu menggunakan aplikasi Python untuk mengujinya.

Meskipun latihan ini didasarkan pada Python, Anda dapat mengembangkan aplikasi klasifikasi teks menggunakan beberapa SDK khusus bahasa; termasuk:

- [Pustaka klien Analitik Teks Azure AI untuk Python](https://pypi.org/project/azure-ai-textanalytics/)
- [Pustaka klien Analitik Teks Azure AI untuk .NET](https://www.nuget.org/packages/Azure.AI.TextAnalytics)
- [Pustaka klien Analitik Teks Azure AI untuk JavaScript](https://www.npmjs.com/package/@azure/ai-text-analytics)

Latihan ini memakan waktu sekitar **35** menit.

## Provisi sumber daya *Azure AI Bahasa*

Jika Anda belum memilikinya di langganan, Anda harus memprovisikan sumber daya **layanan Azure AI Bahasa**. Selain itu, untuk menggunakan klasifikasi teks kustom, Anda perlu mengaktifkan fitur **Klasifikasi teks kustom & ekstraksi**.

1. Buka portal Microsoft Azure di `https://portal.azure.com`, dan masuk menggunakan akun Microsoft yang terkait dengan langganan Azure Anda.
1. Pilih **Buat sumber daya**.
1. Di bidang pencarian, cari **Layanan bahasa**. Kemudian, dalam hasil, pilih **Buat** di bawah **Layanan Bahasa**.
1. Pilih kotak yang menyertakan **Klasifikasi teks kustom**. Lalu pilih **Lanjutkan untuk membuat sumber daya Anda**.
1. Buat sumber daya dengan pengaturan berikut:
    - **Langganan**: *Langganan Azure Anda*.
    - **Grup sumber daya**: *Pilih atau buat grup sumber daya*.
    - **Wilayah**: *: Pilih dari salah satu wilayah berikut*\*
        - Australia Timur
        - India Tengah
        - US Timur
        - US Timur 2
        - Eropa Utara
        - US Tengah Selatan
        - Swiss Utara
        - UK Selatan
        - Eropa Barat
        - US Barat 2
        - AS Barat 3
    - **Nama**: *Masukkan nama unik*.
    - **Tingkat harga**: Pilih **F0** (*gratis*), atau **S** (*standar*) jika F tidak tersedia.
    - **Storage account**: Akun penyimpanan baru
      - **Nama akun penyimpanan**: *Masukkan nama yang unik*.
      - **Tipe akun penyimpanan**: LRS Standar
    - **Pemberitahuan AI yang bertanggung jawab**: Dipilih.

1. Pilih **Tinjau + buat,** lalu pilih **Buat** untuk memprovisikan sumber daya.
1. Tunggu hingga penyebaran selesai, lalu buka grup sumber daya.
1. Temukan akun penyimpanan yang Anda buat, pilih, dan verifikasi _Jenis akun_ adalah **StorageV2**. Jika v1, tingkatkan jenis akun penyimpanan Anda di halaman sumber daya tersebut.

## Mengonfigurasi akses berbasis peran untuk pengguna Anda

> **CATATAN**: Jika melewati langkah ini, Anda akan mengalami kesalahan 403 ketika mencoba terhubung ke proyek khusus Anda. Penting bahwa pengguna Anda saat ini memiliki peran ini untuk mengakses data blob akun penyimpanan, meskipun Anda adalah pemilik akun penyimpanannya.

1. Buka akun penyimpanan di portal Microsoft Azure.
2. Pilih **Access Control (IAM)** di menu panel navigasi kiri.
3. Pilih **Tambahkan** untuk Menambahkan Penetapan Peran, dan pilih peran **Pemilik Data Blob Penyimpanan** di akun penyimpanan.
4. Dalam **Tetapkan akses ke**, pilih **Pengguna, grup, atau prinsipal layanan**
5. Pilih **Pilih anggota**.
6. Pilih pengguna Anda. Anda dapat mencari nama pengguna di bidang **Pilih**.

## Mengunggah artikel sampel

Setelah membuat layanan Azure AI Bahasa dan akun penyimpanan, Anda harus mengunggah artikel contoh untuk melatih model Anda nanti.

1. Di tab browser baru, unduh artikel sampel dari `https://aka.ms/classification-articles` dan ekstrak file ke folder pilihan Anda.

1. Di portal Azure, buka akun penyimpanan yang Anda buat, dan pilih.

1. Di akun penyimpanan Anda, pilih **Konfigurasi**, yang terletak di bawah **Pengaturan**. Di layar Konfigurasi, aktifkan opsi **Izinkan Akses anonim blob**, lalu pilih **Simpan**.

1. Pilih **Kontainer** di menu sebelah kiri, yang terletak di bawah **Penyimpanan data**. Pada layar yang muncul, pilih **+ Kontainer**. Beri kontainer nama `articles`, dan atur **Tingkat akses anonim** ke **Kontainer (akses baca anonim untuk kontainer dan blob)**.

    > **CATATAN**: Saat Anda mengonfigurasi akun penyimpanan untuk solusi nyata, berhati-hatilah untuk menetapkan tingkat akses yang sesuai. Untuk mempelajari selengkapnya tentang setiap tingkat akses, lihat [Dokumentasi Azure Storage](https://learn.microsoft.com/azure/storage/blobs/anonymous-read-access-configure).

1. Setelah Anda membuat kontainer, pilih kontainer lalu pilih tombol **Unggah**. Pilih **Telusuri file** untuk menelusuri artikel sampel yang Anda unduh. Lalu pilih **Unggah**.

## Membuat proyek klasifikasi kustom

Setelah konfigurasi selesai, buat proyek klasifikasi teks kustom. Proyek ini menyediakan tempat kerja untuk membangun, melatih, dan menyebarkan model Anda.

> **CATATAN**: Lab ini menggunakan **Language Studio**, tetapi Anda juga dapat membuat, membangun, melatih, dan menyebarkan model Anda melalui REST API.

1. Di tab browser baru, buka portal Studio Azure AI Bahasa di `https://language.cognitive.azure.com/` dan masuk menggunakan akun Microsoft yang terkait dengan langganan Azure Anda.
1. Jika diminta untuk memilih sumber daya Bahasa, pilih pengaturan berikut:

    - **Direktori Azure**: Direktori Azure yang berisi langganan Anda.
    - **Langganan Azure**: Langganan Azure Anda.
    - **Jenis sumber daya**: Bahasa.
    - **Sumber daya bahasa**: Sumber daya Azure AI Bahasa yang Anda buat sebelumnya.

    Jika Anda <u>tidak</u> diminta untuk memilih sumber daya bahasa, hal tersebut mungkin karena Anda memiliki beberapa sumber daya Bahasa dalam langganan Anda; dalam hal ini:

    1. Pada bilah di bagian atas halaman, pilih tombol **Pengaturan (&#9881;)**.
    2. Pada halaman **Pengaturan**, lihat tab **Sumber Daya**.
    3. Pilih sumber daya bahasa yang baru saja Anda buat, dan klik **Ganti sumber daya**.
    4. Di bagian atas halaman, klik **Language Studio** untuk kembali ke beranda Language Studio

1. Di bagian atas portal, di menu **Buat baru**, pilih **Klasifikasi teks kustom**.
1. Halaman **Hubungkan penyimpanan** akan muncul. Semua nilai akan sudah diisi. Jadi pilih **Berikutnya**.
1. Di halaman **Pilih jenis proyek**, pilih **Klasifikasi label tunggal**. Kemudian pilih **Berikutnya**.
1. Di panel **Masukkan informasi dasar**, atur hal berikut:
    - **Nama**: `ClassifyLab`  
    - **Bahasa utama teks**: Inggris (AS)
    - **Deskripsi**: `Custom text lab`

1. Pilih **Selanjutnya**.
1. Di halaman **Pilih kontainer**, atur menu drop-down **Kontainer penyimpanan blob** ke kontainer *artikel* Anda.
1. Pilih opsi **Tidak, saya perlu memberi label file saya sebagai bagian dari proyek ini**. Kemudian pilih **Berikutnya**.
1. Pilih **Buat proyek**.

> **Tips**: Jika Anda mendapatkan kesalahan tentang tidak berwenang untuk melakukan operasi ini, Anda harus menambahkan penetapan peran. Untuk memperbaikinya, kami menambahkan peran "Kontributor Data Blob Penyimpanan" pada akun penyimpanan untuk pengguna yang menjalankan lab. Rincian lebih lanjut dapat ditemukan [di halaman dokumentasi](https://learn.microsoft.com/azure/ai-services/language-service/custom-named-entity-recognition/how-to/create-project?tabs=portal%2Clanguage-studio#enable-identity-management-for-your-resource).

## Memberi label pada data Anda

Sekarang setelah proyek dibuat, Anda perlu memberi label, atau memberi tag data Anda untuk melatih model cara mengklasifikasikan teks.

1. Di sebelah kiri, pilih **Pelabelan data**, jika belum dipilih. Anda akan melihat daftar file yang Anda unggah ke akun penyimpanan.
1. Di sebelah kanan, di panel **Aktivitas**, pilih **+ Tambahkan kelas**.  Artikel di lab ini terbagi dalam empat kelas yang harus Anda buat: `Classifieds`, `Sports`, `News`, dan `Entertainment`.

    ![Cuplikan layar yang menampilkan halaman data tag dan tombol tambahkan kelas.](../media/tag-data-add-class-new.png#lightbox)

1. Setelah Anda membuat empat kelas, pilih **Artikel 1** untuk memulai. Di sini Anda dapat membaca artikel, menentukan kelas mana file ini, dan ke himpunan data mana (pelatihan atau pengujian) file ini akan ditetapkan.
1. Tetapkan setiap artikel ke kelas dan himpunan data yang sesuai (pelatihan atau pengujian) menggunakan panel **Aktivitas** di kanan.  Anda dapat memilih label dari daftar label di kanan, dan mengatur setiap artikel ke **pelatihan** atau **pengujian** menggunakan opsi di bagian bawah panel Aktivitas. Anda memilih **Dokumen berikutnya** untuk pindah ke dokumen berikutnya. Untuk tujuan lab ini, kita akan menentukan mana yang akan digunakan untuk melatih model dan menguji model:

    | Artikel  | Kelas  | Dataset  |
    |---------|---------|---------|
    | Artikel 1 | Olahraga | Pelatihan |
    | Artikel 10 | Berita | Pelatihan |
    | Artikel 11 | Hiburan | Pengujian |
    | Artikel 12 | Berita | Pengujian |
    | Artikel 13 | Olahraga | Pengujian |
    | Artikel 2 | Olahraga | Pelatihan |
    | Artikel 3 | Iklan Baris | Pelatihan |
    | Artikel 4 | Iklan Baris | Pelatihan |
    | Artikel 5 | Hiburan | Pelatihan |
    | Artikel 6 | Hiburan | Pelatihan |
    | Artikel 7 | Berita | Pelatihan |
    | Artikel 8 | Berita | Pelatihan |
    | Artikel 9 | Hiburan | Pelatihan |

    > **CATATAN** File di Language Studio dicantumkan menurut abjad, itulah sebabnya daftar di atas tidak berurutan. Pastikan Anda mengunjungi kedua halaman dokumen saat memberi label pada artikel.

1. Pilih **Simpan label** untuk menyimpan label Anda.

## Melatih model

Setelah memberi label pada data, Anda perlu melatih model.

1. Pilih **Pekerjaan pelatihan** di menu sebelah kiri.
1. Pilih **Mulai pekerjaan pelatihan**.
1. Latih model baru bernama `ClassifyArticles`.
1. Pilih **Gunakan pemisahan manual data pelatihan dan pengujian**.

    > **TIP** Dalam proyek klasifikasi Anda sendiri, layanan Azure AI Bahasa akan secara otomatis membagi kumpulan pengujian berdasarkan persentase yang berguna dengan himpunan data yang besar. Dengan himpunan data yang lebih kecil, penting untuk berlatih dengan distribusi kelas yang tepat.

1. Pilih **Latih**

> **PENTING** Melatih model Anda terkadang bisa memakan waktu beberapa menit. Anda akan mendapatkan pemberitahuan jika sudah selesai.

## Mengevaluasi model Anda

Dalam aplikasi klasifikasi teks dunia nyata, penting untuk mengevaluasi dan meningkatkan model Anda guna memverifikasi performanya seperti yang Anda harapkan.

1. Pilih **Performa model**, dan pilih model **ClassifyArticles** Anda. Di sana Anda dapat melihat penilaian model Anda, metrik performa, dan kapan model tersebut dilatih. Jika skor model Anda tidak 100%, itu berarti salah satu dokumen yang digunakan untuk pengujian tidak dievaluasi sesuai dengan labelnya. Kegagalan ini dapat membantu Anda memahami mana yang harus diperbaiki.
1. Pilih tab **Detail kumpulan pengujian**. Jika ada kesalahan, tab ini memungkinkan Anda melihat artikel yang Anda tentukan untuk pengujian, sebagai apa model memprediksinya, dan apakah itu bertentangan dengan label pengujian. Default tab ini adalah hanya menampilkan prediksi yang salah. Anda dapat menghidupkan/mematikan opsi **Tampilkan hanya ketidakcocokan** untuk melihat semua artikel yang Anda tentukan untuk pengujian dan sebagai apa masing-masing artikel itu diprediksi.

## Sebarkan model anda

Saat Anda puas dengan pelatihan model Anda, saatnya untuk menyebarkannya, yang memungkinkan Anda mulai mengklasifikasikan teks melalui API.

1. Di panel kiri, pilih **Menyebarkan model**.
1. Pilih **Tambahkan penyebaran**, lalu masukkan `articles` di bidang **Buat nama penyebaran baru**, dan pilih **ClassifyArticles** di bidang **Model**.
1. Pilih **Sebarkan** untuk menyebarkan model Anda.
1. Setelah model Anda disebarkan, biarkan halaman tersebut tetap terbuka. Anda akan memerlukan nama proyek dan penyebaran di langkah berikutnya.

## Persiapan untuk mengembangkan aplikasi di Cloud Shell

Untuk menguji kemampuan klasifikasi teks kustom layanan Azure AI Bahasa, Anda akan mengembangkan aplikasi konsol sederhana di Azure Cloud Shell.

1. Gunakan tombol **[\>_]** di sebelah kanan bilah pencarian di bagian atas halaman untuk membuat Cloud Shell baru di portal Azure, dengan memilih lingkungan ***PowerShell***. Cloud shell menyediakan antarmuka baris perintah dalam panel di bagian bawah portal Azure.

    > **Catatan**: Jika sebelumnya Anda telah membuat cloud shell yang menggunakan lingkungan *Bash* , alihkan ke ***PowerShell***.

1. Di toolbar cloud shell, di menu **Pengaturan**, pilih **Buka versi Klasik** (ini diperlukan untuk menggunakan editor kode).

    **<font color="red">Pastikan Anda telah beralih ke versi klasik cloud shell sebelum melanjutkan.</font>**

1. Di panel PowerShell, masukkan perintah berikut untuk mengkloning repositori GitHub untuk latihan ini:

    ```
   rm -r mslearn-ai-language -f
   git clone https://github.com/microsoftlearning/mslearn-ai-language
    ```

    > **Tips**: Saat Anda menempelkan perintah ke cloudshell, ouput mungkin mengambil sejumlah besar buffer layar. Anda dapat menghapus layar dengan memasukkan `cls` perintah untuk mempermudah fokus pada setiap tugas.

1. Setelah repositori dikloning, navigasikan ke folder yang berisi file kode aplikasi obrolan:  

    ```
   cd mslearn-ai-language/Labfiles/04-text-classification/Python/classify-text
    ```

## Mengonfigurasi aplikasi Anda

1. Di panel baris perintah, jalankan perintah berikut untuk menampilkan file kode di folder **classify-text**:

    ```
   ls -a -l
    ```

    File menyertakan file konfigurasi (**.env**) dan file kode (**classify-text.py**). Teks yang akan dianalisis aplikasi Anda ada di subfolder **artikel**.

1. Buat lingkungan virtual Python dan instal paket SDK Analitik Teks Azure AI Bahasa dan paket lain yang diperlukan dengan menjalankan perintah berikut:

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-ai-textanalytics==5.3.0
    ```

1. Masukkan perintah berikut untuk mengedit file konfigurasi aplikasi:

    ```
   code .env
    ```

    File dibuka dalam editor kode.

1. Perbarui nilai konfigurasi untuk menyertakan **titik akhir** dan **kunci **dari sumber daya Bahasa Azure yang Anda buat (tersedia di halaman **Kunci dan Titik Akhir** untuk sumber daya Azure AI Bahasa Anda di portal Azure). Seharusnya file sudah berisi nama proyek dan penyebaran untuk model klasifikasi teks Anda.
1. Setelah Anda mengganti tempat penampung, gunakan perintah **CTRL+S** atau **Klik kanan > Simpan** untuk menyimpan perubahan Anda dan kemudian gunakan perintah **CTRL+Q** atau **Klik kanan > Keluar** untuk menutup editor kode sambil tetap membuka baris perintah cloud shell.

## Menambahkan kode untuk mengklasifikasikan dokumen

1. Masukkan perintah berikut untuk mengedit file kode aplikasi:

    ```
    code classify-text.py
    ```

1. Tinjau kode yang sudah ada. Anda akan menambahkan kode untuk bekerja dengan SDK Analitik Teks AI Bahasa.

    > **Tips**: Saat Anda menambahkan kode ke file kode, pastikan untuk mempertahankan indentasi yang benar.

1. Di bagian atas file kode, di bawah referensi namespace layanan yang ada, temukan komentar **Impor namespace layanan** dan tambahkan kode berikut untuk mengimpor namespace layanan yang akan Anda perlukan untuk menggunakan SDK Analitik Teks:

    ```python
   # import namespaces
   from azure.core.credentials import AzureKeyCredential
   from azure.ai.textanalytics import TextAnalyticsClient
    ```

1. Dalam fungsi **utama**, perhatikan bahwa kode untuk memuat titik akhir dan kunci layanan Azure AI Bahasa serta nama proyek dan penyebaran dari file konfigurasi telah disediakan. Kemudian, temukan komentar **Buat klien menggunakan titik akhir dan kunci**, dan tambahkan kode berikut untuk membuat klien analisis teks:

    ```Python
   # Create client using endpoint and key
   credential = AzureKeyCredential(ai_key)
   ai_client = TextAnalyticsClient(endpoint=ai_endpoint, credential=credential)
    ```

1. Perhatikan bahwa kode yang ada membaca semua file di folder **artikel** dan membuat daftar yang berisi kontennya. Kemudian, temukan komentar **Dapatkan Klasifikasi** dan tambahkan kode berikut:

     ```Python
   # Get Classifications
   operation = ai_client.begin_single_label_classify(
        batchedDocuments,
        project_name=project_name,
        deployment_name=deployment_name
   )

   document_results = operation.result()

   for doc, classification_result in zip(files, document_results):
        if classification_result.kind == "CustomDocumentClassification":
            classification = classification_result.classifications[0]
            print("{} was classified as '{}' with confidence score {}.".format(
                doc, classification.category, classification.confidence_score)
            )
        elif classification_result.is_error is True:
            print("{} has an error with code '{}' and message '{}'".format(
                doc, classification_result.error.code, classification_result.error.message)
            )
    ```

1. Simpan perubahan Anda (CTRL+S), lalu masukkan perintah berikut untuk menjalankan program (Anda memaksimalkan panel cloud shell dan mengubah ukuran panel untuk melihat lebih banyak teks di panel baris perintah):

    ```
   python classify-text.py
    ```

1. Amati output. Aplikasi harus mencantumkan klasifikasi dan skor keyakinan untuk setiap file teks.

## Penghapusan

Jika Anda tidak membutuhkan proyek lagi, Anda dapat menghapusnya dari halaman **Proyek** di Language Studio. Anda juga dapat menghapus layanan Azure AI Bahasa dan akun penyimpanan terkait di [portal Azure](https://portal.azure.com).
