# Analisis Jaringan Sungai dan Ordo Strahler — Sub-DAS Gede–Pangrango

Project Python/Jupyter Notebook untuk melakukan analisis hidrologi berbasis DEM
(Digital Elevation Model) dan menghasilkan peta jaringan sungai + orde Strahler,
**seluruhnya diturunkan dari satu file DEM** (`DEM_Gede_Pangrango_Clip.tif`) —
tanpa shapefile sungai, DAS, outlet, atau data tambahan lain.

---

## 1. Tujuan Project

Notebook ini melakukan, secara berurutan dan dengan validasi di setiap tahap:

1. Membaca & memvalidasi DEM, menangani CRS yang kosong pada metadata.
2. Preprocessing hidrologi (sink filling) dengan WhiteboxTools.
3. Menghitung flow direction (D8) dan flow accumulation.
4. Mengekstrak jaringan sungai dari flow accumulation (threshold dapat diatur).
5. Memilih titik outlet (interaktif / manual / otomatis) dan men-snap ke sungai.
6. Mendelineasi Sub-DAS dari outlet tersebut.
7. Menghitung orde sungai Strahler yang benar secara topologis.
8. Mengidentifikasi sungai utama dan titik confluence (pertemuan sungai) yang valid.
9. Menyusun peta akhir profesional (DEM + hillshade + sungai + legenda dinamis +
   colorbar + north arrow + scale bar + grid koordinat).
10. Mengekspor peta (PNG 400 dpi, PDF) dan seluruh data vektor (GeoPackage).

Semua angka pada peta (luas, panjang sungai, elevasi, orde, koordinat outlet)
**dihitung dari DEM dan pilihan outlet yang sesungguhnya** — tidak ada nilai yang
dikarang atau di-hardcode.

---

## 2. Struktur Folder

```
PROJECT/
│
├── DEM_Gede_Pangrango_Clip.tif                       <- DEM Anda (letakkan di sini)
├── Analisis_DEM_SubDAS_Strahler_Gede_Pangrango.ipynb <- notebook utama
├── requirements.txt
├── environment.yml
├── README.md
│
└── HASIL_ANALISIS_DAS/
    ├── 01_preprocessing/   <- DEM hasil reproyeksi ke UTM 48S
    ├── 02_hydrology/       <- DEM filled, flow direction, flow accumulation
    ├── 03_streams/         <- jaringan sungai raster & vector, orde Strahler
    ├── 04_watershed/       <- watershed, outlet, DEM ter-clip, hillshade
    ├── 05_maps/            <- PETA FINAL (PNG & PDF) + tabel statistik CSV
    └── 06_vector/          <- data vektor final (.gpkg): Sub-DAS, sungai, outlet
```

Semua sub-folder dibuat otomatis oleh notebook (Tahap 2) jika belum ada.

---

## 3. Instalasi Python

Gunakan **Python 3.11** (direkomendasikan) atau minimal Python 3.9, pada Windows
10/11.

- Unduh dari [python.org](https://www.python.org/downloads/), **atau**
- Gunakan distribusi [Anaconda](https://www.anaconda.com/download) /
  [Miniconda](https://docs.conda.io/en/latest/miniconda.html) (direkomendasikan
  karena mempermudah instalasi `geopandas`/`rasterio` di Windows).

Cek versi Python di Command Prompt / Anaconda Prompt:

```
python --version
```

---

## 4. Instalasi Dependencies

### Opsi A — pip (venv biasa)

```
python -m venv venv
venv\Scripts\activate
pip install --upgrade pip
pip install -r requirements.txt
```

### Opsi B — conda (direkomendasikan di Windows)

```
conda env create -f environment.yml
conda activate subdas-strahler-gede-pangrango
```

Library yang dipakai: `rasterio`, `numpy`, `pandas`, `geopandas`, `shapely`,
`pyproj`, `matplotlib`, `scipy`, `whitebox`, `networkx`, `matplotlib-scalebar`,
`jupyter`.

Notebook (**Tahap 1**) akan otomatis memeriksa seluruh library ini setiap kali
dijalankan dan memberi instruksi instalasi yang jelas jika ada yang belum
terpasang.

---

## 5. Cara Membuka Notebook

1. Letakkan `DEM_Gede_Pangrango_Clip.tif` di folder yang sama dengan file
   `.ipynb` (folder `PROJECT/` di atas).
2. Buka salah satu dari:
   - **Jupyter Notebook**: `jupyter notebook` di terminal, lalu klik file `.ipynb`.
   - **JupyterLab**: `jupyter lab`.
   - **VS Code**: buka folder project, klik file `.ipynb` (extension "Jupyter"
     dari Microsoft harus terpasang), pilih kernel Python yang sudah berisi
     dependencies di atas.

---

## 6. Cara Menjalankan Notebook

Jalankan sel **secara berurutan dari atas ke bawah** (`Run All` / `Restart &
Run All`). Setiap tahap memvalidasi outputnya sendiri sebelum lanjut — jika ada
tahap yang gagal, notebook akan **berhenti** dengan pesan diagnostik lengkap
(nama proses, input, output yang diharapkan, working directory, isi folder,
status tool) alih-alih melanjutkan dengan hasil yang salah.

Notebook aman dijalankan ulang dari awal kapan pun (`Restart & Run All`) tanpa
perlu mengubah path apa pun.

---

## 7. Cara Memilih Outlet (Tahap 9)

DEM yang Anda miliki **tidak dilengkapi titik outlet**, sehingga Anda perlu
menentukannya sendiri melalui variabel `OUTLET_MODE` pada sel konfigurasi
Tahap 9:

| Mode | Cara pakai |
|---|---|
| `"interactive"` | Jalankan `%matplotlib qt` atau `%matplotlib tk` di sel sebelum Tahap 9 (agar jendela plot interaktif muncul), lalu klik satu titik pada peta yang ditampilkan. |
| `"manual"` | Lihat peta hillshade + sungai yang ditampilkan, perkirakan koordinat UTM 48S (meter) outlet yang diinginkan, isi `MANUAL_OUTLET_X` dan `MANUAL_OUTLET_Y`. |
| `"auto"` | Fallback otomatis: memilih titik pada jaringan sungai yang menghasilkan luas tangkapan pada rentang `AUTO_TARGET_AREA_HA_MIN`–`AUTO_TARGET_AREA_HA_MAX` (default 500–1500 ha). Berguna untuk uji coba/verifikasi cepat, **namun untuk analisis sesungguhnya sangat disarankan memakai mode `interactive` atau `manual`** agar lokasi outlet benar-benar sesuai maksud Anda. |

Setelah outlet ditentukan, notebook otomatis melakukan **snapping** ke sel
sungai terdekat (`Jenson Snap Pour Points`) agar watershed yang dihasilkan
akurat.

---

## 8. Cara Mengubah `STREAM_THRESHOLD` (Tahap 8)

`STREAM_THRESHOLD` (default `500`) menentukan jumlah sel tangkapan minimum agar
sebuah sel dianggap bagian dari jaringan sungai:

- **Threshold kecil** (mis. 100–300) → jaringan sungai lebih padat/detail.
- **Threshold besar** (mis. 1000–3000) → jaringan sungai lebih jarang, hanya
  sungai-sungai besar.

Ubah nilainya pada sel konfigurasi Tahap 8, lalu jalankan ulang dari sel
tersebut (semua tahap sesudahnya akan otomatis memakai jaringan sungai yang
baru).

---

## 9. Penjelasan Output

| File | Isi |
|---|---|
| `01_preprocessing/DEM_Gede_Pangrango_UTM48S.tif` | DEM hasil reproyeksi ke EPSG:32748 |
| `02_hydrology/dem_filled.tif` | DEM setelah sink filling |
| `02_hydrology/flow_direction.tif` | Arah aliran D8 |
| `02_hydrology/flow_accumulation.tif` | Akumulasi aliran D8 |
| `03_streams/streams.tif` | Jaringan sungai (raster biner) |
| `03_streams/stream_order.tif` | Orde Strahler per sel sungai |
| `04_watershed/watershed.tif`, `dem_subdas.tif`, `hillshade.tif` | Watershed, DEM ter-clip, hillshade |
| `05_maps/Peta_SubDAS_Strahler_Gede_Pangrango.png` | **Peta akhir**, 400 dpi |
| `05_maps/Peta_SubDAS_Strahler_Gede_Pangrango.pdf` | Peta akhir, format PDF |
| `05_maps/statistik_subdas.csv`, `panjang_sungai_per_orde.csv` | Tabel statistik |
| `06_vector/SubDAS_Gede_Pangrango.gpkg` | Polygon batas Sub-DAS + atribut luas/elevasi |
| `06_vector/Jaringan_Sungai_Strahler.gpkg` | Garis sungai + atribut orde |
| `06_vector/Outlet.gpkg` | Titik outlet (setelah snapping) |

---

## 10. Troubleshooting WhiteboxTools

- **Unduhan binary WhiteboxTools gagal / lambat**: paket `whitebox` mengunduh
  binary WhiteboxTools secara otomatis saat pertama kali dipakai. Pastikan
  komputer terkoneksi internet saat pertama kali menjalankan notebook.
  Jika gagal, unduh manual dari
  [whiteboxgeo.com](https://www.whiteboxgeo.com/download-whiteboxtools/) dan
  ikuti instruksi instalasi manual di dokumentasi paket `whitebox` Python.
- **Error: file output tidak ditemukan padahal proses "sukses" (return code 0)**:
  ini adalah kelas error yang notebook ini didesain khusus untuk mencegahnya.
  Semua path yang dikirim ke WhiteboxTools **wajib absolute** (bukan relatif) —
  ini sudah ditangani otomatis oleh fungsi `run_whitebox_checked()`. Jika Anda
  memodifikasi notebook dan menambahkan pemanggilan tool baru, selalu panggil
  lewat `run_whitebox_checked()`, jangan memanggil method `wbt.<tool>()` secara
  langsung.
- **WhiteboxTools hanya mendukung Shapefile (`.shp`) untuk vector**, bukan
  GeoPackage. Notebook ini menangani hal ini dengan memproses semua vector
  perantara (outlet, watershed) sebagai `.shp`, baru mengonversinya ke `.gpkg`
  di akhir dengan GeoPandas — Anda tidak perlu melakukan apa pun secara manual.
- **File output "hilang" sesaat setelah proses selesai (jarang terjadi)**: pada
  disk yang lambat atau saat antivirus sedang memindai file baru, penulisan
  file bisa tertunda singkat setelah WhiteboxTools melaporkan selesai. Fungsi
  `run_whitebox_checked()` sudah menangani ini dengan mencoba ulang (retry)
  beberapa kali sebelum benar-benar menyatakan gagal.
- **Antivirus Windows memblokir `whitebox_tools.exe`**: tambahkan folder
  instalasi paket `whitebox` (biasanya di dalam `site-packages/whitebox/WBT/`)
  sebagai pengecualian (exclusion) pada Windows Defender / antivirus Anda.

---

## 11. Troubleshooting Interactive Matplotlib (Pemilihan Outlet)

- **`ginput()` tidak merespons klik / langsung timeout**: backend matplotlib
  Anda kemungkinan `inline` (default di banyak setup Jupyter), yang tidak
  mendukung interaksi. Jalankan salah satu perintah berikut di sel baru sebelum
  Tahap 9, lalu jalankan ulang sel Tahap 9:

  ```
  %matplotlib qt
  ```
  atau, jika `qt` tidak tersedia:
  ```
  %matplotlib tk
  ```

  Setelah selesai memilih outlet, Anda dapat mengembalikan tampilan inline
  dengan `%matplotlib inline` bila diperlukan untuk sel-sel visualisasi
  berikutnya.

- **Backend interaktif tetap tidak berfungsi** (mis. di JupyterLab tanpa paket
  GUI backend terpasang, atau di VS Code Web): gunakan `OUTLET_MODE = "manual"`
  dan isi koordinat `MANUAL_OUTLET_X` / `MANUAL_OUTLET_Y` berdasarkan peta yang
  ditampilkan (Anda bisa memperkirakan koordinat dari sumbu X/Y pada gambar
  hasil `plt.show()`), atau gunakan `OUTLET_MODE = "auto"` sebagai fallback.

- **Jendela `qt`/`tk` muncul tapi membeku**: pastikan Anda benar-benar mengklik
  di dalam area gambar sebelum waktu `timeout` (60 detik) habis; jika terlewat,
  jalankan ulang sel Tahap 9.

---

## Catatan Penting

- DEM asli **tidak pernah diubah** — semua hasil olahan ditulis ke folder
  `HASIL_ANALISIS_DAS/`.
- Jika DEM Anda berbeda (nama file, lokasi, cakupan wilayah lain), notebook
  akan tetap berjalan selama bounds koordinatnya masih berupa lintang/bujur
  yang masuk akal; jika bukan Gede–Pangrango, sesuaikan asumsi CRS pada Tahap 4.
- Ganti nilai `STREAM_THRESHOLD` (Tahap 8) dan `OUTLET_MODE`/koordinat outlet
  (Tahap 9) sesuai kondisi lapangan yang Anda ketahui — nilai default hanyalah
  titik awal yang wajar, bukan nilai yang pasti benar untuk semua kasus.
