---
lab:
  title: Buat model pemahaman bahasa dengan layanan Azure AI Bahasa
  module: Module 5 - Create language understanding solutions
---

# Buat model pemahaman bahasa dengan layanan Bahasa

> **CATATAN** Fitur pemahaman bahasa percakapan dari layanan Azure AI Bahasa saat ini dalam pratinjau, dan dapat berubah. Dalam beberapa kasus, pelatihan model mungkin gagal - jika ini terjadi, coba lagi.  

Layanan Azure AI Bahasa memungkinkan Anda menentukan model *pemahaman bahasa percakapan* yang dapat digunakan aplikasi untuk menafsirkan input bahasa alami dari pengguna,  memprediksi *niat* pengguna (apa yang ingin mereka capai), dan mengidentifikasi *entitas* yang harus diterapkan niat tersebut.

Misalnya, model bahasa percakapan untuk aplikasi jam mungkin diharapkan memproses input seperti:

*Jam berapa di London?*

Jenis input ini adalah contoh dari *ucapan* (sesuatu yang mungkin dikatakan atau diketik pengguna), dengan *niat* yang diinginkan adalah untuk mendapatkan waktu di lokasi tertentu (*entitas*); dalam hal ini, London.

> **CATATAN** Tugas model bahasa percakapan adalah memprediksi niat pengguna dan mengidentifikasi entitas yang diterapkan niat tersebut. <u>Bukan</u> pekerjaan model bahasa percakapan untuk benar-benar melakukan tindakan yang diperlukan untuk memenuhi niat. Misalnya, aplikasi jam dapat menggunakan model bahasa percakapan untuk membedakan bahwa pengguna ingin mengetahui waktu di London; tetapi aplikasi klien itu sendiri kemudian harus menerapkan logika untuk menentukan waktu yang tepat dan menyajikannya kepada pengguna.

## Provisikan sumber daya *Azure AI Bahasa*

Jika belum memilikinya di langganan, Anda harus menyediakan sumber daya **layanan Azure AI Bahasa** di langganan Azure Anda.

1. Buka portal Microsoft Azure di `https://portal.azure.com`, dan masuk menggunakan akun Microsoft yang terkait dengan langganan Azure Anda.
1. Di bidang pencarian di bagian atas, cari **layanan Azure AI**. Kemudian, dalam hasil, pilih **Buat** di bawah **Layanan Bahasa**.
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
1. Tampilkan halaman **Kunci dan Titik Akhir**. Anda akan memerlukan informasi di halaman ini nanti dalam latihan.

## Membuat proyek pemahaman bahasa percakapan

Sekarang setelah Anda membuat sumber penulisan, Anda dapat menggunakannya untuk membuat proyek pemahaman bahasa percakapan.

1. Di tab browser baru, buka portal Studio Azure AI Bahasa di `https://language.cognitive.azure.com/` dan masuk menggunakan akun Microsoft yang terkait dengan langganan Azure Anda.

1. Jika diminta untuk memilih sumber daya Bahasa, pilih pengaturan berikut:

    - **Direktori Azure**: Direktori Azure yang berisi langganan Anda.
    - **Langganan Azure**: Langganan Azure Anda.
    - **Jenis sumber daya**: Bahasa.
    - **Sumber daya bahasa**: Sumber daya Azure AI Bahasa yang Anda buat sebelumnya.

    Jika Anda <u>tidak</u> diminta untuk memilih sumber daya bahasa, hal tersebut mungkin karena Anda memiliki beberapa sumber daya Bahasa dalam langganan Anda; dalam hal ini:

    1. Pada bilah di bagian atas halaman, pilih tombol **Pengaturan (&#9881;)**.
    2. Pada halaman **Pengaturan**, lihat tab **Sumber Daya**.
    3. Pilih sumber daya bahasa yang baru saja Anda buat, dan klik **Ganti sumber daya**.
    4. Di bagian atas halaman, klik **Language Studio** untuk kembali ke beranda Language Studio

1. Di bagian atas portal, di menu **Buat baru**, pilih **Pemahaman bahasa percakapan**.

1. Di kotak dialog **Buat proyek**, pada halaman **Masukkan informasi dasar**, masukkan detail berikut, lalu pilih **Berikutnya**:
    - **Nama**: `Clock`
    - **Bahasa utama ucapan**: Bahasa Inggris
    - **Aktifkan beberapa bahasa dalam proyek?**: *Tidak dipilih*
    - **Deskripsi**: `Natural language clock`

1. Pada halaman **Tinjau dan selesai**, pilih **Buat**.

### Membuat niat

Hal pertama yang akan kita lakukan dalam proyek baru adalah mendefinisikan beberapa niat. Model pada akhirnya akan memprediksi niat mana yang diminta pengguna saat mengirimkan ucapan bahasa alami.

> **Tips**: Saat mengerjakan proyek Anda, jika beberapa tips ditampilkan, baca dan klik **Mengerti** untuk menutupnya, atau pilih **Lompati semua**.

1. Pada halaman **Definisi skema**, pada tab **Niat**, pilih **&#65291; Tambahkan** untuk menambahkan niat baru bernama `GetTime`.
1. Verifikasi bahwa niat**GetTime** tercantum (bersama dengan niat **Tidak Ada** default). Kemudian tambahkan niat tambahan berikut:
    - `GetDay`
    - `GetDate`

### Beri label setiap niat dengan sampel ucapan

Untuk membantu model memprediksi niat mana yang diminta pengguna, Anda harus memberi label setiap niat dengan beberapa sampel ucapan.

1. Di panel di sebelah kiri, pilih halaman**Pelabelan Data**.

> **Tips**: Anda bisa memperluas panel dengan ikon **>>** untuk melihat nama halaman, dan menyembunyikannya lagi dengan ikon **<<**.

1. Pilih niat **GetTime baru** dan masukkan ucapan `what is the time?`. Ini menambahkan ucapan sebagai sampel input untuk niat.
1. Tambahkan ucapan tambahan berikut untuk niat**GetTime**:
    - `what's the time?`
    - `what time is it?`
    - `tell me the time`

    > **CATATAN** Untuk menambahkan ucapan baru, tulis ucapan di kotak teks di samping niat lalu tekan ENTER. 

1. Pilih niat **GetDay**dan tambahkan ucapan berikut sebagai sampel input untuk niat tersebut:
    - `what day is it?`
    - `what's the day?`
    - `what is the day today?`
    - `what day of the week is it?`

1. Pilih niat **GetDate** dan tambahkan ucapan berikut untuknya:
    - `what date is it?`
    - `what's the date?`
    - `what is the date today?`
    - `what's today's date?`

1. Setelah Anda menambahkan ucapan untuk setiap niat Anda, pilih **Simpan perubahan**.

### Terapkan dan uji model tersebut

Sekarang setelah Anda menambahkan beberapa niat, mari latih model bahasa dan lihat apakah model dapat memprediksinya dengan benar dari input pengguna.

1. Di panel sebelah kiri, pilih **Tugas pelatihan**. Lalu pilih **+ Mulai tugas pelatihan**.

1. Pada dialog **Mulai tugas pelatihan**, pilih opsi untuk melatih model baru, beri nama `Clock`. Pilih mode**Pelatihan standar** dan opsi **Pemisahan data** default.

1. Untuk memulai proses pelatihan model Anda, pilih **Latih**.

1. Saat pelatihan selesai (yang mungkin memerlukan waktu beberapa menit) **Status** pekerjaan akan berubah menjadi **Pelatihan berhasil**.

1. Pilih halaman **Performa model**, lalu pilih model **Jam**. Tinjau metrik evaluasi keseluruhan dan per niat (*presisi*, *recall*, dan *skor F1*) dan *matriks kebingungan* yang dihasilkan oleh evaluasi yang dilakukan saat pelatihan (perhatikan bahwa karena sedikitnya jumlah sampel ucapan, tidak semua niat dapat disertakan dalam hasil).

    > **CATATAN** Untuk mempelajari selengkapnya tentang metrik evaluasi, lihat [dokumentasi](https://learn.microsoft.com/azure/ai-services/language-service/conversational-language-understanding/concepts/evaluation-metrics)

1. Buka halaman **Menyebarkan model**, lalu pilih **Tambahkan penyebaran**.

1. Pada dialog **Tambahkan penyebaran**, pilih **Buat nama penyebaran baru**, lalu masukkan `production`.

1. Pilih model **Jam** di bidang **Model** lalu pilih **Sebarkan**. Penyebaran ini mungkin memerlukan waktu.

1. Saat model telah disebarkan, pilih halaman ** Penyebaran Pengujian**, lalu pilih penyebaran **produksi** di bidang **Nama penyebaran**.

1. Masukkan teks berikut di kotak teks kosong, lalu pilih **Jalankan pengujian**:

    `what's the time now?`

    Tinjau hasil yang dikembalikan, dengan mencatat bahwa hasil tersebut mencakup niat yang diprediksi (yang seharusnya **GetTime**) dan skor keyakinan yang menunjukkan probabilitas yang dihitung model untuk niat yang diprediksi. Tab JSON menunjukkan keyakinan komparatif untuk setiap niat potensial (yang memiliki skor kepercayaan tertinggi adalah tujuan yang diprediksi)

1. Kosongkan kotak teks, lalu jalankan pengujian lain dengan teks berikut:

    `tell me the time`

    Sekali lagi, tinjau niat dan skor keyakinan yang diprediksi.

1. Coba teks berikut:

    `what's the day today?`

    Semoga model memprediksi niat **GetDay**.

## Menambahkan entitas

Sejauh ini Anda telah mendefinisikan beberapa ucapan sederhana yang memetakan niat. Sebagian besar aplikasi nyata menyertakan ucapan yang lebih kompleks dari mana entitas data tertentu harus diekstraksi untuk mendapatkan lebih banyak konteks untuk niat tersebut.

### Menambahkan entitas learned

Jenis entitas yang paling umum adalah entitas *learned*, di mana model belajar mengidentifikasi nilai entitas berdasarkan contoh.

1. Di Language Studio, kembali ke halaman **Definisi skema**, lalu pada tab **Entitas**, pilih **&#65291; Tambahkan** untuk menambahkan entitas baru.

1. Di kotak dialog **Tambahkan entitas**, masukkan nama entitas `Location`dan pastikan bahwa tab **Dipelajari** dipilih. Lalu pilih **Tambahkan entitas**.

1. Setelah entitas **Lokasi** dibuat, kembali ke halaman **Pelabelan data**.
1. Pilih niat **GetTime** dan masukkan sampel ucapan baru berikut:

    `what time is it in London?`

1. Saat ucapan telah ditambahkan, pilih kata **London**, dan di daftar menu dropdown yang muncul, pilih **Lokasi** untuk menunjukkan bahwa "London" adalah contoh lokasi.

1. Tambahkan sampel ucapan lain untuk niat **GetTime**:

    `Tell me the time in Paris?`

1. Saat ucapan telah ditambahkan, pilih kata **Paris**, dan petakan ke entitas **Lokasi**.

1. Tambahkan sampel ucapan lain untuk niat **GetTime**:

    `what's the time in New York?`

1. Saat ucapan telah ditambahkan, pilih kata **New York**, dan petakan ke entitas **Lokasi**.

1. Pilih **Simpan perubahan** untuk menyimpan ucapan baru.

### Tambahkan entitas *list*

Dalam beberapa kasus, nilai yang valid untuk suatu entitas dapat dibatasi pada daftar istilah dan sinonim tertentu; yang dapat membantu aplikasi mengidentifikasi contoh entitas dalam ucapan.

1. Di Language Studio, kembali ke halaman **Definisi skema**, lalu pada tab **Entitas**, pilih **&#65291; Tambahkan** untuk menambahkan entitas baru.

1. Di kotak dialog **Tambahkan entitas**, masukkan nama entitas `Weekday` dan pilih tab entitas**Daftar**. Lalu pilih **Tambahkan entitas**.

1. Pada halaman untuk entitas **Hari Kerja**, di bagian **Dipelajari**, pastikan **Tidak diperlukan** dipilih. Kemudian, di bagian **Daftar**, pilih **&#65291; Tambahkan daftar baru**. Kemudian masukkan nilai dan sinonim berikut lalu pilih **Simpan**:

    | Daftar kunci | sinonim|
    |-------------------|---------|
    | `Sunday` | `Sun` |

    > **CATATAN** Untuk memasukkan bidang daftar baru, sisipkan nilai `Sunday` di bidang teks, lalu klik bidang di mana 'Ketik nilai dan tekan enter...' ditampilkan, masukkan sinonim, lalu tekan ENTER.

1. Ulangi langkah sebelumnya untuk menambahkan komponen daftar berikut:

    | Nilai | sinonim|
    |-------------------|---------|
    | `Monday` | `Mon` |
    | `Tuesday` | `Tue, Tues` |
    | `Wednesday` | `Wed, Weds` |
    | `Thursday` | `Thur, Thurs` |
    | `Friday` | `Fri` |
    | `Saturday` | `Sat` |

1. Setelah menambahkan dan menyimpan nilai daftar, kembali ke halaman **Pelabelan data**.
1. Pilih niat **GetDate** dan masukkan sampel ucapan baru berikut:

    `what date was it on Saturday?`

1. Saat ucapan telah ditambahkan, pilih kata ***Sabtu***, dan di daftar menu dropdown yang muncul, pilih **Hari Kerja**.

1. Tambahkan sampel ucapan lain untuk niat **GetDate**:

    `what date will it be on Friday?`

1. Ketika ucapan telah ditambahkan, petakan **Jumat** ke entitas **Weekday**.

1. Tambahkan sampel ucapan lain untuk niat **GetDate**:

    `what will the date be on Thurs?`

1. Ketika ucapan telah ditambahkan, petakan **Kamis** ke entitas **Weekday**.

1. pilih **Simpan perubahan** untuk menyimpan ucapan baru.

### Tambahkan entitas *yang dibuat sebelumnya*

Layanan Azure AI Bahasa menyediakan sekumpulan entitas *bawaan* yang biasanya digunakan dalam aplikasi percakapan.

1. Di Language Studio, kembali ke halaman **Definisi skema**, lalu pada tab **Entitas**, pilih **&#65291; Tambahkan** untuk menambahkan entitas baru.

1. Di kotak dialog **Tambahkan entitas**, masukkan nama entitas `Date` dan pilih tab entitas **Bawaan**. Lalu pilih **Tambahkan entitas**.

1. Pada halaman untuk entitas**Tanggal**, di bagian **Dipelajari**, pastikan **Tidak diperlukan** dipilih. Kemudian, di bagian **Bawaan**, pilih **&#65291; Tambahkan bawaan baru**.

1. Dalam daftar **Pilih bawaan**, pilih **DateTime** lalu pilih **Simpan**.
1. Setelah menambahkan entitas bawaan, kembali ke halaman **Pelabelan data**
1. Pilih niat **GetDay** dan masukkan sampel ucapan baru berikut:

    `what day was 01/01/1901?`

1. Saat ucapan telah ditambahkan, pilih ***01/01/1901***, dan di daftar menu dropdown yang muncul, pilih **Tanggal**.

1. Tambahkan sampel ucapan lain untuk niat **GetDay**:

    `what day will it be on Dec 31st 2099?`

1. Ketika ucapan telah ditambahkan, petakan **31 Desember 2099** ke entitas **Date**.

1. Pilih **Simpan perubahan** untuk menyimpan ucapan baru.

### Latih ulang model

Sekarang setelah Anda memodifikasi skema, Anda perlu melatih ulang dan menguji ulang model.

1. Pada halaman **Tugas pelatihan**, pilih **Mulai tugas pelatihan**.

1. Pada dialog **Mulai tugas pelatihan**,  pilih  **timpa model yang ada** dan tentukan model **Jam**. Pilih **Latih** untuk melatih model. Jika diminta, konfirmasikan bahwa Anda ingin menimpa model yang ada.

1. Saat pelatihan selesai, **Status** pekerjaan akan diperbarui menjadi **Pelatihan berhasil**.

1. Pilih halaman **Performa model** lalu pilih model **Jam**. Tinjau metrik evaluasi (*presisi*, *pengenalan*, dan *f-measure*) dan *matriks kebingungan* yang dihasilkan oleh evaluasi yang dilakukan saat pelatihan (perhatikan bahwa karena sejumlah kecil sampel ucapan, tidak semua niat dapat disertakan dalam hasil).

1. Pada halaman **Menyebarkan model**, pilih **Tambahkan penyebaran**.

1. Pada dialog **Tambahkan penyebaran**, pilih **Ganti nama penyebaran yang ada**, lalu pilih **produksi**.

1. Pilih model **Jam** di bidang **Model** lalu pilih **Sebarkan** untuk menyebarkannya. Ini mungkin memakan waktu.

1. Saat model disebarkan, pada halaman **Penyebaran pengujian**, pilih penyebaran **produksi** di bawah bidang **Nama penyebaran**, lalu uji dengan teks berikut:

    `what's the time in Edinburgh?`

1. Tinjau hasil yang dikembalikan, yang diharapkan dapat memprediksi niat **GetTime** dan entitas **Location** dengan nilai teks "Edinburgh".

1. Coba uji ucapan berikut:

    `what time is it in Tokyo?`

    `what date is it on Friday?`

    `what's the date on Weds?`

    `what day was 01/01/2020?`

    `what day will Mar 7th 2030 be?`

## Menggunakan model dari aplikasi klien

Dalam proyek nyata, Anda akan secara berulang memperbaiki niat dan entitas, melatih ulang, dan menguji ulang hingga Anda puas dengan performa prediktif. Kemudian, ketika Anda telah mengujinya dan puas dengan performa prediktifnya, Anda dapat menggunakannya di aplikasi klien dengan panggilan antarmuka REST atau SDK khusus runtime bahasa umum.

### Bersiap untuk mengembangkan aplikasi di Visual Studio Code

Anda akan mengembangkan aplikasi pemahaman bahasa menggunakan Visual Studio Code. File kode untuk aplikasi Anda telah disediakan dalam repositori GitHub.

> **Tips**: Jika Anda telah mengkloning repositori **mslearn-ai-language**, buka di Visual Studio Code. Jika belum melakukannya, ikuti langkah-langkah berikut untuk mengkloningnya ke lingkungan pengembangan Anda.

1. Memulai Visual Studio Code.
2. Buka palet (SHIFT+CTRL+P) dan jalankan **Git: Perintah klon** untuk mengkloning repositori `https://github.com/MicrosoftLearning/mslearn-ai-language` ke folder lokal (tidak masalah folder mana).
3. Setelah repositori dikloning, buka folder di Visual Studio Code.

    > **Catatan**: Jika Visual Studio Code menampilkan pesan pop-up yang meminta Anda memercayai kode yang Anda buka, klik opsi **Ya, saya memercayai pembuatnya** di pop-up.

4. Tunggu sementara file tambahan diinstal untuk mendukung proyek kode C# di repositori.

    > **Catatan**: Jika Anda diminta untuk menambahkan aset yang diperlukan guna membangun dan men-debug, pilih **Tidak Sekarang**.

### Mengonfigurasi aplikasi Anda

Aplikasi untuk C# dan Python telah disediakan, beserta contoh file teks yang akan Anda gunakan untuk menguji peringkasan. Kedua aplikasi memiliki fungsionalitas yang sama. Pertama, Anda akan menyelesaikan beberapa bagian kunci aplikasi untuk mengaktifkannya agar dapat menggunakan sumber daya Azure AI Bahasa Anda.

1. Di Visual Studio Code, di panel **Penjelajah**, telusuri ke folder **Labfiles/03-language** dan luaskan folder **CSharp** atau **Python** tergantung pada preferensi bahasa Anda dan folder **jam-klien** yang ada di dalamnya. Setiap folder berisi file khusus bahasa untuk aplikasi yang akan Anda integrasikan dengan fungsionalitas jawaban atas pertanyaan Azure AI Bahasa.
2. Klik kanan folder **jam-klien** yang berisi file kode Anda dan buka terminal terintegrasi. Kemudian instal paket SDK pemahaman bahasa percakapan Azure AI Bahasa dengan menjalankan perintah yang sesuai untuk preferensi bahasa Anda:

    **C#**:

    ```
    dotnet add package Azure.AI.Language.Conversations --version 1.1.0
    ```

    **Python**:

    ```
    pip install azure-ai-language-conversations
    ```

3. Di panel **Penjelajah**, di folder **jam-klien**, buka file konfigurasi untuk bahasa pilihan Anda

    - **C#**: appsettings.json
    - **Python**: .env
    
4. Perbarui nilai konfigurasi untuk menyertakan  **titik akhir** dan **kunci **dari sumber daya Bahasa Azure yang Anda buat (tersedia di halaman **Kunci dan Titik Akhir** untuk sumber daya Bahasa Azure AI Anda di portal Microsoft Azure).
5. Simpan file konfigurasi.

### Tambahkan kode ke aplikasi

Sekarang Anda siap untuk menambahkan kode yang diperlukan untuk mengimpor pustaka SDK yang diperlukan, membuat koneksi terautentikasi ke proyek yang disebarkan, dan mengirimkan pertanyaan.

1. Perhatikan bahwa folder **clock-client** berisi file kode untuk aplikasi klien:

    - **C#**: Program.cs
    - **Python**: clock-client.py

    Buka file kode dan di bagian atas, di bawah referensi namespace yang ada, temukan komentar **Impor namespace**. Kemudian, di bawah komentar ini, tambahkan kode khusus bahasa berikut untuk mengimpor ruang nama yang Anda perlukan untuk menggunakan SDK Analisis Teks:

    **C#**: Programs.cs

    ```c#
    // import namespaces
    using Azure;
    using Azure.AI.Language.Conversations;
    ```

    **Python**: clock-client.py

    ```python
    # Import namespaces
    from azure.core.credentials import AzureKeyCredential
    from azure.ai.language.conversations import ConversationAnalysisClient
    ```

1. Dalam fungsi **Utama**, perhatikan bahwa kode untuk memuat kunci dan wilayah layanan kognitif dari file konfigurasi telah disediakan. Kemudian temukan komentar **Buat klien untuk model layanan Bahasa** dan tambahkan kode berikut untuk membuat klien prediksi untuk aplikasi Layanan Bahasa Anda:

    **C#**: Programs.cs

    ```c#
    // Create a client for the Language service model
    Uri endpoint = new Uri(predictionEndpoint);
    AzureKeyCredential credential = new AzureKeyCredential(predictionKey);

    ConversationAnalysisClient client = new ConversationAnalysisClient(endpoint, credential);
    ```

    **Python**: clock-client.py

    ```python
    # Create a client for the Language service model
    client = ConversationAnalysisClient(
        ls_prediction_endpoint, AzureKeyCredential(ls_prediction_key))
    ```

1. Perhatikan bahwa kode dalam fungsi **Utama** meminta masukan pengguna hingga pengguna memasukkan "berhenti". Dalam perulangan ini, temukan komentar **Panggil model layanan Bahasa untuk mendapatkan niat dan entitas** dan tambahkan kode berikut:

    **C#**: Programs.cs

    ```c#
    // Call the Language service model to get intent and entities
    var projectName = "Clock";
    var deploymentName = "production";
    var data = new
    {
        analysisInput = new
        {
            conversationItem = new
            {
                text = userText,
                id = "1",
                participantId = "1",
            }
        },
        parameters = new
        {
            projectName,
            deploymentName,
            // Use Utf16CodeUnit for strings in .NET.
            stringIndexType = "Utf16CodeUnit",
        },
        kind = "Conversation",
    };
    // Send request
    Response response = await client.AnalyzeConversationAsync(RequestContent.Create(data));
    dynamic conversationalTaskResult = response.Content.ToDynamicFromJson(JsonPropertyNames.CamelCase);
    dynamic conversationPrediction = conversationalTaskResult.Result.Prediction;   
    var options = new JsonSerializerOptions { WriteIndented = true };
    Console.WriteLine(JsonSerializer.Serialize(conversationalTaskResult, options));
    Console.WriteLine("--------------------\n");
    Console.WriteLine(userText);
    var topIntent = "";
    if (conversationPrediction.Intents[0].ConfidenceScore > 0.5)
    {
        topIntent = conversationPrediction.TopIntent;
    }
    ```

    **Python**: clock-client.py

    ```python
    # Call the Language service model to get intent and entities
    cls_project = 'Clock'
    deployment_slot = 'production'

    with client:
        query = userText
        result = client.analyze_conversation(
            task={
                "kind": "Conversation",
                "analysisInput": {
                    "conversationItem": {
                        "participantId": "1",
                        "id": "1",
                        "modality": "text",
                        "language": "en",
                        "text": query
                    },
                    "isLoggingEnabled": False
                },
                "parameters": {
                    "projectName": cls_project,
                    "deploymentName": deployment_slot,
                    "verbose": True
                }
            }
        )

    top_intent = result["result"]["prediction"]["topIntent"]
    entities = result["result"]["prediction"]["entities"]

    print("view top intent:")
    print("\ttop intent: {}".format(result["result"]["prediction"]["topIntent"]))
    print("\tcategory: {}".format(result["result"]["prediction"]["intents"][0]["category"]))
    print("\tconfidence score: {}\n".format(result["result"]["prediction"]["intents"][0]["confidenceScore"]))

    print("view entities:")
    for entity in entities:
        print("\tcategory: {}".format(entity["category"]))
        print("\ttext: {}".format(entity["text"]))
        print("\tconfidence score: {}".format(entity["confidenceScore"]))

    print("query: {}".format(result["result"]["query"]))
    ```

    Panggilan ke model layanan Bahasa mengembalikan prediksi/hasil, yang mencakup niat teratas (kemungkinan besar) serta entitas apa pun yang terdeteksi dalam ucapan input. Aplikasi klien Anda sekarang harus menggunakan prediksi tersebut untuk menentukan dan melakukan tindakan yang sesuai.

1. Temukan komentar **Terapkan tindakan yang sesuai**, dan tambahkan kode berikut, yang memeriksa maksud yang didukung oleh aplikasi (**GetTime**, **GetDate**, dan **GetDay**) dan menentukan apakah entitas relevan telah terdeteksi, sebelum memanggil fungsi yang ada untuk menghasilkan respons yang sesuai.

    **C#**: Programs.cs

    ```c#
    // Apply the appropriate action
    switch (topIntent)
    {
        case "GetTime":
            var location = "local";           
            // Check for a location entity
            foreach (dynamic entity in conversationPrediction.Entities)
            {
                if (entity.Category == "Location")
                {
                    //Console.WriteLine($"Location Confidence: {entity.ConfidenceScore}");
                    location = entity.Text;
                }
            }
            // Get the time for the specified location
            string timeResponse = GetTime(location);
            Console.WriteLine(timeResponse);
            break;
        case "GetDay":
            var date = DateTime.Today.ToShortDateString();            
            // Check for a Date entity
            foreach (dynamic entity in conversationPrediction.Entities)
            {
                if (entity.Category == "Date")
                {
                    //Console.WriteLine($"Location Confidence: {entity.ConfidenceScore}");
                    date = entity.Text;
                }
            }            
            // Get the day for the specified date
            string dayResponse = GetDay(date);
            Console.WriteLine(dayResponse);
            break;
        case "GetDate":
            var day = DateTime.Today.DayOfWeek.ToString();
            // Check for entities            
            // Check for a Weekday entity
            foreach (dynamic entity in conversationPrediction.Entities)
            {
                if (entity.Category == "Weekday")
                {
                    //Console.WriteLine($"Location Confidence: {entity.ConfidenceScore}");
                    day = entity.Text;
                }
            }          
            // Get the date for the specified day
            string dateResponse = GetDate(day);
            Console.WriteLine(dateResponse);
            break;
        default:
            // Some other intent (for example, "None") was predicted
            Console.WriteLine("Try asking me for the time, the day, or the date.");
            break;
    }
    ```

    **Python**: clock-client.py

    ```python
    # Apply the appropriate action
    if top_intent == 'GetTime':
        location = 'local'
        # Check for entities
        if len(entities) > 0:
            # Check for a location entity
            for entity in entities:
                if 'Location' == entity["category"]:
                    # ML entities are strings, get the first one
                    location = entity["text"]
        # Get the time for the specified location
        print(GetTime(location))

    elif top_intent == 'GetDay':
        date_string = date.today().strftime("%m/%d/%Y")
        # Check for entities
        if len(entities) > 0:
            # Check for a Date entity
            for entity in entities:
                if 'Date' == entity["category"]:
                    # Regex entities are strings, get the first one
                    date_string = entity["text"]
        # Get the day for the specified date
        print(GetDay(date_string))

    elif top_intent == 'GetDate':
        day = 'today'
        # Check for entities
        if len(entities) > 0:
            # Check for a Weekday entity
            for entity in entities:
                if 'Weekday' == entity["category"]:
                # List entities are lists
                    day = entity["text"]
        # Get the date for the specified day
        print(GetDate(day))

    else:
        # Some other intent (for example, "None") was predicted
        print('Try asking me for the time, the day, or the date.')
    ```

1. Simpan perubahan Anda dan kembali ke terminal terintegrasi untuk folder **clock-client**, dan masukkan perintah berikut untuk menjalankan program:

    - **C#:** `dotnet run`
    - **Python**: `python clock-client.py`

    > **Tips**: Anda dapat menggunakan ikon **Maksimalkan ukuran panel** (**^**) di bar alat terminal untuk melihat teks konsol lainnya.

1. Saat diminta, masukkan ucapan untuk menguji aplikasi. Misalnya, coba:

    *Halo*

    *Jam berapa sekarang?*

    *Jam berapa di London?*

    *Tanggal berapa?*

    *Tanggal berapa hari Minggu?*

    *Sekarang hari apa?*

    *Hari apa 01/01/2025?*

    > **Catatan**: Logika dalam aplikasi ini sengaja dibuat sederhana, dan memiliki sejumlah keterbatasan. Misalnya, saat mendapatkan waktu, hanya kumpulan kota terbatas yang didukung dan waktu musim panas diabaikan. Tujuannya adalah untuk melihat contoh pola tipikal untuk menggunakan Layanan Bahasa di mana aplikasi Anda harus:
    >   1. Menyambungkan ke titik akhir prediksi.
    >   2. Kirim ucapan untuk mendapatkan prediksi.
    >   3. Terapkan logika untuk merespons dengan tepat maksud dan entitas yang diprediksi.

1. Setelah Anda selesai menguji, masukkan *berhenti*.

## Membersihkan sumber daya

Jika sudah selesai menjelajahi layanan Azure AI Bahasa, Anda dapat menghapus sumber daya yang Anda buat dalam latihan ini. Berikut caranya:

1. Buka portal Microsoft Azure di `https://portal.azure.com`, dan masuk menggunakan akun Microsoft yang terkait dengan langganan Azure Anda.
2. Telusuri ke sumber daya Azure AI Bahasa yang Anda buat di lab ini.
3. Pada halaman sumber daya, pilih **Hapus** dan ikuti instruksi untuk menghapus sumber daya.

## Informasi selengkapnya

Untuk mempelajari selengkapnya tentang pemahaman bahasa percakapan dalam Azure AI Bahasa, lihat [dokumentasi Azure AI Bahasa](https://learn.microsoft.com/azure/ai-services/language-service/conversational-language-understanding/overview).
