# Mini Project — Pengambilan dan Pembersihan Data Melalui API

## Pengambilan Data Lokasi Menggunakan Geoapify Places API

Mini project ini merupakan implementasi proses **pengambilan data melalui API**, **penerapan konsep Object-Oriented Programming (OOP)**, **data cleaning**, dan **penyimpanan hasil** ke dalam file CSV.

Data diperoleh menggunakan **Geoapify Places API** melalui endpoint `Places`. Data yang dicari berfokus pada lokasi **kafe, restoran, dan supermarket** di area sekitar Monas, Jakarta.

---

## 📌 Informasi Project

| Informasi | Detail |
|---|---|
| Nama | Fajrin Efantri |
| Pelatihan | AI Automation Engineer |
| Project | Pengambilan dan Pembersihan Data Melalui API |
| Sumber Data | Geoapify Places API |
| Endpoint | `https://api.geoapify.com/v2/places` |
| Area Pencarian | Radius 3.000 meter dari titik Monas |
| Jumlah Data Awal | 114 baris |
| Jumlah Data Akhir | 114 baris |
| Jumlah Kolom | 5 kolom |

---

## 🎯 Tujuan

Project ini dibuat untuk menerapkan beberapa tahapan pengolahan data dari sumber API, yaitu:

1. Mengambil data lokasi menggunakan API.
2. Mengamankan API Key menggunakan file `.env`.
3. Membungkus proses pengambilan data ke dalam sebuah class.
4. Melakukan pemeriksaan dan pembersihan data.
5. Menyimpan dataset yang telah dibersihkan ke dalam file CSV.

---

## 🔌 API yang Digunakan

API yang digunakan adalah **Geoapify Places API**.

### Endpoint

```text
https://api.geoapify.com/v2/places
```

### Parameter yang digunakan

| Parameter | Fungsi |
|---|---|
| `categories` | Menentukan jenis lokasi yang ingin dicari |
| `filter` | Menentukan area pencarian berdasarkan koordinat |
| `limit` | Membatasi jumlah data yang diminta |
| `apiKey` | Kunci autentikasi untuk mengakses API |

### Kategori lokasi

Project ini menggunakan tiga kategori:

```text
catering.cafe
catering.restaurant
commercial.supermarket
```

### Area pencarian

Pencarian dilakukan pada radius **3.000 meter dari koordinat Monas**.

```text
circle:106.827153,-6.175392,3000
```

---

## 🔐 Pengelolaan API Key

API Key tidak dituliskan langsung di dalam kode program. Key disimpan dalam file `.env` agar kredensial tidak menjadi bagian dari kode utama.

Contoh isi file `.env`:

```env
GEOAPIFY_API_KEY=isi_api_key_di_sini
```

API Key kemudian dibaca menggunakan `python-dotenv`.

```python
from dotenv import load_dotenv
import os

load_dotenv()

API_KEY = os.getenv("GEOAPIFY_API_KEY")
```

> **Catatan:** file `.env` sebaiknya tidak diunggah ke repository publik.

---

## 🧩 Alur Pengambilan Data

Proses pengambilan data dilakukan melalui beberapa tahap:

```text
API Key
   ↓
Geoapify Places API
   ↓
Pencarian berdasarkan kategori
   ↓
Pencarian dalam radius Monas
   ↓
Ekstraksi data dari response GeoJSON
   ↓
DataFrame
   ↓
Data Cleaning
   ↓
Dataset CSV
```

Geoapify mengembalikan hasil dalam bentuk **GeoJSON**. Data lokasi berada pada bagian `features`, kemudian informasi penting dari setiap lokasi diambil dari `properties`.

---

## 🏗️ Class yang Dibangun

Untuk membuat proses request lebih terstruktur, project ini menggunakan class:

```python
KlienTempat
```

Class tersebut bertugas menyimpan konfigurasi API dan menjalankan proses pengambilan data.

### Atribut

| Atribut | Fungsi |
|---|---|
| `self.api_key` | Menyimpan API Key Geoapify |
| `self.alamat_api` | Menyimpan endpoint Places API |

### Method

| Method | Fungsi |
|---|---|
| `__init__()` | Menginisialisasi API Key dan endpoint |
| `ambil_data()` | Mengambil data berdasarkan kategori, area, dan jumlah data |

Method `ambil_data()` menerima parameter:

```python
ambil_data(kategori, area_filter, jumlah=50)
```

Dengan demikian, proses pencarian dapat digunakan kembali tanpa harus menuliskan request API dari awal.

---

## 📦 Data yang Diambil

Dari response Geoapify, data kemudian disederhanakan menjadi beberapa kolom utama:

| Kolom | Keterangan |
|---|---|
| `Nama` | Nama tempat |
| `Alamat Lengkap` | Alamat lengkap lokasi |
| `Kategori Detail` | Kategori tempat |
| `Latitude` | Koordinat lintang |
| `Longitude` | Koordinat bujur |

Kategori yang berasal dari API juga digabung menjadi satu nilai teks pada kolom `Kategori Detail`.

---

## 🧹 Data Cleaning

Sebelum dataset digunakan lebih lanjut, dilakukan pemeriksaan terhadap:

- nilai kosong,
- data kembar,
- tipe data.

### 1. Nilai kosong

Hasil pemeriksaan menunjukkan terdapat:

```text
Nama               : 1 nilai kosong
Alamat Lengkap     : 0
Kategori Detail    : 0
Latitude           : 0
Longitude          : 0
```

Nilai kosong pada kolom `Nama` tidak menyebabkan baris data dihapus. Nilai tersebut diisi dengan:

```text
Nama Tidak Diketahui
```

Function yang digunakan:

```python
def bersihkan_nama(teks):
    if pd.isna(teks) or teks == "":
        return "Nama Tidak Diketahui"
    return teks
```

### 2. Baris kembar

Pemeriksaan duplikat menggunakan kolom `Alamat Lengkap`.

Hasil pemeriksaan:

```text
Data kembar ditemukan : 0
```

Karena tidak ditemukan alamat lengkap yang sama, tidak ada baris yang dihapus.

```python
df_bersih = df_tempat.drop_duplicates(
    subset="Alamat Lengkap"
).copy()
```

### 3. Tipe data

Pemeriksaan tipe data menunjukkan:

```text
Nama               → str
Alamat Lengkap     → str
Kategori Detail    → str
Latitude           → float64
Longitude          → float64
```

Kolom `Latitude` dan `Longitude` dipastikan tetap bertipe numerik menggunakan function:

```python
def pastikan_tipe_angka(nilai):
    return pd.to_numeric(nilai, errors="coerce")
```

Karena tipe data koordinat sudah sesuai, proses ini berfungsi untuk memastikan konsistensi tipe data.

---

## 📊 Hasil Akhir

Setelah seluruh proses pemeriksaan dan cleaning selesai:

```text
Baris sebelum dibersihkan : 114
Baris dataset akhir       : 114
Baris kembar dibuang      : 0
Nama kosong ditangani     : 1
Jumlah kolom              : 5
```

Dataset akhir memiliki struktur:

```text
Nama
Alamat Lengkap
Kategori Detail
Latitude
Longitude
```

---

## 💾 Penyimpanan Dataset

Dataset yang telah dibersihkan disimpan dalam file:

```text
dataset_tempat_geoapify.csv
```

Proses penyimpanan dilakukan menggunakan:

```python
df_bersih.to_csv(
    "dataset_tempat_geoapify.csv",
    index=False
)
```

Parameter `index=False` digunakan agar nomor index DataFrame tidak ikut disimpan sebagai kolom tambahan.

---

## 📁 Struktur File Repository

Struktur project yang digunakan dapat dibuat seperti berikut:

```text
mp1-aiautomation-2026/
│
├── mini_project1.ipynb
├── dataset_tempat_geoapify.csv
├── Mini_Project_1.pptx
├── .gitignore
└── README.md
```

### Keterangan

- `mini_project1.ipynb` → notebook utama yang berisi seluruh proses project.
- `dataset_tempat_geoapify.csv` → hasil akhir data setelah cleaning.
- `Mini_Project_1.pptx` → bahan presentasi project.
- `.gitignore` → mengatur file yang tidak perlu dimasukkan ke repository.
- `README.md` → dokumentasi project.

---

## ▶️ Cara Menjalankan Project

### 1. Clone repository

```bash
git clone https://github.com/primus2709/mp1-aiautomation-2026.git
cd mp1-aiautomation-2026
```

### 2. Siapkan environment

Pastikan Python dan Jupyter Notebook/Jupyter di VS Code sudah tersedia.

### 3. Install library

Jalankan pada notebook:

```python
!pip install requests python-dotenv --quiet
```

Library utama yang digunakan:

```text
requests
pandas
python-dotenv
time
os
```

### 4. Buat file `.env`

Di folder project, buat file:

```text
.env
```

Kemudian isi:

```env
GEOAPIFY_API_KEY=isi_api_key_di_sini
```

### 5. Jalankan notebook

Buka:

```text
mini_project1.ipynb
```

Kemudian jalankan cell dari atas ke bawah.

---

## ✅ Kesimpulan

Project ini menunjukkan alur sederhana pengolahan data berbasis API mulai dari memperoleh API Key, melakukan request ke Geoapify Places API, membangun class `KlienTempat`, mengubah response API menjadi DataFrame, melakukan data cleaning, hingga menyimpan hasil akhir ke CSV.

Hasil akhirnya adalah dataset lokasi dengan **114 baris dan 5 kolom** yang telah diperiksa dan dibersihkan sehingga siap digunakan untuk proses pengolahan atau analisis berikutnya.

---

## 👤 Author

**Fajrin Efantri**  
AI Automation Engineer

---

## 📌 Catatan

Project ini dibuat sebagai bagian dari **Mini Project Pelatihan AI Automation Engineer**.
