---
lab:
  title: Mengembangkan agen suara Azure AI Voice Live
  description: Pelajari cara membuat aplikasi web untuk mengaktifkan interaksi suara real time dengan agen Azure AI Voice Live.
---

# Mengembangkan agen suara Azure AI Voice Live

Dalam latihan ini, Anda menyelesaikan aplikasi web Python berbasis Flask yang memungkinkan interaksi suara real time dengan agen. Anda menambahkan kode untuk menginisialisasi sesi dan menangani peristiwa sesi. Anda menggunakan skrip penyebaran yang: menyebarkan model AI; membuat gambar aplikasi di Azure Container Registry (ACR) menggunakan tugas ACR; lalu membuat instans Azure App Service yang menarik gambar tersebut. Untuk menguji aplikasi, Anda memerlukan perangkat audio dengan kemampuan mikrofon dan speaker.

Meskipun latihan ini didasarkan pada Python, Anda dapat mengembangkan aplikasi serupa menggunakan SDK khusus bahasa lainnya; termasuk:

- [Pustaka klien Azure VoiceLive untuk .NET](https://www.nuget.org/packages/Azure.AI.VoiceLive/)

Tugas yang dilakukan dalam latihan ini:

* Mengunduh file dasar untuk aplikasi
* Menambahkan kode untuk menyelesaikan aplikasi web
* Meninjau basis kode keseluruhan
* Memperbarui dan menjalankan skrip penyebaran
* Melihat dan menguji aplikasi

Latihan ini memakan waktu sekitar **30** menit.

## Meluncurkan Azure Cloud Shell dan mengunduh file

Di bagian latihan ini, Anda mengunduh file zip yang berisi file dasar untuk aplikasi.

1. Di browser Anda, navigasikan ke portal Azure [https://portal.azure.com](https://portal.azure.com); masuk dengan kredensial Azure Anda jika diminta.

1. Gunakan tombol **[\>_]** di sebelah kanan bilah pencarian di bagian atas halaman untuk membuat shell cloud baru di portal Azure, memilih lingkungan ***Bash***. Cloud shell menyediakan antarmuka baris perintah dalam panel di bagian bawah portal Azure.

    > **Catatan**: Jika sebelumnya Anda telah membuat cloud shell yang menggunakan lingkungan *PowerShell* , alihkan ke ***Bash***.

1. Di toolbar cloud shell, di menu **Pengaturan**, pilih **Buka versi Klasik** (ini diperlukan untuk menggunakan editor kode).

1. Jalankan perintah berikut di shell **Bash** untuk mengunduh dan membuka zip file latihan. Perintah kedua juga akan mengubah ke direktori untuk file latihan.

    ```bash
    wget https://github.com/MicrosoftLearning/mslearn-ai-language/raw/refs/heads/main/downloads/python/voice-live-web.zip
    ```

    ```
    unzip voice-live-web.zip && cd voice-live-web
    ```

## Menambahkan kode untuk menyelesaikan aplikasi web

Setelah file latihan diunduh, langkah selanjutnya adalah menambahkan kode untuk menyelesaikan aplikasi. Langkah-langkah berikut dilakukan di shell cloud. 

>**Tips:** Mengubah ukuran shell cloud untuk menampilkan informasi dan kode lainnya, dengan menyeret batas atas. Anda juga dapat menggunakan tombol minimalkan dan maksimalkan untuk beralih antara shell cloud dan antarmuka portal utama.

Jalankan perintah berikut untuk mengubah ke direktori *src* sebelum Anda melanjutkan latihan.

```bash
cd src
```

### Menambahkan kode untuk mengimplementasikan asisten langsung suara

Di bagian ini, Anda menambahkan kode untuk mengimplementasikan asisten langsung suara. Metode **\_\_init\_\_** menginisialisasi asisten suara dengan menyimpan parameter koneksi Azure VoiceLive (titik akhir, kredensial, model, suara, dan instruksi sistem) dan menyiapkan variabel status runtime bahasa umum untuk mengelola siklus hidup koneksi dan menangani interupsi pengguna selama percakapan. Metode **start** mengimpor komponen SDK Azure VoiceLive esensial yang akan digunakan untuk membuat koneksi WebSocket dan mengonfigurasi sesi suara real time.

1. Jalankan perintah berikut untuk membuka file *flask_app.py* untuk pengeditan.

    ```bash
    code flask_app.py
    ```

1. Cari komentar **# BEGIN VOICE LIVE ASSISTANT IMPLEMENTATION - ALIGN CODE WITH COMMENT** dalam kode. Salin kode di bawah ini dan masukkan tepat di bawah komentar. Pastikan untuk memeriksa indentasi.

    ```python
    def __init__(
        self,
        endpoint: str,
        credential,
        model: str,
        voice: str,
        instructions: str,
        state_callback=None,
    ):
        # Store Azure Voice Live connection and configuration parameters
        self.endpoint = endpoint
        self.credential = credential
        self.model = model
        self.voice = voice
        self.instructions = instructions
        
        # Initialize runtime state - connection established in start()
        self.connection = None
        self._response_cancelled = False  # Used to handle user interruptions
        self._stopping = False  # Signals graceful shutdown
        self.state_callback = state_callback or (lambda *_: None)

    async def start(self):
        # Import Voice Live SDK components needed for establishing connection and configuring session
        from azure.ai.voicelive.aio import connect  # type: ignore
        from azure.ai.voicelive.models import (
            RequestSession,
            ServerVad,
            AzureStandardVoice,
            Modality,
            InputAudioFormat,
            OutputAudioFormat,
        )  # type: ignore
    ```

1. Masukkan **ctrl+s** untuk menyimpan perubahan dan biarkan editor tetap terbuka untuk bagian berikutnya.

### Menambahkan kode untuk mengimplementasikan asisten langsung suara

Di bagian ini, Anda menambahkan kode untuk mengonfigurasi sesi langsung suara. Langkah ini menentukan modalitas (audio saja tidak didukung oleh API), instruksi sistem yang menentukan perilaku asisten, suara Azure TTS untuk respons, format audio untuk aliran input dan output, dan Deteksi Aktivitas Suara (VAD) sisi Server yang menentukan bagaimana model mendeteksi kapan pengguna mulai dan berhenti berbicara.

1. Cari komentar **# BEGIN CONFIGURE VOICE LIVE SESSION - ALIGN CODE WITH COMMENT** dalam kode. Salin kode di bawah ini dan masukkan tepat di bawah komentar. Pastikan untuk memeriksa indentasi.

    ```python
    # Configure VoiceLive session with audio/text modalities and voice activity detection
    session_config = RequestSession(
        modalities=[Modality.TEXT, Modality.AUDIO],
        instructions=self.instructions,
        voice=voice_cfg,
        input_audio_format=InputAudioFormat.PCM16,
        output_audio_format=OutputAudioFormat.PCM16,
        turn_detection=ServerVad(threshold=0.5, prefix_padding_ms=300, silence_duration_ms=500),
    )
    await conn.session.update(session=session_config)
    ```

1. Masukkan **ctrl+s** untuk menyimpan perubahan dan biarkan editor tetap terbuka untuk bagian berikutnya.

### Menambahkan kode untuk menangani peristiwa sesi

Di bagian ini, Anda menambahkan kode untuk menambahkan penanganan aktivitas untuk sesi kolaborasi suara. Penanganan peristiwa merespons peristiwa sesi VoiceLive utama selama siklus hidup percakapan: sinyal **_handle_session_updated** ketika sesi siap untuk input pengguna, **_handle_speech_started** mendeteksi saat pengguna mulai berbicara dan menerapkan logika interupsi dengan menghentikan pemutaran audio asisten yang sedang berlangsung dan membatalkan respons yang sedang berlangsung untuk memungkinkan alur percakapan alami, dan **_handle_speech_stopped** menangani ketika pengguna telah selesai berbicara dan asisten mulai memproses input.

1. Cari komentar **# BEGIN HANDLE SESSION EVENTS - ALIGN CODE WITH COMMENT** dalam kode. Salin kode di bawah ini dan masukkan tepat di bawah komentar. Pastikan untuk memeriksa indentasi.

    ```python
    async def _handle_event(self, event, conn, verbose=False):
        """Handle Voice Live events with clear separation by event type."""
        # Import event types for processing different Voice Live server events
        from azure.ai.voicelive.models import ServerEventType
        
        event_type = event.type
        if verbose:
            _broadcast({"type": "log", "level": "debug", "event_type": str(event_type)})
        
        # Route Voice Live server events to appropriate handlers
        if event_type == ServerEventType.SESSION_UPDATED:
            await self._handle_session_updated()
        elif event_type == ServerEventType.INPUT_AUDIO_BUFFER_SPEECH_STARTED:
            await self._handle_speech_started(conn)
        elif event_type == ServerEventType.INPUT_AUDIO_BUFFER_SPEECH_STOPPED:
            await self._handle_speech_stopped()
        elif event_type == ServerEventType.RESPONSE_AUDIO_DELTA:
            await self._handle_audio_delta(event)
        elif event_type == ServerEventType.RESPONSE_AUDIO_DONE:
            await self._handle_audio_done()
        elif event_type == ServerEventType.RESPONSE_DONE:
            # Reset cancellation flag but don't change state - _handle_audio_done already did
            self._response_cancelled = False
        elif event_type == ServerEventType.ERROR:
            await self._handle_error(event)

    async def _handle_session_updated(self):
        """Session is ready for conversation."""
        self.state_callback("ready", "Session ready. You can start speaking now.")

    async def _handle_speech_started(self, conn):
        """User started speaking - handle interruption if needed."""
        self.state_callback("listening", "Listening… speak now")
        
        try:
            # Stop any ongoing audio playback on the client side
            _broadcast({"type": "control", "action": "stop_playback"})
            
            # If assistant is currently speaking or processing, cancel the response to allow interruption
            current_state = assistant_state.get("state")
            if current_state in {"assistant_speaking", "processing"}:
                self._response_cancelled = True
                await conn.response.cancel()
                _broadcast({"type": "log", "level": "debug", 
                          "msg": f"Interrupted assistant during {current_state}"})
            else:
                _broadcast({"type": "log", "level": "debug", 
                          "msg": f"User speaking during {current_state} - no cancellation needed"})
        except Exception as e:
            _broadcast({"type": "log", "level": "debug", 
                      "msg": f"Exception in speech handler: {e}"})

    async def _handle_speech_stopped(self):
        """User stopped speaking - processing input."""
        self.state_callback("processing", "Processing your input…")

    async def _handle_audio_delta(self, event):
        """Stream assistant audio to clients."""
        if self._response_cancelled:
            return  # Skip cancelled responses
            
        # Update state when assistant starts speaking
        if assistant_state.get("state") != "assistant_speaking":
            self.state_callback("assistant_speaking", "Assistant speaking…")
        
        # Extract and broadcast Voice Live audio delta as base64 to WebSocket clients
        audio_data = getattr(event, "delta", None)
        if audio_data:
            audio_b64 = base64.b64encode(audio_data).decode("utf-8")
            _broadcast({"type": "audio", "audio": audio_b64})

    async def _handle_audio_done(self):
        """Assistant finished speaking."""
        self._response_cancelled = False
        self.state_callback("ready", "Assistant finished. You can speak again.")

    async def _handle_error(self, event):
        """Handle Voice Live errors."""
        error = getattr(event, "error", None)
        message = getattr(error, "message", "Unknown error") if error else "Unknown error"
        self.state_callback("error", f"Error: {message}")

    def request_stop(self):
        self._stopping = True
    ```

1. Masukkan **ctrl+s** untuk menyimpan perubahan dan biarkan editor tetap terbuka untuk bagian berikutnya.

### Meninjau kode di aplikasi

Sejauh ini, Anda telah menambahkan kode ke aplikasi untuk mengimplementasikan agen dan menangani peristiwa agen. Luangkan beberapa menit untuk meninjau kode lengkap dan komentar untuk mendapatkan pemahaman yang lebih baik tentang bagaimana aplikasi menangani status dan operasi klien.

1. Setelah selesai, masukkan **ctrl+q** untuk keluar dari editor. 

## Memperbarui dan menjalankan skrip penyebaran

Di bagian ini Anda membuat perubahan kecil pada skrip penyebaran **azdeploy.sh**, lalu menjalankan penyebaran. 

### Memperbarui skrip penyebaran

Hanya ada dua nilai yang harus Anda ubah di bagian atas skrip penyebaran **azdeploy.sh**. 

* Nilai **rg** menentukan grup sumber daya yang berisi penyebaran. Anda dapat menerima nilai default, atau memasukkan nilai kustom jika Anda perlu menyebarkan ke grup sumber daya tertentu.

* Nilai **location** menetapkan wilayah untuk penyebaran. Model *gpt-4o* yang digunakan dalam latihan dapat disebarkan ke wilayah lain, tetapi mungkin ada batasan di wilayah tertentu. Jika penyebaran gagal di wilayah yang Anda pilih, coba **eastus2** atau **swedencentral**. 

    ```
    rg="rg-voicelive" # Replace with your resource group
    location="eastus2" # Or a location near you
    ```

1. Jalankan perintah berikut di Cloud Shell untuk mulai mengedit skrip penyebaran.

    ```bash
    cd ~/voice-live-web
    ```
    
    ```bash
    code azdeploy.sh
    ```

1. Perbarui nilai untuk **rg** dan **location** untuk memenuhi kebutuhan Anda, lalu masukkan **ctrl+s** untuk menyimpan perubahan dan **ctrl+q** untuk keluar dari editor.

### Jalankan skrip penyebaran

Skrip penyebaran menyebarkan model AI dan membuat sumber daya yang diperlukan di Azure untuk menjalankan aplikasi kontainer di App Service.

1. Jalankan perintah berikut di Cloud Shell untuk mulai menyebarkan sumber daya Azure dan aplikasi.

    ```bash
    bash azdeploy.sh
    ```

1. Pilih **option 1** untuk penyebaran awal.

    Penyebaran harus selesai dalam 5-10 menit. Selama penyebaran, Anda mungkin dimintai informasi/diminta melakukan tindakan berikut:
    
    * Jika Anda diminta untuk mengautentikasi ke Azure, ikuti petunjuk yang diberikan.
    * Jika Anda diminta untuk memilih langganan, gunakan tombol panah untuk menyorot langganan Anda dan tekan **Enter**. 
    * Anda mungkin akan melihat beberapa peringatan selama penyebaran dan peringatan tersebut dapat diabaikan.
    * Jika penyebaran gagal selama penyebaran model AI, ubah wilayah dalam skrip penyebaran dan coba lagi. 
    * Wilayah di Azure terkadang sibuk dan mengganggu waktu penyebaran. Jalankan kembali skrip penyebaran jika penyebaran gagal setelah penyebaran model.

## Menampilkan dan menguji aplikasi

Ketika penyebaran selesai, pesan "Penyebaran selesai!" akan ditampilkan di shell bersama dengan tautan ke aplikasi web. Anda dapat memilih tautan tersebut, atau menavigasi ke sumber daya App Service dan meluncurkan aplikasi dari sana. Dibutuhkan beberapa menit agar aplikasi dimuat. 

1. Pilih tombol **Mulai sesi** untuk menyambungkan ke model.
1. Anda akan diminta untuk memberikan akses aplikasi ke perangkat audio.
1. Mulailah berbicara dengan model saat aplikasi meminta Anda untuk mulai berbicara.

Pemecahan Masalah:

* Jika aplikasi melaporkan variabel lingkungan yang hilang, mulai ulang aplikasi di App Service.
* Jika Anda melihat pesan *gugus audio* yang berlebihan di log yang ditampilkan di aplikasi, pilih **Hentikan sesi**, lalu mulai ulang sesi. 
* Jika aplikasi gagal berfungsi sama sekali, periksa kembali semua kode yang ditambahkan dan apakah indentasi sudah tepat. Jika Anda perlu membuat perubahan apa pun, jalankan kembali penyebaran dan pilih **opsi 2** untuk memperbarui gambar saja.

## Membersihkan sumber daya

Jalankan perintah berikut di Cloud Shell untuk menghapus semua sumber daya yang disebarkan untuk latihan ini. Anda akan diminta untuk mengonfirmasi penghapusan sumber daya.

```
azd down --purge
```