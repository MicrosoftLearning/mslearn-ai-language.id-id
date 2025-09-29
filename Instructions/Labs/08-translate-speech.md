---
lab:
  title: Menerjemahkan Ucapan
  description: Terjemahkan ucapan bahasa ke ucapan dan terapkan di aplikasi Anda sendiri.
---

# Menerjemahkan Ucapan

Azure AI Speech menyertakan API terjemahan ucapan yang dapat Anda gunakan untuk menerjemahkan bahasa lisan. Misalnya, Anda ingin mengembangkan aplikasi penerjemah yang dapat digunakan orang saat bepergian di tempat-tempat di mana mereka tidak berbicara bahasa lokal. Mereka akan dapat mengucapkan frasa seperti "Di mana stasiunnya?" atau "Saya perlu mencari apotek" dalam bahasa mereka sendiri, dan memintanya menerjemahkannya ke bahasa lokal. Dalam latihan ini, Anda akan menggunakan SDK Azure AI Speech untuk Python guna membuat aplikasi sederhana berdasarkan contoh ini.

Meskipun latihan ini didasarkan pada Python, Anda dapat mengembangkan aplikasi terjemahan ucapan menggunakan beberapa SDK khusus bahasa; termasuk:

- [SDK Azure AI Speech untuk Python](https://pypi.org/project/azure-cognitiveservices-speech/)
- [SDK Azure AI Speech untuk .NET](https://www.nuget.org/packages/Microsoft.CognitiveServices.Speech)
- [SDK Azure AI Speech untuk JavaScript](https://www.npmjs.com/package/microsoft-cognitiveservices-speech-sdk)

Latihan ini memakan waktu sekitar **30** menit.

> **CATATAN** Latihan ini dirancang untuk diselesaikan di azure cloud shell, di mana akses langsung ke perangkat keras suara komputer Anda tidak didukung. Oleh karena itu lab akan menggunakan file audio untuk aliran input dan output ucapan. Kode untuk mencapai hasil yang sama menggunakan mikrofon dan speaker disediakan untuk referensi Anda.

## Membuat sumber daya Azure AI Speech

Mari kita mulai dengan membuat sumber daya Azure AI Speech.

1. Buka [portal Azure](https://portal.azure.com) di `https://portal.azure.com`, dan masuk menggunakan akun Microsoft yang terkait dengan langganan Azure Anda.
1. Di bidang pencarian, cari **Layanan ucapan**. Pilih dari daftar, lalu pilih **Buat**.
1. Provisikan sumber daya menggunakan pengaturan berikut:
    - **Langganan**: *Langganan Azure Anda*.
    - **Grup sumber daya**: *Memilih atau membuat grup sumber daya*.
    - **Wilayah**: *Pilih wilayah yang tersedia*
    - **Nama**: *Masukkan nama unik*.
    - **Tingkat harga**: Pilih **F0** (*gratis*), atau **S** (*standar*) jika F tidak tersedia.
1. Pilih **Tinjau + buat**, lalu pilih **Buat** untuk menyediakan sumber daya.
1. Tunggu hingga penyebaran selesai, lalu buka sumber daya yang disebarkan.
1. Lihat halaman **Titik Akhir dan Kunci** di bagian **Manajemen Sumber Daya**. Anda akan memerlukan informasi di halaman ini nanti dalam latihan.

## Persiapan untuk mengembangkan aplikasi di Cloud Shell

1. Biarkan halaman **Kunci dan Titik Akhir** terbuka, lalu gunakan tombol **[\>_]** di sebelah kanan bilah pencarian di bagian atas halaman untuk membuat Cloud Shell baru di portal Azure, dengan memilih lingkungan ***PowerShell***. Cloud shell menyediakan antarmuka baris perintah dalam panel di bagian bawah portal Azure.

    > **Catatan**: Jika sebelumnya Anda telah membuat cloud shell yang menggunakan lingkungan *Bash* , alihkan ke ***PowerShell***.

1. Di toolbar cloud shell, di menu **Pengaturan**, pilih **Buka versi Klasik** (ini diperlukan untuk menggunakan editor kode).

    **<font color="red">Pastikan Anda telah beralih ke versi klasik cloud shell sebelum melanjutkan.</font>**

1. Di panel PowerShell, masukkan perintah berikut untuk mengkloning repositori GitHub untuk latihan ini:

    ```
   rm -r mslearn-ai-language -f
   git clone https://github.com/microsoftlearning/mslearn-ai-language
    ```

    > **Tips**: Saat Anda memasukkan perintah ke cloudshell, output-nya mungkin mengambil sejumlah besar buffer layar. Anda dapat menghapus layar dengan memasukkan `cls` perintah untuk mempermudah fokus pada setiap tugas.

1. Setelah repositori dikloning, navigasikan ke folder yang berisi file kode:

    ```
   cd mslearn-ai-language/Labfiles/08-speech-translation/Python/translator
    ```

1. Di panel baris perintah, jalankan perintah berikut untuk menampilkan file kode di folder **translator**:

    ```
   ls -a -l
    ```

    File menyertakan file konfigurasi (**.env**) dan file kode (**translator.py**).

1. Buat lingkungan virtual Python dan instal paket SDK Azure AI Speech dan paket lain yang diperlukan dengan menjalankan perintah berikut:

    ```
    python -m venv labenv
    ./labenv/bin/Activate.ps1
    pip install -r requirements.txt azure-cognitiveservices-speech==1.42.0
    ```

1. Masukkan perintah berikut untuk mengedit file konfigurasi yang telah disediakan:

    ```
   code .env
    ```

    File dibuka dalam editor kode.

1. Perbarui nilai konfigurasi untuk menyertakan **wilayah** dan **kunci** dari sumber daya Azure AI Speech yang Anda buat (tersedia di halaman **Kunci dan Titik Akhir** untuk sumber daya Azure AI Penerjemah Anda di portal Azure).
1. Setelah Anda mengganti tempat penampung, gunakan perintah **CTRL+S** untuk menyimpan perubahan Anda lalu gunakan perintah **CTRL+Q** untuk menutup editor kode sambil menjaga baris perintah cloud shell tetap terbuka.

## Menambahkan kode untuk menggunakan SDK Azure AI Speech

> **Tips**: Saat Anda menambahkan kode, pastikan untuk mempertahankan indentasi yang benar.

1. Masukkan perintah berikut untuk mengedit file kode yang telah disediakan:

    ```
   code translator.py
    ```

1. Buka file kode dan di bagian atas, di bawah referensi kumpulan nama yang ada, temukan komentar **Impor kumpulan nama**. Kemudian, di bawah komentar ini, tambahkan kode khusus bahasa berikut untuk mengimpor namespace layanan yang akan Anda perlukan untuk menggunakan SDK Azure AI Speech:

    ```python
   # Import namespaces
   from azure.core.credentials import AzureKeyCredential
   import azure.cognitiveservices.speech as speech_sdk
    ```

1. Di fungsi **utama**, di bawah komentar **Dapatkan pengaturan konfigurasi**, perhatikan bahwa kode memuat kunci dan wilayah yang Anda tentukan dalam file konfigurasi.

1. Temukan kode berikut di bawah komentar **Konfigurasikan terjemahan**, dan tambahkan kode berikut untuk mengonfigurasi koneksi Anda ke titik akhir Layanan Azure AI Speech:

    ```python
   # Configure translation
   translation_config = speech_sdk.translation.SpeechTranslationConfig(speech_key, speech_region)
   translation_config.speech_recognition_language = 'en-US'
   translation_config.add_target_language('fr')
   translation_config.add_target_language('es')
   translation_config.add_target_language('hi')
   print('Ready to translate from',translation_config.speech_recognition_language)
    ```

1. Anda akan menggunakan **SpeechTranslationConfig** untuk menerjemahkan ucapan menjadi teks, namun Anda juga akan menggunakan **SpeechConfig** untuk mensintesis terjemahan menjadi ucapan. Tambahkan kode berikut di bawah komentar **Konfigurasikan ucapan**:

    ```python
   # Configure speech
   speech_config = speech_sdk.SpeechConfig(speech_key, speech_region)
   print('Ready to use speech service in:', speech_config.region)
    ```

1. Simpan perubahan Anda (*CTRL+S*), tetapi biarkan editor kode terbuka.

## Menjalankan aplikasi

Sejauh ini, aplikasi tidak melakukan apa pun selain menyambungkan ke sumber daya Azure AI Speech, tetapi menjalankannya dan memeriksa apakah aplikasi berfungsi sebelum menambahkan fungsionalitas ucapan dapat bermanfaat.

1. Di baris perintah, masukkan perintah berikut untuk menjalankan aplikasi penerjemah:

    ```
   python translator.py
    ```

    Kode harus menampilkan wilayah sumber daya layanan ucapan yang akan digunakan aplikasi, pesan yang siap diterjemahkan dari en-US dan meminta Anda menentukan bahasa target. Eksekusi yang berhasil menunjukkan bahwa aplikasi telah tersambung ke layanan Azure AI Speech Anda. Tekan ENTER untuk mengakhiri program.

## Menerapkan terjemahan ucapan

Sekarang setelah Anda memiliki **SpeechTranslationConfig** untuk layanan Azure AI Speech, Anda dapat menggunakan API terjemahan Azure AI Speech untuk mengenali dan menerjemahkan ucapan.

1. Dalam file kode, perhatikan bahwa kode menggunakan fungsi **Terjemahkan** untuk menerjemahkan input lisan. Kemudian dalam fungsi **Terjemahkan**, di bawah komentar **Terjemahkan ucapan**, tambahkan kode berikut untuk membuat klien **TranslationRecognizer** yang dapat digunakan untuk mengenali dan menerjemahkan ucapan dari mengajukan.

    ```python
   # Translate speech
   current_dir = os.getcwd()
   audioFile = current_dir + '/station.wav'
   audio_config_in = speech_sdk.AudioConfig(filename=audioFile)
   translator = speech_sdk.translation.TranslationRecognizer(translation_config, audio_config = audio_config_in)
   print("Getting speech from file...")
   result = translator.recognize_once_async().get()
   print('Translating "{}"'.format(result.text))
   translation = result.translations[targetLanguage]
   print(translation)
    ```

1. Simpan perubahan Anda (*CTRL+S*), dan jalankan kembali program:

    ```
   python translator.py
    ```

1. Saat diminta, masukkan kode bahasa yang valid (*fr*, *es*, atau *hi*). Program ini harus mentranskripsikan file input Anda dan menerjemahkannya ke bahasa yang Anda tentukan (Prancis, Spanyol, atau Hindi). Ulangi proses ini, mencoba setiap bahasa yang didukung oleh aplikasi.

    > **CATATAN**: Terjemahan ke bahasa Hindi mungkin tidak selalu ditampilkan dengan benar di jendela Konsol karena masalah pengkodean karakter.

1. Setelah selesai, tekan ENTER untuk mengakhiri program.

> **CATATAN**: Kode di aplikasi Anda menerjemahkan input ke ketiga bahasa dalam satu panggilan. Hanya terjemahan untuk bahasa tertentu yang ditampilkan, tetapi Anda dapat mengambil terjemahan mana pun dengan menetapkan kode bahasa target di kumpulan hasil **terjemahan**.

## Sintesiskan terjemahan menjadi ucapan

Sejauh ini, aplikasi Anda menerjemahkan masukan lisan ke teks; yang mungkin cukup jika Anda perlu meminta bantuan seseorang saat bepergian. Namun, akan lebih baik jika terjemahan diucapkan dengan lantang dengan suara yang sesuai.

> **Catatan**: Karena keterbatasan perangkat keras cloud shell, kita akan mengarahkan output ucapan yang disintesis ke file.

1. Dalam fungsi **Terjemahkan**, temukan komentar **Sintesis terjemahan**, lalu tambahkan kode berikut untuk menggunakan klien **SpeechSynthesizer** guna menyintesis terjemahan sebagai ucapan dan menyimpannya sebagai file .wav:

    ```python
   # Synthesize translation
   output_file = "output.wav"
   voices = {
            "fr": "fr-FR-HenriNeural",
            "es": "es-ES-ElviraNeural",
            "hi": "hi-IN-MadhurNeural"
   }
   speech_config.speech_synthesis_voice_name = voices.get(targetLanguage)
   audio_config_out = speech_sdk.audio.AudioConfig(filename=output_file)
   speech_synthesizer = speech_sdk.SpeechSynthesizer(speech_config, audio_config_out)
   speak = speech_synthesizer.speak_text_async(translation).get()
   if speak.reason != speech_sdk.ResultReason.SynthesizingAudioCompleted:
        print(speak.reason)
   else:
        print("Spoken output saved in " + output_file)
    ```

1. Simpan perubahan Anda (*CTRL+S*), dan jalankan kembali program:

    ```
   python translator.py
    ```

1. Tinjau output dari aplikasi, yang harus menunjukkan bahwa terjemahan output lisan disimpan dalam file. Setelah selesai, tekan **ENTER** untuk mengakhiri program.
1. Jika Anda memiliki pemutar media yang mampu memutar file audio .wav, unduh file yang dihasilkan dengan memasukkan perintah berikut:

    ```
   download ./output.wav
    ```

    Perintah unduh membuat tautan popup di kanan bawah browser Anda, yang dapat Anda pilih untuk mengunduh dan membuka file.

> **Catatan**
> *Dalam contoh ini, Anda telah menggunakan **SpeechTranslationConfig** untuk menerjemahkan ucapan ke teks, lalu gunakan **SpeechConfig** untuk menyintesis terjemahan sebagai ucapan. Anda sebenarnya dapat menggunakan **SpeechTranslationConfig** untuk menyintesis terjemahan secara langsung, tetapi ini hanya berfungsi saat menerjemahkan ke satu bahasa, dan menghasilkan aliran audio yang biasanya disimpan sebagai file.*

## Membersihkan sumber daya

Jika sudah selesai menjelajahi layanan Azure AI Speech, Anda dapat menghapus sumber daya yang Anda buat dalam latihan ini. Berikut caranya:

1. Tutup panel Azure Cloud Shell
1. Di portal Azure, telusuri sumber daya Azure AI Speech yang Anda buat di lab ini.
1. Pada halaman sumber daya, pilih **Hapus** dan ikuti instruksi untuk menghapus sumber daya.

## Bagaimana jika Anda memiliki mikrofon dan speaker?

Dalam latihan ini, lingkungan Azure Cloud Shell yang kami gunakan tidak mendukung perangkat keras audio. Jadi, Anda menggunakan file audio untuk input dan output ucapan. Mari lihat bagaimana kode dapat dimodifikasi untuk menggunakan perangkat keras audio jika Anda memilikinya.

### Menggunakan terjemahan ucapan dengan mikrofon

1. Jika memiliki mikrofon, Anda dapat menggunakan kode berikut untuk mengambil input lisan untuk terjemahan ucapan:

    ```python
   # Translate speech
   audio_config_in = speech_sdk.AudioConfig(use_default_microphone=True)
   translator = speech_sdk.translation.TranslationRecognizer(translation_config, audio_config = audio_config_in)
   print("Speak now...")
   result = translator.recognize_once_async().get()
   print('Translating "{}"'.format(result.text))
   translation = result.translations[targetLanguage]
   print(translation)
    ```

> **Catatan**: Mikrofon default sistem adalah input audio default, sehingga Anda juga dapat mengabaikan AudioConfig sekalian!

### Menggunakan sintesis ucapan dengan pembicara

1. Jika Anda memiliki pembicara, Anda dapat menggunakan kode berikut untuk mensintesis ucapan.
    
    ```python
   # Synthesize translation
   voices = {
            "fr": "fr-FR-HenriNeural",
            "es": "es-ES-ElviraNeural",
            "hi": "hi-IN-MadhurNeural"
   }
   speech_config.speech_synthesis_voice_name = voices.get(targetLanguage)
   audio_config_out = speech_sdk.audio.AudioOutputConfig(use_default_speaker=True)
   speech_synthesizer = speech_sdk.SpeechSynthesizer(speech_config, audio_config_out)
   speak = speech_synthesizer.speak_text_async(translation).get()
   if speak.reason != speech_sdk.ResultReason.SynthesizingAudioCompleted:
        print(speak.reason)
    ```

> **Catatan**: Speaker default sistem adalah output audio default, sehingga Anda juga dapat menghilangkan AudioConfig sama sekali!

## Informasi selengkapnya

Untuk informasi selengkapnya tentang menggunakan API terjemahan Azure AI Speech, lihat [dokumentasi terjemahan Ucapan](https://learn.microsoft.com/azure/ai-services/speech-service/speech-translation).
