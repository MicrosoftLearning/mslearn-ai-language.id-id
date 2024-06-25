---
lab:
  title: Membuat Solusi Jawaban atas Pertanyaan
  module: Module 6 - Create question answering solutions with Azure AI Language
---

# Membuat Solusi Jawaban atas Pertanyaan

Salah satu skenario percakapan yang paling umum adalah memberikan dukungan melalui basis pengetahuan tentang pertanyaan yang sering diajukan (FAQ). Banyak organisasi menerbitkan FAQ sebagai dokumen atau halaman web, yang berfungsi dengan baik untuk kumpulan pertanyaan dan jawaban yang kecil, tetapi dokumen besar dapat sulit dan memakan waktu untuk dicari.

**Azure AI Bahasa** mencakup kemampuan *menjawab pertanyaan* yang memungkinkan Anda membuat basis pengetahuan dari pasangan pertanyaan dan jawaban yang dapat ditanyakan menggunakan input bahasa alami, dan paling sering digunakan sebagai sumber daya yang dapat digunakan bot untuk mencari jawaban atas pertanyaan yang diajukan oleh pengguna.

## Provisi sumber daya *Azure AI Bahasa*

Jika belum berlangganan, Anda harus menyediakan sumber daya **layanan Azure AI Bahasa**. Selain itu, untuk membuat dan menghosting basis pengetahuan untuk menjawab pertanyaan, Anda perlu mengaktifkan fitur **Jawaban Atas Pertanyaan**.

1. Buka portal Microsoft Azure di `https://portal.azure.com`, dan masuk menggunakan akun Microsoft yang terkait dengan langganan Azure Anda.
1. Di bidang pencarian di bagian atas masukkan **layanan Azure AI**, lalu tekan **Enter**.
1. Pilih **Buat** di bawah sumber daya **Layanan Bahasa** pada hasil.
1. **Pilih** blok **Jawaban atas pertanyaan kustom**. Lalu pilih **Lanjutkan untuk membuat sumber daya Anda**. Anda harus memasukkan pengaturan berikut:

    - **Langganan**: *Langganan Azure Anda*
    - **Grup sumber daya**: *Pilih atau buat grup sumber daya*.
    - **Wilayah**: *Pilih lokasi yang tersedia*
    - **Nama**: *Masukkan nama unik*
    - **Tingkat harga**: Pilih **F0** (*gratis*), atau **S** (*standar*) jika F tidak tersedia.
    - **Wilayah Azure Search**: *Pilih lokasi di wilayah global yang sama dengan sumber daya Bahasa Anda*
    - **Tingkat harga Penelusuran Azure**: Gratis (F) (*Jika tingkat ini tidak tersedia, pilih Dasar (B)*)
    - **Pemberitahuan AI yang Bertanggung Jawab**: *Setuju*

1. Pilih **Buat + tinjau**, lalu pilih **Buat**.

    > **CATATAN** Jawaban atas Pertanyaan Kustom menggunakan Azure Cognitive Search untuk mengindeks dan mengkueri basis pengetahuan pertanyaan dan jawaban.

1. Tunggu hingga penyebaran selesai, lalu buka sumber daya yang disebarkan.
1. Tampilkan halaman Loop **Kunci dan Titik Akhir**. Anda akan membutuhkan informasi di halaman ini nanti dalam latihan.

## Membuat Solusi Jawaban atas Pertanyaan

Untuk membuat basis pengetahuan untuk jawaban atas pertanyaan di sumber daya Azure AI Bahasa, Anda dapat menggunakan portal Language Studio untuk membuat proyek jawaban atas pertanyaan. Dalam hal ini, Anda akan membuat basis pengetahuan yang berisi pertanyaan dan jawaban tentang [Microsoft Learn](https://docs.microsoft.com/learn).

1. Di tab browser baru, buka portal Language Studio di [https://language.cognitive.azure.com/](https://language.cognitive.azure.com/) dan masuk menggunakan akun Microsoft yang terkait dengan langganan Azure.
1. Jika diminta untuk memilih sumber daya Bahasa, pilih pengaturan berikut:
    - **Direktori Azure**: Direktori Azure yang berisi langganan Anda.
    - **Langganan Azure**: Langganan Azure Anda.
    - **Tipe sumber daya**: Bahasa
    - **Nama sumber daya**: Sumber daya Azure AI Bahasa yang Anda buat sebelumnya.

    Jika Anda <u>tidak</u> diminta untuk memilih sumber daya bahasa, hal tersebut mungkin karena Anda memiliki beberapa sumber daya Bahasa dalam langganan Anda; dalam hal ini:

    1. Pada bilah di bagian atas halaman, pilih tombol **Pengaturan (&#9881;)**.
    2. Pada halaman **Pengaturan**, lihat tab **Sumber Daya**.
    3. Pilih sumber daya bahasa yang baru saja Anda buat, dan klik **Ganti sumber daya**.
    4. Di bagian atas halaman, klik **Language Studio** untuk kembali ke beranda Language Studio.

1. Di bagian atas portal Language Studio, di menu **Buat baru**, pilih **Jawaban atas pertanyaan kustom**.
1. Pada wizard ***Buat proyek**, pada halaman **Pilih pengaturan bahasa**, pilih opsi untuk **Atur bahasa untuk semua proyek di sumber daya ini**, dan pilih **bahasa Inggris** sebagai bahasa. Kemudian pilih **Berikutnya**.
1. Pada halaman **Masukkan informasi dasar**, masukkan detail berikut:
    - **Nama** `LearnFAQ`
    - **Deskripsi**: `FAQ for Microsoft Learn`
    - **Jawaban default ketika tidak ada jawaban yang dikembalikan**: `Sorry, I don't understand the question`
1. Pilih **Selanjutnya**.
1. Pada halaman **Tinjau dan selesaikan**, pilih **Buat proyek**.

## Tambahkan sumber ke basis pengetahuan

Anda dapat membuat basis pengetahuan dari awal, tetapi biasanya dimulai dengan mengimpor pertanyaan dan jawaban dari halaman FAQ atau dokumen yang ada. Dalam hal ini, Anda akan mengimpor data dari halaman web FAQ yang ada untuk dipelajari Microsoft, dan Anda juga akan mengimpor beberapa pertanyaan dan jawaban "obrolan" yang telah ditentukan sebelumnya untuk mendukung pertukaran percakapan umum.

1. Pada halaman **Kelola sumber** untuk proyek jawaban atas pertanyaan Anda, di daftar **&#9547; Tambahkan sumber**, pilih **URL**. Lalu pada kotak dialog **Tambahkan URL**, pilih **&#9547; Tambahkan URL** dan atur nama dan URL berikut sebelum memilih **Tambahkan semua** untuk menambahkannya ke basis pengetahuan:
    - **Nama**: `Learn FAQ Page`
    - **URL**: `https://docs.microsoft.com/en-us/learn/support/faq`
1. Pada halaman **Kelola sumber** untuk proyek jawaban atas pertanyaan Anda, di daftar **&#9547; Tambahkan sumber**, pilih **Chitchat**. Pada kotak dialog **Tambahkan obrolan**, pilih **Ramah** dan pilih **Tambahkan obrolan**.

## Mengedit Pangkalan Pengetahuan

Basis pengetahuan Anda telah diisi dengan pasangan pertanyaan dan jawaban dari FAQ Microsoft Learn, dilengkapi dengan satu set pasangan tanya jawab percakapan *obrolan*. Anda dapat memperluas basis pengetahuan dengan menambahkan pasangan pertanyaan dan jawaban tambahan.

1. Pada proyek **LearnFAQ** Anda di Language Studio, pilih halaman **Edit basis pengetahuan** untuk melihat pasangan pertanyaan dan jawaban yang ada (jika ada tips yang ditampilkan, baca dan pilih **Mengerti** untuk menutupnya, atau pilih **Lewati semua**)
1. Di basis pengetahuan, pada tab **Pasangan pertanyaan dan jawaban**, pilih **&#65291;**, dan buat pasangan pertanyaan jawaban baru dengan pengaturan berikut:
    - **Sumber**:  `https://docs.microsoft.com/en-us/learn/support/faq`
    - **Pertanyaan**: `What are Microsoft credentials?`
    - **Jawaban**: `Microsoft credentials enable you to validate and prove your skills with Microsoft technologies.`
1. Pilih **Selesai**.
1. Di halaman untuk pertanyaan **Apa itu kredensial Microsoft?** yang dibuat, perluas **Pertanyaan alternatif**. Kemudian tambahkan pertanyaan alternatif `How can I demonstrate my Microsoft technology skills?`.

    Dalam beberapa kasus, masuk akal untuk mengizinkan pengguna menindaklanjuti jawaban untuk membuat percakapan *multi giliran* yang memungkinkan mereka menyempurnakan pertanyaan secara berulang agar mendapatkan jawaban yang dibutuhkan.

1. Di bawah jawaban yang Anda masukkan untuk pertanyaan sertifikasi, perluas **Petunjuk tindak lanjut** dan tambahkan petunjuk tindak lanjut berikut:
    - **Teks yang ditampilkan dalam perintah kepada pengguna**: `Learn more about credentials`.
    - Pilih tab **Buat tautan ke pasangan baru**, dan masukkan teks ini: `You can learn more about credentials on the [Microsoft credentials page](https://docs.microsoft.com/learn/credentials/).`
    - Pilih **Tampilkan hanya dalam alur kontekstual**. Opsi ini memastikan bahwa jawabannya hanya dikembalikan dalam konteks pertanyaan lanjutan dari pertanyaan sertifikasi asli.
1. Pilih **Tambahkan perintah**.

## Melatih dan menguji pangkalan pengetahuan

Sekarang setelah Anda memiliki basis pengetahuan, Anda dapat mengujinya di Language Studio.

1. Simpan perubahan ke basis pengetahuan Anda dengan memilih tombol **Simpan** di bawah tab **Pasangan pertanyaan jawaban** di sebelah kiri.
1. Setelah perubahan disimpan, pilih tombol **Uji** untuk membuka panel pengujian.
1. Pada panel pengujian, di bagian atas, batalkan pilihan **Sertakan respons jawaban singkat** (jika belum dipilih). Kemudian di bagian bawah masukkan pesan `Hello`. Tanggapan yang sesuai harus dikembalikan.
1. Pada panel pengujian, di bagian bawah, masukkan pesan `What is Microsoft Learn?`. Respons yang tepat dari FAQ akan muncul.
1. Masukkan pesan `Thanks!` Respons obrolan yang sesuai harus dikembalikan.
1. Masukkan pesan `Tell me about Microsoft credentials`. Jawaban yang Anda buat harus dikembalikan bersama dengan tautan petunjuk tindak lanjut.
1. Pilih tautan tindak lanjut**Pelajari selengkapnya tentang kredensial**. Jawaban tindak lanjut dengan tautan ke halaman sertifikasi harus dikembalikan.
1. Setelah selesai menguji basis pengetahuan, tutup panel pengujian.

## Sebarkan basis pengetahuan

Basis pengetahuan menyediakan layanan back-end yang dapat digunakan aplikasi klien untuk menjawab pertanyaan. Sekarang Anda siap untuk mempublikasikan basis pengetahuan Anda dan mengakses antarmuka REST dari klien.

1. Dalam proyek **LearnFAQ** di Language Studio, pilih halaman **Sebarkan basis pengetahuan**.
1. Di bagian atas halaman, pilih **Sebarkan**. Kemudian pilih **Sebarkan** untuk mengonfirmasi bahwa Anda ingin menyebarkan basis pengetahuan.
1. Saat penyebaran selesai, pilih **Dapatkan URL prediksi** untuk menampilkan titik akhir REST untuk basis pengetahuan Anda dan perhatikan bahwa permintaan sampel menyertakan parameter untuk:
    - **projectName**: Nama proyek Anda (yang seharusnya adalah *LearnFAQ*)
    - **deploymentName**: Nama penerapan Anda (yang seharusnya adalah *produksi*)
1. Tutup kotak dialog URL prediksi.

## Bersiap untuk mengembangkan aplikasi di Visual Studio Code

Anda akan mengembangkan aplikasi jawaban atas pertanyaan menggunakan Visual Studio Code. File kode untuk aplikasi Anda telah disediakan di repositori GitHub.

> **Tips**: Jika Anda telah mengkloning repositori **mslearn-ai-language**, buka di kode Visual Studio. Atau, ikuti langkah-langkah ini untuk mengkloningnya ke lingkungan pengembangan Anda.

1. Memulai Visual Studio Code.
2. Buka palet (SHIFT+CTRL+P) dan jalankan **Git: Perintah klon** untuk mengkloning repositori `https://github.com/MicrosoftLearning/mslearn-ai-language` ke folder lokal (tidak masalah folder mana).
3. Setelah repositori dikloning, buka folder di Visual Studio Code.

    > **Catatan**: Jika Visual Studio Code menampilkan pesan pop-up yang meminta Anda untuk memercayai kode yang Anda buka, klik opsi **Ya, saya memercayai penulisnya** pada pop-up tersebut.

4. Tunggu sementara file tambahan diinstal untuk mendukung proyek kode C# di repositori.

    > **Catatan**: Jika Anda diminta untuk menambahkan aset yang diperlukan guna membangun dan men-debug, pilih **Tidak Sekarang**.

## Mengonfigurasi aplikasi Anda

Aplikasi untuk C# dan Python telah disediakan, beserta contoh file teks yang akan Anda gunakan untuk menguji peringkasan. Kedua aplikasi memiliki fungsionalitas yang sama. Pertama, Anda akan menyelesaikan beberapa bagian kunci dari aplikasi agar dapat menggunakan sumber daya Azure AI Language.

1. Di Visual Studio Code, pada panel **Explorer**, telusuri folder **Labfiles/02-qna** dan perluas folder **CSharp** atau **Python**, tergantung pada preferensi bahasa Anda dan folder **qna-app** yang ada di dalamnya. Setiap folder berisi file khusus bahasa untuk aplikasi yang akan Anda integrasikan dengan fungsionalitas jawaban atas pertanyaan Azure AI Bahasa.
2. Klik kanan folder **qna-app** yang berisi file kode Anda dan buka terminal terintegrasi. Kemudian instal paket SDK jawaban atas pertanyaan Azure AI Bahasa dengan menjalankan perintah yang sesuai berdasarkan preferensi bahasa Anda:

    **C#**:

    ```
    dotnet add package Azure.AI.Language.QuestionAnswering
    ```

    **Python**:

    ```
    pip install azure-ai-language-questionanswering
    ```

3. Di panel **Explorer**, pada folder **qna-app**, buka file konfigurasi sesuai bahasa pilihan Anda

    - **C#**: appsettings.json
    - **Python**: .env
    
4. Perbarui nilai konfigurasi untuk menyertakan  **titik akhir** dan **kunci** dari sumber daya Bahasa Azure yang Anda buat (tersedia di halaman web **Kunci dan Titik Akhir** untuk sumber daya Azure AI Bahasa Anda di portal Azure). Nama proyek dan nama penyebaran untuk basis pengetahuan yang Anda sebarkan juga harus ada di file ini.
5. Simpan file konfigurasi.

## Tambahkan kode ke aplikasi

Sekarang Anda sudah siap untuk menambahkan kode yang diperlukan untuk mengimpor pustaka SDK yang diperlukan, membuat koneksi terautentikasi ke proyek yang telah disebarkan, dan mengirimkan pertanyaan.

1. Perhatikan bahwa folder **qna-app** berisi file kode untuk aplikasi klien:

    - **C#**: Program.cs
    - **Python**: qna-app.py

    Buka file kode dan di bagian atas, di bawah referensi namespace yang ada, temukan komentar **Impor namespace**. Kemudian, di bawah komentar ini, tambahkan kode khusus bahasa berikut untuk mengimpor ruang nama yang Anda perlukan untuk menggunakan SDK Analisis Teks:

    **C#**: Programs.cs

    ```csharp
    // import namespaces
    using Azure;
    using Azure.AI.Language.QuestionAnswering;
    ```

    **Python**: qna-app.py

    ```python
    # import namespaces
    from azure.core.credentials import AzureKeyCredential
    from azure.ai.language.questionanswering import QuestionAnsweringClient
    ```

1. Dalam fungsi **Utama**, perhatikan bahwa kode untuk memuat titik akhir dan kunci layanan Azure AI Bahasa dari file konfigurasi telah disediakan. Kemudian temukan komentar **Buat klien menggunakan titik akhir dan kunci**, dan tambahkan kode berikut untuk membuat klien untuk API Analisis Teks:

    **C#**: Programs.cs

    ```C#
    // Create client using endpoint and key
    AzureKeyCredential credentials = new AzureKeyCredential(aiSvcKey);
    Uri endpoint = new Uri(aiSvcEndpoint);
    QuestionAnsweringClient aiClient = new QuestionAnsweringClient(endpoint, credentials);
    ```

    **Python**: qna-app.py

    ```Python
    # Create client using endpoint and key
    credential = AzureKeyCredential(ai_key)
    ai_client = QuestionAnsweringClient(endpoint=ai_endpoint, credential=credential)
    ```

1. Di fungsi **Utama**, temukan komentar **Kirim pertanyaan dan tampilkan jawaban**, dan tambahkan kode berikut agar dapat membaca pertanyaan berulang kali dari baris perintah, mengirimkannya ke layanan, dan menampilkan detail jawabannya:

    **C#**: Programs.cs

    ```C#
    // Submit a question and display the answer
    string user_question = "";
    while (user_question.ToLower() != "quit")
        {
            Console.Write("Question: ");
            user_question = Console.ReadLine();
            QuestionAnsweringProject project = new QuestionAnsweringProject(projectName, deploymentName);
            Response<AnswersResult> response = aiClient.GetAnswers(user_question, project);
            foreach (KnowledgeBaseAnswer answer in response.Value.Answers)
            {
                Console.WriteLine(answer.Answer);
                Console.WriteLine($"Confidence: {answer.Confidence:P2}");
                Console.WriteLine($"Source: {answer.Source}");
                Console.WriteLine();
            }
        }
    ```

    **Python**: qna-app.py

    ```Python
    # Submit a question and display the answer
    user_question = ''
    while user_question.lower() != 'quit':
        user_question = input('\nQuestion:\n')
        response = ai_client.get_answers(question=user_question,
                                        project_name=ai_project_name,
                                        deployment_name=ai_deployment_name)
        for candidate in response.answers:
            print(candidate.answer)
            print("Confidence: {}".format(candidate.confidence))
            print("Source: {}".format(candidate.source))
    ```

1. Simpan perubahan Anda dan kembali ke terminal terintegrasi untuk folder **qna-app**, dan masukkan perintah berikut untuk menjalankan program:

    - **C#:** `dotnet run`
    - **Python**: `python qna-app.py`

    > **Tips**: Anda dapat menggunakan ikon **Maksimalkan ukuran panel** (**^**) di bar alat terminal untuk melihat teks konsol lainnya.

1. Saat diminta, masukkan pertanyaan yang akan dikirimkan ke proyek jawaban atas pertanyaan Anda; misalnya `What is a learning path?`.
1. Tinjau jawaban yang dikembalikan.
1. Ajukan pertanyaan lainnya. Setelah selesai, masukkan `quit`.

## Membersihkan sumber daya

Jika sudah selesai menjelajahi layanan Azure AI Bahasa, Anda dapat menghapus sumber daya yang Anda buat dalam latihan ini. Berikut caranya:

1. Buka portal Microsoft Azure di `https://portal.azure.com`, dan masuk menggunakan akun Microsoft yang terkait dengan langganan Azure Anda.
2. Telusuri ke sumber daya Azure AI Bahasa yang Anda buat di lab ini.
3. Pada halaman sumber daya, pilih **Hapus** dan ikuti instruksi untuk menghapus sumber daya.

## Informasi selengkapnya

Untuk mempelajari selengkapnya tentang jawaban atas pertanyaan dalam Azure AI Bahasa, lihat [Dokumentasi Azure AI Bahasa](https://learn.microsoft.com/azure/ai-services/language-service/question-answering/overview).
