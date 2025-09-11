---
lab:
  title: Menerjemahkan Teks
  description: Terjemahkan teks yang disediakan antara bahasa yang didukung dengan Azure AI Penerjemah.
---

# Menerjemahkan Teks

**Azure AI Penerjemah** adalah layanan yang memungkinkan Anda menerjemahkan teks antar bahasa. Dalam latihan ini, Anda akan menggunakannya untuk membuat aplikasi sederhana yang menerjemahkan input dalam bahasa apa pun yang didukung ke bahasa target pilihan Anda.

Meskipun latihan ini didasarkan pada Python, Anda dapat mengembangkan aplikasi terjemahan teks menggunakan beberapa SDK khusus bahasa; termasuk:

- [Pustaka klien Azure AI Translation untuk Python](https://pypi.org/project/azure-ai-translation-text/)
- [Pustaka klien Azure AI Translation untuk .NET](https://www.nuget.org/packages/Azure.AI.Translation.Text)
- [Pustaka klien Azure AI Translation untuk JavaScript](https://www.npmjs.com/package/@azure-rest/ai-translation-text)

Latihan ini memakan waktu sekitar **30** menit.

## Memprovisikan sumber daya *Azure AI Penerjemah*

Jika Anda belum memilikinya di langganan, Anda harus menyediakan sumber daya **Azure AI Penerjemah**

1. Buka portal Microsoft Azure di `https://portal.azure.com`, dan masuk menggunakan akun Microsoft yang terkait dengan langganan Azure Anda.
1. Di bidang pencarian di bagian atas, cari **Penerjemah** lalu pilih **Penerjemah** dalam hasil.
1. Buat sumber daya dengan pengaturan berikut:
    - **Langganan**: *Langganan Azure Anda*
    - **Grup sumber daya**: *Pilih atau buat grup sumber daya*
    - **Wilayah**: *Pilih wilayah yang tersedia*
    - **Nama**: *Masukkan nama unik*
    - **Tingkat harga**: Pilih **F0** (*gratis*), atau **S** (*standar*) jika F tidak tersedia.
1. Pilih **Tinjau + buat**, lalu pilih **Buat** untuk menyediakan sumber daya.
1. Tunggu hingga penyebaran selesai, lalu buka sumber daya yang disebarkan.
1. Tampilkan halaman **Kunci dan Titik Akhir**. Anda akan memerlukan informasi di halaman ini nanti dalam latihan.

## Persiapan untuk mengembangkan aplikasi di Cloud Shell

Untuk menguji kemampuan terjemahan teks Azure AI Penerjemah, Anda akan mengembangkan aplikasi konsol sederhana di Azure Cloud Shell.

1. Gunakan tombol **[\>_]** di sebelah kanan bilah pencarian di bagian atas halaman untuk membuat Cloud Shell baru di portal Azure, dengan memilih lingkungan ***PowerShell***. Cloud shell menyediakan antarmuka baris perintah dalam panel di bagian bawah portal Azure.

    > **Catatan**: Jika sebelumnya Anda telah membuat cloud shell yang menggunakan lingkungan *Bash* , alihkan ke ***PowerShell***.

1. Di toolbar cloud shell, di menu **Pengaturan**, pilih **Buka versi Klasik** (ini diperlukan untuk menggunakan editor kode).

    **<font color="red">Pastikan Anda telah beralih ke versi klasik cloud shell sebelum melanjutkan.</font>**

1. Di panel PowerShell, masukkan perintah berikut untuk mengkloning repositori GitHub untuk latihan ini:

    ```
   rm -r mslearn-ai-language -f
   git clone https://github.com/microsoftlearning/mslearn-ai-language
    ```

    > **Tips**: Saat Anda memasukkan perintah ke cloudshell, output-nya mungkin mengambil sejumlah besar buffer layar. Anda dapat menghapus layar dengan memasukkan `cls` perintah untuk mempermudah fokus pada setiap tugas.

1. Setelah repositori dikloning, navigasikan ke folder yang berisi file kode aplikasi obrolan:  

    ```
   cd mslearn-ai-language/Labfiles/06-translator-sdk/Python/translate-text
    ```

## Mengonfigurasi aplikasi Anda

1. Di panel baris perintah, jalankan perintah berikut untuk menampilkan file kode di folder **translate-text**:

    ```
   ls -a -l
    ```

    File menyertakan file konfigurasi (**.env**) dan file kode (**translate.py**).

1. Buat lingkungan virtual Python dan instal paket SDK Azure AI Translation dan paket lain yang diperlukan dengan menjalankan perintah berikut:

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-ai-translation-text==1.0.1
    ```

1. Masukkan perintah berikut untuk mengedit file konfigurasi aplikasi:

    ```
   code .env
    ```

    File dibuka dalam editor kode.

1. Perbarui nilai konfigurasi untuk menyertakan **wilayah** dan **kunci** dari sumber daya Penerjemah Azure AI yang Anda buat (tersedia di halaman **Kunci dan Titik Akhir **untuk sumber daya Penerjemah Azure AI Anda di portal Microsoft Azure).

    > **CATATAN**: Pastikan untuk menambahkan *wilayah* untuk sumber daya Anda, <u>bukan</u> titik akhir!

1. Setelah Anda mengganti tempat penampung, gunakan perintah **CTRL+S** atau **Klik kanan > Simpan** untuk menyimpan perubahan Anda dan kemudian gunakan perintah **CTRL+Q** atau **Klik kanan > Keluar** untuk menutup editor kode sambil tetap membuka baris perintah cloud shell.

## Menambahkan kode untuk menerjemahkan teks

1. Masukkan perintah berikut untuk mengedit file kode aplikasi:

    ```
   code translate.py
    ```

1. Tinjau kode yang sudah ada. Anda akan menambahkan kode untuk bekerja dengan SDK Azure AI Translation.

    > **Tips**: Saat Anda menambahkan kode ke file kode, pastikan untuk mempertahankan indentasi yang benar.

1. Di bagian atas file kode, di bawah referensi namespace layanan yang ada, temukan komentar **Impor namespace layanan** dan tambahkan kode berikut untuk mengimpor namespace layanan yang akan Anda perlukan untuk menggunakan SDK Translation:

    ```python
   # import namespaces
   from azure.core.credentials import AzureKeyCredential
   from azure.ai.translation.text import *
   from azure.ai.translation.text.models import InputTextItem
    ```

1. Dalam fungsi **utama**, perhatikan bahwa kode yang ada membaca pengaturan konfigurasi.
1. Temukan komentar **Buat klien menggunakan titik akhir dan** kunci dan tambahkan kode berikut:

    ```python
   # Create client using endpoint and key
   credential = AzureKeyCredential(translatorKey)
   client = TextTranslationClient(credential=credential, region=translatorRegion)
    ```

1. Temukan komentar **Pilih bahasa target** dan tambahkan kode berikut, yang menggunakan layanan Penerjemah Teks untuk menampilkan daftar bahasa yang didukung untuk terjemahan, dan meminta pengguna untuk memilih kode bahasa untuk bahasa target:

    ```python
   # Choose target language
   languagesResponse = client.get_supported_languages(scope="translation")
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

1. Temukan komentar **Terjemahkan teks** dan tambahkan kode berikut, yang berulang kali meminta pengguna untuk menerjemahkan teks, menggunakan layanan Azure AI Penerjemah untuk menerjemahkannya ke bahasa target (mendeteksi bahasa sumber secara otomatis), dan menampilkan hasilnya sampai pengguna memasukkan *quit*:

    ```python
   # Translate text
   inputText = ""
   while inputText.lower() != "quit":
        inputText = input("Enter text to translate ('quit' to exit):")
        if inputText != "quit":
            input_text_elements = [InputTextItem(text=inputText)]
            translationResponse = client.translate(body=input_text_elements, to_language=[targetLanguage])
            translation = translationResponse[0] if translationResponse else None
            if translation:
                sourceLanguage = translation.detected_language
                for translated_text in translation.translations:
                    print(f"'{inputText}' was translated from {sourceLanguage.language} to {translated_text.to} as '{translated_text.text}'.")
    ```

1. Simpan perubahan Anda (CTRL+S), lalu masukkan perintah berikut untuk menjalankan program (Anda memaksimalkan panel cloud shell dan mengubah ukuran panel untuk melihat lebih banyak teks di panel baris perintah):

    ```
   python translate.py
    ```

1. Saat diminta, masukkan bahasa target yang valid dari daftar yang ditampilkan.
1. Masukkan frasa yang akan diterjemahkan (misalnya `This is a test` atau `C'est un test`) dan lihat hasilnya, yang harus mendeteksi bahasa sumber dan menerjemahkan teks ke bahasa target.
1. Setelah selesai, masukkan `quit`. Anda dapat menjalankan aplikasi lagi dan memilih bahasa target yang berbeda.

## Membersihkan sumber daya

Jika sudah selesai menjelajahi layanan Azure AI Penerjemah, Anda dapat menghapus sumber daya yang Anda buat dalam latihan ini. Berikut caranya:

1. Tutup panel Azure Cloud Shell
1. Di portal Azure, telusuri sumber daya Azure AI Penerjemah yang Anda buat di lab ini.
1. Pada halaman sumber daya, pilih **Hapus** dan ikuti instruksi untuk menghapus sumber daya.

## Informasi selengkapnya

Untuk informasi selengkapnya tentang menggunakan layanan **Azure AI Penerjemah**, lihat [dokumentasi Azure AI Penerjemah](https://learn.microsoft.com/azure/ai-services/translator/).
