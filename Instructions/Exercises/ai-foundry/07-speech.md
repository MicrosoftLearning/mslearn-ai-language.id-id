---
lab:
  title: Mengenali dan Menyintesis Ucapan (versi Azure AI Foundry)
  module: Module 4 - Create speech-enabled apps with Azure AI services
---

<!--
Possibly update to use standalone AI Service instead of Foundry?
-->

# Mengenali dan mensintesis ucapan

**Azure AI Speech** adalah layanan yang menyediakan fungsionalitas terkait ucapan, termasuk:

- API *ucapan ke teks* yang memungkinkan Anda menerapkan pengenalan ucapan (mengonversi kata-kata lisan yang dapat didengar menjadi teks).
- API *teks ke ucapan* yang memungkinkan Anda menerapkan sintesis ucapan (mengonversi teks menjadi ucapan yang dapat didengar).

Dalam latihan ini, Anda akan menggunakan kedua API ini untuk mengimplementasikan aplikasi jam yang berbicara.

> **CATATAN** Latihan ini dirancang untuk diselesaikan di azure cloud shell, di mana akses langsung ke perangkat keras suara komputer Anda tidak didukung. Oleh karena itu lab akan menggunakan file audio untuk aliran input dan output ucapan. Kode untuk mencapai hasil yang sama menggunakan mikrofon dan speaker disediakan untuk referensi Anda.

## Membuat proyek Azure OpenAI

Mari kita mulai dengan membuat proyek Azure AI Foundry.

1. Di browser web, buka [portal Azure AI Foundry](https://ai.azure.com) di `https://ai.azure.com` dan masuk menggunakan kredensial Azure Anda. Tutup semua tip atau panel mulai cepat yang terbuka saat pertama kali Anda masuk, dan jika perlu, gunakan logo **Azure AI Foundry** di kiri atas untuk menavigasi ke halaman beranda, yang terlihat sama dengan gambar berikut:

    ![Tangkapan layar portal Azure AI Foundry.](./media/ai-foundry-home.png)

1. Di beranda, pilih **+ Buat proyek**.
1. Di wizard **Buat proyek**, masukkan nama proyek yang sesuai untuk (misalnya, `my-ai-project`) lalu tinjau sumber daya Azure yang akan dibuat secara otomatis untuk mendukung proyek Anda.
1. Pilih **Kustomisasi** dan tentukan pengaturan berikut untuk hub Anda:
    - **Nama hub**: *Nama unik - misalnya `my-ai-hub`*
    - **Langganan**: *Langganan Azure Anda*
    - **Grup sumber daya**: *Pilih atau buat grup sumber daya dengan nama unik (misalnya, `my-ai-resources`), atau pilih yang sudah ada*
    - **Lokasi**: Pilih wilayah yang tersedia
    - **Menyambungkan Layanan Azure AI atau Azure OpenAI**: *Membuat sumber daya Layanan AI baru dengan nama yang sesuai (misalnya, `my-ai-services`) atau menggunakan yang sudah ada*
    - **Menyambungkan Azure AI Search**: Lewati koneksi

1. Pilih **Berikutnya** dan tinjau konfigurasi Anda. Lalu pilih **Buat** dan tunggu hingga prosesnya selesai.
1. Saat proyek Anda dibuat, tutup tips apa pun yang ditampilkan dan tinjau halaman proyek di portal Azure AI Foundry, yang akan terlihat mirip dengan gambar berikut:

    ![Tangkapan layar detail proyek Azure AI di portal Azure AI Foundry.](./media/ai-foundry-project.png)

## Menyiapkan dan mengonfigurasi aplikasi jam berbicara

1. Di portal Azure AI Foundry, lihat halaman **Gambaran Umum** untuk proyek Anda.
1. Di area **Detail proyek**, perhatikan **string koneksi Proyek** dan **lokasi** untuk proyek Anda, Anda akan menggunakan string koneksi untuk menyambungkan ke proyek Anda dalam aplikasi klien, dan Anda akan memerlukan lokasi untuk menyambungkan ke titik akhir Ucapan Layanan Azure AI.
1. Buka tab browser baru (biarkan portal Azure AI Foundry tetap terbuka di tab yang sudah ada). Kemudian di tab baru, telusuri [Portal Azure](https://portal.azure.com) di `https://portal.azure.com`; masuk menggunakan kredensial Azure Anda jika diminta.
1. Gunakan tombol **[\>_]** di sebelah kanan bilah pencarian di bagian atas halaman untuk membuat Cloud Shell baru di portal Azure, dengan memilih lingkungan ***PowerShell***. Cloud shell menyediakan antarmuka baris perintah dalam panel di bagian bawah portal Azure.

    > **Catatan**: Jika sebelumnya Anda telah membuat cloud shell yang menggunakan lingkungan *Bash* , alihkan ke ***PowerShell***.

1. Di toolbar cloud shell, di menu **Pengaturan**, pilih **Buka versi Klasik** (ini diperlukan untuk menggunakan editor kode).

    > **Tips**: Saat Anda menempelkan perintah ke cloudshell, ouput mungkin mengambil sejumlah besar buffer layar. Anda dapat menghapus layar dengan memasukkan `cls` perintah untuk mempermudah fokus pada setiap tugas.

1. Di panel PowerShell, masukkan perintah berikut untuk mengkloning repositori GitHub untuk latihan ini:

    ```
   rm -r mslearn-ai-language -f
   git clone https://github.com/microsoftlearning/mslearn-ai-language mslearn-ai-language
    ```

    ***Sekarang ikuti langkah-langkah untuk bahasa pemrograman yang Anda pilih.***

1. Setelah repositori dikloning, navigasikan ke folder yang berisi file kode aplikasi jam berbicara:  

    **Python**

    ```
   cd mslearn-ai-language/labfiles/07b-speech/python/speaking-clock
    ```

    **C#**

    ```
   cd mslearn-ai-language/labfiles/07b-speech/c-sharp/speaking-clock
    ```

1. Di panel baris perintah cloud shell, masukkan perintah berikut untuk menginstal pustaka yang akan Anda gunakan:

    **Python**

    ```
   pip install python-dotenv azure-identity azure-ai-projects azure-cognitiveservices-speech==1.42.0
    ```

    **C#**

    ```
   dotnet add package Azure.Identity
   dotnet add package Azure.AI.Projects --prerelease
   dotnet add package Microsoft.CognitiveServices.Speech --version 1.42.0
    ```

1. Masukkan perintah berikut untuk mengedit file konfigurasi yang telah disediakan:

    **Python**

    ```
   code .env
    ```

    **C#**

    ```
   code appsettings.json
    ```

    File dibuka dalam editor kode.

1. Dalam file kode, ganti tempat penampung **your_project_endpoint** dan **your_location**dengan string koneksi dan lokasi untuk proyek Anda (disalin dari halaman **Gambaran Umum** di portal Azure AI Foundry).
1. Setelah Anda mengganti tempat penampung, gunakan perintah **CTRL+S** untuk menyimpan perubahan Anda lalu gunakan perintah **CTRL+Q** untuk menutup editor kode sambil menjaga baris perintah cloud shell tetap terbuka.

## Menambahkan kode untuk menggunakan SDK Azure AI Speech

> **Tips**: Saat Anda menambahkan kode, pastikan untuk mempertahankan indentasi yang benar.

1. Masukkan perintah berikut untuk mengedit file kode yang telah disediakan:

    **Python**

    ```
   code speaking-clock.py
    ```

    **C#**

    ```
   code Program.cs
    ```

1. Buka file kode dan di bagian atas, di bawah referensi kumpulan nama yang ada, temukan komentar **Impor kumpulan nama**. Kemudian, di bawah komentar ini, tambahkan kode khusus bahasa berikut untuk mengimpor kumpulan nama yang akan Anda perlukan untuk menggunakan SDK Azure AI Speech dengan sumber daya Layanan Azure AI di proyek Azure AI Foundry Anda:

    **Python**

    ```python
   # Import namespaces
   from dotenv import load_dotenv
   from azure.ai.projects.models import ConnectionType
   from azure.identity import DefaultAzureCredential
   from azure.core.credentials import AzureKeyCredential
   from azure.ai.projects import AIProjectClient
   import azure.cognitiveservices.speech as speech_sdk
    ```

    **C#**

    ```csharp
   // Import namespaces
   using Azure.Identity;
   using Azure.AI.Projects;
   using Microsoft.CognitiveServices.Speech;
   using Microsoft.CognitiveServices.Speech.Audio;
    ```

1. Dalam fungsi **utama**, di bawah komentar **Dapatkan pengaturan konfigurasi**, perhatikan bahwa kode memuat string koneksi proyek dan nilai nama penyebaran model yang Anda tentukan dalam file konfigurasi.

1. Tambahkan kode berikut di bawah komentar **Dapatkan titik akhir dan kunci Ucapan AI dari proyek**:

    **Python**

    ```python
   # Get AI Services key from the project
   project_client = AIProjectClient.from_connection_string(
        conn_str=project_connection,
        credential=DefaultAzureCredential())

   ai_svc_connection = project_client.connections.get_default(
      connection_type=ConnectionType.AZURE_AI_SERVICES,
      include_credentials=True, 
    )

   ai_svc_key = ai_svc_connection.key

    ```

    **C#**

    ```csharp
   // Get AI Services key from the project
   var projectClient = new AIProjectClient(project_connection,
                        new DefaultAzureCredential());

   ConnectionResponse aiSvcConnection = projectClient.GetConnectionsClient().GetDefaultConnection(ConnectionType.AzureAIServices, true);

   var apiKeyAuthProperties = aiSvcConnection.Properties as ConnectionPropertiesApiKeyAuth;

   var aiSvcKey = apiKeyAuthProperties.Credentials.Key;
    ```

    Kode ini terhubung ke proyek Azure AI Foundry Anda, mendapatkan sumber daya yang terhubung dengan Layanan AI default, dan mengambil kunci autentikasi yang diperlukan untuk menggunakannya.

1. Di bawah komentar **Konfigurasikan layanan** ucapan, tambahkan kode berikut untuk menggunakan kunci Layanan AI dan wilayah proyek Anda untuk mengonfigurasi koneksi Anda ke titik akhir Ucapan Layanan Azure AI

   **Python**

    ```python
   # Configure speech service
   speech_config = speech_sdk.SpeechConfig(ai_svc_key, location)
   print('Ready to use speech service in:', speech_config.region)
    ```

    **C#**

    ```csharp
   // Configure speech service
   speechConfig = SpeechConfig.FromSubscription(aiSvcKey, location);
   Console.WriteLine("Ready to use speech service in " + speechConfig.Region);
    ```

1. Simpan perubahan Anda (*CTRL+S*), tetapi biarkan editor kode terbuka.

## Menjalankan aplikasi

Sejauh ini, aplikasi tidak melakukan apa pun selain menyambungkan ke proyek Azure AI Foundry Anda untuk mengambil detail yang diperlukan untuk menggunakan layanan Ucapan, tetapi akan berguna untuk menjalankannya dan memeriksa apakah aplikasi berfungsi sebelum menambahkan fungsionalitas ucapan.

1. Di baris perintah di bawah editor kode, masukkan perintah Azure CLI berikut untuk menentukan akun Azure yang masuk untuk sesi tersebut:

    ```
   az account show
    ```

    Output JSON yang dihasilkan harus menyertakan detail akun Azure Anda dan langganan tempat Anda bekerja (yang seharusnya merupakan langganan yang sama tempat Anda membuat proyek Azure AI Foundry.)

    Aplikasi Anda menggunakan kredensial Azure untuk konteks di mana aplikasi dijalankan untuk mengautentikasi koneksi ke proyek Anda. Di lingkungan produksi, aplikasi mungkin dikonfigurasi untuk berjalan menggunakan identitas terkelola. Di lingkungan pengembangan ini, ini akan menggunakan kredensial sesi cloud shell terautentikasi Anda.

    > **Catatan**: Anda dapat masuk ke Azure di lingkungan pengembangan Anda dengan menggunakan perintah `az login`Azure CLI. Dalam hal ini, cloud shell telah masuk menggunakan kredensial Azure yang Anda gunakan untuk masuk ke portal; jadi masuk secara eksplisit tidak diperlukan. Untuk mempelajari selengkapnya tentang menggunakan Azure CLI untuk mengautentikasi ke Azure, lihat [Mengautentikasi ke Azure menggunakan Azure CLI](https://learn.microsoft.com/cli/azure/authenticate-azure-cli).

1. Di baris perintah, masukkan perintah khusus bahasa berikut untuk menjalankan aplikasi jam berbicara:

    **Python**

    ```
   python speaking-clock.py
    ```

    **C#**

    ```
   dotnet run
    ```

1. Jika Anda menggunakan C#, Anda dapat mengabaikan peringatan apa pun tentang menggunakan operator **tunggu** dalam metode asinkron - kami akan memperbaikinya nanti. Kode harus menampilkan wilayah sumber daya layanan ucapan yang akan digunakan aplikasi. Eksekusi yang berhasil menunjukkan bahwa aplikasi telah terhubung ke proyek Azure AI Foundry Anda dan mengambil kunci yang diperlukan untuk menggunakan layanan Azure AI Speech.

## Menambahkan kode untuk mengenali ucapan

Sekarang setelah Anda memiliki **SpeechConfig** untuk layanan ucapan di sumber daya Azure AI Speech, Anda dapat menggunakan API **Ucapan ke teks** untuk mengenali ucapan dan mentranskripsikannya ke teks.

Dalam prosedur ini, input ucapan diambil dari file audio, yang dapat Anda putar di sini:

<video controls src="media/Time.mp4" title="Jam berapa sekarang?" width="150"></video>

1. Dalam fungsi **Utama**, perhatikan bahwa kode menggunakan fungsi **TranscribeCommand** untuk menerima masukan lisan. Kemudian dalam fungsi **TranscribeCommand**, di bawah komentar **Konfigurasikan pengenalan ucapan**, tambahkan kode yang sesuai di bawah ini untuk membuat klien **SpeechRecognizer** yang dapat digunakan untuk mengenali dan mentranskripsikan ucapan dari file audio:

    **Python**

    ```python
   # Configure speech recognition
   current_dir = os.getcwd()
   audioFile = current_dir + '/time.wav'
   audio_config = speech_sdk.AudioConfig(filename=audioFile)
   speech_recognizer = speech_sdk.SpeechRecognizer(speech_config, audio_config)
    ```

    **C#**

    ```csharp
   // Configure speech recognition
   string audioFile = "time.wav";
   using AudioConfig audioConfig = AudioConfig.FromWavFileInput(audioFile);
   using SpeechRecognizer speechRecognizer = new SpeechRecognizer(speechConfig, audioConfig);
    ```

1. Dalam fungsi **TranscribeCommand**, di bawah **input ucapan Proses** komentar, tambahkan kode berikut untuk mendengarkan input lisan, berhati-hatilah untuk tidak mengganti kode di akhir fungsi yang mengembalikan perintah:

    **Python**

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

    **C#**

    ```csharp
   // Process speech input
   Console.WriteLine("Listening...");
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

1. Simpan perubahan Anda (*CTRL+S*), lalu di baris perintah di bawah editor kode, masukkan perintah berikut untuk menjalankan program:

    **Python**

    ```
   python speaking-clock.py
    ```

    **C#**

    ```
   dotnet run
    ```

1. Tinjau output dari aplikasi, yang seharusnya berhasil "mendengar" ucapan dalam file audio dan mengembalikan respons yang sesuai (perhatikan bahwa Azure Cloud Shell Anda mungkin berjalan di server yang berada di zona waktu yang berbeda dengan milik Anda!)

    > **Tip**: Jika SpeechRecognizer mengalami kesalahan, maka akan muncul hasil "Dibatalkan". Kode dalam aplikasi kemudian akan menampilkan pesan kesalahan. Penyebab yang paling mungkin adalah nilai wilayah yang salah dalam file konfigurasi.

## Mensintesis ucapan

Aplikasi jam berbicara Anda menerima masukan lisan, tetapi tidak benar-benar berbicara! Mari kita perbaiki dengan menambahkan kode untuk mensintesis ucapan.

Sekali lagi, karena keterbatasan perangkat keras cloud shell, kita akan mengarahkan output ucapan yang disintesis ke file.

1. Dalam fungsi **Utama** untuk program Anda, perhatikan bahwa kode menggunakan fungsi **TellTime** untuk memberi tahu pengguna waktu saat ini.
1. Dalam fungsi **TellTime**, di bawah komentar **Konfigurasikan sintesis ucapan**, tambahkan kode berikut untuk membuat klien **SpeechSynthesizer** yang dapat digunakan untuk menghasilkan output lisan:

    **Python**

    ```python
   # Configure speech synthesis
   output_file = "output.wav"
   speech_config.speech_synthesis_voice_name = "en-GB-RyanNeural"
   audio_config = speech_sdk.audio.AudioConfig(filename=output_file)
   speech_synthesizer = speech_sdk.SpeechSynthesizer(speech_config, audio_config,)
    ```

    **C#**

    ```csharp
   // Configure speech synthesis
   var outputFile = "output.wav";
   speechConfig.SpeechSynthesisVoiceName = "en-GB-RyanNeural";
   using var audioConfig = AudioConfig.FromWavFileOutput(outputFile);
   using SpeechSynthesizer speechSynthesizer = new SpeechSynthesizer(speechConfig, audioConfig);
    ```

1. Dalam fungsi **TellTime**, di bawah **komentar Sintesis output lisan**, tambahkan kode berikut untuk menghasilkan output lisan, berhati-hatilah untuk tidak mengganti kode di akhir fungsi yang mencetak respons:

    **Python**

    ```python
   # Synthesize spoken output
   speak = speech_synthesizer.speak_text_async(response_text).get()
   if speak.reason != speech_sdk.ResultReason.SynthesizingAudioCompleted:
       print(speak.reason)
   else:
       print("Spoken output saved in " + outputFile)
    ```

    **C#**

    ```csharp
   // Synthesize spoken output
   SpeechSynthesisResult speak = await speechSynthesizer.SpeakTextAsync(responseText);
   if (speak.Reason != ResultReason.SynthesizingAudioCompleted)
   {
       Console.WriteLine(speak.Reason);
   }
   else
   {
       Console.WriteLine("Spoken output saved in " + outputFile);
   }
    ```

1. Simpan perubahan Anda (*CTRL+S*), lalu di baris perintah di bawah editor kode, masukkan perintah berikut untuk menjalankan program:

   **Python**

    ```
   python speaking-clock.py
    ```

    **C#**

    ```
   dotnet run
    ```

1. Tinjau output dari aplikasi, yang harus menunjukkan bahwa output lisan disimpan dalam file.
1. Jika Anda memiliki pemutar media yang mampu memutar file audio .wav, di toolbar untuk panel cloud shell, gunakan tombol **Unggah/Unduh file** untuk mengunduh file audio dari folder aplikasi Anda, lalu putar:

    **Python**

    /beranda/*pengguna*`/mslearn-ai-language/Labfiles/07b-speech/Python/speaking-clock/output.wav`

    **C#**

    /beranda/*pengguna*`/mslearn-ai-language/Labfiles/07b-speech/C-Sharp/speaking-clock/output.wav`

    File akan terdengar serupa dengan yang ini:

    <video controls src="./media/Output.mp4" title="Waktunya adalah 2:15" width="150"></video>

## Menggunakan Speech Synthesis Markup Language

Speech Synthesis Markup Language (SSML) memungkinkan Anda menyesuaikan cara ucapan Anda disintesis menggunakan format berbasis XML.

1. Dalam fungsi **TellTime**, ganti semua kode saat ini di bawah **komentar Sintesis output lisan** dengan kode berikut (biarkan kode di bawah komentar **Cetak respons**):

    **Python**

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
       print("Spoken output saved in " + outputFile)
    ```

   **C#**

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
   else
   {
        Console.WriteLine("Spoken output saved in " + outputFile);
   }
    ```

1. Simpan perubahan Anda dan kembali ke terminal terintegrasi untuk folder **speaking-clock**, dan masukkan perintah berikut untuk menjalankan program:

    **Python**

    ```
   python speaking-clock.py
    ```

    **C#**

    ```
   dotnet run
    ```

1. Tinjau output dari aplikasi, yang harus menunjukkan bahwa output lisan disimpan dalam file.
1. Sekali lagi, jika Anda memiliki pemutar media yang mampu memutar file audio .wav, di toolbar untuk panel cloud shell, gunakan tombol **Unggah/Unduh file** untuk mengunduh file audio dari folder aplikasi Anda, lalu mainkan:

    **Python**

    /beranda/*pengguna*`/mslearn-ai-language/Labfiles/07b-speech/Python/speaking-clock/output.wav`

    **C#**

    /beranda/*pengguna*`/mslearn-ai-language/Labfiles/07b-speech/C-Sharp/speaking-clock/output.wav`

    File akan terdengar serupa dengan yang ini:
    
    <video controls src="./media/Output2.mp4" title="Waktunya adalah 5:30. Waktu untuk mengakhiri lab ini." width="150"></video>

## Penghapusan

Setelah selesai menjelajahi Azure AI Speech, Anda harus menghapus sumber daya yang telah Anda buat dalam latihan ini untuk menghindari biaya Azure yang tidak perlu.

1. Kembali ke tab browser yang berisi portal Azure (atau buka kembali [portal Azure](https://portal.azure.com) di `https://portal.azure.com` tab browser baru) dan lihat konten grup sumber daya tempat Anda menyebarkan sumber daya yang digunakan dalam latihan ini.
1. Pada toolbar pilih **Hapus grup sumber daya**.
1. Masukkan nama grup sumber daya untuk mengonfirmasi bahwa Anda ingin menghapusnya, dan pilih Hapus.

## Bagaimana jika Anda memiliki mikrofon dan speaker?

Dalam latihan ini, Anda menggunakan file audio untuk input dan output ucapan. Mari kita lihat bagaimana kode dapat dimodifikasi untuk menggunakan perangkat keras audio.

### Menggunakan pengenalan ucapan dengan mikrofon

Jika Anda memiliki mikrofon, Anda dapat menggunakan kode berikut untuk mengambil input lisan untuk pengenalan ucapan:

**Python**

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

**C#**

```csharp
// Configure speech recognition
using AudioConfig audioConfig = AudioConfig.FromDefaultMicrophoneInput();
using SpeechRecognizer speechRecognizer = new SpeechRecognizer(speechConfig, audioConfig);
Console.WriteLine("Speak now...");

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

> **Catatan**: Mikrofon default sistem adalah input audio default, sehingga Anda juga dapat mengabaikan AudioConfig sekalian!

### Menggunakan sintesis ucapan dengan pembicara

Jika Anda memiliki pembicara, Anda dapat menggunakan kode berikut untuk mensintesis ucapan.

**Python**

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

**C#**

```csharp
var now = DateTime.Now;
string responseText = "The time is " + now.Hour.ToString() + ":" + now.Minute.ToString("D2");

// Configure speech synthesis
speechConfig.SpeechSynthesisVoiceName = "en-GB-RyanNeural";
using var audioConfig = AudioConfig.FromDefaultSpeakerOutput();
using SpeechSynthesizer speechSynthesizer = new SpeechSynthesizer(speechConfig, audioConfig);

// Synthesize spoken output
SpeechSynthesisResult speak = await speechSynthesizer.SpeakTextAsync(responseText);
if (speak.Reason != ResultReason.SynthesizingAudioCompleted)
{
    Console.WriteLine(speak.Reason);
}
```

> **Catatan**: Speaker default sistem adalah output audio default, sehingga Anda juga dapat menghilangkan AudioConfig sama sekali!

## Informasi selengkapnya

Untuk informasi selengkapnya tentang menggunakan API **Ucapan ke teks** dan **Teks ke ucapan**, lihat [dokumentasi Ucapan ke teks](https://learn.microsoft.com/azure/ai-services/speech-service/index-speech-to-text) dan [dokumentasi Teks ke ucapan](https://learn.microsoft.com/azure/ai-services/speech-service/index-text-to-speech).
