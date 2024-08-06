---
lab:
  title: Menerjemahkan Ucapan
  module: Module 8 - Translate speech with Azure AI Speech
---

# Menerjemahkan Ucapan

Azure AI Speech menyertakan API terjemahan ucapan yang dapat Anda gunakan untuk menerjemahkan bahasa lisan. Misalnya, Anda ingin mengembangkan aplikasi penerjemah yang dapat digunakan orang saat bepergian di tempat-tempat di mana mereka tidak berbicara bahasa lokal. Mereka akan dapat mengucapkan frasa seperti "Di mana stasiunnya?" atau "Saya perlu mencari apotek" dalam bahasa mereka sendiri, dan memintanya menerjemahkannya ke bahasa lokal.

> **CATATAN** Latihan ini mengharuskan Anda menggunakan komputer dengan speaker/headphone. Untuk pengalaman terbaik, mikrofon juga diperlukan. Beberapa lingkungan virtual yang dihosting mungkin dapat menangkap audio dari mikrofon lokal Anda, tetapi jika ini tidak berhasil (atau Anda tidak memiliki mikrofon sama sekali), Anda dapat menggunakan file audio yang disediakan untuk input ucapan. Ikuti petunjuknya dengan cermat, karena Anda harus memilih opsi yang berbeda tergantung pada apakah Anda menggunakan mikrofon atau file audio.

## Provisikan sumber daya *Azure AI Speech*

Jika Anda belum memilikinya dalam langganan, Anda harus menyediakan sumber daya **Azure AI Speech**.

1. Buka portal Microsoft Azure di `https://portal.azure.com`, dan masuk menggunakan akun Microsoft yang terkait dengan langganan Azure Anda.
1. Di bidang pencarian di bagian atas, cari **layanan Azure AI** dan tekan **Enter**, lalu pilih **Buat** di bawah **Layanan ucapan** dalam hasil.
1. Buat sumber daya dengan pengaturan berikut:
    - **Langganan**: *Langganan Azure Anda*
    - **Grup sumber daya**: *Pilih atau buat grup sumber daya*
    - **Wilayah**: *Pilih wilayah yang tersedia*
    - **Nama**: *Masukkan nama unik*
    - **Tingkat harga**: Pilih **F0** (*gratis*), atau **S** (*standar*) jika F tidak tersedia.
    - **Pemberitahuan AI yang Bertanggung Jawab**: Setuju.
1. Pilih **Tinjau + buat**, pilih **Buat** untuk menyediakan sumber daya.
1. Tunggu hingga penyebaran selesai, lalu buka sumber daya yang disebarkan.
1. Tampilkan halaman **Kunci dan Titik Akhir**. Anda akan memerlukan informasi di halaman ini nanti dalam latihan.

## Bersiap untuk mengembangkan aplikasi di Visual Studio Code

Anda akan mengembangkan aplikasi ucapan menggunakan Visual Studio Code. File kode untuk aplikasi Anda telah disediakan dalam repositori GitHub.

> **Tips**: Jika Anda telah mengkloning repositori **mslearn-ai-language**, buka di Visual Studio code. Jika tidak, ikuti langkah-langkah ini untuk mengkloningnya ke lingkungan pengembangan Anda.

1. Memulai Visual Studio Code.
1. Buka palet (SHIFT+CTRL+P) dan jalankan **Git: Perintah klon** untuk mengkloning repositori `https://github.com/MicrosoftLearning/mslearn-ai-language` ke folder lokal (tidak masalah folder mana).
1. Setelah repositori dikloning, buka folder di Visual Studio Code.

    > **Catatan**: Jika Visual Studio Code menampilkan pesan pop-up untuk meminta Anda memercayai kode yang Anda buka, klik opsi **Ya, saya memercayai penulis** di pop-up tersebut.

1. Tunggu sementara file tambahan diinstal untuk mendukung proyek kode C# di repositori.

    > **Catatan**: Jika Anda diminta untuk menambahkan aset yang diperlukan guna membangun dan men-debug, pilih **Tidak Sekarang**.

## Mengonfigurasi aplikasi Anda

Aplikasi untuk C# dan Python telah disediakan. Kedua aplikasi memiliki fungsionalitas yang sama. Pertama, Anda akan menyelesaikan beberapa bagian kunci aplikasi untuk mengaktifkannya menggunakan sumber daya Azure AI Speech Anda.

1. Di Visual Studio Code, di panel **Penjelajah**, telusuri ke folder **Labfiles/08-speech-translation** dan luaskan folder **CSharp** atau **Python** tergantung pada preferensi bahasa Anda dan folder **translator** yang ada di dalamnya. Setiap folder berisi file kode khusus bahasa untuk aplikasi tempat Anda akan mengintegrasikan fungsionalitas Azure AI Speech.
1. Klik kanan folder **translator** yang berisi file kode Anda dan buka terminal terintegrasi. Kemudian instal paket SDK Azure AI Speech dengan menjalankan perintah yang sesuai untuk preferensi bahasa Anda:

    **C#**

    ```
    dotnet add package Microsoft.CognitiveServices.Speech --version 1.30.0
    ```

    **Python**

    ```
    pip install azure-cognitiveservices-speech==1.30.0
    ```

1. Di panel **Penjelajah**, di folder **translator**, buka file konfigurasi untuk bahasa pilihan Anda

    - **C#**: appsettings.json
    - **Python**: .env

1. Perbarui nilai konfigurasi untuk menyertakan  **wilayah** dan **kunci** dari sumber daya Azure AI Speech yang Anda buat (tersedia di halaman **Kunci dan Titik Akhir** untuk sumber daya Azure AI Speech Anda di portal Microsoft Azure).

    > **CATATAN**: Pastikan untuk menambahkan *wilayah* untuk sumber daya Anda, <u>bukan</u> titik akhir!

1. Simpan file konfigurasi.

## Tambahkan kode untuk menggunakan SDK Ucapan

1. Perhatikan bahwa folder **translator** berisi file kode untuk aplikasi klien:

    - **C#**: Program.cs
    - **Python**: translator.py

    Buka file kode dan di bagian atas, di bawah referensi namespace yang ada, temukan komentar **Impor namespace**. Kemudian, di bawah komentar ini, tambahkan kode khusus bahasa berikut untuk mengimpor namespace layanan yang akan Anda perlukan untuk menggunakan SDK Azure AI Speech:

    **C#**: Program.cs

    ```csharp
    // Import namespaces
    using Microsoft.CognitiveServices.Speech;
    using Microsoft.CognitiveServices.Speech.Audio;
    using Microsoft.CognitiveServices.Speech.Translation;
    ```

    **Python**: translator.py

    ```python
    # Import namespaces
    import azure.cognitiveservices.speech as speech_sdk
    ```

1. Dalam fungsi **Utama**, perhatikan bahwa kode untuk memuat kunci dan wilayah layanan Azure AI Speech dari file konfigurasi telah disediakan. Anda harus menggunakan variabel ini untuk membuat **SpeechTranslationConfig** untuk sumber daya Azure AI Speech Anda, yang akan Anda gunakan untuk menerjemahkan input lisan. Tambahkan kode berikut di bawah komentar **Konfigurasikan terjemahan**:

    **C#**: Program.cs

    ```csharp
    // Configure translation
    translationConfig = SpeechTranslationConfig.FromSubscription(aiSvcKey, aiSvcRegion);
    translationConfig.SpeechRecognitionLanguage = "en-US";
    translationConfig.AddTargetLanguage("fr");
    translationConfig.AddTargetLanguage("es");
    translationConfig.AddTargetLanguage("hi");
    Console.WriteLine("Ready to translate from " + translationConfig.SpeechRecognitionLanguage);
    ```

    **Python**: translator.py

    ```python
    # Configure translation
    translation_config = speech_sdk.translation.SpeechTranslationConfig(ai_key, ai_region)
    translation_config.speech_recognition_language = 'en-US'
    translation_config.add_target_language('fr')
    translation_config.add_target_language('es')
    translation_config.add_target_language('hi')
    print('Ready to translate from',translation_config.speech_recognition_language)
    ```

1. Anda akan menggunakan **SpeechTranslationConfig** untuk menerjemahkan ucapan menjadi teks, namun Anda juga akan menggunakan **SpeechConfig** untuk mensintesis terjemahan menjadi ucapan. Tambahkan kode berikut di bawah komentar **Konfigurasikan ucapan**:

    **C#**: Program.cs

    ```csharp
    // Configure speech
    speechConfig = SpeechConfig.FromSubscription(aiSvcKey, aiSvcRegion);
    ```

    **Python**: translator.py

    ```python
    # Configure speech
    speech_config = speech_sdk.SpeechConfig(ai_key, ai_region)
    ```

1. Simpan perubahan Anda dan kembali ke terminal terintegrasi untuk folder **penerjemah**, dan masukkan perintah berikut untuk menjalankan program:

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python translator.py
    ```

1. Jika Anda menggunakan C#, Anda dapat mengabaikan peringatan apa pun tentang menggunakan operator **tunggu** dalam metode asinkron - kami akan memperbaikinya nanti. Kode harus menampilkan pesan bahwa kode siap untuk diterjemahkan dari en-US dan meminta bahasa target. Tekan ENTER untuk mengakhiri program.

## Menerapkan terjemahan ucapan

Sekarang setelah Anda memiliki **SpeechTranslationConfig** untuk layanan Azure AI Speech, Anda dapat menggunakan API terjemahan Azure AI Speech untuk mengenali dan menerjemahkan ucapan.

> **PENTING**: Bagian ini mencakup instruksi untuk dua prosedur alternatif. Ikuti prosedur pertama jika Anda memiliki mikrofon yang berfungsi. Ikuti prosedur kedua jika Anda ingin menyimulasikan input lisan dengan menggunakan file audio.

### Jika Anda memiliki mikrofon yang berfungsi

1. Dalam fungsi **Utama** untuk program Anda, perhatikan bahwa kode menggunakan fungsi **Terjemahkan** untuk menerjemahkan masukan lisan.
1. Dalam fungsi **Terjemahkan**, di bawah komentar **Terjemahkan ucapan**, tambahkan kode berikut untuk membuat klien **TranslationRecognizer** yang dapat digunakan untuk mengenali dan menerjemahkan ucapan menggunakan default mikrofon sistem untuk input.

    **C#**: Program.cs

    ```csharp
    // Translate speech
    using AudioConfig audioConfig = AudioConfig.FromDefaultMicrophoneInput();
    using TranslationRecognizer translator = new TranslationRecognizer(translationConfig, audioConfig);
    Console.WriteLine("Speak now...");
    TranslationRecognitionResult result = await translator.RecognizeOnceAsync();
    Console.WriteLine($"Translating '{result.Text}'");
    translation = result.Translations[targetLanguage];
    Console.OutputEncoding = Encoding.UTF8;
    Console.WriteLine(translation);
    ```

    **Python**: translator.py

    ```python
    # Translate speech
    audio_config = speech_sdk.AudioConfig(use_default_microphone=True)
    translator = speech_sdk.translation.TranslationRecognizer(translation_config, audio_config = audio_config)
    print("Speak now...")
    result = translator.recognize_once_async().get()
    print('Translating "{}"'.format(result.text))
    translation = result.translations[targetLanguage]
    print(translation)
    ```

    > **CATATAN** Kode dalam aplikasi Anda menerjemahkan input ke ketiga bahasa dalam satu panggilan. Hanya terjemahan untuk bahasa tertentu yang ditampilkan, tetapi Anda dapat mengambil terjemahan mana pun dengan menetapkan kode bahasa target di kumpulan hasil **terjemahan**.

1. Sekarang lanjutkan ke bagian **Jalankan program** di bawah.

---

### Atau, gunakan input audio dari file

1. Di jendela terminal, masukkan perintah berikut untuk memasang pustaka yang dapat Anda gunakan untuk memutar file audio:

    **C#**: Program.cs

    ```csharp
    dotnet add package System.Windows.Extensions --version 4.6.0 
    ```

    **Python**: translator.py

    ```python
    pip install playsound==1.3.0
    ```

1. Dalam file kode untuk program Anda, di bawah impor namespace yang ada, tambahkan kode berikut untuk mengimpor pustaka yang baru saja Anda pasang:

    **C#**: Program.cs

    ```csharp
    using System.Media;
    ```

    **Python**: translator.py

    ```python
    from playsound import playsound
    ```

1. Dalam fungsi **Utama** untuk program Anda, perhatikan bahwa kode menggunakan fungsi **Terjemahkan** untuk menerjemahkan masukan lisan. Kemudian dalam fungsi **Terjemahkan**, di bawah komentar **Terjemahkan ucapan**, tambahkan kode berikut untuk membuat klien **TranslationRecognizer** yang dapat digunakan untuk mengenali dan menerjemahkan ucapan dari mengajukan.

    **C#**: Program.cs

    ```csharp
    // Translate speech
    string audioFile = "station.wav";
    SoundPlayer wavPlayer = new SoundPlayer(audioFile);
    wavPlayer.Play();
    using AudioConfig audioConfig = AudioConfig.FromWavFileInput(audioFile);
    using TranslationRecognizer translator = new TranslationRecognizer(translationConfig, audioConfig);
    Console.WriteLine("Getting speech from file...");
    TranslationRecognitionResult result = await translator.RecognizeOnceAsync();
    Console.WriteLine($"Translating '{result.Text}'");
    translation = result.Translations[targetLanguage];
    Console.OutputEncoding = Encoding.UTF8;
    Console.WriteLine(translation);
    ```

    **Python**: translator.py

    ```python
    # Translate speech
    audioFile = 'station.wav'
    playsound(audioFile)
    audio_config = speech_sdk.AudioConfig(filename=audioFile)
    translator = speech_sdk.translation.TranslationRecognizer(translation_config, audio_config = audio_config)
    print("Getting speech from file...")
    result = translator.recognize_once_async().get()
    print('Translating "{}"'.format(result.text))
    translation = result.translations[targetLanguage]
    print(translation)
    ```

---

### Jalankan program

1. Simpan perubahan Anda dan kembali ke terminal terintegrasi untuk folder **penerjemah**, dan masukkan perintah berikut untuk menjalankan program:

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python translator.py
    ```

1. Saat diminta, masukkan kode bahasa yang valid (*fr*, *es*, atau *hi*), lalu, jika menggunakan mikrofon, ucapkan dengan jelas dan ucapkan "di mana Stasiun?" atau frasa lain yang mungkin Anda gunakan saat bepergian ke luar negeri. Program ini harus mentranskripsikan input lisan Anda dan menerjemahkannya ke bahasa yang Anda tentukan (Prancis, Spanyol, atau Hindi). Ulangi proses ini, mencoba setiap bahasa yang didukung oleh aplikasi. Setelah selesai, tekan ENTER untuk mengakhiri program.

    TranslationRecognizer memberi Anda waktu sekitar 5 detik untuk berbicara. Jika tidak mendeteksi adanya input lisan, itu menghasilkan hasil "Tidak ada kecocokan". Terjemahan ke bahasa Hindi mungkin tidak selalu ditampilkan dengan benar di jendela Konsol karena masalah pengkodean karakter.

> **CATATAN**: Kode di aplikasi Anda menerjemahkan input ke ketiga bahasa dalam satu panggilan. Hanya terjemahan untuk bahasa tertentu yang ditampilkan, tetapi Anda dapat mengambil terjemahan mana pun dengan menetapkan kode bahasa target di kumpulan hasil **terjemahan**.

## Sintesiskan terjemahan menjadi ucapan

Sejauh ini, aplikasi Anda menerjemahkan masukan lisan ke teks; yang mungkin cukup jika Anda perlu meminta bantuan seseorang saat bepergian. Namun, akan lebih baik jika terjemahan diucapkan dengan lantang dengan suara yang sesuai.

1. Dalam fungsi **Terjemahkan**, di bawah komentar **Sintesis terjemahan**, tambahkan kode berikut untuk menggunakan klien **SpeechSynthesizer** guna menyintesis terjemahan sebagai ucapan melalui speaker default:

    **C#**: Program.cs

    ```csharp
    // Synthesize translation
    var voices = new Dictionary<string, string>
                    {
                        ["fr"] = "fr-FR-HenriNeural",
                        ["es"] = "es-ES-ElviraNeural",
                        ["hi"] = "hi-IN-MadhurNeural"
                    };
    speechConfig.SpeechSynthesisVoiceName = voices[targetLanguage];
    using SpeechSynthesizer speechSynthesizer = new SpeechSynthesizer(speechConfig);
    SpeechSynthesisResult speak = await speechSynthesizer.SpeakTextAsync(translation);
    if (speak.Reason != ResultReason.SynthesizingAudioCompleted)
    {
        Console.WriteLine(speak.Reason);
    }
    ```

    **Python**: translator.py

    ```python
    # Synthesize translation
    voices = {
            "fr": "fr-FR-HenriNeural",
            "es": "es-ES-ElviraNeural",
            "hi": "hi-IN-MadhurNeural"
    }
    speech_config.speech_synthesis_voice_name = voices.get(targetLanguage)
    speech_synthesizer = speech_sdk.SpeechSynthesizer(speech_config)
    speak = speech_synthesizer.speak_text_async(translation).get()
    if speak.reason != speech_sdk.ResultReason.SynthesizingAudioCompleted:
        print(speak.reason)
    ```

1. Simpan perubahan Anda dan kembali ke terminal terintegrasi untuk folder **penerjemah**, dan masukkan perintah berikut untuk menjalankan program:

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python translator.py
    ```

1. Saat diminta, masukkan kode bahasa yang valid (*fr*, *es*, atau *hi*), lalu ucapkan dengan jelas ke mikrofon dan ucapkan frasa yang mungkin Anda gunakan saat jalan-jalan keluar negeri. Program harus menyalin masukan lisan Anda dan merespons dengan terjemahan lisan. Ulangi proses ini, mencoba setiap bahasa yang didukung oleh aplikasi. Setelah selesai, tekan **ENTER** untuk mengakhiri program.

> **Catatan**
> *Dalam contoh ini, Anda telah menggunakan **SpeechTranslationConfig** untuk menerjemahkan ucapan ke teks, lalu gunakan **SpeechConfig** untuk menyintesis terjemahan sebagai ucapan. Anda sebenarnya dapat menggunakan **SpeechTranslationConfig** untuk mensintesis terjemahan secara langsung, tetapi ini hanya berfungsi saat menerjemahkan ke satu bahasa, dan menghasilkan aliran audio yang biasanya disimpan sebagai file daripada dikirim langsung ke pembicara.*

## Informasi selengkapnya

Untuk informasi selengkapnya tentang menggunakan API terjemahan Azure AI Speech, lihat [dokumentasi terjemahan Ucapan](https://learn.microsoft.com/azure/ai-services/speech-service/speech-translation).
