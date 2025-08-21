# Continuous Delivery with Jenkins in Kubernetes Engine

Dokumen ini merangkum proses otomatisasi build dan deployment menggunakan **Jenkins** di atas **Google Kubernetes Engine (GKE)**, dengan pemicu sederhana berupa perintah, Ini hanya rangkuman singkat dan repository ini kodenya dari event GCP (Ini saya gunakan untuk pembelajaran saya dan mengasah skill saya di bidang DevOps) :

Untuk Penjelasan lebih lengkap cek di link berikut 
[![Logo](https://drive.google.com/file/d/1GpXPpqa5Qm01M4xsushwGLBcaB9sd6EZ/view?usp=sharing)](https://example.com)


```bash
git push origin <branch>
```

Integrasi antara Jenkins dan GitHub memungkinkan pipeline berjalan otomatis mulai dari build, push image Docker, hingga deployment ke Kubernetes.

---

## 1. Penyiapan Awal: Kubernetes Cluster & Jenkins

Sebelum Jenkins dapat melakukan build otomatis, siapkan lingkungan dasar terlebih dahulu:

* **Membuat Kubernetes Cluster**
  Jalankan:

  ```bash
  gcloud container clusters create jenkins-cd
  ```

  Perintah ini memberikan akses agar Jenkins dapat berinteraksi dengan **Google Container Registry (GCR)** dan repositori kode sumber.

* **Instalasi Jenkins menggunakan Helm**
  Jenkins diinstal ke cluster dengan **Helm**, menggunakan *chart* yang sudah disiapkan.
  Konfigurasi tambahan, seperti plugin Jenkins, dilakukan melalui file `values.yaml`.

---

## 2. Konfigurasi GitHub Repository & Jenkins

Setelah Jenkins berjalan, hubungkan dengan repositori aplikasi Anda:

* **Inisialisasi Repositori Git**
  Buat repositori privat di GitHub, lalu push kode aplikasi ke repositori tersebut.

* **Otentikasi Jenkins ke GitHub**

  * Buat kunci SSH baru (`id_github` & `id_github.pub`).
  * Tambahkan kunci publik ke **GitHub**.
  * Simpan kunci privat sebagai **Credentials** di Jenkins.
    → Jenkins kini dapat menarik kode dari repositori privat Anda.

---

## 3. Pembuatan & Konfigurasi Pipeline Jenkins

Pipeline dibuat untuk mengotomatisasi proses CI/CD:

* **Job Multibranch Pipeline**
  Buat job baru dengan tipe *Multibranch Pipeline*. Jenkins otomatis mendeteksi semua branch pada repositori.

* **Hubungkan ke Repositori Git**
  Masukkan URL GitHub (`git@github.com:<username>/<repo>.git`) dan pilih credentials SSH yang telah ditambahkan.

* **Pengaturan Trigger**
  Aktifkan opsi **Scan Multibranch Pipeline Triggers**, misalnya setiap 1 menit.
  Jenkins akan mendeteksi setiap perubahan baru (`git push`) dan menjalankan pipeline terkait.

---

## 4. Jenkinsfile: Pipeline as Code

Pipeline didefinisikan dalam file `Jenkinsfile` di dalam repositori:

### Contoh Alur Stages

1. **Build**
   Kompilasi kode & bangun image Docker aplikasi.
2. **Push**
   Upload image Docker ke **Google Container Registry (GCR)**.
3. **Deploy**
   Terapkan manifest Kubernetes (`.yaml`) untuk deployment aplikasi.
   Branch berbeda bisa diarahkan ke environment berbeda (dev, canary, production).

> 🔧 Sebelum menjalankan pipeline, sesuaikan variabel di `Jenkinsfile` seperti `PROJECT_ID` dan `CLUSTER_ZONE`.

---

## 5. Alur Kerja Otomatis

Dengan konfigurasi di atas, alur otomatis berjalan seperti ini:

1. **Developer melakukan `git push`**
   Commit dan push ke repositori GitHub.
2. **Jenkins mendeteksi perubahan**
   Jenkins memindai repositori dan menemukan commit baru.
3. **Pipeline terpicu**
   Jenkins menjalankan `Jenkinsfile` untuk branch tersebut.
4. **Build → Push → Deploy**

   * Bangun image Docker
   * Push ke GCR
   * Deploy ke cluster Kubernetes

---

## Apa itu GCR?

**Google Container Registry (GCR)** adalah layanan penyimpanan image Docker dari Google Cloud.
Dengan GCR, Anda dapat:

* Menyimpan dan mengelola image container.
* Melakukan push image hasil build Jenkins.
* Menarik (pull) image tersebut saat deployment di Kubernetes.

Format penamaan image di GCR:

```bash
gcr.io/<PROJECT_ID>/<IMAGE_NAME>:<TAG>
```

---

## Kesimpulan

Dengan setup ini, setiap perubahan kode yang dipush ke GitHub akan secara otomatis:

* Dibangun menjadi image Docker
* Dikirim ke GCR
* Dideploy ke Kubernetes

Sehingga mewujudkan praktik **Continuous Integration/Continuous Delivery (CI/CD)** hanya dengan satu perintah sederhana:

```bash
git push
```
