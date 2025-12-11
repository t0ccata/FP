# Cerebral Vessel Segmentation Using U-Net for 3D Vascular Reconstruction

Repositori ini berisi implementasi pipeline *Deep Learning* otomatis untuk segmentasi pembuluh darah otak dari citra TOF-MRA 3D dan rekonstruksi 3D vaskular. Proyek ini bertujuan untuk mengatasi keterbatasan analisis konvensional 2D dengan menghasilkan model 3D fidelitas tinggi untuk membantu diagnosis klinis.

## 1\. Latar Belakang dan Metodologi

### Latar Belakang

Visualisasi struktur serebrovaskular yang akurat sangat penting untuk mendiagnosis penyakit otak. Namun, analisis konvensional *Time-of-Flight Magnetic Resonance Angiography* (TOF-MRA) seringkali terbatas pada pandangan 2D atau *Maximum Intensity Projections* (MIP), yang mungkin tidak sepenuhnya merepresentasikan kedalaman spasial yang kompleks dari topologi vaskular. Selain itu, segmentasi manual yang diperlukan untuk pemodelan 3D tidak efisien dan rentan terhadap variabilitas antar-pengamat.

### Metodologi

Studi ini mengembangkan pipeline segmentasi otomatis menggunakan arsitektur **3D U-Net** yang dilatih pada COSTA Database. Metodologi ini mengintegrasikan:

1.  **Preprocessing Berbasis MONAI**: Untuk standarisasi data, termasuk *resampling* isotropik dan normalisasi intensitas.
2.  **Model Segmentasi 3D U-Net**: Dilatih menggunakan PyTorch untuk memisahkan piksel pembuluh darah dari latar belakang.
3.  **Rekonstruksi Permukaan 3D**: Menggunakan algoritma *Marching Cubes* (via VTK/skimage) untuk menghasilkan rekonstruksi permukaan 3D yang akurat dari hasil segmentasi.

-----

## 2\. Struktur Folder dan Alur Kerja (Workflow)

Berikut adalah struktur direktori proyek dan penjelasan mengenai aliran data antar file:

```bash
.
├── model_training.ipynb   # Notebook untuk melatih model
├── inference.ipynb        # Notebook untuk pengujian/prediksi
├── niftitovtp.ipynb       # Notebook untuk konversi hasil ke 3D Mesh
└── README.md              # Dokumentasi proyek
```

### Alur Data (Pipeline Flow)

Proses pengolahan data berjalan secara berurutan melalui notebook berikut:

1.  **`model_training.ipynb`**:

      * **Input**: Mengambil data mentah dari folder `data/`.
      * **Proses**: Preprocessing -\> Training U-Net 3D.
      * **Output**: Menyimpan model terbaik (`model.pt`) ke folder `models/`.

2.  **`inference.ipynb`**:

      * **Input**: Memuat `model.pt` dari `models/` dan data uji dari `data/`.
      * **Proses**: Sliding Window Inference -\> Thresholding.
      * **Output**: Menghasilkan file segmentasi mask (misal: `prediction.nii.gz`) di folder `output/`.

3.  **`niftitovtp.ipynb`**:

      * **Input**: Mengambil file segmentasi (`prediction.nii.gz`) dari `output/`.
      * **Proses**: Algoritma Marching Cubes -\> Mesh Generation.
      * **Output**: File objek 3D (`.vtp`) di folder `output/` yang siap divisualisasikan.

-----

## 3\. Setup Environment

Untuk menjalankan proyek ini, pastikan Anda memiliki Python dan lingkungan yang mendukung GPU (disarankan NVIDIA CUDA).

### Instalasi Dependencies

Install library yang diperlukan menggunakan pip:

```bash
pip install torch torchvision torchaudio
pip install monai nibabel numpy matplotlib
pip install scikit-learn scikit-image vtk
```

-----

## 4\. Cara Menjalankan Projek

Ikuti langkah-langkah berikut secara berurutan:

### Tahap 1: Pelatihan Model

Buka dan jalankan `model_training.ipynb`.

  * Script ini akan memuat dataset, melakukan augmentasi data (rotasi, scaling), dan melatih model U-Net.
  * Pastikan path dataset sudah sesuai dengan lokasi folder `data/` Anda.
  * Hasil training berupa file `model.pt` akan tersimpan otomatis.

### Tahap 2: Inference (Prediksi)

Buka dan jalankan `inference.ipynb`.

  * Pastikan path ke `model.pt` sudah benar.
  * Script ini akan melakukan segmentasi pada data test.
  * Hasilnya adalah file NIfTI (.nii.gz) yang berisi mask biner pembuluh darah.

### Tahap 3: Konversi ke 3D Model

Buka dan jalankan `niftitovtp.ipynb`.

  * Masukkan path file hasil inference dari tahap sebelumnya.
  * Script akan mengonversi voxel volume menjadi surface mesh.
  * Output final berupa file `.vtp` (misal: `output_mask_clean.vtp`).

-----

## 5\. Hasil Pipeline Modeling

### Evaluasi Model

Berdasarkan pengujian, model ini menghasilkan performa tinggi dengan metrik sebagai berikut:

  * **Dice Similarity Coefficient**: 0.9118
  * **IoU (Jaccard Index)**: 0.8391
  * **Accuracy**: 99.75%
  * **Average Hausdorff Distance**: 0.3313 mm

### Visualisasi (Nifti to VTP)

Hasil akhir proyek ini adalah rekonstruksi 3D pembuluh darah yang dapat dibuka menggunakan software visualisasi medis seperti **ParaView** atau **3D Slicer**. File `.vtp` memungkinkan inspeksi topologi vaskular secara mendetail untuk keperluan analisis medis.
