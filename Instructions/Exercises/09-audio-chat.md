---
lab:
  title: Mengembangkan aplikasi obrolan berkemampuan audio
  description: Mempelajari cara menggunakan Azure AI Foundry untuk membuat aplikasi AI generatif yang mendukung input audio.
---

# Mengembangkan aplikasi obrolan berkemampuan audio

Dalam latihan ini, Anda menggunakan model AI generatif *Phi-4-multimodal-instruct* untuk menghasilkan respons terhadap perintah yang mencakup file audio. Anda akan menyediakan aplikasi bantuan AI untuk perusahaan pemasok produsen dengan menggunakan Azure AI Foundry dan layanan Inferensi Model Azure AI untuk meringkas pesan suara pelanggan.

Latihan ini memakan waktu sekitar **30** menit.

## Membuat proyek Azure OpenAI

Mari kita mulai dengan membuat proyek Azure AI Foundry.

1. Di browser web, buka [portal Azure AI Foundry](https://ai.azure.com) di `https://ai.azure.com` dan masuk menggunakan kredensial Azure Anda. Tutup semua tip atau panel mulai cepat yang terbuka saat pertama kali Anda masuk, dan jika perlu, gunakan logo **Azure AI Foundry** di kiri atas untuk menavigasi ke halaman beranda, yang terlihat sama dengan gambar berikut:

    ![Tangkapan layar portal Azure AI Foundry.](../media/ai-foundry-home.png)

1. Di beranda, pilih **+ Buat proyek**.
1. Di wizard **Buat proyek**, masukkan nama yang valid untuk proyek Anda dan jika hub yang telah ada disarankan, pilih opsi untuk membuat yang baru. Kemudian tinjau sumber daya Azure yang akan dibuat secara otomatis untuk mendukung hub dan proyek Anda.
1. Pilih **Kustomisasi** dan tentukan pengaturan berikut untuk hub Anda:
    - **Nama hub** : *Nama yang valid untuk hub Anda*
    - **Langganan**: *Langganan Azure Anda*
    - **Grup sumber daya**: *Buat atau pilih grup sumber daya*
    - **Wilayah**: Pilih salah satu wilayah berikut\*:
        - US Timur
        - US Timur 2
        - US Tengah Utara
        - US Tengah Selatan
        - Swedia Tengah
        - US Barat
        - US Barat 3
    - **Menyambungkan Layanan Azure AI atau Azure OpenAI**: *Membuat sumber daya Layanan AI baru*
    - **Menyambungkan Azure AI Search**: Lewati koneksi

    > \* Pada saat penulisan, model Microsoft *Phi-4-multimodal-instruct* yang akan kita gunakan dalam latihan ini tersedia di wilayah ini. Anda dapat memeriksa ketersediaan regional terbaru untuk model tertentu dalam [dokumentasi Azure AI Foundry](https://learn.microsoft.com/azure/ai-foundry/how-to/deploy-models-serverless-availability#region-availability). Jika batas kuota tercapai di akhir latihan, Anda mungkin perlu membuat sumber daya lain di wilayah yang berbeda.

1. Pilih **Berikutnya** dan tinjau konfigurasi Anda. Lalu pilih **Buat** dan tunggu hingga prosesnya selesai.
1. Saat proyek Anda dibuat, tutup tips apa pun yang ditampilkan dan tinjau halaman proyek di portal Azure AI Foundry, yang akan terlihat mirip dengan gambar berikut:

    ![Tangkapan layar detail proyek Azure AI di portal Azure AI Foundry.](../media/ai-foundry-project.png)

## Menyebarkan model multimodal

Sekarang Anda siap untuk menyebarkan model multimodal yang dapat mendukung input berbasis audio. Ada beberapa model yang dapat Anda pilih, termasuk model OpenAI *gpt-4o* . Dalam latihan ini, kita akan menggunakan model *Phi-4-multimodal-instruct*.

1. Di toolbar di kanan atas halaman proyek Azure AI Foundry Anda, gunakan ikon **Fitur pratinjau** (**&#9215;**) untuk memastikan bahwa fitur **Sebarkan model ke layanan inferensi model Azure AI** diaktifkan. Fitur ini memastikan penyebaran model Anda tersedia untuk layanan Inferensi Azure AI, yang akan Anda gunakan dalam kode aplikasi Anda.
1. Di panel sebelah kiri untuk proyek Anda, di bagian **Aset saya**, pilih halaman **Model + titik akhir**.
1. Pada halaman **Model + titik akhir** , di tab **Penyebaran model** di menu **+ Sebarkan model** pilih **Sebarkan model dasar**.
1. Cari model **Phi-4-multimodal-instruct** dalam daftar, lalu pilih dan konfirmasikan.
1. Setujui perjanjian lisensi jika diminta, lalu sebarkan model dengan pengaturan berikut dengan memilih **Sesuaikan** dalam detail penyebaran:
    - **Nama penyebaran**: *Nama yang valid untuk penyebaran model Anda*
    - **Tipe penyebaran**: Standar Global
    - **Detail penyebaran**: *Gunakan pengaturan default*
1. Tunggu hingga status penyediaan penyebaran menjadi **Selesai**.

## Membuat aplikasi klien

Setelah menyebarkan model, Anda dapat menggunakan penyebaran dalam aplikasi klien.

> **Tips**: Anda dapat memilih untuk mengembangkan solusi Anda sendiri menggunakan Python atau Microsoft C#. Ikuti instruksi di bagian yang sesuai untuk bahasa yang Anda pilih.

### Menyiapkan konfigurasi aplikasi

1. Di portal Azure AI Foundry, lihat halaman **Gambaran Umum** untuk proyek Anda.
1. Di area **Detail proyek**, perhatikan **string koneksi Proyek**. Anda akan menggunakan string koneksi ini untuk menyambungkan ke proyek Anda di aplikasi klien.
1. Buka tab browser baru (biarkan portal Azure AI Foundry tetap terbuka di tab yang sudah ada). Kemudian di tab baru, telusuri [Portal Azure](https://portal.azure.com) di `https://portal.azure.com`; masuk menggunakan kredensial Azure Anda jika diminta.

    Tutup pemberitahuan selamat datang apa pun untuk melihat halaman beranda portal Azure.

1. Gunakan tombol **[\>_]** di sebelah kanan bilah pencarian di bagian atas halaman untuk membuat Cloud Shell baru di portal Azure, dengan memilih lingkungan ***PowerShell*** dengan tidak ada penyimpanan pada langganan Anda.

    Cloud shell menyediakan antarmuka baris perintah dalam panel di bagian bawah portal Azure. Anda dapat mengubah ukuran atau memaksimalkan panel ini untuk mempermudah pekerjaan.

    > **Catatan**: Jika sebelumnya Anda telah membuat cloud shell yang menggunakan lingkungan *Bash* , alihkan ke ***PowerShell***.

1. Di toolbar cloud shell, di menu **Pengaturan**, pilih **Buka versi Klasik** (ini diperlukan untuk menggunakan editor kode).

    **<font color="red">Pastikan Anda telah beralih ke versi klasik cloud shell sebelum melanjutkan.</font>**

1. Di panel cloud shell, masukkan perintah berikut untuk mengkloning repo GitHub yang berisi file kode untuk latihan ini (ketik perintah, atau salin ke clipboard lalu klik kanan di baris perintah dan tempel sebagai teks biasa):


    ```
    rm -r mslearn-ai-audio -f
    git clone https://github.com/MicrosoftLearning/mslearn-ai-language mslearn-ai-audio
    ```

    > **Tips**: Saat Anda menempelkan perintah ke cloudshell, ouput mungkin mengambil sejumlah besar buffer layar. Anda dapat menghapus layar dengan memasukkan `cls` perintah untuk mempermudah fokus pada setiap tugas.

1. Setelah repositori dikloning, navigasikan ke folder yang berisi file kode aplikasi obrolan:  

    **Python**

    ```
    cd mslearn-ai-audio/Labfiles/09-audio-chat/Python
    ```

    **C#**

    ```
    cd mslearn-ai-audio/Labfiles/09-audio-chat/C-sharp
    ```

1. Di panel baris perintah cloud shell, masukkan perintah berikut untuk menginstal pustaka yang akan Anda gunakan:

    **Python**

    ```
    python -m venv labenv
    ./labenv/bin/Activate.ps1
    pip install -r requirements.txt azure-identity azure-ai-projects azure-ai-inference
    ```

    **C#**

    ```
    dotnet add package Azure.Identity
    dotnet add package Azure.AI.Inference --version 1.0.0-beta.3
    dotnet add package Azure.AI.Projects --version 1.0.0-beta.3
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

1. Dalam file kode, ganti penanda **your_project_connection_string** dengan string koneksi untuk proyek Anda (disalin dari halaman **Gambaran Umum** proyek di portal Azure AI Foundry), dan penanda **your_model_deployment** dengan nama yang Anda tetapkan ke penyebaran model Phi-4-multimodal-instruct Anda.

1. Setelah Anda mengganti placeholder, di editor kode,gunakan perintah **CTRL+S** atau **Klik kanan > Simpan** untuk menyimpan perubahan Anda dan kemudian gunakan perintah **CTRL+Q** atau **Klik kanan > Keluar** untuk menutup editor kode sambil tetap membuka baris perintah cloud shell.

### Menulis kode untuk menyambungkan ke proyek Anda dan mendapatkan klien obrolan untuk model Anda

> **Tips**: Saat Anda menambahkan kode, pastikan untuk mempertahankan indentasi yang benar.

1. Masukkan perintah berikut untuk mengedit file kode yang telah disediakan:

    **Python**

    ```
    code audio-chat.py
    ```

    **C#**

    ```
    code Program.cs
    ```

1. Dalam file kode, perhatikan pernyataan yang sudah ada yang telah ditambahkan di bagian atas file untuk mengimpor namespace SDK yang diperlukan. Kemudian, temukan komentar **Tambahkan referensi**, tambahkan kode berikut untuk mereferensikan namespace di pustaka yang Anda instal sebelumnya:

    **Python**

    ```python
    # Add references
    from dotenv import load_dotenv
    from azure.identity import DefaultAzureCredential
    from azure.ai.projects import AIProjectClient
    from azure.ai.inference.models import (
        SystemMessage,
        UserMessage,
        TextContentItem,
    )
    ```

    **C#**

    ```csharp
    // Add references
    using Azure.Identity;
    using Azure.AI.Projects;
    using Azure.AI.Inference;
    ```

1. Dalam fungsi **utama**, di bawah komentar **Dapatkan pengaturan konfigurasi**, perhatikan bahwa kode memuat string koneksi proyek dan nilai nama penyebaran model yang Anda tentukan dalam file konfigurasi.

1. Temukan komentar **Inisialisasi klien proyek**, tambahkan kode berikut untuk menghubungkan proyek Azure AI Foundry Anda dengan menggunakan kredensial Azure yang Anda gunakan untuk masuk saat ini:

    **Python**

    ```python
    # Initialize the project client
    project_client = AIProjectClient.from_connection_string(
        conn_str=project_connection,
        credential=DefaultAzureCredential())
    ```

    **C#**

    ```csharp
    // Initialize the project client
    var projectClient = new AIProjectClient(project_connection,
                        new DefaultAzureCredential());
    ```

1. Di bawah komentar **Dapatkan klien obrolan**, tambahkan kode berikut untuk membuat objek klien untuk mengobrol dengan model Anda:

    **Python**

    ```python
    # Get a chat client
    chat_client = project_client.inference.get_chat_completions_client(model=model_deployment)
    ```

    **C#**

    ```csharp
    // Get a chat client
    ChatCompletionsClient chat = projectClient.GetChatCompletionsClient();
    ```
    

### Menulis kode untuk mengirimkan perintah berbasis audio berbasis URL

1. Di editor kode untuk file **audio-chat.py**, di bagian loop, di bawah komentar **Dapatkan respons terhadap input audio**, tambahkan kode berikut untuk mengirimkan perintah yang menyertakan audio berikut:

    <video controls src="../media/avocados.mp4" title="Permintaan untuk avocado" width="150"></video>

    **Python**

    ```python
    # Get a response to audio input
    file_path = "https://github.com/MicrosoftLearning/mslearn-ai-language/raw/refs/heads/main/Labfiles/09-audio-chat/data/avocados.mp3"
    response = chat_client.complete(
        messages=[
            SystemMessage(system_message),
            UserMessage(
                [
                    TextContentItem(text=prompt),
                    {
                        "type": "audio_url",
                        "audio_url": {"url": file_path}
                    }
                ]
            )
        ]
    )
    print(response.choices[0].message.content)
    ```

    **C#**

    ```csharp
    // Get a response to audio input
    string audioUrl = "https://github.com/MicrosoftLearning/mslearn-ai-language/raw/refs/heads/main/Labfiles/09-audio-chat/data/avocados.mp3";
    var requestOptions = new ChatCompletionsOptions()
    {
        Messages =
        {
            new ChatRequestSystemMessage(system_message),
            new ChatRequestUserMessage(
                new ChatMessageTextContentItem(prompt),
                new ChatMessageAudioContentItem(new Uri(audioUrl))),
        },
        Model = model_deployment
    };
    var response = chat.Complete(requestOptions);
    Console.WriteLine(response.Value.Content);
    ```

1. Gunakan perintah **CTRL+S** untuk menyimpan perubahan Anda ke file kode. Anda juga dapat menutup editor kode (**CTRL+Q**) jika mau.

1. Di panel baris perintah cloud shell di bawah editor kode, masukkan perintah berikut untuk menjalankan aplikasi:

    **Python**

    ```
    python audio-chat.py
    ```

    **C#**

    ```
    dotnet run
    ```

1. Saat diminta, masukkan perintah  

    ```
    Can you summarize this customer's voice message?
    ```

1. Tinjau responsnya.

### Menggunakan file audio lain

1. Di editor kode untuk kode aplikasi Anda, temukan kode yang Anda tambahkan sebelumnya di bawah komentar **Dapatkan respons terhadap input audio**. Kemudian ubah kode sebagai berikut untuk memilih file audio yang berbeda:

    <video controls src="../media/fresas.mp4" title="Permintaan untuk stroberi" width="150"></video>

    **Python**

    ```python
    # Get a response to audio input
    file_path = "https://github.com/MicrosoftLearning/mslearn-ai-language/raw/refs/heads/main/Labfiles/09-audio-chat/data/fresas.mp3"
    response = chat_client.complete(
        messages=[
            SystemMessage(system_message),
            UserMessage(
                [
                    TextContentItem(text=prompt),
                    {
                        "type": "audio_url",
                        "audio_url": {"url": file_path}
                    }
                ]
            )
        ]
    )
    print(response.choices[0].message.content)
    ```

    **C#**

    ```csharp
    // Get a response to audio input
    string audioUrl = "https://github.com/MicrosoftLearning/mslearn-ai-language/raw/refs/heads/main/Labfiles/09-audio-chat/data/fresas.mp3";
    var requestOptions = new ChatCompletionsOptions()
    {
        Messages =
        {
            new ChatRequestSystemMessage(system_message),
            new ChatRequestUserMessage(
                new ChatMessageTextContentItem(prompt),
                new ChatMessageAudioContentItem(new Uri(audioUrl))),
        },
        Model = model_deployment
    };
    var response = chat.Complete(requestOptions);
    Console.WriteLine(response.Value.Content);
    ```

1. Gunakan perintah **CTRL+S** untuk menyimpan perubahan Anda ke file kode. Anda juga dapat menutup editor kode (**CTRL+Q**) jika mau.

1. Di panel baris perintah cloud shell di bawah editor kode, masukkan perintah berikut untuk menjalankan aplikasi:

    **Python**

    ```
    python audio-chat.py
    ```

    **C#**

    ```
    dotnet run
    ```

1. Saat diminta, masukkan perintah berikut: 
    
    ```
    Can you summarize this customer's voice message? Is it time-sensitive?
    ```

1. Tinjau responsnya. Kemudian masukkan `quit` untuk keluar dari program.

    > **Catatan**: Dalam aplikasi sederhana ini, kita belum menerapkan logika untuk mempertahankan riwayat percakapan; sehingga model akan memperlakukan setiap perintah sebagai permintaan baru tanpa konteks perintah sebelumnya.

1. Anda dapat terus menjalankan aplikasi, memilih jenis perintah yang berbeda dan mencoba permintaan yang berbeda. Setelah selesai, tekan enter `quit` untuk mengakhiri program.

    Jika Anda punya waktu, Anda dapat memodifikasi kode untuk menggunakan perintah sistem yang berbeda dan file audio yang dapat diakses di internet.

    > **Catatan**: Dalam aplikasi sederhana ini, kami belum menerapkan logika untuk mempertahankan riwayat percakapan; sehingga model akan memperlakukan setiap permintaan sebagai permintaan baru tanpa konteks perintah sebelumnya.

## Ringkasan

Dalam latihan ini, Anda menggunakan Azure AI Foundry dan SDK Azure AI Inference untuk membuat aplikasi klien menggunakan model multimodal untuk menghasilkan respons terhadap audio.

## Penghapusan

Setelah selesai menjelajahi Azure AI Foundry, Anda harus menghapus sumber daya yang telah Anda buat di latihan ini untuk menghindari biaya Azure yang tidak perlu.

1. Kembali ke tab browser yang berisi portal Azure (atau buka kembali [portal Azure](https://portal.azure.com) di `https://portal.azure.com` tab browser baru) dan lihat konten grup sumber daya tempat Anda menyebarkan sumber daya yang digunakan dalam latihan ini.
1. Pada toolbar pilih **Hapus grup sumber daya**.
1. Masukkan nama grup sumber daya untuk mengonfirmasi bahwa Anda ingin menghapusnya, dan pilih Hapus.
