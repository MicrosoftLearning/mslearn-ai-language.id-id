---
lab:
  title: Menerjemahkan Teks
  module: Module 3 - Getting Started with Natural Language Processing
---
{% assign site.title = page.lab.title %}

# Menerjemahkan Teks

**Azure AI Penerjemah** adalah layanan yang memungkinkan Anda menerjemahkan teks antar bahasa. Dalam latihan ini, Anda akan menggunakannya untuk membuat aplikasi sederhana yang menerjemahkan input dalam bahasa apa pun yang didukung ke bahasa target pilihan Anda.

## Memprovisikan sumber daya *Azure AI Penerjemah*

Jika Anda belum memilikinya di langganan, Anda harus menyediakan sumber daya **Azure AI Penerjemah**

1. Buka portal Microsoft Azure di `https://portal.azure.com`, dan masuk menggunakan akun Microsoft yang terkait dengan langganan Azure Anda.
1. Di bidang pencarian di bagian atas, cari **layanan Azure AI** dan tekan **Enter**, lalu pilih **Buat** di bawah **Penerjemah** dalam hasil.
1. Buat sumber daya dengan pengaturan berikut:
    - **Langganan**: *Langganan Azure Anda*
    - **Grup sumber daya**: *Pilih atau buat grup sumber daya*
    - **Wilayah**: *Pilih wilayah yang tersedia*
    - **Nama**: *Masukkan nama unik*
    - **Tingkat harga**: Pilih **F0** (*gratis*), atau **S** (*standar*) jika F tidak tersedia.
    - **Pemberitahuan AI yang Bertanggung Jawab**: Setuju.
1. Pilih **Tinjau + buat**, lalu pilih **Buat** untuk menyediakan sumber daya.
1. Tunggu hingga penyebaran selesai, lalu buka sumber daya yang disebarkan.
1. Tampilkan halaman **Kunci dan Titik Akhir**. Anda akan memerlukan informasi di halaman ini nanti dalam latihan.

## Bersiap untuk mengembangkan aplikasi di Visual Studio Code

Anda akan mengembangkan aplikasi terjemahan teks menggunakan Visual Studio Code. File kode untuk aplikasi Anda telah disediakan dalam repositori GitHub.

> **Tips**: Jika Anda telah mengkloning repositori **mslearn-ai-language**, buka di Visual Studio Code. Jika belum melakukannya, ikuti langkah-langkah berikut untuk mengkloningnya ke lingkungan pengembangan Anda.

1. Memulai Visual Studio Code.
2. Buka palet (SHIFT+CTRL+P) dan jalankan **Git: Perintah klon** untuk mengkloning repositori `https://github.com/MicrosoftLearning/mslearn-ai-language` ke folder lokal (tidak masalah folder mana).
3. Setelah repositori dikloning, buka folder di Visual Studio Code.

    > **Catatan**: Jika Visual Studio Code menampilkan pesan pop-up yang meminta Anda memercayai kode yang Anda buka, klik opsi **Ya, saya memercayai pembuatnya** di pop-up.

4. Tunggu sementara file tambahan diinstal untuk mendukung proyek kode C# di repositori.

    > **Catatan**: Jika Anda diminta untuk menambahkan aset yang diperlukan guna membangun dan men-debug, pilih **Tidak Sekarang**.

## Mengonfigurasi aplikasi Anda

Aplikasi untuk C# dan Python telah disediakan. Kedua aplikasi memiliki fungsionalitas yang sama. Pertama, Anda akan menyelesaikan beberapa bagian utama aplikasi untuk mengaktifkannya menggunakan sumber daya Penerjemah Azure AI Anda.

1. Di Visual Studio Code, di panel **Explorer**, telusuri folder **Labfiles/06b-translator-sdk** Labfiles/06b-translator-sdk dan perluas folder **CSharp** atau **Python** tergantung pada preferensi bahasa Anda dan folder **translate-text** yang ada di dalamnya. Setiap folder berisi file kode khusus bahasa untuk aplikasi tempat Anda akan mengintegrasikan fungsionalitas Penerjemah Azure AI.
2. Klik kanan folder **translate-text** yang berisi file kode Anda dan buka terminal terintegrasi. Kemudian instal paket Azure AI Translator SDK dengan menjalankan perintah yang sesuai untuk preferensi bahasa Anda:

    **C#**:

    ```
    dotnet add package Azure.AI.Translation.Text --version 1.0.0-beta.1
    ```

    **Python**:

    ```
    pip install azure-ai-translation-text==1.0.0b1
    ```

3. Di panel **Penjelajah**, di folder **translate-text**, buka file konfigurasi untuk bahasa pilihan Anda

    - **C#**: appsettings.json
    - **Python**: .env
    
4. Perbarui nilai konfigurasi untuk menyertakan **wilayah** dan **kunci** dari sumber daya Penerjemah Azure AI yang Anda buat (tersedia di halaman **Kunci dan Titik Akhir **untuk sumber daya Penerjemah Azure AI Anda di portal Microsoft Azure).

    > **CATATAN**: Pastikan untuk menambahkan *wilayah* untuk sumber daya Anda, <u>bukan</u> titik akhir!

5. Simpan file konfigurasi.

## Menambahkan kode untuk menerjemahkan teks

Sekarang Anda siap menggunakan Penerjemah Azure AI untuk menerjemahkan teks.

1. Perhatikan bahwa folder **translate-text** berisi file kode untuk aplikasi klien:

    - **C#**: Program.cs
    - **Python**: translate.py

    Buka file kode dan di bagian atas, di bawah referensi namespace yang ada, temukan komentar **Impor namespace**. Kemudian, di bawah komentar ini, tambahkan kode khusus bahasa berikut untuk mengimpor ruang nama yang Anda perlukan untuk menggunakan SDK Analisis Teks:

    **C#**: Programs.cs

    ```csharp
    // import namespaces
    using Azure;
    using Azure.AI.Translation.Text;
    ```

    **Python**: translate.py

    ```python
    # import namespaces
    from azure.ai.translation.text import *
    from azure.ai.translation.text.models import InputTextItem
    ```

1. Dalam fungsi **Utama**, perhatikan bahwa kode yang ada membaca pengaturan konfigurasi.
1. Temukan komentar **Buat klien menggunakan titik akhir dan** kunci dan tambahkan kode berikut:

    **C#**: Programs.cs

    ```csharp
    // Create client using endpoint and key
    AzureKeyCredential credential = new(translatorKey);
    TextTranslationClient client = new(credential, translatorRegion);
    ```

    **Python**: translate.py

    ```python
    # Create client using endpoint and key
    credential = TranslatorCredential(translatorKey, translatorRegion)
    client = TextTranslationClient(credential)
    ```

1. Temukan komentar **Pilih bahasa target** dan tambahkan kode berikut, yang menggunakan layanan Penerjemah Teks untuk mengembalikan daftar bahasa yang didukung untuk terjemahan, dan meminta pengguna untuk memilih kode bahasa untuk bahasa target.

    **C#**: Programs.cs

    ```csharp
    // Choose target language
    Response<GetLanguagesResult> languagesResponse = await client.GetLanguagesAsync(scope:"translation").ConfigureAwait(false);
    GetLanguagesResult languages = languagesResponse.Value;
    Console.WriteLine($"{languages.Translation.Count} languages available.\n(See https://learn.microsoft.com/azure/ai-services/translator/language-support#translation)");
    Console.WriteLine("Enter a target language code for translation (for example, 'en'):");
    string targetLanguage = "xx";
    bool languageSupported = false;
    while (!languageSupported)
    {
        targetLanguage = Console.ReadLine();
        if (languages.Translation.ContainsKey(targetLanguage))
        {
            languageSupported = true;
        }
        else
        {
            Console.WriteLine($"{targetLanguage} is not a supported language.");
        }

    }
    ```

    **Python**: translate.py

    ```python
    # Choose target language
    languagesResponse = client.get_languages(scope="translation")
    print("{} languages supported.".format(len(languagesResponse.translation)))
    print("(See https://learn.microsoft.com/azure/ai-services/translator/language-support#translation)")
    print("Enter a target language code for translation (for example, 'en'):")
    targetLanguage = "xx"
    supportedLanguage = False
    while supportedLanguage == False:
        targetLanguage = input()
        if  targetLanguage in languagesResponse.translation.keys():
            supportedLanguage = True
        else:
            print("{} is not a supported language.".format(targetLanguage))
    ```

1. Temukan komentar **Menerjemahkan teks** dan menambahkan kode berikut, yang berulang kali meminta pengguna untuk teks diterjemahkan, menggunakan layanan Penerjemah Azure AI untuk menerjemahkannya ke bahasa target (mendeteksi bahasa sumber secara otomatis), dan menampilkan hasilnya sampai pengguna memasukkan *keluar*.

    **C#**: Programs.cs

    ```csharp
    // Translate text
    string inputText = "";
    while (inputText.ToLower() != "quit")
    {
        Console.WriteLine("Enter text to translate ('quit' to exit)");
        inputText = Console.ReadLine();
        if (inputText.ToLower() != "quit")
        {
            Response<IReadOnlyList<TranslatedTextItem>> translationResponse = await client.TranslateAsync(targetLanguage, inputText).ConfigureAwait(false);
            IReadOnlyList<TranslatedTextItem> translations = translationResponse.Value;
            TranslatedTextItem translation = translations[0];
            string sourceLanguage = translation?.DetectedLanguage?.Language;
            Console.WriteLine($"'{inputText}' translated from {sourceLanguage} to {translation?.Translations[0].To} as '{translation?.Translations?[0]?.Text}'.");
        }
    } 
    ```

    **Python**: translate.py

    ```python
    # Translate text
    inputText = ""
    while inputText.lower() != "quit":
        inputText = input("Enter text to translate ('quit' to exit):")
        if inputText != "quit":
            input_text_elements = [InputTextItem(text=inputText)]
            translationResponse = client.translate(content=input_text_elements, to=[targetLanguage])
            translation = translationResponse[0] if translationResponse else None
            if translation:
                sourceLanguage = translation.detected_language
                for translated_text in translation.translations:
                    print(f"'{inputText}' was translated from {sourceLanguage.language} to {translated_text.to} as '{translated_text.text}'.")
    ```

1. Simpan perubahan ke file kode Anda.

## Menguji aplikasi Anda

Sekarang aplikasi Anda siap untuk diuji.

1. Di terminal terintegrasi untuk folder **Teks terjemahkan** Terjemahkan, dan masukkan perintah berikut untuk menjalankan program:

    - **C#:** `dotnet run`
    - **Python**: `python translate.py`

    > **Tips**: Anda dapat menggunakan ikon **Maksimalkan ukuran panel** (**^**) di toolbar terminal untuk melihat lebih banyak teks konsol.

1. Saat diminta, masukkan bahasa target yang valid dari daftar yang ditampilkan.
1. Masukkan frasa yang akan diterjemahkan (misalnya `This is a test` atau `C'est un test`) dan lihat hasilnya, yang harus mendeteksi bahasa sumber dan menerjemahkan teks ke bahasa target.
1. Setelah selesai, masukkan `quit`. Anda dapat menjalankan aplikasi lagi dan memilih bahasa target yang berbeda.

## Penghapusan

Saat Anda tidak memerlukan proyek lagi, Anda dapat menghapus sumber daya Penerjemah Azure AI di [portal Microsoft Azure](https://portal.azure.com).
