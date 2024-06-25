---
lab:
  title: Mengenali dan Menyintesis Ucapan
  module: Module 4 - Create speech-enabled apps with Azure AI services
---

# Mengenali dan mensintesis ucapan

**Azure AI Speech** adalah layanan yang menyediakan fungsionalitas terkait ucapan, termasuk:

- API *ucapan ke teks* yang memungkinkan Anda menerapkan pengenalan ucapan (mengonversi kata-kata lisan yang dapat didengar menjadi teks).
- API *teks ke ucapan* yang memungkinkan Anda menerapkan sintesis ucapan (mengonversi teks menjadi ucapan yang dapat didengar).

Dalam latihan ini, Anda akan menggunakan kedua API ini untuk mengimplementasikan aplikasi jam yang berbicara.

> **CATATAN** Latihan ini mengharuskan Anda menggunakan komputer dengan speaker/headphone. Untuk pengalaman terbaik, mikrofon juga diperlukan. Beberapa lingkungan virtual yang dihosting mungkin dapat menangkap audio dari mikrofon lokal Anda, tetapi jika ini tidak berhasil (atau Anda tidak memiliki mikrofon sama sekali), Anda dapat menggunakan file audio yang disediakan untuk input ucapan. Ikuti petunjuknya dengan cermat, karena Anda harus memilih opsi yang berbeda tergantung pada apakah Anda menggunakan mikrofon atau file audio.

## Menyediakan sumber daya *Azure AI Speech*

Jika belum memilikinya di langganan, Anda harus menyediakan sumber daya **Azure AI Speech**.

1. Buka portal Microsoft Azure di `https://portal.azure.com`, dan masuk menggunakan akun Microsoft yang terkait dengan langganan Azure Anda.
1. Di bidang pencarian di bagian atas, cari **layanan Azure AI** dan tekan **Enter**. Kemudian, pada hasil pencarian, pilih **Buat** di bawah **Layanan ucapan**.
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

Anda akan mengembangkan aplikasi ucapan menggunakan Visual Studio Code. File kode untuk aplikasi Anda telah disediakan dalam repositori GitHub.

> **Tips**: Jika Anda telah mengkloning repositori **mslearn-ai-language**, buka di Visual Studio Code. Jika belum melakukannya, ikuti langkah-langkah berikut untuk mengkloningnya ke lingkungan pengembangan Anda.

1. Memulai Visual Studio Code.
1. Buka palet (SHIFT+CTRL+P) dan jalankan **Git: Perintah klon** untuk mengkloning repositori `https://github.com/MicrosoftLearning/mslearn-ai-language` ke folder lokal (tidak masalah folder mana).
1. Setelah repositori dikloning, buka folder di Visual Studio Code.

    > **Catatan**: Jika Visual Studio Code menampilkan pesan pop-up yang meminta Anda memercayai kode yang Anda buka, klik opsi **Ya, saya memercayai pembuatnya** di pop-up.

1. Tunggu sementara file tambahan diinstal untuk mendukung proyek kode C# di repositori.

    > **Catatan**: Jika Anda diminta untuk menambahkan aset yang diperlukan guna membangun dan men-debug, pilih **Tidak Sekarang**.

## Mengonfigurasi aplikasi Anda

Aplikasi untuk C# dan Python telah disediakan. Kedua aplikasi memiliki fungsionalitas yang sama. Pertama, Anda akan menyelesaikan beberapa bagian penting aplikasi untuk mengaktifkannya agar dapat menggunakan sumber daya Azure AI Speech.

1. Di Visual Studio Code, di panel **Explorer**, telusuri folder **Labfiles/07-speech** dan luaskan folder **CSharp** atau **Python**, tergantung pada preferensi bahasa Anda dan folder **speaking-clock** yang ada di dalamnya. Setiap folder berisi file kode khusus bahasa untuk aplikasi tempat Anda akan mengintegrasikan fungsionalitas Azure AI Speech.
1. Klik kanan folder **speaking-clock** yang berisi file kode Anda dan buka terminal terintegrasi. Kemudian, instal paket SDK Azure AI Speech dengan menjalankan perintah yang sesuai dengan preferensi bahasa Anda:

    **C#**

    ```
    dotnet add package Microsoft.CognitiveServices.Speech --version 1.30.0
    ```

    **Python**

    ```
    pip install azure-cognitiveservices-speech==1.30.0
    ```

1. Di panel **Explorer**, pada folder **speaking-clock**, buka file konfigurasi sesuai bahasa pilihan Anda

    - **C#**: appsettings.json
    - **Python**: .env

1. Perbarui nilai konfigurasi untuk menyertakan **wilayah** dan **kunci** dari sumber daya Azure AI Speech yang Anda buat (tersedia di halaman **Kunci dan Titik Akhir** untuk sumber daya Azure AI Speech Anda di portal Azure).

    > **CATATAN**: Pastikan untuk menambahkan *wilayah* untuk sumber daya Anda, <u>bukan</u> titik akhir!

1. Simpan file konfigurasi.

## Menambahkan kode untuk menggunakan SDK Azure AI Speech

1. Perhatikan bahwa folder **speaking-clock** berisi file kode untuk aplikasi klien:

    - **C#**: Program.cs
    - **Python**: speaking-clock.py

    Buka file kode dan di bagian atas, di bawah referensi namespace yang ada, temukan komentar **Impor namespace**. Kemudian, di bawah komentar ini, tambahkan kode khusus bahasa berikut untuk mengimpor namespace layanan yang akan Anda perlukan untuk menggunakan SDK Azure AI Speech:

    **C#**: Program.cs

    ```csharp
    // Import namespaces
    using Microsoft.CognitiveServices.Speech;
    using Microsoft.CognitiveServices.Speech.Audio;
    ```

    **Python**: speaking-clock.py

    ```python
    # Import namespaces
    import azure.cognitiveservices.speech as speech_sdk
    ```

1. Dalam fungsi **Main**, perhatikan bahwa kode untuk memuat kunci dan wilayah layanan dari file konfigurasi telah disediakan. Anda harus menggunakan variabel ini untuk membuat **SpeechConfig** untuk sumber daya Azure AI Speech Anda. Tambahkan kode berikut di bawah komentar **Konfigurasikan layanan ucapan**:

    **C#**: Program.cs

    ```csharp
    // Configure speech service
    speechConfig = SpeechConfig.FromSubscription(aiSvcKey, aiSvcRegion);
    Console.WriteLine("Ready to use speech service in " + speechConfig.Region);
    
    // Configure voice
    speechConfig.SpeechSynthesisVoiceName = "en-US-AriaNeural";
    ```

    **Python**: speaking-clock.py

    ```python
    # Configure speech service
    speech_config = speech_sdk.SpeechConfig(ai_key, ai_region)
    print('Ready to use speech service in:', speech_config.region)
    ```

1. Simpan perubahan Anda dan kembali ke terminal terintegrasi untuk folder **speaking-clock**, dan masukkan perintah berikut untuk menjalankan program:

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python speaking-clock.py
    ```

1. Jika Anda menggunakan C#, Anda dapat mengabaikan peringatan apa pun tentang menggunakan operator **tunggu** dalam metode asinkron - kami akan memperbaikinya nanti. Kode harus menampilkan wilayah sumber daya layanan ucapan yang akan digunakan aplikasi.

## Menambahkan kode untuk mengenali ucapan

Sekarang setelah Anda memiliki **SpeechConfig** untuk layanan ucapan di sumber daya Azure AI Speech, Anda dapat menggunakan API **Speech-to-text** untuk mengenali ucapan dan mentranskripsikannya ke teks.

> **PENTING**: Bagian ini mencakup instruksi untuk dua prosedur alternatif. Ikuti prosedur pertama jika Anda memiliki mikrofon yang berfungsi. Ikuti prosedur kedua jika Anda ingin menyimulasikan input lisan dengan menggunakan file audio.

### Jika Anda memiliki mikrofon yang berfungsi

1. Dalam fungsi **Utama** untuk program Anda, perhatikan bahwa kode menggunakan fungsi **TranscribeCommand** untuk menerima masukan lisan.
1. Dalam fungsi **TranscribeCommand**, di bawah komentar **Konfigurasikan pengenalan ucapan**, tambahkan kode yang sesuai di bawah ini untuk membuat klien **SpeechRecognizer** yang dapat digunakan untuk mengenali dan mentranskripsikan ucapan menggunakan mikrofon sistem default:

    **C#**

    ```csharp
    // Configure speech recognition
    using AudioConfig audioConfig = AudioConfig.FromDefaultMicrophoneInput();
    using SpeechRecognizer speechRecognizer = new SpeechRecognizer(speechConfig, audioConfig);
    Console.WriteLine("Speak now...");
    ```

    **Python**

    ```python
    # Configure speech recognition
    audio_config = speech_sdk.AudioConfig(use_default_microphone=True)
    speech_recognizer = speech_sdk.SpeechRecognizer(speech_config, audio_config)
    print('Speak now...')
    ```

1. Sekarang lewati ke bagian **Tambahkan kode untuk memproses perintah yang ditranskripsikan** di bawah ini.

---

### Atau, gunakan input audio dari file

1. Di jendela terminal, masukkan perintah berikut untuk memasang pustaka yang dapat Anda gunakan untuk memutar file audio:

    **C#**

    ```
    dotnet add package System.Windows.Extensions --version 4.6.0 
    ```

    **Python**

    ```
    pip install playsound==1.2.2
    ```

1. Dalam file kode untuk program Anda, di bawah impor namespace yang ada, tambahkan kode berikut untuk mengimpor pustaka yang baru saja Anda pasang:

    **C#**: Program.cs

    ```csharp
    using System.Media;
    ```

    **Python**: speaking-clock.py

    ```python
    from playsound import playsound
    ```

1. Dalam fungsi **Utama**, perhatikan bahwa kode menggunakan fungsi **TranscribeCommand** untuk menerima masukan lisan. Kemudian dalam fungsi **TranscribeCommand**, di bawah komentar **Konfigurasikan pengenalan ucapan**, tambahkan kode yang sesuai di bawah ini untuk membuat klien **SpeechRecognizer** yang dapat digunakan untuk mengenali dan mentranskripsikan ucapan dari file audio:

    **C#**: Program.cs

    ```csharp
    // Configure speech recognition
    string audioFile = "time.wav";
    SoundPlayer wavPlayer = new SoundPlayer(audioFile);
    wavPlayer.Play();
    using AudioConfig audioConfig = AudioConfig.FromWavFileInput(audioFile);
    using SpeechRecognizer speechRecognizer = new SpeechRecognizer(speechConfig, audioConfig);
    ```

    **Python**: speaking-clock.py

    ```python
    # Configure speech recognition
    current_dir = os.getcwd()
    audioFile = current_dir + '\\time.wav'
    playsound(audioFile)
    audio_config = speech_sdk.AudioConfig(filename=audioFile)
    speech_recognizer = speech_sdk.SpeechRecognizer(speech_config, audio_config)
    ```

---

### Menambahkan kode untuk memproses perintah yang ditranskripsikan

1. Dalam fungsi **TranscribeCommand**, di bawah **input ucapan Proses** komentar, tambahkan kode berikut untuk mendengarkan input lisan, berhati-hatilah untuk tidak mengganti kode di akhir fungsi yang mengembalikan perintah:

    **C#**: Program.cs

    ```csharp
    // Process speech input
    SpeechRecognitionResult speech = await speechRecognizer.RecognizeOnceAsync();
    if (speech.Reason == ResultReason.RecognizedSpeech)
    {
        command = speech.Text;
        Console.WriteLine(command);
    }
    else
    {
        Console.WriteLine(speech.Reason);
        if (speech.Reason == ResultReason.Canceled)
        {
            var cancellation = CancellationDetails.FromResult(speech);
            Console.WriteLine(cancellation.Reason);
            Console.WriteLine(cancellation.ErrorDetails);
        }
    }
    ```

    **Python**: speaking-clock.py

    ```python
    # Process speech input
    speech = speech_recognizer.recognize_once_async().get()
    if speech.reason == speech_sdk.ResultReason.RecognizedSpeech:
        command = speech.text
        print(command)
    else:
        print(speech.reason)
        if speech.reason == speech_sdk.ResultReason.Canceled:
            cancellation = speech.cancellation_details
            print(cancellation.reason)
            print(cancellation.error_details)
    ```

1. Simpan perubahan Anda dan kembali ke terminal terintegrasi untuk folder **speaking-clock**, dan masukkan perintah berikut untuk menjalankan program:

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python speaking-clock.py
    ```

1. Jika menggunakan mikrofon, bicaralah dengan jelas dan ucapkan "jam berapa sekarang?". Program harus mentranskripsikan input lisan Anda dan menampilkan waktu (berdasarkan waktu lokal komputer tempat kode berjalan, yang mungkin bukan waktu yang benar di mana Anda berada).

    SpeechRecognizer memberi Anda waktu sekitar 5 detik untuk berbicara. Jika tidak mendeteksi adanya input lisan, itu menghasilkan hasil "Tidak ada kecocokan".

    Jika SpeechRecognizer mengalami kesalahan, itu menghasilkan hasil dari "Dibatalkan". Kode dalam aplikasi kemudian akan menampilkan pesan kesalahan. Penyebab yang paling mungkin adalah kunci atau wilayah yang salah dalam file konfigurasi.

## Mensintesis ucapan

Aplikasi jam berbicara Anda menerima masukan lisan, tetapi tidak benar-benar berbicara! Mari kita perbaiki dengan menambahkan kode untuk mensintesis ucapan.

1. Dalam fungsi **Utama** untuk program Anda, perhatikan bahwa kode menggunakan fungsi **TellTime** untuk memberi tahu pengguna waktu saat ini.
1. Dalam fungsi **TellTime**, di bawah komentar **Konfigurasikan sintesis ucapan**, tambahkan kode berikut untuk membuat klien **SpeechSynthesizer** yang dapat digunakan untuk menghasilkan output lisan:

    **C#**: Program.cs

    ```csharp
    // Configure speech synthesis
    speechConfig.SpeechSynthesisVoiceName = "en-GB-RyanNeural";
    using SpeechSynthesizer speechSynthesizer = new SpeechSynthesizer(speechConfig);
    ```

    **Python**: speaking-clock.py

    ```python
    # Configure speech synthesis
    speech_config.speech_synthesis_voice_name = "en-GB-RyanNeural"
    speech_synthesizer = speech_sdk.SpeechSynthesizer(speech_config)
    ```

    > **CATATAN** Konfigurasi audio default menggunakan perangkat audio sistem default untuk output sehingga Anda tidak perlu menyediakan **AudioConfig** secara eksplisit. Jika Anda perlu mengalihkan output audio ke file, Anda dapat menggunakan **AudioConfig** dengan jalur file untuk melakukannya.

1. Dalam fungsi **TellTime**, di bawah **komentar Sintesis output lisan**, tambahkan kode berikut untuk menghasilkan output lisan, berhati-hatilah untuk tidak mengganti kode di akhir fungsi yang mencetak respons:

    **C#**: Program.cs

    ```csharp
    // Synthesize spoken output
    SpeechSynthesisResult speak = await speechSynthesizer.SpeakTextAsync(responseText);
    if (speak.Reason != ResultReason.SynthesizingAudioCompleted)
    {
        Console.WriteLine(speak.Reason);
    }
    ```

    **Python**: speaking-clock.py

    ```python
    # Synthesize spoken output
    speak = speech_synthesizer.speak_text_async(response_text).get()
    if speak.reason != speech_sdk.ResultReason.SynthesizingAudioCompleted:
        print(speak.reason)
    ```

1. Simpan perubahan Anda dan kembali ke terminal terintegrasi untuk folder **speaking-clock**, dan masukkan perintah berikut untuk menjalankan program:

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python speaking-clock.py
    ```

1. Ketika diminta, bicaralah dengan jelas ke mikrofon dan ucapkan "jam berapa sekarang?". Program harus berbicara, memberi tahu Anda waktu.

## Gunakan suara yang berbeda

Aplikasi jam berbicara Anda menggunakan suara default, yang dapat Anda ubah. Layanan Ucapan mendukung berbagai suara *standar* serta suara *saraf* yang lebih mirip manusia. Anda juga dapat membuat suara *kustom*.

> **Catatan**: Untuk daftar suara neural dan standar, lihat [Galeri Suara](https://speech.microsoft.com/portal/voicegallery) di Speech Studio.

1. Dalam fungsi **TellTime**, di bawah komentar **Konfigurasikan sintesis ucapan**, ubah kode sebagai berikut untuk menentukan suara alternatif sebelum membuat klien **SpeechSynthesizer** :

   **C#**: Program.cs

    ```csharp
    // Configure speech synthesis
    speechConfig.SpeechSynthesisVoiceName = "en-GB-LibbyNeural"; // change this
    using SpeechSynthesizer speechSynthesizer = new SpeechSynthesizer(speechConfig);
    ```

    **Python**: speaking-clock.py

    ```python
    # Configure speech synthesis
    speech_config.speech_synthesis_voice_name = 'en-GB-LibbyNeural' # change this
    speech_synthesizer = speech_sdk.SpeechSynthesizer(speech_config)
    ```

1. Simpan perubahan Anda dan kembali ke terminal terintegrasi untuk folder **speaking-clock**, dan masukkan perintah berikut untuk menjalankan program:

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python speaking-clock.py
    ```

1. Ketika diminta, bicaralah dengan jelas ke mikrofon dan ucapkan "jam berapa sekarang?". Program harus berbicara dalam suara yang ditentukan, memberi tahu Anda waktu.

## Menggunakan Speech Synthesis Markup Language

Speech Synthesis Markup Language (SSML) memungkinkan Anda menyesuaikan cara ucapan Anda disintesis menggunakan format berbasis XML.

1. Dalam fungsi **TellTime**, ganti semua kode saat ini di bawah **komentar Sintesis output lisan** dengan kode berikut (biarkan kode di bawah komentar **Cetak respons**):

   **C#**: Program.cs

    ```csharp
    // Synthesize spoken output
    string responseSsml = $@"
        <speak version='1.0' xmlns='http://www.w3.org/2001/10/synthesis' xml:lang='en-US'>
            <voice name='en-GB-LibbyNeural'>
                {responseText}
                <break strength='weak'/>
                Time to end this lab!
            </voice>
        </speak>";
    SpeechSynthesisResult speak = await speechSynthesizer.SpeakSsmlAsync(responseSsml);
    if (speak.Reason != ResultReason.SynthesizingAudioCompleted)
    {
        Console.WriteLine(speak.Reason);
    }
    ```

    **Python**: speaking-clock.py

    ```python
    # Synthesize spoken output
    responseSsml = " \
        <speak version='1.0' xmlns='http://www.w3.org/2001/10/synthesis' xml:lang='en-US'> \
            <voice name='en-GB-LibbyNeural'> \
                {} \
                <break strength='weak'/> \
                Time to end this lab! \
            </voice> \
        </speak>".format(response_text)
    speak = speech_synthesizer.speak_ssml_async(responseSsml).get()
    if speak.reason != speech_sdk.ResultReason.SynthesizingAudioCompleted:
        print(speak.reason)
    ```

1. Simpan perubahan Anda dan kembali ke terminal terintegrasi untuk folder **speaking-clock**, dan masukkan perintah berikut untuk menjalankan program:

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python speaking-clock.py
    ```

1. Ketika diminta, bicaralah dengan jelas ke mikrofon dan ucapkan "jam berapa sekarang?". Program harus berbicara dalam suara yang ditentukan dalam SSML (menggantikan suara yang ditentukan dalam SpeechConfig) untuk memberi tahu Anda waktu saat ini. Setelah jeda, Anda akan diberi tahu bahwa sudah saatnya untuk mengakhiri lab ini, inilah saatnya!

## Informasi selengkapnya

Untuk informasi selengkapnya tentang menggunakan API **Ucapan ke teks** dan **Teks ke ucapan**, lihat [dokumentasi Ucapan ke teks](https://learn.microsoft.com/azure/ai-services/speech-service/index-speech-to-text) dan [dokumentasi Teks ke ucapan](https://learn.microsoft.com/azure/ai-services/speech-service/index-text-to-speech).
