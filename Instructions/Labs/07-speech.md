---
lab:
  title: Mengenali dan Menyintesis Ucapan
  description: 'Terapkan jam bicara yang mengonversi ucapan ke teks, dan teks ke ucapan.'
---

# Mengenali dan mensintesis ucapan

**Azure AI Speech** adalah layanan yang menyediakan fungsionalitas terkait ucapan, termasuk:

- API *ucapan ke teks* yang memungkinkan Anda menerapkan pengenalan ucapan (mengonversi kata-kata lisan yang dapat didengar menjadi teks).
- API *teks ke ucapan* yang memungkinkan Anda menerapkan sintesis ucapan (mengonversi teks menjadi ucapan yang dapat didengar).

Dalam latihan ini, Anda akan menggunakan kedua API ini untuk mengimplementasikan aplikasi jam yang berbicara.

Meskipun latihan ini didasarkan pada Python, Anda dapat mengembangkan aplikasi ucapan menggunakan beberapa SDK khusus bahasa; termasuk:

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

## Menyiapkan dan mengonfigurasi aplikasi jam berbicara

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

1. Setelah repositori dikloning, navigasikan ke folder yang berisi file kode aplikasi jam berbicara:  

    ```
   cd mslearn-ai-language/Labfiles/07-speech/Python/speaking-clock
    ```

1. Di panel baris perintah, jalankan perintah berikut untuk menampilkan file kode di folder **speaking-clock**:

    ```
   ls -a -l
    ```

    File menyertakan file konfigurasi (**.env**) dan file kode (**speaking-clock.py**). File audio yang akan digunakan aplikasi Anda ada di subfolder **audio**.

1. Buat lingkungan virtual Python dan instal paket SDK Azure AI Speech dan paket lain yang diperlukan dengan menjalankan perintah berikut:

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-cognitiveservices-speech==1.42.0
    ```

1. Masukkan perintah berikut untuk mengedit file konfigurasi:

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
   code speaking-clock.py
    ```

1. Buka file kode dan di bagian atas, di bawah referensi kumpulan nama yang ada, temukan komentar **Impor kumpulan nama**. Kemudian, di bawah komentar ini, tambahkan kode khusus bahasa berikut untuk mengimpor namespace layanan yang akan Anda perlukan untuk menggunakan SDK Azure AI Speech:

    ```python
   # Import namespaces
   from azure.core.credentials import AzureKeyCredential
   import azure.cognitiveservices.speech as speech_sdk
    ```

1. Di fungsi **utama**, di bawah komentar **Dapatkan pengaturan konfigurasi**, perhatikan bahwa kode memuat kunci dan wilayah yang Anda tentukan dalam file konfigurasi.

1. Temukan komentar **Konfigurasikan layanan ucapan**, dan tambahkan kode berikut untuk menggunakan kunci Layanan AI dan wilayah Anda untuk mengonfigurasi koneksi Anda ke titik akhir layanan Azure AI Speech:

    ```python
   # Configure speech service
   speech_config = speech_sdk.SpeechConfig(speech_key, speech_region)
   print('Ready to use speech service in:', speech_config.region)
    ```

1. Simpan perubahan Anda (*CTRL+S*), tetapi biarkan editor kode terbuka.

## Menjalankan aplikasi

Sejauh ini, aplikasi tidak melakukan apa pun selain menyambungkan ke proyek Azure AI Speech, tetapi akan berguna untuk menjalankannya dan memeriksa apakah aplikasi berfungsi sebelum menambahkan fungsionalitas ucapan.

1. Di baris perintah, masukkan perintah berikut untuk menjalankan aplikasi jam bicara:

    ```
   python speaking-clock.py
    ```

    Kode harus menampilkan wilayah sumber daya layanan ucapan yang akan digunakan aplikasi. Eksekusi yang berhasil menunjukkan bahwa aplikasi telah tersambung ke sumber daya Azure AI Speech Anda.

## Menambahkan kode untuk mengenali ucapan

Sekarang setelah Anda memiliki **SpeechConfig** untuk layanan ucapan di sumber daya Azure AI Speech, Anda dapat menggunakan API **Ucapan ke teks** untuk mengenali ucapan dan mentranskripsikannya ke teks.

Dalam prosedur ini, input ucapan diambil dari file audio, yang dapat Anda putar di sini:

<video controls src="https://github.com/MicrosoftLearning/mslearn-ai-language/raw/refs/heads/main/Instructions/media/Time.mp4" title="Jam berapa sekarang?" width="150"></video>

1. Dalam file kode, perhatikan bahwa kode menggunakan fungsi **TranscribeCommand** untuk menerima input lisan. Kemudian dalam fungsi **TranscribeCommand**, temukan komentar **Konfigurasikan pengenalan ucapan**, lalu tambahkan kode yang sesuai di bawah ini untuk membuat klien **SpeechRecognizer** yang dapat digunakan untuk mengenali dan mentranskripsikan ucapan dari file audio:

    ```python
   # Configure speech recognition
   current_dir = os.getcwd()
   audioFile = current_dir + '/time.wav'
   audio_config = speech_sdk.AudioConfig(filename=audioFile)
   speech_recognizer = speech_sdk.SpeechRecognizer(speech_config, audio_config)
    ```

1. Dalam fungsi **TranscribeCommand**, di bawah **input ucapan Proses** komentar, tambahkan kode berikut untuk mendengarkan input lisan, berhati-hatilah untuk tidak mengganti kode di akhir fungsi yang mengembalikan perintah:

    ```python
   # Process speech input
   print("Listening...")
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

1. Simpan perubahan Anda (*CTRL+S*), lalu di baris perintah di bawah editor kode, jalankan kembali program:
1. Tinjau output, yang seharusnya berhasil "mendengar" ucapan dalam file audio dan mengembalikan respons yang sesuai (perhatikan bahwa Azure Cloud Shell Anda mungkin berjalan di server yang berada di zona waktu yang berbeda dengan milik Anda!)

    > **Tip**: Jika SpeechRecognizer mengalami kesalahan, maka akan muncul hasil "Dibatalkan". Kode dalam aplikasi kemudian akan menampilkan pesan kesalahan. Penyebab yang paling mungkin adalah nilai wilayah yang salah dalam file konfigurasi.

## Mensintesis ucapan

Aplikasi jam berbicara Anda menerima masukan lisan, tetapi tidak benar-benar berbicara! Mari kita perbaiki dengan menambahkan kode untuk mensintesis ucapan.

Sekali lagi, karena keterbatasan perangkat keras cloud shell, kita akan mengarahkan output ucapan yang disintesis ke file.

1. Dalam file kode, perhatikan bahwa kode menggunakan fungsi **TellTime** untuk memberi tahu pengguna waktu saat ini.
1. Dalam fungsi **TellTime**, di bawah komentar **Konfigurasikan sintesis ucapan**, tambahkan kode berikut untuk membuat klien **SpeechSynthesizer** yang dapat digunakan untuk menghasilkan output lisan:

    ```python
   # Configure speech synthesis
   output_file = "output.wav"
   speech_config.speech_synthesis_voice_name = "en-GB-RyanNeural"
   audio_config = speech_sdk.audio.AudioConfig(filename=output_file)
   speech_synthesizer = speech_sdk.SpeechSynthesizer(speech_config, audio_config,)
    ```

1. Dalam fungsi **TellTime**, di bawah **komentar Sintesis output lisan**, tambahkan kode berikut untuk menghasilkan output lisan, berhati-hatilah untuk tidak mengganti kode di akhir fungsi yang mencetak respons:

    ```python
   # Synthesize spoken output
   speak = speech_synthesizer.speak_text_async(response_text).get()
   if speak.reason != speech_sdk.ResultReason.SynthesizingAudioCompleted:
        print(speak.reason)
   else:
        print("Spoken output saved in " + output_file)
    ```

1. Simpan perubahan Anda (*CTRL+S*) dan jalankan kembali program, yang seharusnya menunjukkan bahwa output lisan disimpan dalam file.

1. Jika Anda memiliki pemutar media yang mampu memutar file audio .wav, unduh file yang dihasilkan dengan memasukkan perintah berikut:

    ```
   download ./output.wav
    ```

    Perintah unduh membuat tautan popup di kanan bawah browser Anda, yang dapat Anda pilih untuk mengunduh dan membuka file.

    File akan terdengar serupa dengan yang ini:

    <video controls src="https://github.com/MicrosoftLearning/mslearn-ai-language/raw/refs/heads/main/Instructions/media/Output.mp4" title="Waktunya adalah 2:15" width="150"></video>

## Menggunakan Speech Synthesis Markup Language

Speech Synthesis Markup Language (SSML) memungkinkan Anda menyesuaikan cara ucapan Anda disintesis menggunakan format berbasis XML.

1. Dalam fungsi **TellTime**, ganti semua kode saat ini di bawah **komentar Sintesis output lisan** dengan kode berikut (biarkan kode di bawah komentar **Cetak respons**):

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
   else:
       print("Spoken output saved in " + output_file)
    ```

1. Simpan perubahan Anda dan jalankan kembali program, yang seharusnya menunjukkan sekali lagi bahwa output lisan disimpan dalam file.
1. Unduh dan putar file yang dihasilkan, yang seharusnya terdengar seperti ini:
    
    <video controls src="https://github.com/MicrosoftLearning/mslearn-ai-language/raw/refs/heads/main/Instructions/media/Output2.mp4" title="Waktunya adalah 5:30. Waktu untuk mengakhiri lab ini." width="150"></video>

## (OPSIONAL) Bagaimana jika Anda memiliki mikrofon dan speaker?

Dalam latihan ini, Anda menggunakan file audio untuk input dan output ucapan. Mari kita lihat bagaimana kode dapat dimodifikasi untuk menggunakan perangkat keras audio.

### Menggunakan pengenalan ucapan dengan mikrofon

Jika Anda memiliki mikrofon, Anda dapat menggunakan kode berikut untuk mengambil input lisan untuk pengenalan ucapan:

```python
# Configure speech recognition
audio_config = speech_sdk.AudioConfig(use_default_microphone=True)
speech_recognizer = speech_sdk.SpeechRecognizer(speech_config, audio_config)
print('Speak now...')

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

> **Catatan**: Mikrofon default sistem adalah input audio default, sehingga Anda juga dapat mengabaikan AudioConfig sekalian!

### Menggunakan sintesis ucapan dengan pembicara

Jika Anda memiliki pembicara, Anda dapat menggunakan kode berikut untuk mensintesis ucapan.

```python
response_text = 'The time is {}:{:02d}'.format(now.hour,now.minute)

# Configure speech synthesis
speech_config.speech_synthesis_voice_name = "en-GB-RyanNeural"
audio_config = speech_sdk.audio.AudioOutputConfig(use_default_speaker=True)
speech_synthesizer = speech_sdk.SpeechSynthesizer(speech_config, audio_config)

# Synthesize spoken output
speak = speech_synthesizer.speak_text_async(response_text).get()
if speak.reason != speech_sdk.ResultReason.SynthesizingAudioCompleted:
    print(speak.reason)
```

> **Catatan**: Speaker default sistem adalah output audio default, sehingga Anda juga dapat menghilangkan AudioConfig sama sekali!

## Penghapusan

Setelah selesai menjelajahi Azure AI Speech, Anda harus menghapus sumber daya yang telah Anda buat dalam latihan ini untuk menghindari biaya Azure yang tidak perlu.

1. Tutup panel Azure Cloud Shell
1. Di portal Azure, telusuri sumber daya Azure AI Speech yang Anda buat di lab ini.
1. Pada halaman sumber daya, pilih **Hapus** dan ikuti instruksi untuk menghapus sumber daya.

## Informasi selengkapnya

Untuk informasi selengkapnya tentang menggunakan API **Ucapan ke teks** dan **Teks ke ucapan**, lihat [dokumentasi Ucapan ke teks](https://learn.microsoft.com/azure/ai-services/speech-service/index-speech-to-text) dan [dokumentasi Teks ke ucapan](https://learn.microsoft.com/azure/ai-services/speech-service/index-text-to-speech).
