# Instance Segmentation Indoor

Perbandingan performa segmentasi instance pada lingkungan indoor menggunakan **YOLOv8n-seg**, **YOLO11n-seg**, dan **Mask R-CNN**. Setiap arsitektur dibandingkan dalam dua konfigurasi: bobot *pretrained* dan inisialisasi *from scratch*.

Tugas ini disusun untuk mata kuliah **Visi Komputer (EA234503)**, Institut Teknologi Sepuluh Nopember.

**Repositori:** [github.com/JustDitz/InstanceSegmentation-Indoor](https://github.com/JustDitz/InstanceSegmentation-Indoor) · **Laporan:** [laporan.pdf](laporan.pdf)

---

## Gambaran eksperimen

| Aspek | Konfigurasi |
|---|---|
| Dataset | [Panoptic Indoor Segmentation v5](https://universe.roboflow.com/panopticindoor/panoptic-indoor-segmentation) |
| Arsitektur | YOLOv8n-seg, YOLO11n-seg, Mask R-CNN ResNet50-FPN |
| Inisialisasi | Pretrained dan scratch untuk setiap arsitektur |
| Pelatihan | 10 epoch, batch size 2, seed 42 |
| Evaluasi | Mask AP50 dan AP50:95 pada data uji original dan foto personal |
| Foto personal | 41 foto: 28 normal dan 13 difficult, dianotasi menggunakan Roboflow |

Dataset original memiliki **922 gambar**: 397 train, 166 validation, dan 359 test. Anotasi menggunakan format COCO; subset YOLO dikonversi ke format YOLO-seg pada alur pelatihan.

> Perbandingan ini merupakan tugas dengan anggaran pelatihan terbatas, bukan uji kausal terkontrol sepenuhnya. Framework, preprocessing, dan konfigurasi antararsitektur berbeda.

## Hasil utama

Nilai berikut adalah **mask AP50:95 dalam persen**. Original merujuk pada test split dataset Roboflow; personal merujuk pada keseluruhan 41 foto. Nilai dibulatkan hingga dua desimal.

| Model | Original | Personal | Normal | Difficult |
|---|---:|---:|---:|---:|
| YOLOv8n pretrained | 5,25 | 5,34 | 6,93 | 10,22 |
| YOLOv8n scratch | 0,00 | 0,01 | 0,01 | 0,00 |
| **YOLO11n pretrained** | **7,33** | **7,95** | **14,74** | 11,27 |
| YOLO11n scratch | 0,00 | 0,00 | 0,00 | 0,00 |
| Mask R-CNN pretrained | 4,83 | 6,89 | 6,83 | **13,59** |
| Mask R-CNN scratch | 0,06 | 0,20 | 0,29 | 1,02 |

- Pada konfigurasi ini, model pretrained unggul atas pasangan scratch di semua kelompok evaluasi.
- YOLO11n pretrained memperoleh AP tertinggi pada test original dan keseluruhan foto personal. Mask R-CNN pretrained memperoleh AP tertinggi pada subset difficult.
- Nilai `0,00` dapat berasal dari pembulatan. AP personal juga berasal dari snapshot yang telah dibulatkan; AP50 personal tidak tersedia dalam ringkasan yang tersimpan.
- AP personal yang lebih tinggi tidak membuktikan bahwa *domain shift* meningkatkan performa: distribusi kelas dan karakteristik kedua set berbeda.

Angka lengkap dan asalnya tersedia di [`results/analisis_inisialisasi/final_table_percent.csv`](results/analisis_inisialisasi/final_table_percent.csv) dan [`metric_sources.json`](results/analisis_inisialisasi/metric_sources.json).

Lima foto pada analisis kegagalan **sengaja dipilih sebagai kasus kegagalan**, bukan sampel acak atau gambaran keseluruhan dataset. Diagnostik menggunakan confidence 0,14 dan pencocokan mask sekelas dengan IoU ≥ 0,50; pengaturan ini berbeda dari evaluasi AP dataset.

## Isi repositori

```text
.
├── viskom-train.ipynb                 # Persiapan data, training, dan evaluasi
├── viskom-final-no-retraining.ipynb   # Analisis hasil tersimpan dan kasus kegagalan
├── inference.ipynb                    # Demo inferensi pada satu foto
├── Dataset_Roboflow/                  # Ekspor dataset COCO dan split bawaan
├── Model/                             # Checkpoint enam konfigurasi
├── test/                              # Foto personal dan anotasi COCO
├── metadata_fixed.csv                 # Metadata kondisi foto personal
├── results/                           # Metrik, log, grafik, dan visualisasi
└── paper/                             # Laporan PDF dan sumber LaTeX
```

Notebook training semula dikonfigurasi untuk direktori **Kaggle**. Path dataset, checkpoint, dan output perlu disesuaikan jika dijalankan di lingkungan lain. Beberapa berkas model berukuran besar; pastikan berkas dan bobot yang diperlukan tersedia setelah clone.

## Menjalankan notebook

### 1. Siapkan lingkungan

Clone repositori dan buka direktori kerja:

```bash
git clone https://github.com/JustDitz/InstanceSegmentation-Indoor.git
cd InstanceSegmentation-Indoor
```

Gunakan Python dan Jupyter dengan pasangan **PyTorch/torchvision** yang kompatibel dengan perangkat. Petunjuk instalasi PyTorch tersedia di [pytorch.org](https://pytorch.org/get-started/locally/). Dependensi lain yang digunakan notebook meliputi `numpy`, `pandas`, `matplotlib`, `Pillow`, `pillow-heif`, `PyYAML`, `pycocotools`, `ultralytics`, dan `roboflow`.

```bash
python -m pip install jupyter numpy pandas matplotlib Pillow pillow-heif PyYAML pycocotools ultralytics roboflow
jupyter lab
```

Instalasi paket dan dukungan CUDA bergantung pada sistem operasi serta versi Python/PyTorch. Notebook ini tidak disertai environment dengan versi yang dikunci.

### 2. Pilih alur

| Kebutuhan | Notebook | Hal yang perlu diperiksa |
|---|---|---|
| Melihat dan menganalisis hasil tersimpan | `viskom-final-no-retraining.ipynb` | Buka dari root repositori. Notebook memakai metrik tersimpan/snapshot; jika cache diagnostik hilang, inferensi terbatas dapat dijalankan sesuai konfigurasi. |
| Mencoba foto baru | `inference.ipynb` | Periksa checkpoint dan mapping kelas di Blok 01; sesuaikan `PERSONAL_PHOTO_PATH` dan `COMPARE_MODELS` di Blok 04, lalu jalankan Blok 05B. |
| Melatih atau mengevaluasi ulang | `viskom-train.ipynb` | Sesuaikan path Kaggle dan dependensi terlebih dahulu. Cell training dijalankan secara terpisah dan memerlukan sumber daya komputasi. |

**Perhatian:** jangan mengandalkan `RUN_TRAINING = False` sebagai pengaman untuk semua cell: menjalankan cell training secara langsung tetap dapat memulai pelatihan. Untuk demo Mask R-CNN, verifikasi mapping kelas terhadap data training asli; kesetaraan mapping tidak dapat diasumsikan hanya dari nama checkpoint.

## Keterbatasan

- Dataset personal kecil dan distribusi kelas tidak seimbang; beberapa kategori hanya memiliki sedikit anotasi.
- Hasil berasal dari satu konfigurasi pelatihan 10 epoch, tanpa replikasi beberapa seed atau pengujian signifikansi statistik.
- Perbedaan preprocessing, framework, dan pengaturan inferensi membatasi perbandingan langsung dan atribusi efek hanya pada inisialisasi.
- Sebagian metrik berasal dari output notebook yang tersimpan, bukan file evaluasi lengkap. Lihat [provenans dan batas bukti](paper/README.md#provenans-dan-batas-bukti).
- Anotasi personal dan lima kasus terpilih tetap perlu ditinjau sebelum menarik kesimpulan yang lebih luas.

## Penulis

**Muhammad Aditya Nugraha**  
NRP 5054241004 · Kelas RKA  
Institut Teknologi Sepuluh Nopember, Surabaya, Indonesia  
Dosen: Anny Yuniarti · 2026

Dataset Panoptic Indoor Segmentation v5 mencantumkan lisensi CC BY 4.0 pada metadata ekspornya. Periksa ketentuan dataset, bobot model, serta izin penggunaan foto personal secara terpisah sebelum mendistribusikan ulang.
