---
lab:
  title: Mengekstrak entitas kustom
  description: Latih model untuk mengekstrak entitas yang disesuaikan dari input teks menggunakan Azure AI Bahasa.
---

# Mengekstrak entitas kustom

Selain kemampuan pemrosesan bahasa alami lainnya, Layanan Azure AI Bahasa memungkinkan Anda menentukan entitas kustom, dan mengekstrak instansnya dari teks.

Untuk menguji ekstraksi entitas kustom, kita akan membuat model dan melatihnya melalui Studio Azure AI Bahasa, lalu menggunakan aplikasi Python untuk mengujinya.

Meskipun latihan ini didasarkan pada Python, Anda dapat mengembangkan aplikasi klasifikasi teks menggunakan beberapa SDK khusus bahasa; termasuk:

- [Pustaka klien Analitik Teks Azure AI untuk Python](https://pypi.org/project/azure-ai-textanalytics/)
- [Pustaka klien Analitik Teks Azure AI untuk .NET](https://www.nuget.org/packages/Azure.AI.TextAnalytics)
- [Pustaka klien Analitik Teks Azure AI untuk JavaScript](https://www.npmjs.com/package/@azure/ai-text-analytics)

Latihan ini memakan waktu sekitar **35** menit.

## Provisi sumber daya *Azure AI Bahasa*

Jika Anda belum memilikinya di langganan, Anda harus memprovisikan sumber daya **layanan Azure AI Bahasa**. Selain itu, untuk menggunakan klasifikasi teks kustom, Anda perlu mengaktifkan fitur **Klasifikasi teks kustom & ekstraksi**.

1. Di browser, buka portal Microsoft Azure di `https://portal.azure.com`, dan masuk dengan akun Microsoft Anda.
1. Pilih tombol **Buat sumber daya**, cari *Bahasa*, dan buat sumber daya **Layanan Bahasa**. Saat berada di halaman untuk *Pilih fitur tambahan*, pilih fitur kustom yang berisi **Ekstraksi pengenalan entitas bernama kustom**. Buat sumber daya dengan pengaturan berikut:
    - **Langganan**: *Langganan Azure Anda*
    - **Grup sumber daya**: *Memilih atau membuat grup sumber daya*
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
    - **Nama**: *Masukkan nama unik*
    - **Tingkat harga**: Pilih **F0** (*gratis*), atau **S** (*standar*) jika F tidak tersedia.
    - **Storage account**: Akun penyimpanan baru:
      - **Nama akun penyimpanan**: *Masukkan nama yang unik*.
      - **Tipe akun penyimpanan**: LRS Standar
    - **Pemberitahuan AI yang bertanggung jawab**: Dipilih.

1. Pilih **Tinjau + buat,** lalu pilih **Buat** untuk memprovisikan sumber daya.
1. Tunggu hingga penyebaran selesai, lalu buka sumber daya yang disebarkan.
1. Tampilkan halaman **Kunci dan Titik Akhir**. Anda akan memerlukan informasi di halaman ini nanti dalam latihan.

## Mengonfigurasi akses berbasis peran untuk pengguna Anda

> **CATATAN**: Jika Anda melewati langkah ini, Anda akan mengalami kesalahan 403 ketika mencoba menyambung ke proyek khusus Anda. Penting bahwa pengguna Anda saat ini memiliki peran ini untuk mengakses data blob akun penyimpanan, meskipun Anda adalah pemilik akun penyimpanannya.

1. Buka akun penyimpanan di portal Microsoft Azure.
2. Pilih **Access Control (IAM)** di menu panel navigasi kiri.
3. Pilih **Tambahkan** untuk Menambahkan Penetapan Peran, dan pilih peran **Kontributor Data Blob Penyimpanan** di akun penyimpanan.
4. Dalam **Tetapkan akses ke**, pilih **Pengguna, grup, atau prinsipal layanan**
5. Pilih **Pilih anggota**.
6. Pilih pengguna Anda. Anda dapat mencari nama pengguna di bidang **Pilih**.

## Unggah iklan contoh

Setelah membuat Layanan Azure AI Bahasa dan akun penyimpanan, Anda harus mengunggah contoh iklan untuk melatih model Nanti.

1. Di tab browser baru, unduh sampel iklan rahasia dari `https://aka.ms/entity-extraction-ads` dan ekstrak file ke folder pilihan Anda.

2. Di portal Azure, buka akun penyimpanan yang Anda buat, dan pilih.

3. Di akun penyimpanan Anda pilih **Konfigurasi**, yang terletak di bawah **Pengaturan**, dan layar mengaktifkan opsi untuk **Izinkan akses anonim Blob** lalu pilih **Simpan**.

4. Pilih **Kontainer** dari menu sebelah kiri, yang terletak di bawah **Penyimpanan data**. Pada layar yang muncul, pilih **+ Kontainer**. Beri kontainer nama `classifieds`, dan atur **Tingkat akses anonim** ke **Kontainer (akses baca anonim untuk kontainer dan blob)**.

    > **CATATAN**: Saat Anda mengonfigurasi akun penyimpanan untuk solusi nyata, berhati-hatilah untuk menetapkan tingkat akses yang sesuai. Untuk mempelajari selengkapnya tentang setiap tingkat akses, lihat [dokumentasi Azure Storage](https://learn.microsoft.com/azure/storage/blobs/anonymous-read-access-configure).

5. Setelah membuat kontainer, pilih kontainer dan klik tombol **Unggah** dan unggah iklan sampel yang Anda unduh.

## Membuat proyek pengenalan entitas karakter kustom

Sekarang Anda siap untuk membuat proyek pengenalan entitas bernama kustom. Proyek ini menyediakan tempat kerja untuk membangun, melatih, dan menyebarkan model Anda.

> **CATATAN**: Anda juga dapat membuat, membangun, melatih, dan menyebarkan model Anda melalui REST API.

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
    4. Di bagian atas halaman, klik **Language Studio** untuk kembali ke beranda Language Studio.

1. Di bagian atas portal, di menu **Buat** baru, pilih **Pengenalan entitas bernama kustom**.

1. Buat proyek baru dengan pengaturan berikut:
    - **Menyambungkan penyimpanan**: *Nilai ini kemungkinan sudah terisi. Ubah ke akun penyimpanan Anda jika belum*
    - **Informasi Dasar:**
    - **Nama**: `CustomEntityLab`
        - **Bahasa utama teks**: Inggris (AS)
        - **Apakah himpunan data Anda menyertakan dokumen yang tidak dalam bahasa yang sama?** : *Tidak ada*
        - **Deskripsi**: `Custom entities in classified ads`
    - **Kontainer:**
        - **kontainer penyimpanan Blob**: diklasifikasikan
        - **Apakah file Anda diberi label dengan kelas?**: Tidak, saya perlu memberi label pada file saya sebagai bagian dari proyek ini

> **Tips**: Jika Anda mendapatkan kesalahan tentang tidak berwenang untuk melakukan operasi ini, Anda harus menambahkan penetapan peran. Untuk memperbaikinya, kami menambahkan peran "Kontributor Data Blob Penyimpanan" pada akun penyimpanan untuk pengguna yang menjalankan lab. Rincian lebih lanjut dapat ditemukan [di halaman dokumentasi](https://learn.microsoft.com/azure/ai-services/language-service/custom-named-entity-recognition/how-to/create-project?tabs=portal%2Clanguage-studio#enable-identity-management-for-your-resource).

## Memberi label pada data Anda

Sekarang setelah proyek dibuat, Anda perlu memberi label pada data untuk melatih model cara mengidentifikasi entitas.

1. Jika halaman **Pelabelan data** belum terbuka, di panel di sebelah kiri, pilih **Pelabelan data**. Anda akan melihat daftar file yang Anda unggah ke akun penyimpanan.
1. Di sisi kanan, di panel **Aktivitas**, pilih **Tambahkan** entitas dan tambahkan entitas baru bernama `ItemForSale`.
1.  Ulangi langkah sebelumnya untuk membuat entitas berikut:
    - `Price`
    - `Location`
1. Setelah membuat tiga entitas, pilih **Ad 1.txt** sehingga Anda dapat membacanya.
1. Di *Ad 1.txt*: 
    1. Sorot teks *seikat kayu bakar* dan pilih entitas **ItemForSale**.
    1. Sorot teks *Denver, CO* dan pilih entitas **Lokasi**.
    1. Sorot teks *$90* dan pilih entitas **Harga**.
1. Dalam panel **Aktivitas**, perhatikan bahwa dokumen ini akan ditambahkan ke himpunan data untuk melatih model.
1. Gunakan tombol **Dokumen berikutnya** untuk berpindah ke dokumen berikutnya, dan lanjutkan menetapkan teks ke entitas yang sesuai untuk seluruh kumpulan dokumen, menambahkan semuanya ke himpunan data pelatihan.
1. Ketika Anda telah memberi label dokumen terakhir (*Ad 9.txt*), simpan label.

## Melatih model

Setelah memberi label pada data, Anda perlu melatih model.

1. Pilih **Pekerjaan pelatihan** di panel sebelah kiri.
2. Pilih **Mulai pekerjaan pelatihan**
3. Melatih model baru bernama `ExtractAds`
4. Pilih ** isahkan kumpulan pengujian secara otomatis dari data pelatihan**

    > **TIP**: Dalam proyek ekstraksi Anda sendiri, gunakan pemisahan pengujian yang paling sesuai dengan data Anda. Untuk data yang lebih konsisten dan himpunan data yang lebih besar, Layanan Azure AI Bahasa akan secara otomatis membagi rangkaian pengujian berdasarkan persentase. Dengan kumpulan data yang lebih kecil, penting untuk berlatih dengan berbagai kemungkinan dokumen input yang tepat.

5. Klik **Latih**

    > **PENTING**: Melatih model Anda terkadang bisa memakan waktu beberapa menit. Anda akan mendapatkan pemberitahuan jika sudah selesai.

## Mengevaluasi model Anda

Dalam aplikasi dunia nyata, penting untuk mengevaluasi dan meningkatkan model Anda untuk memverifikasi performanya seperti yang Anda harapkan. Dua halaman di sebelah kiri menunjukkan detail model terlatih Anda, dan pengujian apa pun yang gagal.

Pilih **performa Model** di menu sisi kiri, dan pilih model `ExtractAds` Anda. Di sana Anda dapat melihat penilaian model Anda, metrik performa, dan kapan model tersebut dilatih. Anda akan dapat melihat apakah ada dokumen pengujian yang gagal, dan kegagalan ini membantu Anda memahami bagian mana yang harus ditingkatkan.

## Sebarkan model anda

Saat Anda puas dengan pelatihan model Anda, inilah saatnya untuk menerapkannya, yang lalu memungkinkan Anda untuk mulai mengekstrak entitas melalui API.

1. Di panel kiri, pilih **Menyebarkan model**.
2. Pilih **Tambahkan penyebaran**, lalu masukkan nama `AdEntities` dan pilih model **ExtractAds**.
3. Klik **Sebarkan** untuk menyebarkan model Anda.

## Persiapan untuk mengembangkan aplikasi di Cloud Shell

Untuk menguji kemampuan ekstraksi entitas kustom layanan Azure AI Bahasa, Anda akan mengembangkan aplikasi konsol sederhana di Azure Cloud Shell.

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
    ```

1. After the repo has been cloned, navigate to the folder containing the application code files:  

    ```
    cd mslearn-ai-language/Labfiles/05-custom-entity-recognition/Python/custom-entities
    ```

## Configure your application

1. In the command line pane, run the following command to view the code files in the **custom-entities** folder:

    ```
   ls -a -l
    ```

    The files include a configuration file (**.env**) and a code file (**custom-entities.py**). The text your application will analyze is in the **ads** subfolder.

1. Create a Python virtual environment and install the Azure AI Language Text Analytics SDK package and other required packages by running the following command:

    ```
   python -m venv labenv ./labenv/bin/Activate.ps1 pip install -r requirements.txt azure-ai-textanalytics==5.3.0
    ```

1. Enter the following command to edit the application configuration file:

    ```
   code .env
    ```

    The file is opened in a code editor.

1. Update the configuration values to include the  **endpoint** and a **key** from the Azure Language resource you created (available on the **Keys and Endpoint** page for your Azure AI Language resource in the Azure portal).The file should already contain the project and deployment names for your custom entity extraction model.
1. After you've replaced the placeholders, within the code editor, use the **CTRL+S** command or **Right-click > Save** to save your changes and then use the **CTRL+Q** command or **Right-click > Quit** to close the code editor while keeping the cloud shell command line open.

## Add code to extract entities

1. Enter the following command to edit the application code file:

    ```
    code custom-entities.py
    ```

1. Review the existing code. You will add code to work with the AI Language Text Analytics SDK.

    > **Tip**: As you add code to the code file, be sure to maintain the correct indentation.

1. At the top of the code file, under the existing namespace references, find the comment **Import namespaces** and add the following code to import the namespaces you will need to use the Text Analytics SDK:

    ```python
   # import namespaces
   from azure.core.credentials import AzureKeyCredential
   from azure.ai.textanalytics import TextAnalyticsClient
    ```

1. Dalam fungsi **utama**, perhatikan bahwa kode untuk memuat titik akhir dan kunci layanan Azure AI Bahasa serta nama proyek dan penyebaran dari file konfigurasi telah disediakan. Kemudian, temukan komentar **Buat klien menggunakan titik akhir dan kunci**, dan tambahkan kode berikut untuk membuat klien analitik teks:

    ```Python
   # Create client using endpoint and key
   credential = AzureKeyCredential(ai_key)
   ai_client = TextAnalyticsClient(endpoint=ai_endpoint, credential=credential)
    ```

1. Perhatikan bahwa kode yang ada membaca semua file di folder **iklan** dan membuat daftar yang berisi kontennya. Kemudian, temukan komentar **Ekstrak entitas** dan tambahkan kode berikut:

    ```Python
   # Extract entities
   operation = ai_client.begin_recognize_custom_entities(
        batchedDocuments,
        project_name=project_name,
        deployment_name=deployment_name
   )

   document_results = operation.result()

   for doc, custom_entities_result in zip(files, document_results):
        print(doc)
        if custom_entities_result.kind == "CustomEntityRecognition":
            for entity in custom_entities_result.entities:
                print(
                    "\tEntity '{}' has category '{}' with confidence score of '{}'".format(
                        entity.text, entity.category, entity.confidence_score
                    )
                )
        elif custom_entities_result.is_error is True:
            print("\tError with code '{}' and message '{}'".format(
                custom_entities_result.error.code, custom_entities_result.error.message
                )
            )
    ```

1. Simpan perubahan Anda (CTRL+S), lalu masukkan perintah berikut untuk menjalankan program (Anda memaksimalkan panel cloud shell dan mengubah ukuran panel untuk melihat lebih banyak teks di panel baris perintah):

    ```
   python custom-entities.py
    ```

1. Amati output. Aplikasi harus mencantumkan detail entitas yang ditemukan di setiap file teks.

## Penghapusan

Jika Anda tidak membutuhkan proyek lagi, Anda dapat menghapusnya dari halaman **Proyek** di Language Studio. Anda juga dapat menghapus layanan Azure AI Bahasa dan akun penyimpanan terkait di [portal Azure](https://portal.azure.com).
