---
lab:
  title: Membuat solusi Jawaban Atas Pertanyaan
  description: Gunakan Azure AI Bahasa untuk membuat solusi jawaban atas pertanyaan kustom.
---

# Membuat Solusi Jawaban atas Pertanyaan

Salah satu skenario percakapan yang paling umum adalah memberikan dukungan melalui basis pengetahuan tentang pertanyaan yang sering diajukan (FAQ). Banyak organisasi menerbitkan FAQ sebagai dokumen atau halaman web, yang berfungsi dengan baik untuk kumpulan pertanyaan dan jawaban yang kecil, tetapi dokumen besar dapat sulit dan memakan waktu untuk dicari.

**Azure AI Bahasa** mencakup kemampuan*menjawab pertanyaan* yang memungkinkan Anda membuat basis pengetahuan dari pasangan pertanyaan dan jawaban yang dapat ditanyakan menggunakan input bahasa alami, dan paling sering digunakan sebagai sumber daya yang dapat digunakan bot untuk mencari jawaban atas pertanyaan yang diajukan oleh pengguna. Dalam latihan ini, Anda akan menggunakan SDK Python Azure AI Bahasa untuk analitik teks guna menerapkan aplikasi jawaban atas pertanyaan sederhana.

Meskipun latihan ini didasarkan pada Python, Anda dapat mengembangkan aplikasi jawaban atas pertanyaan menggunakan beberapa SDK khusus bahasa; termasuk:

- [Pustaka klien Jawaban Atas Pertanyaan Layanan Azure AI Bahasa untuk Python](https://pypi.org/project/azure-ai-language-questionanswering/)
- [Pustaka klien Jawaban Atas Pertanyaan Layanan Azure AI Bahasa untuk .NET](https://www.nuget.org/packages/Azure.AI.Language.QuestionAnswering)

Latihan ini memakan waktu sekitar**20** menit.

## Provisi sumber daya*Azure AI Bahasa*

Jika belum berlangganan, Anda harus menyediakan sumber daya**layanan Azure AI Bahasa**. Selain itu, untuk membuat dan menghosting basis pengetahuan untuk menjawab pertanyaan, Anda perlu mengaktifkan fitur**Jawaban Atas Pertanyaan**.

1. Buka portal Microsoft Azure di`https://portal.azure.com`, dan masuk menggunakan akun Microsoft yang terkait dengan langganan Azure Anda.
1. Pilih**Buat sumber daya**.
1. Di bidang pencarian, cari**Layanan bahasa**. Kemudian, dalam hasil, pilih**Buat** di bawah**Layanan Bahasa**.
1. Pilih blok**Menjawab pertanyaan kustom**. Lalu pilih**Lanjutkan untuk membuat sumber daya Anda**. Anda harus memasukkan pengaturan berikut:

    - **Langganan**:*Langganan Azure Anda*
    - **Grup sumber daya**:*Pilih atau buat grup sumber daya*.
    - **Wilayah**:*Pilih lokasi yang tersedia*
    - **Nama**:*Masukkan nama unik*
    - **Tingkat harga**: Pilih**F0** (*gratis*), atau**S** (*standar*) jika F tidak tersedia.
    - **Wilayah Azure Search**:*Pilih lokasi di wilayah global yang sama dengan sumber daya Bahasa Anda*
    - **Tingkat harga Penelusuran Azure**: Gratis (F) (*Jika tingkat ini tidak tersedia, pilih Dasar (B)*)
    - **Pemberitahuan AI yang Bertanggung Jawab**:*Setuju*

1. Pilih**Buat + tinjau**, lalu pilih**Buat**.

    > **CATATAN** Jawaban atas Pertanyaan Kustom menggunakan Azure Cognitive Search untuk mengindeks dan mengkueri basis pengetahuan pertanyaan dan jawaban.

1. Tunggu hingga penyebaran selesai, lalu buka sumber daya yang disebarkan.
1. Lihat halaman**Titik Akhir dan Kunci** di bagian**Manajemen Sumber Daya**. Anda akan membutuhkan informasi di halaman ini nanti dalam latihan.

## Membuat Solusi Jawaban atas Pertanyaan

Untuk membuat basis pengetahuan untuk jawaban atas pertanyaan di sumber daya Azure AI Bahasa, Anda dapat menggunakan portal Language Studio untuk membuat proyek jawaban atas pertanyaan. Dalam hal ini, Anda akan membuat basis pengetahuan yang berisi pertanyaan dan jawaban tentang[Microsoft Learn](https://learn.microsoft.com/training/).

1. Di tab browser baru, buka portal Language Studio di[https://language.cognitive.azure.com/](https://language.cognitive.azure.com/) dan masuk menggunakan akun Microsoft yang terkait dengan langganan Azure.
1. Jika diminta untuk memilih sumber daya Bahasa, pilih pengaturan berikut:
    - **Direktori Azure**: Direktori Azure yang berisi langganan Anda.
    - **Langganan Azure**: Langganan Azure Anda.
    - **Tipe sumber daya**: Bahasa
    - **Nama sumber daya**: Sumber daya Azure AI Bahasa yang Anda buat sebelumnya.

    Jika Anda<u>tidak</u> diminta untuk memilih sumber daya bahasa, hal tersebut mungkin karena Anda memiliki beberapa sumber daya Bahasa dalam langganan Anda; dalam hal ini:

    1. Pada bilah di bagian atas halaman, pilih tombol**Pengaturan (&#9881;)**.
    2. Pada halaman**Pengaturan**, lihat tab**Sumber Daya**.
    3. Pilih sumber daya bahasa yang baru saja Anda buat, dan klik**Ganti sumber daya**.
    4. Di bagian atas halaman, klik**Language Studio** untuk kembali ke beranda Language Studio.

1. Di bagian atas portal Language Studio, di menu**Buat baru**, pilih**Jawaban atas pertanyaan kustom**.
1. Pada wizard ***Buat proyek**, pada halaman**Pilih pengaturan bahasa**, pilih opsi untuk**Atur bahasa untuk semua proyek**, dan pilih**bahasa Inggris** sebagai bahasa. Kemudian pilih**Berikutnya**.
1. Pada halaman**Masukkan informasi dasar**, masukkan detail berikut:
    - **Nama**`LearnFAQ`
    - **Deskripsi**:`FAQ for Microsoft Learn`
    - **Jawaban default ketika tidak ada jawaban yang dikembalikan**:`Sorry, I don't understand the question`
1. Pilih**Selanjutnya**.
1. Pada halaman**Tinjau dan selesaikan**, pilih**Buat proyek**.

## Tambahkan sumber ke basis pengetahuan

Anda dapat membuat basis pengetahuan dari awal, tetapi biasanya dimulai dengan mengimpor pertanyaan dan jawaban dari halaman FAQ atau dokumen yang ada. Dalam hal ini, Anda akan mengimpor data dari halaman web FAQ yang ada untuk Microsoft Learn, dan Anda juga akan mengimpor beberapa pertanyaan dan jawaban "obrolan" yang telah ditentukan sebelumnya untuk mendukung pertukaran percakapan umum.

1. Pada halaman**Kelola sumber** untuk proyek jawaban atas pertanyaan Anda, di daftar **&#9547; Tambahkan sumber**, pilih**URL**. Lalu pada kotak dialog**Tambahkan URL**, pilih **&#9547; Tambahkan URL** dan atur nama dan URL berikut sebelum memilih**Tambahkan semua** untuk menambahkannya ke basis pengetahuan:
    - **Nama**:`Learn FAQ Page`
    - **URL**:`https://learn.microsoft.com/en-us/training/support/faq?pivots=general`
1. Pada halaman**Kelola sumber** untuk proyek jawaban atas pertanyaan Anda, di daftar **&#9547; Tambahkan sumber**, pilih**Chitchat**. Pada kotak dialog**Tambahkan obrolan**, pilih**Ramah** dan pilih**Tambahkan obrolan**.

## Mengedit Pangkalan Pengetahuan

Basis pengetahuan Anda telah diisi dengan pasangan pertanyaan dan jawaban dari FAQ Microsoft Learn, dilengkapi dengan satu set pasangan tanya jawab percakapan*obrolan*. Anda dapat memperluas basis pengetahuan dengan menambahkan pasangan pertanyaan dan jawaban tambahan.

1. Pada proyek**LearnFAQ** Anda di Language Studio, pilih halaman**Edit basis pengetahuan** untuk melihat pasangan pertanyaan dan jawaban yang ada (jika ada tips yang ditampilkan, baca dan pilih**Mengerti** untuk menutupnya, atau pilih**Lewati semua**)
1. Di basis pengetahuan, pada tab**Pasangan pertanyaan dan jawaban**, pilih **&#65291;**, dan buat pasangan pertanyaan jawaban baru dengan pengaturan berikut:
    - **Sumber**:`https://learn.microsoft.com/en-us/training/support/faq?pivots=general`
    - **Pertanyaan**:`What are the different types of modules on Microsoft Learn?`
    - **Jawaban**:`Microsoft Learn offers various types of training modules, including role-based learning paths, product-specific modules, and hands-on labs. Each module contains units with lessons and knowledge checks to help you learn at your own pace.`
1. Pilih**Selesai**.
1. Di halaman untuk pertanyaan**Apa saja tipe modul yang tersedia di Microsoft Learn?** yang dibuat, perluas**Pertanyaan alternatif**. Kemudian tambahkan pertanyaan alternatif`How are training modules organized?`.

    Dalam beberapa kasus, masuk akal untuk mengizinkan pengguna menindaklanjuti jawaban untuk membuat percakapan*multi giliran* yang memungkinkan mereka menyempurnakan pertanyaan secara berulang agar mendapatkan jawaban yang dibutuhkan.

1. Di bawah jawaban yang Anda masukkan untuk pertanyaan tipe modul, perluas**Petunjuk tindak lanjut** dan tambahkan petunjuk tindak lanjut berikut:
    - **Teks yang ditampilkan dalam perintah kepada pengguna**:`Learn more about training`.
    - Pilih tab**Buat tautan ke pasangan baru**, dan masukkan teks ini:`You can explore modules and learning paths on the [Microsoft Learn training page](https://learn.microsoft.com/training/).`
    - Pilih**Tampilkan hanya dalam alur kontekstual**. Opsi ini memastikan bahwa jawabannya hanya dikembalikan dalam konteks pertanyaan tindak lanjut dari pertanyaan tipe modul asli.
1. Pilih**Tambahkan perintah**.

## Melatih dan menguji pangkalan pengetahuan

Sekarang setelah Anda memiliki basis pengetahuan, Anda dapat mengujinya di Language Studio.

1. Simpan perubahan ke basis pengetahuan Anda dengan memilih tombol**Simpan** di bawah tab**Pasangan pertanyaan jawaban** di sebelah kiri.
1. Setelah perubahan disimpan, pilih tombol**Uji** untuk membuka panel pengujian.
1. Pada panel pengujian, di bagian atas, batalkan pilihan**Sertakan respons jawaban singkat** (jika belum dipilih). Kemudian di bagian bawah masukkan pesan`Hello`. Tanggapan yang sesuai harus dikembalikan.
1. Pada panel pengujian, di bagian bawah, masukkan pesan`What is Microsoft Learn?`. Respons yang tepat dari FAQ akan muncul.
1. Masukkan pesan`Thanks!` Respons obrolan yang sesuai harus dikembalikan.
1. Masukkan pesan`What are the different types of modules on Microsoft Learn?`. Jawaban yang Anda buat harus dikembalikan bersama dengan tautan petunjuk tindak lanjut.
1. Pilih tautan tindak lanjut**Pelajari selengkapnya tentang pelatihan**. Jawaban tindak lanjut dengan tautan ke halaman pelatihan harus dikembalikan.
1. Setelah selesai menguji basis pengetahuan, tutup panel pengujian.

## Sebarkan basis pengetahuan

Basis pengetahuan menyediakan layanan back-end yang dapat digunakan aplikasi klien untuk menjawab pertanyaan. Sekarang Anda siap untuk mempublikasikan basis pengetahuan Anda dan mengakses antarmuka REST dari klien.

1. Dalam proyek**LearnFAQ** di Language Studio, pilih halaman**Sebarkan basis pengetahuan** dari menu navigasi di kiri.
1. Di bagian atas halaman, pilih**Sebarkan**. Kemudian pilih**Sebarkan** untuk mengonfirmasi bahwa Anda ingin menyebarkan basis pengetahuan.
1. Saat penyebaran selesai, pilih**Dapatkan URL prediksi** untuk menampilkan titik akhir REST untuk basis pengetahuan Anda dan perhatikan bahwa permintaan sampel menyertakan parameter untuk:
    - **projectName**: Nama proyek Anda (yang seharusnya adalah*LearnFAQ*)
    - **deploymentName**: Nama penerapan Anda (yang seharusnya adalah*production*)
1. Tutup kotak dialog URL prediksi.

## Persiapan untuk mengembangkan aplikasi di Cloud Shell

Anda akan mengembangkan aplikasi jawaban atas pertanyaan menggunakan Cloud Shell di portal Azure. File kode untuk aplikasi Anda telah disediakan dalam repositori GitHub.

1. Gunakan tombol **[\>_]** di sebelah kanan bilah pencarian di bagian atas halaman untuk membuat Cloud Shell baru di portal Azure, dengan memilih lingkungan***PowerShell***. Cloud shell menyediakan antarmuka baris perintah dalam panel di bagian bawah portal Azure.

    > **Catatan**: Jika sebelumnya Anda telah membuat cloud shell yang menggunakan lingkungan*Bash* , alihkan ke***PowerShell***.

1. Di toolbar cloud shell, di menu**Pengaturan**, pilih**Buka versi Klasik** (ini diperlukan untuk menggunakan editor kode).

    **<font color="red">Pastikan Anda telah beralih ke versi klasik cloud shell sebelum melanjutkan.</font>**

1. Di panel PowerShell, masukkan perintah berikut untuk mengkloning repositori GitHub untuk latihan ini:

    ```
    rm -r mslearn-ai-language -f
    git clone https://github.com/microsoftlearning/mslearn-ai-language
    ```

    > **Tips**: Saat Anda memasukkan perintah ke cloudshell, output-nya mungkin mengambil sejumlah besar buffer layar. Anda dapat menghapus layar dengan memasukkan`cls` perintah untuk mempermudah fokus pada setiap tugas.

1. Setelah repositori dikloning, navigasikan ke folder yang berisi file kode aplikasi obrolan:  

    ```
    cd mslearn-ai-language/Labfiles/02-qna/Python/qna-app
    ```

## Mengonfigurasi aplikasi Anda

1. Di panel baris perintah, jalankan perintah berikut untuk menampilkan file kode di folder**qna-app**:

    ```
   ls -a -l
    ```

    File menyertakan file konfigurasi (**.env**) dan file kode (**qna-app.py**).

1. Buat lingkungan virtual Python dan instal paket SDK Jawaban Atas Pertanyaan Azure AI Bahasa dan paket lain yang diperlukan dengan menjalankan perintah berikut:

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-ai-language-questionanswering
    ```

1. Masukkan perintah berikut untuk mengedit file konfigurasi:

    ```
    code .env
    ```

    File dibuka dalam editor kode.

1. Dalam file kode, perbarui nilai konfigurasi di dalamnya untuk mencerminkan**titik akhir** dan**kunci** autentikasi untuk sumber daya Bahasa Azure yang Anda buat (tersedia di halaman**Kunci dan Titik Akhir** untuk sumber daya Azure AI Bahasa Anda di portal Azure). Nama proyek dan nama penyebaran untuk basis pengetahuan yang Anda sebarkan juga harus ada di file ini.
1. Setelah Anda mengganti tempat penampung, gunakan perintah**CTRL+S** atau**Klik kanan > Simpan** untuk menyimpan perubahan Anda dan kemudian gunakan perintah**CTRL+Q** atau**Klik kanan > Keluar** untuk menutup editor kode sambil tetap membuka baris perintah cloud shell.

## Menambahkan kode untuk menggunakan basis pengetahuan Anda

1. Masukkan perintah berikut untuk mengedit file kode aplikasi:

    ```
    code qna-app.py
    ```

1. Tinjau kode yang sudah ada. Anda akan menambahkan kode untuk bekerja dengan basis pengetahuan Anda.

    > **Tips**: Saat Anda menambahkan kode ke file kode, pastikan untuk mempertahankan indentasi yang benar.

1. Dalam file kode, temukan komentar**Impor namespace**. Kemudian, di bawah komentar ini, tambahkan kode khusus bahasa berikut untuk mengimpor namespace layanan yang akan Anda perlukan untuk menggunakan SDK Jawaban Atas Pertanyaan:

    ```python
   # import namespaces
   from azure.core.credentials import AzureKeyCredential
   from azure.ai.language.questionanswering import QuestionAnsweringClient
    ```

1. Dalam fungsi**utama**, perhatikan bahwa kode untuk memuat titik akhir dan kunci layanan Azure AI Bahasa dari file konfigurasi telah disediakan. Kemudian, temukan komentar**Buat klien menggunakan titik akhir dan kunci**, dan tambahkan kode berikut untuk membuat klien jawaban atas pertanyaan:

    ```Python
   # Create client using endpoint and key
   credential = AzureKeyCredential(ai_key)
   ai_client = QuestionAnsweringClient(endpoint=ai_endpoint, credential=credential)
    ```

1. Dalam file kode, temukan komentar**Kirim pertanyaan dan tampilkan jawaban**, dan tambahkan kode berikut agar dapat membaca pertanyaan berulang kali dari baris perintah, mengirimkannya ke layanan, dan menampilkan detail jawabannya:

    ```Python
   # Submit a question and display the answer
   user_question = ''
   while True:
        user_question = input('\nQuestion:\n')
        if user_question.lower() == "quit":                
            break
        response = ai_client.get_answers(question=user_question,
                                        project_name=ai_project_name,
                                        deployment_name=ai_deployment_name)
        for candidate in response.answers:
            print(candidate.answer)
            print("Confidence: {}".format(candidate.confidence))
            print("Source: {}".format(candidate.source))
    ```

1. Simpan perubahan Anda (CTRL+S), lalu masukkan perintah berikut untuk menjalankan program (Anda memaksimalkan panel cloud shell dan mengubah ukuran panel untuk melihat lebih banyak teks di panel baris perintah):

    ```
   python qna-app.py
    ```

1. Saat diminta, masukkan pertanyaan yang akan dikirimkan ke proyek jawaban atas pertanyaan Anda; misalnya`What is a learning path?`.
1. Tinjau jawaban yang dikembalikan.
1. Ajukan pertanyaan lainnya. Setelah selesai, masukkan`quit`.

## Membersihkan sumber daya

Jika sudah selesai menjelajahi layanan Azure AI Bahasa, Anda dapat menghapus sumber daya yang Anda buat dalam latihan ini. Berikut caranya:

1. Tutup panel Azure Cloud Shell
1. Di portal Azure, telusuri sumber daya Azure AI Bahasa yang Anda buat di lab ini.
1. Pada halaman sumber daya, pilih**Hapus** dan ikuti instruksi untuk menghapus sumber daya.

## Informasi selengkapnya

Untuk mempelajari selengkapnya tentang jawaban atas pertanyaan dalam Azure AI Bahasa, lihat[Dokumentasi Azure AI Bahasa](https://learn.microsoft.com/azure/ai-services/language-service/question-answering/overview).
