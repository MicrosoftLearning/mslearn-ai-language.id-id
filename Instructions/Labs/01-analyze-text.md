---
lab:
  title: Analisa teks
  description: 'Gunakan Azure AI Bahasa untuk analisis teks, termasuk deteksi bahasa, analisis sentimen, ekstraksi frasa kunci, dan pengenalan entitas.'
---

# Menganalisis Teks

**Azure AI Bahasa** mendukung analisis teks, termasuk deteksi bahasa, analisis sentimen, ekstraksi frasa kunci, dan pengenalan entitas.

Misalnya, agen perjalanan ingin memproses ulasan hotel yang telah dikirimkan ke situs web perusahaan. Dengan menggunakan layanan Azure AI Bahasa, mereka dapat menentukan bahasa yang digunakan untuk menulis setiap ulasan, sentimen (positif, netral, atau negatif) dari ulasan, frasa kunci yang mungkin menunjukkan topik utama yang dibahas dalam ulasan, dan entitas bernama, seperti tempat, landmark, atau orang yang disebutkan dalam ulasan. Dalam latihan ini, Anda akan menggunakan SDK Python Azure AI Bahasa untuk analitik teks guna menerapkan aplikasi ulasan hotel sederhana berdasarkan contoh ini.

Meskipun latihan ini didasarkan pada Python, Anda dapat mengembangkan aplikasi analitik teks menggunakan beberapa SDK khusus bahasa; termasuk:

- [Pustaka klien Analitik Teks Azure AI untuk Python](https://pypi.org/project/azure-ai-textanalytics/)
- [Pustaka klien Analitik Teks Azure AI untuk .NET](https://www.nuget.org/packages/Azure.AI.TextAnalytics)
- [Pustaka klien Analitik Teks Azure AI untuk JavaScript](https://www.npmjs.com/package/@azure/ai-text-analytics)

Latihan ini memakan waktu sekitar **30** menit.

## Provisikan sumber daya *Azure AI Bahasa*

Jika belum memilikinya di langganan, Anda harus menyediakan sumber daya **layanan Azure AI Bahasa** di langganan Azure Anda.

1. Buka portal Microsoft Azure di `https://portal.azure.com`, dan masuk menggunakan akun Microsoft yang terkait dengan langganan Azure Anda.
1. Pilih **Buat sumber daya**.
1. Di bidang pencarian, cari **Layanan bahasa**. Kemudian, dalam hasil, pilih **Buat** di bawah **Layanan Bahasa**.
1. Pilih **Lanjutkan untuk membuat sumber daya Anda**.
1. Provisikan sumber daya menggunakan pengaturan berikut:
    - **Langganan**: *Langganan Azure Anda*.
    - **Grup sumber daya**: *Memilih atau membuat grup sumber daya*.
    - **Wilayah**: *Pilih wilayah yang tersedia*
    - **Nama**: *Masukkan nama unik*.
    - **Tingkat harga**: Pilih **F0** (*gratis*), atau **S** (*standar*) jika F tidak tersedia.
    - **Pemberitahuan AI yang Bertanggung Jawab**: Setuju.
1. Pilih **Tinjau + buat**, lalu pilih **Buat** untuk menyediakan sumber daya.
1. Tunggu hingga penyebaran selesai, lalu buka sumber daya yang disebarkan.
1. Lihat halaman **Titik Akhir dan Kunci** di bagian **Manajemen Sumber Daya**. Anda akan memerlukan informasi di halaman ini nanti dalam latihan.

## Mengkloning repositori untuk kursus ini

Anda akan mengembangkan kode menggunakan Cloud Shell dari portal Azure. File kode untuk aplikasi Anda telah disediakan dalam repositori GitHub.

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
    cd mslearn-ai-language/Labfiles/01-analyze-text/Python/text-analysis
    ```

## Mengonfigurasi aplikasi Anda

1. Di panel baris perintah, jalankan perintah berikut untuk menampilkan file kode di folder **text-analysis**:

    ```
   ls -a -l
    ```

    File menyertakan file konfigurasi (**.env**) dan file kode (**text-analysis.py**). Teks yang akan dianalisis aplikasi Anda ada di subfolder **ulasan**.

1. Buat lingkungan virtual Python dan instal paket SDK Analitik Teks Azure AI Bahasa dan paket lain yang diperlukan dengan menjalankan perintah berikut:

    ```
    python -m venv labenv
    ./labenv/bin/Activate.ps1
    pip install -r requirements.txt azure-ai-textanalytics==5.3.0
    ```

1. Masukkan perintah berikut untuk mengedit file konfigurasi aplikasi:

    ```
   code .env
    ```

    File dibuka dalam editor kode.

1. Perbarui nilai konfigurasi untuk menyertakan **titik akhir** dan **kunci** dari sumber daya Bahasa Azure yang Anda buat (tersedia di halaman **Kunci dan Titik Akhir** untuk sumber daya Azure AI Bahasa Anda di portal Microsoft Azure)
1. Setelah Anda mengganti tempat penampung, gunakan perintah **CTRL+S** atau **Klik kanan > Simpan** untuk menyimpan perubahan Anda dan kemudian gunakan perintah **CTRL+Q** atau **Klik kanan > Keluar** untuk menutup editor kode sambil tetap membuka baris perintah cloud shell.

## Menambahkan kode untuk menyambungkan ke sumber daya Azure AI Bahasa Anda

1. Masukkan perintah berikut untuk mengedit file kode aplikasi:

    ```
    code text-analysis.py
    ```

1. Tinjau kode yang sudah ada. Anda akan menambahkan kode untuk bekerja dengan SDK Analitik Teks AI Bahasa.

    > **Tips**: Saat Anda menambahkan kode ke file kode, pastikan untuk mempertahankan indentasi yang benar.

1. Di bagian atas file kode, di bawah referensi namespace layanan yang ada, temukan komentar **Impor namespace layanan** dan tambahkan kode berikut untuk mengimpor namespace layanan yang akan Anda perlukan untuk menggunakan SDK Analitik Teks:

    ```python
   # import namespaces
   from azure.core.credentials import AzureKeyCredential
   from azure.ai.textanalytics import TextAnalyticsClient
    ```

1. Dalam fungsi **utama**, perhatikan bahwa kode untuk memuat titik akhir dan kunci layanan Azure AI Bahasa dari file konfigurasi telah disediakan. Kemudian temukan komentar **Buat klien menggunakan titik akhir dan kunci**, dan tambahkan kode berikut untuk membuat klien untuk API Analisis Teks:

    ```Python
   # Create client using endpoint and key
   credential = AzureKeyCredential(ai_key)
   ai_client = TextAnalyticsClient(endpoint=ai_endpoint, credential=credential)
    ```

1. Simpan perubahan Anda (CTRL+S), lalu masukkan perintah berikut untuk menjalankan program (Anda memaksimalkan panel cloud shell dan mengubah ukuran panel untuk melihat lebih banyak teks di panel baris perintah):

    ```
   python text-analysis.py
    ```

1. Amati keluaran saat kode harus berjalan tanpa kesalahan, menampilkan konten setiap file teks ulasan dalam folder **ulasan**. Aplikasi berhasil membuat klien untuk Text Analytics API tetapi tidak menggunakannya. Kami akan memperbaikinya di bagian berikutnya.

## Menambahkan kode untuk mendeteksi bahasa

Sekarang setelah Anda membuat klien untuk API, mari kita gunakan untuk mendeteksi bahasa yang digunakan untuk menulis setiap ulasan.

1. Di editor kode, temukan komentar **Dapatkan bahasa**. Kemudian, tambahkan kode yang diperlukan untuk mendeteksi bahasa di setiap dokumen ulasan:

    ```python
   # Get language
   detectedLanguage = ai_client.detect_language(documents=[text])[0]
   print('\nLanguage: {}'.format(detectedLanguage.primary_language.name))
    ```

     > **Catatan**: *Dalam contoh ini, setiap ulasan dianalisis satu per satu, menghasilkan panggilan terpisah ke layanan untuk setiap file. Pendekatan alternatif adalah membuat kumpulan dokumen dan meneruskannya ke layanan dalam satu panggilan. Dalam kedua pendekatan tersebut, respons dari layanan terdiri dari kumpulan dokumen; itulah sebabnya dalam kode Python di atas, indeks dokumen pertama (dan satu-satunya) dalam respons ([0]) ditentukan.*

1. Simpan perubahan Anda. Kemudian jalankan kembali program.
1. Amati hasilnya, perhatikan bahwa kali ini bahasa untuk setiap ulasan diidentifikasi.

## Tambahkan kode untuk mengevaluasi sentimen

*Analisis sentimen* adalah teknik yang umum digunakan untuk mengklasifikasikan teks sebagai *positif* atau *negatif* (atau kemungkinan *netral* atau *campuran* ). Ini biasanya digunakan untuk menganalisis posting media sosial, ulasan produk, dan item lain di mana sentimen teks dapat memberikan wawasan yang berguna.

1. Di editor kode, temukan komentar **Dapatkan sentimen**. Kemudian, tambahkan kode yang diperlukan untuk mendeteksi sentimen dari setiap dokumen ulasan:

    ```python
   # Get sentiment
   sentimentAnalysis = ai_client.analyze_sentiment(documents=[text])[0]
   print("\nSentiment: {}".format(sentimentAnalysis.sentiment))
    ```

1. Simpan perubahan Anda. Kemudian tutup editor kode dan jalankan kembali program.
1. Amati output, perhatikan bahwa sentimen ulasan terdeteksi.

## Tambahkan kode untuk mengidentifikasi frasa kunci

Akan berguna untuk mengidentifikasi frase kunci dalam tubuh teks untuk membantu menentukan topik utama yang dibahas.

1. Di editor kode, temukan komentar **Dapatkan frasa kunci**. Kemudian, tambahkan kode yang diperlukan untuk mendeteksi frasa kunci di setiap dokumen ulasan:

    ```python
   # Get key phrases
   phrases = ai_client.extract_key_phrases(documents=[text])[0].key_phrases
   if len(phrases) > 0:
        print("\nKey Phrases:")
        for phrase in phrases:
            print('\t{}'.format(phrase))
    ```

1. Simpan perubahan Anda dan jalankan kembali program.
1. Amati hasilnya, perhatikan bahwa setiap dokumen berisi frasa kunci yang memberikan beberapa wawasan tentang ulasan tersebut.

## Tambahkan kode untuk mengekstrak entitas

Sering kali, dokumen atau kumpulan teks lainnya menyebutkan orang, tempat, periode waktu, atau entitas lain. API Analytics teks dapat mendeteksi beberapa kategori (dan sub-kategori) entitas dalam teks Anda.

1. Di editor kode, temukan komentar **Dapatkan entitas**. Kemudian, tambahkan kode yang diperlukan untuk mengidentifikasi entitas yang disebutkan di setiap ulasan:

    ```python
   # Get entities
   entities = ai_client.recognize_entities(documents=[text])[0].entities
   if len(entities) > 0:
        print("\nEntities")
        for entity in entities:
            print('\t{} ({})'.format(entity.text, entity.category))
    ```

1. Simpan perubahan Anda dan jalankan kembali program.
1. Amati keluarannya, perhatikan entitas yang telah terdeteksi dalam teks.

## Tambahkan kode untuk mengekstrak entitas tertaut

Selain entitas yang dikategorikan, Text Analytics API dapat mendeteksi entitas yang diketahui memiliki tautan ke sumber data, seperti Wikipedia.

1. Di editor kode, temukan komentar **Dapatkan entitas tertaut**. Kemudian, tambahkan kode yang diperlukan untuk mengidentifikasi entitas tertaut yang disebutkan di setiap ulasan:

    ```python
   # Get linked entities
   entities = ai_client.recognize_linked_entities(documents=[text])[0].entities
   if len(entities) > 0:
        print("\nLinks")
        for linked_entity in entities:
            print('\t{} ({})'.format(linked_entity.name, linked_entity.url))
    ```

1. Simpan perubahan Anda dan jalankan kembali program.
1. Amati output, catat entitas terkait yang diidentifikasi.

## Membersihkan sumber daya

Jika sudah selesai menjelajahi layanan Azure AI Bahasa, Anda dapat menghapus sumber daya yang Anda buat dalam latihan ini. Berikut caranya:

1. Tutup panel Azure Cloud Shell
1. Di portal Azure, telusuri sumber daya Azure AI Bahasa yang Anda buat di lab ini.
1. Pada halaman sumber daya, pilih **Hapus** dan ikuti instruksi untuk menghapus sumber daya.

## Informasi selengkapnya

Untuk informasi selengkapnya tentang menggunakan **Azure AI Bahasa**, lihat [dokumentasi](https://learn.microsoft.com/azure/ai-services/language-service/).
