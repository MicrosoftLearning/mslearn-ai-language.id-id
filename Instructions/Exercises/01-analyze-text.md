---
lab:
  title: Analisa teks
  module: Module 3 - Develop natural language processing solutions
---

# Menganalisis Teks

**Azure Language** mendukung analisis teks, termasuk deteksi bahasa, analisis sentimen, ekstraksi frasa kunci, dan pengenalan entitas.

Misalnya, agen perjalanan ingin memproses ulasan hotel yang telah dikirimkan ke situs web perusahaan. Dengan menggunakan layanan Azure AI Bahasa, mereka dapat menentukan bahasa yang digunakan untuk menulis setiap ulasan, sentimen (positif, netral, atau negatif) dari ulasan, frasa kunci yang mungkin menunjukkan topik utama yang dibahas dalam ulasan, dan entitas bernama, seperti tempat, landmark, atau orang yang disebutkan dalam ulasan.

## Memprovisikan sumber daya * Azure AI Bahasa*

Jika Anda belum memilikinya di langganan, Anda harus menyediakan sumber daya **layanan Azure AI Bahasa** di langganan Azure Anda.

1. Buka portal Microsoft Azure di `https://portal.azure.com`, dan masuk menggunakan akun Microsoft yang terkait dengan langganan Azure Anda.
1. Di bidang pencarian di bagian atas, cari **layanan Azure AI**. Kemudian, dalam hasil, pilih **Buat** di bawah **Layanan Bahasa**.
1. Pilih **Lanjutkan untuk membuat sumber daya Anda**.
1. Provisikan sumber daya menggunakan pengaturan berikut:
    - **Langganan**: *Langganan Azure Anda*.
    - **Grup sumber daya**: *Memilih atau membuat grup sumber daya*.
    - **Wilayah**: *Pilih wilayah yang tersedia*
    - **Nama**: *Masukkan nama unik*.
    - **Tingkat harga**: Pilih **F0** (*gratis*), atau **S** (*standar*) jika F tidak tersedia.
    - **Pemberitahuan AI yang Bertanggung Jawab**: Setuju.
1. Pilih **Tinjau + buat**, lalu pilih **Buat** untuk memprovisikan sumber daya.
1. Tunggu hingga penyebaran selesai, lalu buka sumber daya yang disebarkan.
1. Lihat halaman **Kunci dan Titik Akhir**. Anda akan memerlukan informasi di halaman ini nanti dalam latihan.

## Bersiap untuk mengembangkan aplikasi di Visual Studio Code

Anda akan mengembangkan aplikasi analitik teks menggunakan Visual Studio Code. File kode untuk aplikasi Anda telah disediakan di repositori GitHub.

> **Tips**: Jika Anda telah mengkloning repositori **mslearn-ai-language**, buka di Visual Studio Code. Jika tidak, ikuti langkah-langkah ini untuk mengkloningnya ke lingkungan pengembangan Anda.

1. Memulai Visual Studio Code.
2. Buka palet (SHIFT+CTRL+P) dan jalankan **Git: Perintah klon** untuk mengkloning repositori `https://github.com/MicrosoftLearning/mslearn-ai-language` ke folder lokal (tidak masalah folder mana).
3. Setelah repositori dikloning, buka folder di Visual Studio Code.

    > **Catatan**: Jika Visual Studio Code menampilkan pesan pop-up untuk meminta Anda mempercayai kode yang Anda buka, klik **Ya, saya mempercayai opsi** penulis di pop-up.

4. Tunggu sementara file tambahan diinstal untuk mendukung proyek kode C# di repositori.

    > **Catatan**: Jika Anda diminta untuk menambahkan aset yang diperlukan guna membangun dan men-debug, pilih **Tidak Sekarang**.

## Mengonfigurasi aplikasi Anda

Aplikasi untuk C# dan Python telah disediakan, serta sampel file teks yang akan Anda gunakan untuk menguji ringkasan. Kedua aplikasi memiliki fungsionalitas yang sama. Pertama, Anda akan menyelesaikan beberapa bagian utama aplikasi untuk mengaktifkannya menggunakan sumber daya Azure AI Bahasa Anda.

1. Di Visual Studio Code, di panel **Explorer**, telusuri folder **Labfiles/01-analyze-text** Labfiles/06b-translator-sdk dan perluas folder **CSharp** atau **Python** tergantung pada preferensi bahasa Anda dan folder **text-analysis** yang ada di dalamnya. Setiap folder berisi file khusus bahasa untuk aplikasi tempat Anda akan mengintegrasikan fungsionalitas analitik teks Azure AI Bahasa.
2. Klik kanan folder **text-analysis** yang berisi file kode Anda dan buka terminal terintegrasi. Kemudian instal paket SDK Analisis Teks Azure AI Bahasa dengan menjalankan perintah yang sesuai untuk preferensi bahasa Anda. Untuk latihan Python, instal juga paket `dotenv`:

    **C#**:

    ```
    dotnet add package Azure.AI.TextAnalytics --version 5.3.0
    ```

    **Python**:

    ```
    pip install azure-ai-textanalytics==5.3.0
    pip install python-dotenv
    ```

3. Di panel **Penjelajah**, di folder **text-analysis**, buka file konfigurasi untuk bahasa pilihan Anda

    - **C#**: appsettings.json
    - **Python**: .env
    
4. Perbarui nilai konfigurasi untuk menyertakan **titik akhir** dan **kunci** dari sumber daya Bahasa Azure yang Anda buat (tersedia di halaman **Kunci dan Titik Akhir** untuk sumber daya Azure AI Bahasa Anda di portal Microsoft Azure)
5. Simpan file konfigurasi.

6. Perhatikan bahwa folder **text-analysis** berisi file kode untuk aplikasi klien:

    - **C#**: Program.cs
    - **Python**: text-analysis.py

    Buka file kode dan di bagian atas, di bawah referensi namespace yang ada, temukan komentar **Impor namespace**. Kemudian, di bawah komentar ini, tambahkan kode khusus bahasa berikut untuk mengimpor ruang nama yang Anda perlukan untuk menggunakan SDK Analisis Teks:

    **C#**: Programs.cs

    ```csharp
    // import namespaces
    using Azure;
    using Azure.AI.TextAnalytics;
    ```

    **Python**: text-analysis.py

    ```python
    # import namespaces
    from azure.core.credentials import AzureKeyCredential
    from azure.ai.textanalytics import TextAnalyticsClient
    ```

7. Dalam fungsi **Utama**, perhatikan bahwa kode untuk memuat titik akhir dan kunci layanan Azure AI Bahasa dari file konfigurasi telah disediakan. Kemudian temukan komentar **Buat klien menggunakan titik akhir dan kunci**, dan tambahkan kode berikut untuk membuat klien untuk API Analisis Teks:

    **C#**: Programs.cs

    ```C#
    // Create client using endpoint and key
    AzureKeyCredential credentials = new AzureKeyCredential(aiSvcKey);
    Uri endpoint = new Uri(aiSvcEndpoint);
    TextAnalyticsClient aiClient = new TextAnalyticsClient(endpoint, credentials);
    ```

    **Python**: text-analysis.py

    ```Python
    # Create client using endpoint and key
    credential = AzureKeyCredential(ai_key)
    ai_client = TextAnalyticsClient(endpoint=ai_endpoint, credential=credential)
    ```

8. Simpan perubahan Anda dan kembali ke terminal terintegrasi untuk folder **text-analysis**, dan masukkan perintah berikut untuk menjalankan program:

    - **C#:** `dotnet run`
    - **Python**: `python text-analysis.py`

    > **Tips**: Anda dapat menggunakan ikon **Maksimalkan ukuran panel** (**^**) di toolbar terminal untuk melihat lebih banyak teks konsol.

9. Amati keluaran saat kode harus berjalan tanpa kesalahan, menampilkan konten setiap file teks ulasan dalam folder **ulasan**. Aplikasi berhasil membuat klien untuk Text Analytics API tetapi tidak menggunakannya. Kami akan memperbaikinya di prosedur selanjutnya.

## Menambahkan kode untuk mendeteksi bahasa

Sekarang setelah Anda membuat klien untuk API, mari kita gunakan untuk mendeteksi bahasa yang digunakan untuk menulis setiap ulasan.

1. Dalam fungsi **Utama** untuk program Anda, temukan komentar **Dapatkan bahasa**. Kemudian, di bawah komentar ini, tambahkan kode yang diperlukan untuk mendeteksi bahasa di setiap dokumen ulasan:

    **C#**: Programs.cs

    ```csharp
    // Get language
    DetectedLanguage detectedLanguage = aiClient.DetectLanguage(text);
    Console.WriteLine($"\nLanguage: {detectedLanguage.Name}");
    ```

    **Python**: text-analysis.py

    ```python
    # Get language
    detectedLanguage = ai_client.detect_language(documents=[text])[0]
    print('\nLanguage: {}'.format(detectedLanguage.primary_language.name))
    ```

     > **Catatan**: *Dalam contoh ini, setiap ulasan dianalisis satu per satu, menghasilkan panggilan terpisah ke layanan untuk setiap file. Pendekatan alternatif adalah membuat kumpulan dokumen dan meneruskannya ke layanan dalam satu panggilan. Dalam kedua pendekatan tersebut, tanggapan dari layanan terdiri dari kumpulan dokumen; itulah sebabnya dalam kode Python di atas, indeks dokumen pertama (dan satu-satunya) dalam respons ([0]) ditentukan.*

1. Simpan perubahan Anda. Kemudian, kembali ke terminal terintegrasi untuk membuka folder **text-analysis**, dan jalankan kembali program.
1. Amati hasilnya, perhatikan bahwa kali ini bahasa untuk setiap ulasan diidentifikasi.

## Tambahkan kode untuk mengevaluasi sentimen

*Analisis sentimen* adalah teknik yang umum digunakan untuk mengklasifikasikan teks sebagai *positif* atau *negatif* (atau kemungkinan *netral* atau *campuran* ). Ini biasanya digunakan untuk menganalisis posting media sosial, ulasan produk, dan item lain di mana sentimen teks dapat memberikan wawasan yang berguna.

1. Dalam fungsi **Utama** untuk program Anda, temukan komentar **Dapatkan sentimen**. Kemudian, di bawah komentar ini, tambahkan kode yang diperlukan untuk mendeteksi sentimen dari setiap dokumen ulasan:

    **C#**: Program.cs

    ```csharp
    // Get sentiment
    DocumentSentiment sentimentAnalysis = aiClient.AnalyzeSentiment(text);
    Console.WriteLine($"\nSentiment: {sentimentAnalysis.Sentiment}");
    ```

    **Python**: text-analysis.py

    ```python
    # Get sentiment
    sentimentAnalysis = ai_client.analyze_sentiment(documents=[text])[0]
    print("\nSentiment: {}".format(sentimentAnalysis.sentiment))
    ```

1. Simpan perubahan Anda. Kemudian, kembali ke terminal terintegrasi untuk membuka folder **text-analysis**, dan jalankan kembali program.
1. Amati output, perhatikan bahwa sentimen ulasan terdeteksi.

## Tambahkan kode untuk mengidentifikasi frasa kunci

Akan berguna untuk mengidentifikasi frase kunci dalam tubuh teks untuk membantu menentukan topik utama yang dibahas.

1. Dalam fungsi **Utama** untuk program Anda, temukan komentar **Dapatkan frasa kunci**. Kemudian, di bawah komentar ini, tambahkan kode yang diperlukan untuk mendeteksi frasa kunci di setiap dokumen ulasan:

    **C#**: Program.cs

    ```csharp
    // Get key phrases
    KeyPhraseCollection phrases = aiClient.ExtractKeyPhrases(text);
    if (phrases.Count > 0)
    {
        Console.WriteLine("\nKey Phrases:");
        foreach(string phrase in phrases)
        {
            Console.WriteLine($"\t{phrase}");
        }
    }
    ```

    **Python**: text-analysis.py

    ```python
    # Get key phrases
    phrases = ai_client.extract_key_phrases(documents=[text])[0].key_phrases
    if len(phrases) > 0:
        print("\nKey Phrases:")
        for phrase in phrases:
            print('\t{}'.format(phrase))
    ```

1. Simpan perubahan Anda. Kemudian, kembali ke terminal terintegrasi untuk membuka folder **text-analysis**, dan jalankan kembali program.
1. Amati hasilnya, perhatikan bahwa setiap dokumen berisi frasa kunci yang memberikan beberapa wawasan tentang ulasan tersebut.

## Tambahkan kode untuk mengekstrak entitas

Sering kali, dokumen atau kumpulan teks lainnya menyebutkan orang, tempat, periode waktu, atau entitas lain. API Analytics teks dapat mendeteksi beberapa kategori (dan sub-kategori) entitas dalam teks Anda.

1. Dalam fungsi **Utama** untuk program Anda, temukan komentar **Dapatkan entitas**. Kemudian, di bawah komentar ini, tambahkan kode yang diperlukan untuk mengidentifikasi entitas yang disebutkan di setiap ulasan:

    **C#**: Program.cs

    ```csharp
    // Get entities
    CategorizedEntityCollection entities = aiClient.RecognizeEntities(text);
    if (entities.Count > 0)
    {
        Console.WriteLine("\nEntities:");
        foreach(CategorizedEntity entity in entities)
        {
            Console.WriteLine($"\t{entity.Text} ({entity.Category})");
        }
    }
    ```

    **Python**: text-analysis.py

    ```python
    # Get entities
    entities = ai_client.recognize_entities(documents=[text])[0].entities
    if len(entities) > 0:
        print("\nEntities")
        for entity in entities:
            print('\t{} ({})'.format(entity.text, entity.category))
    ```

1. Simpan perubahan Anda. Kemudian, kembali ke terminal terintegrasi untuk membuka folder **text-analysis**, dan jalankan kembali program.
1. Amati keluarannya, perhatikan entitas yang telah terdeteksi dalam teks.

## Tambahkan kode untuk mengekstrak entitas tertaut

Selain entitas yang dikategorikan, Text Analytics API dapat mendeteksi entitas yang diketahui memiliki tautan ke sumber data, seperti Wikipedia.

1. Dalam fungsi **Utama** untuk program Anda, temukan komentar **Dapatkan entitas tertaut**. Kemudian, di bawah komentar ini, tambahkan kode yang diperlukan untuk mengidentifikasi entitas terkait yang disebutkan di setiap ulasan:

    **C#**: Program.cs

    ```csharp
    // Get linked entities
    LinkedEntityCollection linkedEntities = aiClient.RecognizeLinkedEntities(text);
    if (linkedEntities.Count > 0)
    {
        Console.WriteLine("\nLinks:");
        foreach(LinkedEntity linkedEntity in linkedEntities)
        {
            Console.WriteLine($"\t{linkedEntity.Name} ({linkedEntity.Url})");
        }
    }
    ```

    **Python**: text-analysis.py

    ```python
    # Get linked entities
    entities = ai_client.recognize_linked_entities(documents=[text])[0].entities
    if len(entities) > 0:
        print("\nLinks")
        for linked_entity in entities:
            print('\t{} ({})'.format(linked_entity.name, linked_entity.url))
    ```

1. Simpan perubahan Anda. Kemudian, kembali ke terminal terintegrasi untuk membuka folder **text-analysis**, dan jalankan kembali program.
1. Amati output, catat entitas terkait yang diidentifikasi.

## Membersihkan sumber daya

Setelah menjelajahi layanan Azure AI Bahasa, Anda dapat menghapus sumber daya yang dibuat dalam latihan ini. Berikut caranya:

1. Buka portal Microsoft Azure di `https://portal.azure.com`, dan masuk menggunakan akun Microsoft yang terkait dengan langganan Azure Anda.

2. Telusuri ke sumber daya Azure AI Bahasa yang Anda buat di lab ini.

3. Pada halaman sumber daya, pilih **Hapus** dan ikuti instruksi untuk menghapus sumber daya.

## Informasi selengkapnya

Untuk informasi selengkapnya tentang menggunakan **Azure AI Bahasa**, lihat [dokumentasi](https://learn.microsoft.com/azure/ai-services/language-service/).
