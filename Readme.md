# Profiling & JMeter Benchmark Results

**Nama:** Abigail Namaratonggi P.

**NPM:** 2406495773

**Kelas:** Pemrograman Lanjut - B

---

<details>
<summary><b>Result (JMeter & Profiler)</b></summary>

## JMeter Results

### 1. `all_student`

| Before Optimization | After Optimization                                       |
|-----------------|----------------------------------------------------------|
| ![img.png](Screenshots/Screenshot%202026-04-28%20100737.png) | ![img](Screenshots/Screenshot%202026-04-28%20200956.png) |
| ![img_3.png](Screenshots/Screenshot%202026-04-28%20102213.png) | ![img](Screenshots/Screenshot%202026-04-28%20201609.png)    |

---

### 2. `all_student_name`

| Before Optimization | After Optimization                                       |
|----------------|----------------------------------------------------------|
| ![img_1.png](Screenshots/Screenshot%202026-04-28%20100944.png) | ![img](Screenshots/Screenshot%202026-04-28%20201725.png) |
| ![img_4.png](Screenshots/Screenshot%202026-04-28%20103037.png) | ![img](Screenshots/Screenshot%202026-04-28%20201851.png) |

---

### 3. `highest_gpa`

| Before Optimization | After Optimization                                       |
|----------------|----------------------------------------------------------|
| ![img_2.png](Screenshots/Screenshot%202026-04-28%20101219.png) | ![img](Screenshots/Screenshot%202026-04-28%20202000.png) |
| ![img_5.png](Screenshots/Screenshot%202026-04-28%20061405.png) | ![img](Screenshots/Screenshot%202026-04-28%20202058.png)  |

---

## Profiling Results

| Endpoint | Before Optimization             | After Optimization                                       |
|----------|---------------------------------|----------------------------------------------------------|
| `all_student` | ![img.png](Screenshots/img.png) | ![img](Screenshots/Screenshot%202026-04-28%20202506.png) |
| `all_student_name` | ![img.png](Screenshots/img2.png)| ![img](Screenshots/Screenshot%202026-04-28%20202804.png) |
| `highest_gpa` | ![img.png](Screenshots/img3.png) | ![img](Screenshots/Screenshot%202026-04-28%20203009.png) |

---

## Kesimpulan

### Perbandingan Hasil JMeter Sebelum dan Sesudah Optimasi

Berdasarkan pengujian dengan Apache JMeter (10 sampel per endpoint), terjadi peningkatan performa. Ringkasan perbandingannya adalah sebagai berikut:

| Endpoint | Rata-rata Sebelum (ms) | Rata-rata Sesudah (ms) | Peningkatan             |
|----------|------------------------|------------------------|-------------------------|
| `all_student` | ~459.029               | ~5.455                 | **~98.81% lebih cepat** |
| `all_student_name` | ~16.992                | ~2.120                 | **~87.52% lebih cepat** |
| `highest_gpa` | ~1.639                 | ~16                    | **~99.02% lebih cepat** |

---

### Analisis per Endpoint
#### `/all-student`
Pada pengujian awal, *endpoint* ini mencatat waktu respons **459.029 ms** (hampir 8 menit). Penyebabnya terletak pada eksekusi `StudentService.getAllStudentsWithCourses()` yang terjebak dalam masalah *N+1 query*. Sistem memanggil *database* berulang kali secara tidak efisien. Saat querynya diubah menggunakan `JOIN FETCH`, waktu prosesnya menurun ke angka **5.455 ms** saja.

#### `/all-student-name`
Inefisiensi pada *endpoint* ini (dengan waktu awal **16.992 ms**) disebabkan oleh pemborosan alokasi memori. Aplikasi memuat seluruh entitas tabel mahasiswa ke dalam RAM hanya untuk mengekstrak namanya lewat `joinStudentNames()`. Masalah ini diselesaikan dengan menerapkan fitur *Projection* pada *repository*, sehingga *database* murni hanya mengirimkan string nama dan waktu respons berhasil berkurang menjadi **2.120 ms**.

#### `/highest-gpa`
Kasus pencarian IPK tertinggi ini memakan waktu **1.639 ms** karena aplikasi memaksa memori Java untuk melakukan iterasi (*looping*) pada puluhan ribu data. Logika ini kurang tepat sasaran. Solusinya adalah memindahkan beban kerja (*sorting*) tersebut langsung ke *database engine*. Perubahan ini membuat waktu eksekusi menjadi **16 ms** (meningkat lebih dari 99%).

---

### Kesimpulan Evaluasi Performa

Ya, terdapat peningkatan performa yang sangat signifikan dari hasil pengukuran JMeter. Sebelum optimasi, aplikasi berjalan lambat karena adanya *N+1 query problem* dan inefisiensi pencarian di level Java yang membebani memori dan CPU.

Setelah melakukan *refactoring*, seperti menerapkan `JOIN FETCH`, menggunakan *projection*, dan mendelegasikan proses *sorting* ke level *database*, *Sample Time* (waktu respons) pada JMeter menurun drastis. Aplikasi kini mampu menangani *request* dengan jauh lebih cepat, efisien, dan stabil.

---
</details>

<details>
<summary><b>Reflection</b></summary>

#### 1. What is the difference between the approach of performance testing with JMeter and profiling with IntelliJ Profiler in the context of optimizing application performance?
JMeter menguji dari **luar** (sebagai *user* yang memberi beban/trafik dan mengukur waktu respons server). IntelliJ Profiler membedah dari **dalam** (melihat secara detail baris kode/metode mana di dalam Java yang paling memakan CPU dan memori).

####  2. How does the profiling process help you in identifying and understanding the weak points in your application?
Profiler memetakan alur eksekusi kode (melalui *Call Tree* atau *Flame Graph*) dan menunjukkan persentase waktu CPU yang dihabiskan. Jadi, kita bisa langsung melihat bagian kode mana yang menjadi *bottleneck* (titik lemah) tanpa perlu menebak-nebak.

#### 3. Do you think IntelliJ Profiler is effective in assisting you to analyze and identify bottlenecks in your application code?
Sangat efektif. Karena terintegrasi langsung dengan IDE, kita bisa langsung melompat dari grafik visual (yang menunjukkan masalah seperti N+1 query) ke baris kode yang bermasalah untuk segera diperbaiki.

#### 4. What are the main challenges you face when conducting performance testing and profiling, and how do you overcome these challenges?
**Tantangan:** Membaca tumpukan *Call Tree* yang sangat dalam dan membedakan mana proses internal Spring/Tomcat dan mana kode buatan sendiri.
**Solusi:** Fokus menelusuri metode yang memakan waktu eksekusi terbesar (*Total Time* terbesar) dan mencari nama *package* aplikasi kita sendiri.

#### 5. What are the main benefits you gain from using IntelliJ Profiler for profiling your application code?
Sangat menghemat waktu *debugging* performa, memberikan bukti nyata letak inefisiensi, dan memiliki fitur *Comparison* untuk memvalidasi apakah *refactoring* yang kita lakukan benar-benar berdampak positif atau tidak.

#### 6. How do you handle situations where the results from profiling with IntelliJ Profiler are not entirely consistent with findings from performance testing using JMeter?
Jika Profiler menunjukkan kode sudah cepat tapi JMeter masih lambat, masalahnya biasanya ada di luar kode Java. Saya akan mengecek faktor eksternal seperti latensi jaringan, koneksi *database*, atau memastikan JVM sudah melakukan *warm-up* sebelum diukur oleh JMeter.

#### 7. What strategies do you implement in optimizing application code after analyzing results from performance testing and profiling? How do you ensure the changes you make do not affect the application's functionality?
**Strategi:** Memindahkan beban pemrosesan data (seperti *looping*, *sorting*, dan filter kolom) dari level Java ke level *Database* (menggunakan *Query*, *JOIN FETCH*, atau *Projection*).
**Menjaga Fungsi:** Selalu menjalankan **Unit Test** yang sudah dibuat sebelumnya setiap kali selesai melakukan *refactoring* untuk memastikan *output* aplikasi tidak berubah.

**Kesimpulan Evaluasi Performa:**

Ya, terdapat peningkatan performa yang sangat signifikan dari hasil pengukuran JMeter. Sebelum optimasi, aplikasi berjalan lambat karena adanya *N+1 query problem* dan inefisiensi pencarian di level Java yang membebani memori dan CPU.

Setelah melakukan *refactoring*, seperti menerapkan `JOIN FETCH`, menggunakan *projection*, dan mendelegasikan proses *sorting* ke level *database*, *Sample Time* (waktu respons) pada JMeter menurun drastis. Metode `getAllStudentsWithCourses` berhasil berkurang dari 15.935 ms menjadi hanya 3.748 ms (terjadi peningkatan performa lebih dari 75%). Begitu juga dengan metode `findStudentWithHighestGpa` dan `joinStudentNames`. Aplikasi jadi mampu menangani *request* dengan jauh lebih cepat, efisien, dan stabil.

**Sebelum refactoring**
![sebelum](Screenshots/Screenshot%202026-04-28%20081153.png)
**Sesudah refactoring**
![sesudah](Screenshots/Screenshot%202026-04-28%20083157.png)
</details>