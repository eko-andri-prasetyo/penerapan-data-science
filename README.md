# Submission Pertama: Menyelesaikan Permasalahan Human Resources (HR) — Jaya Jaya Maju

## Business Understanding

**Latar belakang:**  
Jaya Jaya Maju adalah perusahaan multinasional (berdiri sejak 2000) dengan >1000 karyawan. Perusahaan menghadapi **attrition rate >10%**, yang berdampak pada biaya rekrutmen, knowledge loss, dan produktivitas. Tujuan proyek ini adalah mengidentifikasi faktor yang memengaruhi attrition dan menyediakan **business dashboard** untuk HR agar bisa memantau faktor risiko secara berkala.

### Permasalahan Bisnis
1. Berapa **attrition rate** saat ini (berdasarkan data historis)?
2. Faktor apa saja yang paling berkorelasi dengan attrition (misalnya **overtime**, **business travel**, **job role**, **income**, **umur**, dll.)?
3. Segmen karyawan mana yang memiliki risiko attrition paling tinggi dan perlu intervensi?

### Cakupan Proyek
- Data understanding + data cleaning.
- Analisis deskriptif untuk mencari faktor risiko attrition.
- Pembuatan database (SQLite) untuk BI tool.
- Pembuatan **business dashboard** (Metabase).
- *(Opsional)*: pembuatan model ML sederhana untuk prediksi risiko attrition + script prediksi.

---

## Persiapan

### Sumber Data
Dataset: `employee_data.csv` (dari Dicoding / GitHub Dicoding dataset employee).

### Setup Environment
(Windows PowerShell):

```powershell
python -m venv .venv
.\.venv\Scripts\activate
pip install -r requirements.txt
jupyter notebook
```

---

## Data Understanding

### Ringkasan Data
- Total baris data: **1470**
- Data dengan label Attrition (untuk analisis/training): **1058**
- Data tanpa label Attrition (NaN): **412**
- Attrition rate (pada data berlabel): **16.92%** (179 karyawan keluar dari 1058 baris berlabel)

> Catatan: Karena sebagian baris memiliki `Attrition = NaN`, analisis attrition dilakukan pada baris yang berlabel. Baris tanpa label dipakai untuk demo prediksi (opsional).

### Temuan Utama (Deskriptif)
Berikut beberapa faktor dengan perbedaan attrition yang cukup menonjol:

1. **OverTime**
   - Yes: **31.92%**
   - No: **10.79%**

2. **Business Travel**
   - Travel_Frequently: **24.88%**
   - Travel_Rarely: **15.68%**
   - Non-Travel: **10.28%**

3. **Department**
   - Sales: **20.69%**
   - Human Resources: **15.79%**
   - Research & Development: **15.26%**

4. **Job Role (top tertinggi)**
   - Sales Representative: **43.10%**
   - Laboratory Technician: **26.06%**
   - Human Resources: **20.00%**
   - Research Scientist: **17.76%**
   - Sales Executive: **16.81%**

5. **Usia (Age Band)**
   - 18–25: **38.04%**
   - 26–30: **21.35%**
   - 31–35: **17.99%**
   - 36–40: **10.64%** (lebih rendah)

6. **Monthly Income (quartile)**
   - Q1 (low): **29.06%**
   - Q4 (high): **9.81%**

7. **Tenure (YearsAtCompany)**
   - 0–1 tahun: **34.59%**
   - 2–3 tahun: **19.89%**
   - 11–20 tahun: **6.92%** (lebih rendah)

---

## Data Preparation / Preprocessing

- Menghapus kolom konstan: `EmployeeCount`, `Over18`, `StandardHours`
- Menghapus kolom ID: `EmployeeId`
- Menangani missing values:
  - Numerik: imputasi median
  - Kategorikal: imputasi most frequent
- Encoding:
  - One-hot encoding untuk fitur kategorikal

Detail implementasi dapat dilihat pada `notebook.ipynb`.

---

## Modeling (Opsional)

Model yang digunakan:
- **Logistic Regression** (`class_weight="balanced"`) untuk mengatasi ketidakseimbangan kelas.

### Evaluation (hasil sample split 80/20)
- ROC-AUC: **0.816**
- Threshold 0.5:
  - Accuracy: **0.726**
  - Precision: **0.343**
  - Recall: **0.667**
  - F1: **0.453**
- Threshold 0.68 (tuning untuk F1 lebih baik):
  - Accuracy: **0.821**
  - Precision: **0.479**
  - Recall: **0.639**
  - F1: **0.548**

File model:
- `model/attrition_model.joblib`

Script prediksi:
- `prediction.py`

Contoh penggunaan:
```powershell
python prediction.py --input employee_data.csv --output pred_out.csv --threshold 0.68
```

---

## Business Dashboard (Metabase)

Dashboard dibuat untuk membantu HR:
- memantau attrition rate,
- melihat segmen risiko tertinggi,
- dan memonitor faktor yang kuat kaitannya dengan attrition.

### Database untuk Metabase
Proyek ini menggunakan **SQLite** agar ringan dan mudah untuk Metabase. File DB: `hr.db` berisi tabel `employees`.

Jika belum ada `hr.db`, bisa dibuat dari notebook (bagian “Membuat Database SQLite untuk Metabase”).

### Jalankan Metabase (Docker)
```bash
docker run -d --name metabase -p 3000:3000 -v "$PWD":/data metabase/metabase
```

**Login Metabase (disarankan sesuai instruksi Dicoding):**
- Email: `root@mail.com`
- Password: `root123`

### Connect ke SQLite
Saat setup database di Metabase:
- Database type: **SQLite**
- Path: `/data/hr.db`

### Rekomendasi Isi Dashboard (minimal 1 dashboard, berisi visual)
**KPI Cards**
1. Attrition Rate (%)


**Charts**
1. Attrition Rate by OverTime (bar)
2. Attrition Rate by BusinessTravel (bar)
3. Attrition Rate by Department (bar)
4. Attrition Rate by JobRole (bar - top 10)
5. Attrition Rate by AgeBand (bar)


**Filter yang disarankan**
- Department, JobRole, Gender, OverTime, BusinessTravel

### Export metabase database instance
Setelah dashboard selesai, export database Metabase:
```bash
docker cp metabase:/metabase.db/metabase.db.mv.db ./
```

File yang wajib dilampirkan:
- `metabase.db.mv.db`
- Screenshot dashboard: `ekoandriprasetyo-dashboard.png` atau `.jpg`

---

## Conclusion

Attrition rate pada data historis berlabel adalah **16.92%**, lebih tinggi dari target perusahaan (10%). Faktor yang paling menonjol terkait attrition adalah:

- **OverTime** (karyawan lembur jauh lebih tinggi attrition),
- **Frequent business travel**,
- **Job role tertentu** (Sales Representative & Laboratory Technician),
- **Kelompok usia lebih muda (18–25)**,
- **Income lebih rendah**,
- **Tenure pendek (0–1 tahun)**.

---

## Rekomendasi Action Items (Optional)

1. **Kebijakan lembur yang lebih sehat**
   - audit beban kerja tim dengan attrition tinggi (Sales, Lab Technician),
   - batasi overtime kronis, tambah headcount atau rotasi shift.

2. **Program retensi untuk early tenure (0–1 tahun)**
   - onboarding yang lebih kuat, buddy system, dan check-in HR di 30/60/90 hari.

3. **Intervensi untuk job role berisiko tinggi**
   - review kompensasi, jalur karier, dan insentif untuk Sales Representative dan Laboratory Technician.

4. **Kompensasi/benefit untuk income quartile bawah**
   - lakukan benchmarking, dan prioritaskan penyesuaian untuk segmen risiko tinggi.

5. **Perbaikan kebijakan perjalanan dinas**
   - rotasi perjalanan, kompensasi perjalanan, atau opsi hybrid/remote bila memungkinkan.

---

## Struktur Folder Submission

```
submission/
├── model/
│   └── attrition_model.joblib
├── notebook.ipynb
├── prediction.py
├── README.md
├── ekoandriprasetyo-dashboard.png
├── metabase.db.mv.db
├── hr.db
└── requirements.txt
```
