# Persyaratan

## Eksekusi di Cloud Shell

* Langganan Azure dengan akses OpenAI
* Jika dijalankan di Azure Cloud Shell, pilih shell Bash. Azure CLI dan Azure Developer CLI disertakan dalam Cloud Shell.

## Menjalankan secara lokal

* Anda dapat menjalankan aplikasi web secara lokal setelah menjalankan skrip penyebaran:
    * [Azure Developer CLI (azd)](https://learn.microsoft.com/en-us/azure/developer/azure-developer-cli/install-azd)
    * [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli)
    * Langganan Azure dengan akses OpenAI


## Variabel Lingkungan

File `.env` dibuat oleh skrip *azdeploy.sh*. Titik akhir model AI, kunci API, dan nama model ditambahkan selama penyebaran sumber daya.

## Penyebaran sumber daya Azure

`azdeploy.sh` yang disediakan membuat sumber daya yang diperlukan di Azure:

* Ubah dua variabel di bagian atas skrip agar sesuai dengan kebutuhan Anda. Jangan ubah yang lain.
* Skrip:
    * Menyebarkan model *gpt-4o* menggunakan AZD.
    * Membuat layanan Azure Container Registry
    * Menggunakan tugas ACR untuk membuat dan menyebarkan gambar Dockerfile ke ACR
    * Membuat Paket App Service
    * Membuat Aplikasi App Service Web App
    * Mengonfigurasi aplikasi web untuk gambar kontainer di ACR
    * Mengonfigurasi variabel lingkungan aplikasi web
    * Skrip akan menyediakan titik akhir App Service

Skrip menyediakan dua opsi penyebaran: 1. Penyebaran penuh; dan 2. Penyebaran ulang gambar saja. Opsi 2 hanya untuk pascapenyebaran saat Anda ingin bereksperimen dengan perubahan dalam aplikasi. 

> Catatan: Anda dapat menjalankan skrip di PowerShell, atau Bash, menggunakan perintah `bash azdeploy.sh`. Perintah ini juga memungkinkan Anda menjalankan skrip di Bash tanpa harus menjadikannya dapat dieksekusi.

## Pengembangan lokal

### Provisikan model AI ke Azure

Anda dapat menjalankan proyek secara lokal dan hanya menyediakan model AI dengan mengikuti langkah-langkah berikut:

1. **Menginisialisasi lingkungan** (pilih nama deskriptif):

   ```bash
   azd env new gpt-realtime-lab --confirm
   # or: azd env new your-name-gpt-experiment --confirm
   ```
   
   **Penting**: Nama ini menjadi bagian dari nama sumber daya Azure Anda!  
   Bendera `--confirm` mengatur ini sebagai lingkungan default Anda tanpa meminta.

1. **Atur grup sumber daya Anda**:

   ```bash
   azd env set AZURE_RESOURCE_GROUP "rg-your-name-gpt"
   ```

1. **Proses masuk dan provisi sumber daya AI**:

   ```bash
   az login
   azd provision
   ```

    > **Penting**: JANGAN jalankan `azd deploy` - aplikasi ini tidak dikonfigurasi di templat AZD.

Jika Anda hanya menyediakan model menggunakan metode `azd provision`, Anda HARUS membuat file `.env` di akar direktori dengan entri berikut:

```
AZURE_VOICE_LIVE_ENDPOINT=""
AZURE_VOICE_LIVE_API_KEY=""
VOICE_LIVE_MODEL=""
VOICE_LIVE_VOICE="en-US-JennyNeural"
VOICE_LIVE_INSTRUCTIONS="You are a helpful AI assistant with a focus on world history. Respond naturally and conversationally. Keep your responses concise but engaging."
VOICE_LIVE_VERBOSE="" #Suppresses excessive logging to the terminal if running locally
```

Catatan:

1. Titik akhir adalah titik akhir untuk model dan hanya boleh menyertakan `https://<proj-name>.cognitiveservices.azure.com`.
1. Kunci API adalah kunci untuk model.
1. Model adalah nama model yang digunakan selama penyebaran.
1. Anda dapat mengambil nilai-nilai ini dari portal AI Foundry.

### Menjalankan proyek secara lokal

Proyek dibuat dan dikelola menggunakan **uv**, tetapi tidak diperlukan untuk dijalankan. 

Jika Anda menginstal **uv**:

* Jalankan `uv venv` untuk membuat lingkungan
* Jalankan `uv sync` untuk menambahkan paket
* Alias dibuat untuk aplikasi web: `uv run web` untuk memulai skrip `flask_app.py`.
* File requirements.txt dibuat dengan `uv pip compile pyproject.toml -o requirements.txt`

Jika Anda belum menginstal **uv**:

* Buat lingkungan: `python -m venv .venv`
* Aktifkan lingkungan: `.\.venv\Scripts\Activate.ps1`
* Instal dependensi: `pip install -r requirements.txt`
* Jalankan aplikasi (dari akar proyek): `python .\src\real_time_voice\flask_app.py`
