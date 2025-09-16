---
lab:
  title: Menjelajahi Voice Live API
  description: Pelajari cara menggunakan dan menyesuaikan Voice Live API yang tersedia di Playground Azure AI Foundry.
---

# Menjelajahi Voice Live API

Dalam latihan ini, Anda membuat agen di Azure AI Foundry dan menjelajahi Voice Live API di Speech Playground. 

Latihan ini memakan waktu sekitar **30** menit.

> <span style="color:red">**Catatan**:</span> Sebagian teknologi yang digunakan dalam latihan ini sedang dalam pratinjau atau dalam pengembangan aktif. Anda mungkin mengalami beberapa perilaku, peringatan, atau kesalahan tak terduga.

> <span style="color:red">**Catatan**:</span> Latihan ini dirancang untuk diselesaikan di lingkungan browser dengan akses langsung ke mikrofon komputer Anda. Meskipun konsep dapat dieksplorasi di Azure Cloud Shell, fitur suara interaktif memerlukan akses perangkat keras audio lokal.

## Membuat proyek Azure OpenAI

Mari kita mulai dengan membuat proyek Azure AI Foundry.

1. Di browser web, buka [portal Azure AI Foundry](https://ai.azure.com) di `https://ai.azure.com` dan masuk menggunakan kredensial Azure Anda. Tutup semua tips atau panel mulai cepat yang terbuka saat pertama kali Anda masuk, dan jika perlu, gunakan logo **Azure AI Foundry** di kiri atas untuk menavigasi ke beranda, yang tampilannya mirip dengan gambar berikut (tutup panel **Bantuan** jika terbuka):

    ![Tangkapan layar beranda Azure AI Foundry dengan agen terpilih.](../media/ai-foundry-new-home-page.png)

1. Di beranda, pilih **Buat agen**.

1. Di wizard **Buat agen**, masukkan nama yang valid untuk proyek Anda. 

1. Klik **Opsi Tingkat Lanjut** dan tentukan pengaturan berikut:
    - **Sumber daya Azure AI Foundry**: *Pertahankan nama default*
    - **Langganan**: *Langganan Azure Anda*
    - **Grup sumber daya**: *Buat atau pilih grup sumber daya*
    - **Wilayah**: Pilih wilayah secara acak dari opsi berikut:\*
        - AS Timur 2
        - Swedia Tengah

    > \* Pada saat penulisan, Voice Live API hanya didukung di wilayah yang tercantum sebelumnya. Memilih lokasi secara acak membantu memastikan satu wilayah tidak kewalahan dengan lalu lintas, dan membantu agar pengalaman Anda lebih lancar. Jika batas layanan tercapai, Anda mungkin perlu membuat proyek lain di wilayah yang berbeda.

1. Pilih **Buat** dan tinjau konfigurasi Anda. Tunggu hingga proses penyiapan selesai.

    >**Catatan**: Jika Anda mendapat kesalahan izin, pilih tombol **Perbaiki** untuk menambahkan izin yang sesuai untuk melanjutkan.

1. Saat proyek Anda dibuat, Anda akan diarahkan secara default ke playground Agen di portal Azure AI Foundry, yang akan terlihat mirip dengan gambar berikut:

    ![Tangkapan layar detail proyek Azure AI di portal Azure AI Foundry.](../media/ai-foundry-project-2.png)

## Memulai sampel Voice Live

 Di bagian latihan ini, Anda berinteraksi dengan salah satu agen. 

1. Pilih **Playground** di panel navigasi.

1. Temukan grup **Speech playground**, dan pilih tombol **Coba Speech playground**.

1. Speech Playground menawarkan banyak opsi bawaan. Gunakan bilah gulir horizontal untuk menavigasi ke akhir daftar dan pilih petak **Voice Live**. 

    ![Cuplikan layar petak Voice Live.](../media/voice-live-tile.png)

1. Pilih sampel agen **Obrolan santai** di panel **Coba dengan sampel**.

1. Pastikan mikrofon dan speaker Anda berfungsi dan pilih tombol **Mulai** di bagian bawah halaman. 

    Saat Anda berinteraksi dengan agen, perhatikan bahwa Anda dapat menyela agen dan agen akan berhenti sejenak untuk mendengarkan. Cobalah berbicara dengan panjang jeda yang berbeda antara kata dan kalimat. Perhatikan seberapa cepat agen mengenali jeda dan mengisi percakapan. Setelah selesai, pilih tombol **Akhiri**.

1. Mulai agen sampel lainnya untuk mempelajari perilaku mereka.

    Saat Anda mempelajari agen yang berbeda, perhatikan perubahan di bagian **Instruksi respons** di panel **Konfigurasi**.

## Mengonfigurasi agen 

Di bagian ini, Anda mengubah suara agen dan menambahkan avatar ke agen **Obrolan santai**. Panel **Konfigurasi** dibagi menjadi tiga bagian: **GenAI**, **Ucapan**, dan **Avatar**.

>**Catatan:** Jika Anda mengubah, atau berinteraksi dengan, salah satu opsi konfigurasi, Anda perlu memilih tombol **Terapkan** di bagian bawah panel **Konfigurasi** untuk mengaktifkan agen.

Pilih agen **Obrolan santai**. Berikutnya, ubah suara agen dan tambahkan avatar, dengan instruksi berikut:

1. Pilih **> Ucapan** untuk memperluas bagian dan mengakses opsi.

1. Pilih menu drop-down di opsi **Suara** dan pilih suara yang berbeda.

1. Pilih **Terapkan** untuk menyimpan perubahan Anda, lalu **Mulai** luncurkan agen dan dengar perubahan Anda.

    Ulangi langkah-langkah sebelumnya untuk mencoba beberapa suara yang berbeda. Lanjutkan ke langkah berikutnya saat Anda selesai memilih suara.

1. Pilih **> Avatar** untuk memperluas bagian dan mengakses opsi.

1. Pilih tombol dwiarah untuk mengaktifkan fitur dan memilih salah satu avatar. 

1. Pilih **Terapkan** untuk menyimpan perubahan Anda, lalu **Mulai** luncurkan agen. 

    Perhatikan animasi dan sinkronisasi avatar ke audio.

1. Perluas bagian **> GenAI** dan atur tombol **Keterlibatan proaktif** ke posisi nonaktif. Berikutnya, pilih **Terapkan** untuk menyimpan perubahan Anda, lalu **Mulai** luncurkan agen.

    Jika **Keterlibatan proaktif** dinonaktifkan, agen tidak akan memulai percakapan. Tanyakan ke agen "Bisakah Anda memberi tahu saya apa yang Anda lakukan?" untuk memulai percakapan.

>**Tips:** Anda dapat memilih **Reset ke default** lalu **Terapkan** untuk mengembalikan agen ke perilaku defaultnya.

Setelah selesai, lanjutkan ke bagian berikutnya.

## Membuat agen suara

Di bagian ini Anda membuat agen suara sendiri dari awal.

1. Pilih **Mulai dari awal** di bagian **Coba sendiri** pada panel. 

1. Perluas bagian **> GenAI** dari panel **Konfigurasi**.

1. Pilih menu drop-down **Model AI Generatif** dan pilih model **GPT-4o Mini Realtime**.

1. Tambahkan teks berikut di bagian **Instruksi respons**.

    ```
    You are a voice agent named "Ava" who acts as a friendly car rental agent. 
    ```

1. Atur penggeser **Suhu respons** ke nilai **0,8**. 

1. Atur tombol **Keterlibatan proaktif** ke posisi aktif.

1. Pilih **Terapkan** untuk menyimpan perubahan Anda, lalu **Mulai** luncurkan agen.

    Agen akan memperkenalkan dirinya sendiri dan bertanya apa yang dapat dibantu hari ini. Tanyakan kepada agen "Apakah ada sedan yang tersedia untuk disewa pada hari Kamis?" Perhatikan berapa lama waktu yang dibutuhkan agen untuk merespons. Ajukan pertanyaan lain kepada agen untuk melihat responsnya. Setelah selesai, lanjutkan ke langkah berikutnya.

1. Perluas bagian **> Ucapan** dari panel **Konfigurasi**.

1. Atur tombol **Akhir ucapan (EOU)** ke posisi **aktif**.

1. Atur tombol **Penyempurnaan audio** ke posisi **aktif**.

1. Pilih **Terapkan** untuk menyimpan perubahan Anda, lalu **Mulai** luncurkan agen.

    Setelah agen memperkenalkan dirinya, tanyakan "Apakah ada pesawat untuk disewa?" Perhatikan bahwa agen merespons lebih cepat daripada sebelumnya setelah menyelesaikan pertanyaan Anda. Pengaturan **Akhir ucapan (EOU)** mengonfigurasi agen untuk mendeteksi jeda dan akhir ucapan Anda berdasarkan konteks dan semantik. Hal ini memungkinkannya untuk percakapan yang lebih alami.

Setelah selesai, lanjutkan ke bagian berikutnya.

## Membersihkan sumber daya

Setelah menyelesaikan latihan, hapus proyek yang telah Anda buat untuk menghindari penggunaan sumber daya yang tidak perlu.

1. Pilih **Pusat manajemen** di menu navigasi AI Foundry.
1. Pilih **Hapus proyek** di panel informasi sebelah kanan, lalu konfirmasi penghapusan.

