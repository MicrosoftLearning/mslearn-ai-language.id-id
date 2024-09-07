---
lab:
  title: Mengekstrak entitas kustom
  module: Module 3 - Getting Started with Natural Language Processing
---

# Mengekstrak entitas kustom

Selain kemampuan pemrosesan bahasa alami lainnya, Layanan Azure AI Bahasa memungkinkan Anda menentukan entitas kustom, dan mengekstrak instansnya dari teks.

Untuk menguji ekstraksi entitas kustom, kita akan membuat model dan melatihnya melalui Studio Azure AI Bahasa, lalu menggunakan aplikasi baris perintah untuk mengujinya.

## Memprovisikan sumber daya * Azure AI Bahasa*

Jika Anda belum memilikinya di langganan, Anda harus memprovisikan sumber daya **layanan Azure AI Bahasa**. Selain itu, untuk menggunakan klasifikasi teks kustom, Anda perlu mengaktifkan fitur **Klasifikasi teks kustom & ekstraksi**.

1. Di browser, buka portal Microsoft Azure di `https://portal.azure.com`, dan masuk dengan akun Microsoft Anda.
1. Pilih tombol **Buat sumber daya**, cari *Bahasa*, dan buat sumber daya **Layanan Bahasa**. Saat berada di halaman untuk *Pilih fitur tambahan*, pilih fitur kustom yang berisi **Ekstraksi pengenalan entitas bernama kustom**. Buat sumber daya dengan pengaturan berikut:
    - **Langganan**: *Langganan Azure Anda*
    - **Grup sumber daya**: *Memilih atau membuat grup sumber daya*
    - **Wilayah**: *Pilih wilayah yang tersedia*
    - **Nama**: *Masukkan nama unik*
    - **Tingkat harga**: Pilih **F0** (*gratis*), atau **S** (*standar*) jika F tidak tersedia.
    - **Storage account**: Akun penyimpanan baru:
      - **Nama akun penyimpanan**: *Masukkan nama yang unik*.
      - **Tipe akun penyimpanan**: LRS Standar
    - **Pemberitahuan AI yang bertanggung jawab**: Dipilih.

1. Pilih **Tinjau + buat,** lalu pilih **Buat** untuk memprovisikan sumber daya.
1. Tunggu hingga penyebaran selesai, lalu buka sumber daya yang disebarkan.
1. Tampilkan halaman **Kunci dan Titik Akhir**. Anda akan memerlukan informasi di halaman ini nanti dalam latihan.

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
1.  Ulangi langkah sebelumnya o buat entitas berikut:
    - `Price`
    - `Location`
1. Setelah membuat tiga entitas, pilih **Ad 1.txt** sehingga Anda dapat membacanya.
1. Di *Ad 1.txt*: 
    1. Sorot teks *seikat kayu bakar* dan pilih entitas **ItemForSale**.
    1. Sorot teks *Denver, CO* dan pilih entitas **Lokasi**.
    1. Sorot teks *$90* dan pilih entitas **Harga**.
1.In panel **Aktivitas**, perhatikan bahwa dokumen ini akan ditambahkan ke himpunan data untuk melatih model.
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

## Bersiap untuk mengembangkan aplikasi di Visual Studio Code

Untuk menguji kemampuan ekstraksi entitas kustom layanan Azure AI Bahasa, Anda akan mengembangkan aplikasi konsol sederhana di Visual Studio Code.

> **Tips**: Jika Anda telah mengkloning repositori **mslearn-ai-language**, buka di kode Visual Studio. Jika belum melakukannya, ikuti langkah-langkah berikut untuk mengkloningnya ke lingkungan pengembangan Anda.

1. Memulai Visual Studio Code.
2. Buka palet (SHIFT+CTRL+P) dan jalankan **Git: Perintah klon** untuk mengkloning repositori `https://github.com/MicrosoftLearning/mslearn-ai-language` ke folder lokal (tidak masalah folder mana).
3. Setelah repositori dikloning, buka folder di Visual Studio Code.

    > **Catatan**: Jika Visual Studio Code menampilkan pesan pop-up yang meminta Anda memercayai kode yang Anda buka, klik opsi **Ya, saya memercayai pembuatnya** di pop-up.

4. Tunggu sementara file tambahan diinstal untuk mendukung proyek kode C# di repositori.

    > **Catatan**: Jika Anda diminta untuk menambahkan aset yang diperlukan guna membangun dan men-debug, pilih **Tidak Sekarang**.

## Mengonfigurasi aplikasi Anda

Aplikasi untuk C# dan Python telah disediakan. Kedua aplikasi memiliki fungsionalitas yang sama. Pertama, Anda akan menyelesaikan beberapa bagian utama aplikasi untuk mengaktifkannya menggunakan sumber daya Azure AI Bahasa Anda.

1. Di Visual Studio Code, di panel **Penjelajah**, telusuri folder **Labfiles/05-custom-entity-recognition** dan perluas folder **CSharp** atau **Python** tergantung pada preferensi bahasa Anda dan **entitas kustom** folder yang ada di dalamnya. Setiap folder berisi file khusus bahasa untuk aplikasi tempat Anda akan mengintegrasikan fungsionalitas klasifikasi teks Azure AI Bahasa.
1. Klik kanan folder **entitas-kustom** yang berisi file kode Anda dan buka terminal terintegrasi. Kemudian instal paket SDK Analisis Teks Azure AI Bahasa dengan menjalankan perintah yang sesuai untuk preferensi bahasa Anda:

    **C#**:

    ```
    dotnet add package Azure.AI.TextAnalytics --version 5.3.0
    ```

    **Python**:

    ```
    pip install azure-ai-textanalytics==5.3.0
    ```

1. Di panel **Penjelajah**, di folder **entitas-kustom**, buka file konfigurasi untuk bahasa pilihan Anda

    - **C#**: appsettings.json
    - **Python**: .env
    
1. Perbarui nilai konfigurasi untuk menyertakan **titik akhir** dan **kunci** dari sumber daya Bahasa Azure yang Anda buat (tersedia di halaman **Kunci dan Titik Akhir** untuk sumber daya Azure AI Bahasa Anda di portal Microsoft Azure). File harus sudah berisi nama proyek dan penyebaran untuk model ekstraksi entitas kustom Anda.
1. Simpan file konfigurasi.

## Tambahkan kode untuk mengekstrak entitas

Sekarang Anda siap untuk menggunakan layanan Azure AI Bahasa untuk mengekstrak entitas kustom dari teks.

1. Perluas folder **iklan** di folder **entitas kustom** untuk melihat iklan rahasia yang akan dianalisis aplikasi Anda.
1. Di folder **entitas-kustom**, buka file kode untuk aplikasi klien:

    - **C#**: Program.cs
    - **Python**: custom-entities.py

1. Temukan komentar **Impor namespace**. Kemudian, di bawah komentar ini, tambahkan kode khusus bahasa berikut untuk mengimpor ruang nama yang Anda perlukan untuk menggunakan SDK Analisis Teks:

    **C#**: Programs.cs

    ```csharp
    // import namespaces
    using Azure;
    using Azure.AI.TextAnalytics;
    ```

    **Python**: custom-entities.py

    ```python
    # import namespaces
    from azure.core.credentials import AzureKeyCredential
    from azure.ai.textanalytics import TextAnalyticsClient
    ```

1. Dalam fungsi **Utama**, perhatikan bahwa kode untuk memuat titik akhir dan kunci layanan Azure AI Bahasa serta nama proyek dan penyebaran dari file konfigurasi telah disediakan. Kemudian temukan komentar **Buat klien menggunakan titik akhir dan kunci**, dan tambahkan kode berikut untuk membuat klien untuk API Analisis Teks:

    **C#**: Programs.cs

    ```csharp
    // Create client using endpoint and key
    AzureKeyCredential credentials = new(aiSvcKey);
    Uri endpoint = new(aiSvcEndpoint);
    TextAnalyticsClient aiClient = new(endpoint, credentials);
    ```

    **Python**: custom-entities.py

    ```Python
    # Create client using endpoint and key
    credential = AzureKeyCredential(ai_key)
    ai_client = TextAnalyticsClient(endpoint=ai_endpoint, credential=credential)
    ```

1. Dalam fungsi **Utama**, perhatikan bahwa kode yang ada membaca semua file di folder **iklan** dan membuat daftar yang berisi kontennya. Dalam kasus kode C#, daftar objek **TextDocumentInput** digunakan untuk menyertakan nama file sebagai ID dan bahasa. Di Python, daftar sederhana konten teks digunakan.
1. Temukan komentar **Ekstrak entitas** dan tambahkan kode berikut:

    **C#**: Program.cs

    ```csharp
    // Extract entities
    RecognizeCustomEntitiesOperation operation = await aiClient.RecognizeCustomEntitiesAsync(WaitUntil.Completed, batchedDocuments, projectName, deploymentName);

    await foreach (RecognizeCustomEntitiesResultCollection documentsInPage in operation.Value)
    {
        foreach (RecognizeEntitiesResult documentResult in documentsInPage)
        {
            Console.WriteLine($"Result for \"{documentResult.Id}\":");

            if (documentResult.HasError)
            {
                Console.WriteLine($"  Error!");
                Console.WriteLine($"  Document error code: {documentResult.Error.ErrorCode}");
                Console.WriteLine($"  Message: {documentResult.Error.Message}");
                Console.WriteLine();
                continue;
            }

            Console.WriteLine($"  Recognized {documentResult.Entities.Count} entities:");

            foreach (CategorizedEntity entity in documentResult.Entities)
            {
                Console.WriteLine($"  Entity: {entity.Text}");
                Console.WriteLine($"  Category: {entity.Category}");
                Console.WriteLine($"  Offset: {entity.Offset}");
                Console.WriteLine($"  Length: {entity.Length}");
                Console.WriteLine($"  ConfidenceScore: {entity.ConfidenceScore}");
                Console.WriteLine($"  SubCategory: {entity.SubCategory}");
                Console.WriteLine();
            }

            Console.WriteLine();
        }
    }
    ```

    **Python**: custom-entities.py

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

1. Simpan perubahan ke file kode Anda.

## Menguji aplikasi Anda

Sekarang aplikasi Anda siap untuk diuji.

1. Di terminal terintegrasi untuk folder **klasifikasi-teks** masukkan perintah berikut untuk menjalankan program:

    - **C#:** `dotnet run`
    - **Python**: `python custom-entities.py`

    > **Tips**: Anda dapat menggunakan ikon **Maksimalkan ukuran panel** (**^**) di toolbar terminal untuk melihat lebih banyak teks konsol.

1. Amati output. Aplikasi harus mencantumkan detail entitas yang ditemukan di setiap file teks.

## Penghapusan

Jika Anda tidak membutuhkan proyek lagi, Anda dapat menghapusnya dari halaman **Proyek** di Language Studio. Anda juga dapat menghapus layanan Azure AI Bahasa dan akun penyimpanan terkait di [portal Azure](https://portal.azure.com).
