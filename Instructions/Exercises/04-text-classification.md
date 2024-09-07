---
lab:
  title: Klasifikasi teks kustom
  module: Module 3 - Getting Started with Natural Language Processing
---

# Klasifikasi teks kustom

Azure AI Bahasa menyediakan beberapa kemampuan NLP, termasuk identifikasi frasa kunci, ringkasan teks, dan analisis sentimen. Layanan Bahasa juga menyediakan fitur kustom seperti jawaban atas pertanyaan kustom dan klasifikasi teks kustom.

Untuk menguji klasifikasi teks kustom layanan Azure AI Bahasa, kami akan mengonfigurasi model menggunakan Language Studio, lalu menggunakan aplikasi baris perintah kecil yang berjalan di Cloud Shell untuk mengujinya. Pola dan fungsionalitas yang sama yang digunakan di sini dapat diikuti untuk aplikasi dunia nyata.

## Memprovisikan sumber daya *Azure AI Bahasa*

Jika Anda belum memilikinya di langganan, Anda harus memprovisikan sumber daya **layanan Azure AI Bahasa**. Selain itu, untuk menggunakan klasifikasi teks kustom, Anda perlu mengaktifkan fitur **Klasifikasi teks kustom & ekstraksi**.

1. Di browser, buka portal Microsoft Azure di `https://portal.azure.com`, dan masuk dengan akun Microsoft Anda.
1. Pilih bidang pencarian di bagian atas portal, cari `Azure AI services`, dan buat sumber daya **Layanan Bahasa**.
1. Pilih kotak yang menyertakan **Klasifikasi teks kustom**. Lalu pilih **Lanjutkan untuk membuat sumber daya Anda**.
1. Buat sumber daya dengan pengaturan berikut:
    - **Langganan**: *Langganan Azure Anda*.
    - **Grup sumber daya**: *Pilih atau buat grup sumber daya*.
    - **Wilayah**: *Pilih wilayah yang tersedia*:
    - **Nama**: *Masukkan nama unik*.
    - **Tingkat harga**: Pilih **F0** (*gratis*), atau **S** (*standar*) jika F tidak tersedia.
    - **Storage account**: Akun penyimpanan baru
      - **Nama akun penyimpanan**: *Masukkan nama yang unik*.
      - **Tipe akun penyimpanan**: LRS Standar
    - **Pemberitahuan AI yang bertanggung jawab**: Dipilih.

1. Pilih **Tinjau + buat,** lalu pilih **Buat** untuk memprovisikan sumber daya.
1. Tunggu hingga penyebaran selesai, lalu buka sumber daya yang disebarkan.
1. Tampilkan halaman **Kunci dan Titik Akhir**. Anda akan memerlukan informasi di halaman ini nanti dalam latihan.

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

## Bersiap untuk mengembangkan aplikasi di Visual Studio Code

Untuk menguji kemampuan klasifikasi teks kustom layanan Azure AI Bahasa, Anda akan mengembangkan aplikasi konsol sederhana di Visual Studio Code.

> **Tips**: Jika Anda telah mengkloning repositori **mslearn-ai-language**, buka di Visual Studio Code. Jika belum melakukannya, ikuti langkah-langkah berikut untuk mengkloningnya ke lingkungan pengembangan Anda.

1. Memulai Visual Studio Code.
2. Buka palet (SHIFT+CTRL+P) dan jalankan **Git: Perintah klon** untuk mengkloning repositori `https://github.com/MicrosoftLearning/mslearn-ai-language` ke folder lokal (tidak masalah folder mana).
3. Setelah repositori dikloning, buka folder di Visual Studio Code.

    > **Catatan**: Jika Visual Studio Code menampilkan pesan pop-up yang meminta Anda memercayai kode yang Anda buka, klik opsi **Ya, saya memercayai pembuatnya** di pop-up.

4. Tunggu sementara file tambahan diinstal untuk mendukung proyek kode C# di repositori.

    > **Catatan**: Jika Anda diminta untuk menambahkan aset yang diperlukan guna membangun dan men-debug, pilih **Tidak Sekarang**.

## Mengonfigurasi aplikasi Anda

Aplikasi untuk C# dan Python telah disediakan, beserta contoh file teks yang akan Anda gunakan untuk menguji peringkasan. Kedua aplikasi memiliki fungsionalitas yang sama. Pertama, Anda akan menyelesaikan beberapa bagian utama aplikasi untuk mengaktifkannya menggunakan sumber daya Azure AI Bahasa Anda.

1. Di Visual Studio Code, di panel **Explorer**, telusuri ke folder **Labfiles/04-text-classification** dan perluas folder **CSharp** atau **Python** tergantung pada preferensi bahasa Anda dan folder **classify-text** yang ada di dalamnya. Setiap folder berisi file khusus bahasa untuk aplikasi tempat Anda akan mengintegrasikan fungsionalitas klasifikasi teks Azure AI Bahasa.
1. Klik kanan folder **classify-text** yang berisi file kode Anda dan buka terminal terintegrasi. Kemudian instal paket SDK Analisis Teks Azure AI Bahasa dengan menjalankan perintah yang sesuai untuk preferensi bahasa Anda:

    **C#**:

    ```
    dotnet add package Azure.AI.TextAnalytics --version 5.3.0
    ```

    **Python**:

    ```
    pip install azure-ai-textanalytics==5.3.0
    ```

1. Di panel **Explorer**, di folder **classify-text**, buka file konfigurasi untuk bahasa pilihan Anda

    - **C#**: appsettings.json
    - **Python**: .env
    
1. Perbarui nilai konfigurasi untuk menyertakan **titik akhir** dan **kunci** dari sumber daya Bahasa Azure yang Anda buat (tersedia di halaman **Kunci dan Titik Akhir** untuk sumber daya Azure AI Bahasa Anda di portal Microsoft Azure). File semestinya sudah berisi nama proyek dan penyebaran untuk model klasifikasi teks Anda.
1. Simpan file konfigurasi.

## Menambahkan kode untuk mengklasifikasikan dokumen

Sekarang Anda siap menggunakan layanan Azure AI Bahasa untuk mengklasifikasikan dokumen.

1. Perluas folder **artikel** di dalam folder **classify-text** untuk melihat artikel teks yang akan diklasifikasikan oleh aplikasi Anda.
1. Di folder **classify-text**, buka file kode untuk aplikasi klien:

    - **C#**: Program.cs
    - **Python**: classify-text.py

1. Temukan komentar **Impor namespace**. Kemudian, di bawah komentar ini, tambahkan kode khusus bahasa berikut untuk mengimpor ruang nama yang Anda perlukan untuk menggunakan SDK Analisis Teks:

    **C#**: Programs.cs

    ```csharp
    // import namespaces
    using Azure;
    using Azure.AI.TextAnalytics;
    ```

    **Python**: classify-text.py

    ```python
    # import namespaces
    from azure.core.credentials import AzureKeyCredential
    from azure.ai.textanalytics import TextAnalyticsClient
    ```

1. Dalam fungsi **Utama**, perhatikan bahwa kode untuk memuat titik akhir dan kunci layanan Azure AI Bahasa serta nama proyek dan penyebaran dari file konfigurasi telah disediakan. Kemudian temukan komentar **Buat klien menggunakan titik akhir dan kunci**, dan tambahkan kode berikut untuk membuat klien untuk API Analisis Teks:

    **C#**: Programs.cs

    ```csharp
    // Create client using endpoint and key
    AzureKeyCredential credentials = new AzureKeyCredential(aiSvcKey);
    Uri endpoint = new Uri(aiSvcEndpoint);
    TextAnalyticsClient aiClient = new TextAnalyticsClient(endpoint, credentials);
    ```

    **Python**: classify-text.py

    ```Python
    # Create client using endpoint and key
    credential = AzureKeyCredential(ai_key)
    ai_client = TextAnalyticsClient(endpoint=ai_endpoint, credential=credential)
    ```

1. Dalam fungsi **Utama**, perhatikan bahwa kode yang ada membaca semua file di folder **artikel** dan membuat daftar yang berisi kontennya. Kemudian, temukan komentar **Dapatkan Klasifikasi** dan tambahkan kode berikut:

    **C#**: Program.cs

    ```csharp
    // Get Classifications
    ClassifyDocumentOperation operation = await aiClient.SingleLabelClassifyAsync(WaitUntil.Completed, batchedDocuments, projectName, deploymentName);

    int fileNo = 0;
    await foreach (ClassifyDocumentResultCollection documentsInPage in operation.Value)
    {
        
        foreach (ClassifyDocumentResult documentResult in documentsInPage)
        {
            Console.WriteLine(files[fileNo].Name);
            if (documentResult.HasError)
            {
                Console.WriteLine($"  Error!");
                Console.WriteLine($"  Document error code: {documentResult.Error.ErrorCode}");
                Console.WriteLine($"  Message: {documentResult.Error.Message}");
                continue;
            }

            Console.WriteLine($"  Predicted the following class:");
            Console.WriteLine();

            foreach (ClassificationCategory classification in documentResult.ClassificationCategories)
            {
                Console.WriteLine($"  Category: {classification.Category}");
                Console.WriteLine($"  Confidence score: {classification.ConfidenceScore}");
                Console.WriteLine();
            }
            fileNo++;
        }
    }
    ```
    
    **Python**: classify-text.py

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

1. Simpan perubahan pada file kode Anda.

## Menguji aplikasi Anda

Sekarang aplikasi Anda siap untuk diuji.

1. Di terminal terintegrasi untuk folder **classify-text**, masukkan perintah berikut untuk menjalankan program:

    - **C#:** `dotnet run`
    - **Python**: `python classify-text.py`

    > **Tips**: Anda dapat menggunakan ikon **Maksimalkan ukuran panel** (**^**) di toolbar terminal untuk melihat lebih banyak teks konsol.

1. Amati output. Aplikasi harus mencantumkan klasifikasi dan skor keyakinan untuk setiap file teks.


## Penghapusan

Jika Anda tidak membutuhkan proyek lagi, Anda dapat menghapusnya dari halaman **Proyek** di Language Studio. Anda juga dapat menghapus layanan Azure AI Bahasa dan akun penyimpanan terkait di [portal Azure](https://portal.azure.com).
