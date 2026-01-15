# 📊 Google Business Review Intelligence & Staff KPI Dashboard

Sistem pemantauan performa layanan pelanggan otomatis yang mengekstraksi wawasan (insights) dari ulasan Google Business Profile. Dengan menggabungkan **Python (Regex & Data Processing)** dan **Looker Studio**, sistem ini mampu mengukur KPI staf secara objektif berdasarkan apresiasi langsung dari pelanggan.

---

## 🎯 Latar Belakang & Motivasi Proyek

### Mengapa Proyek Ini Dibuat?

Dalam industri layanan kesehatan (terutama spa/klinik kecantikan), **penilaian performa karyawan** masih sering menggunakan metode tradisional seperti:
- Penilaian subjektif dari manager
- Target penjualan tanpa mempertimbangkan kepuasan pelanggan
- Kurangnya data objektif dari feedback pelanggan

**Masalah yang Ingin Diselesaikan:**
1. **Penilaian Staf Tidak Objektif**: Performa karyawan tidak terukur dengan baik dari perspektif pelanggan
2. **Data Tersebar**: Ulasan tersebar di berbagai platform (Google, TripAdvisor, dll) tanpa analisis terpadu
3. **Insight Tertunda**: Tidak ada visibility real-time untuk manajemen tentang performa layanan
4. **Sulit Identifikasi Bottleneck**: Mana layanan yang paling disukai? Staf mana yang paling diapresiasi?

**Solusi yang Kami Tawarkan:**
- **Otomasi Ekstraksi Data**: Ambil ulasan langsung dari Google Business Profile secara programmatic
- **Analisis Cerdas**: Gunakan Regex Pattern Matching untuk mendeteksi penyebutan nama staf dalam teks ulasan
- **KPI Terukur**: Hitung frekuensi apresiasi per staf sebagai KPI objektif berbasis feedback pelanggan
- **Dashboard Real-Time**: Visualisasi data di Looker Studio untuk decision-making yang lebih cepat
- **Scalable**: Sistem dapat direplikasi untuk multi-lokasi atau multi-brand

---

## 🚀 Fitur Utama

### ✨ 1. Automated Staff Mention Extraction
- Mendeteksi penyebutan nama staf dalam ulasan menggunakan **Pattern Matching (Regex)**
- Case-insensitive search untuk menangkap variasi penulisan nama
- Contoh: "dr. Josie", "dr. josie", "Dr Josie", "Josie" akan terdeteksi semuanya
- Mendukung alias atau nickname staf

### 📈 2. KPI Generation
- Menghitung **frekuensi apresiasi pelanggan per staf**
- Agregasi metrics: Total ulasan, Rating rata-rata, Jumlah mention
- Indikator kualitas layanan berdasarkan feedback real pelanggan
- Tracking trend performa dari waktu ke waktu

### 💭 3. Sentiment Analysis Ready
- Menyiapkan data untuk kategorisasi ulasan positif vs negatif
- Foundation untuk NLP-based insights di masa depan
- Rating star sebagai proxy sentiment (5⭐ = Positif, 1-3⭐ = Negatif)

### 📊 4. Live Dashboard Integration
- Data terhubung langsung ke **Looker Studio** untuk pemantauan real-time
- Visualisasi interaktif: Bar Chart, Time Series, Scorecard, Pie Chart
- Akses dashboard dapat dibagikan ke manajemen atau HR team
- Update otomatis sesuai refresh schedule Google Sheets

---

## 🛠️ Step-by-Step Workflow: From JSON to Dashboard

### **Step 1: Data Acquisition (Google Business Profile)**

#### 1.1 Dapatkan Akses API atau Export Manual
```
Pilih satu dari dua metode:

✅ Metode A: Google Business API (Recommended)
   - Buka Google Cloud Console
   - Enable "Google My Business API"
   - Buat Service Account & download credentials JSON
   - Gunakan API untuk pull review secara programmatic
   
✅ Metode B: Export Manual (Quick & Simple)
   - Login ke Google Business Profile
   - Navigate ke "Reviews" section
   - Klik "⋮" (three dots) → Export as JSON
   - Download file reviews-*.json
```

#### 1.2 Format Data JSON
Data yang diterima umumnya memiliki struktur:
```json
{
  "reviewId": "ABHRLXU...",
  "reviewer": {
    "displayName": "John Doe",
    "profilePhotoUrl": "..."
  },
  "reviewTime": {
    "seconds": 1735833600,
    "nanos": 0
  },
  "rating": 5,
  "text": "Great treatment! Dr Brill treated me wonderfully. Highly recommended!",
  "status": "PUBLIC",
  "ownerResponse": {
    "text": "Thank you for your positive feedback...",
    "reviewTime": {...}
  }
}
```

**Field Penting:**
- `reviewId`: Unique identifier
- `reviewer.displayName`: Nama reviewer
- `rating`: Jumlah bintang (1-5)
- `text`: Isi ulasan (tempat mencari nama staf)
- `reviewTime`: Timestamp untuk time-series analysis

#### 1.3 Konversi ke DataFrame
Semua file JSON akan dimuat dan dikombinasikan dalam Pandas DataFrame:
```python
# Input: reviews-*.json files di folder /data
# Output: Combined DataFrame dengan semua reviews
```

---

### **Step 2: Intelligent Data Processing (Python Engine)**

#### 2.1 Setup Staff Library
Buat daftar nama staf yang akan dicari dalam ulasan:
```python
STAFF_NAMES = {
    'Ayuk': ['ayuk', 'ayu'],
    'Kathleen': ['kathleen', 'kathlen'],
    'Ria': ['ria', 'rya'],
    'Weby': ['weby', 'web'],
    'Amel': ['amel', 'amelia'],
    'Santi': ['santi', 'santi'],
    'Dr. Brill': ['dr brill', 'dr. brill', 'brill'],
    'Miranda': ['miranda', 'mira'],
    'Dr. Angel': ['dr angel', 'dr. angel', 'angel'],
}
```

#### 2.2 Regex Pattern Matching
Buat pattern pencarian yang tidak sensitif huruf besar/kecil:
```python
import re

def extract_mentioned_staff(review_text, staff_names):
    """
    Ekstrak nama staf yang disebutkan dalam teks ulasan
    
    Args:
        review_text: Teks ulasan dari customer
        staff_names: Dictionary mapping staff name to aliases
        
    Returns:
        List of mentioned staff names
    """
    mentioned = []
    text_lower = review_text.lower()
    
    for staff_name, aliases in staff_names.items():
        # Buat pattern: \b(alias1|alias2|alias3)\b
        # \b = word boundary (avoid matching partial words)
        pattern = r'\b(' + '|'.join(aliases) + r')\b'
        
        if re.search(pattern, text_lower):
            mentioned.append(staff_name)
    
    return mentioned
```

**Penjelasan Regex:**
- `\b`: Word boundary → cari kata yang utuh
- `|`: OR operator → cari alias apapun
- Case-insensitive flag → tangkap semua variasi huruf

**Contoh:**
```
Input: "Great treatment! Dr Brill treated me wonderfully."
Pattern: r'\b(dr brill|dr. brill|brill)\b'
Output: ['Dr. Brill']

Input: "Ayuk dan Kathleen gave me excellent service!"
Pattern: untuk 'Ayuk': r'\b(ayuk|ayu)\b'
Pattern: untuk 'Kathleen': r'\b(kathleen|kathlen)\b'
Output: ['Ayuk', 'Kathleen']
```

#### 2.3 Data Processing & Tagging
```python
# Untuk setiap review, jalankan ekstraksi
df['Mentioned_Staff'] = df['text'].apply(
    lambda x: extract_mentioned_staff(x, STAFF_NAMES)
)

# Contoh hasil:
# Review 1: "Great treatment! Dr Brill treated me..." → ['Dr. Brill']
# Review 2: "Ayuk dan Kathleen gave me..." → ['Ayuk', 'Kathleen']
# Review 3: "Nice place" → []
```

#### 2.4 KPI Calculation
```python
# Hitung frekuensi mention per staf
staff_kpi = {}

for staff_name in STAFF_NAMES.keys():
    count = df['Mentioned_Staff'].apply(
        lambda x: staff_name in x if isinstance(x, list) else False
    ).sum()
    
    staff_kpi[staff_name] = {
        'mention_count': count,
        'avg_rating': df[df['Mentioned_Staff'].apply(
            lambda x: staff_name in x if isinstance(x, list) else False
        )]['rating'].mean()
    }

# Hasil KPI:
# Dr. Brill: 9 mentions, avg rating 4.8
# Ayuk: 7 mentions, avg rating 4.9
# Kathleen: 6 mentions, avg rating 4.7
# ...
```

#### 2.5 Flattening & Export
Karena satu review bisa mention multiple staff, kita flatten data:
```python
# Original: 1 row = 1 review dengan multiple staff mentions
# Flattened: 1 row = 1 review-staff pair

flattened_data = []
for idx, row in df.iterrows():
    if isinstance(row['Mentioned_Staff'], list) and row['Mentioned_Staff']:
        for staff in row['Mentioned_Staff']:
            new_row = row.copy()
            new_row['Staff_Mentioned'] = staff
            flattened_data.append(new_row)
    else:
        new_row = row.copy()
        new_row['Staff_Mentioned'] = None
        flattened_data.append(new_row)

df_final = pd.DataFrame(flattened_data)

# Export ke Excel/CSV
df_final.to_excel('Staff_Extracted_Final.xlsx', index=False)
```

---

### **Step 3: Data Connection (Looker Studio)**

#### 3.1 Upload ke Google Sheets
```
1. Buka Google Sheets (sheets.google.com)
2. Buat spreadsheet baru atau gunakan existing
3. Buka file Excel yang sudah diekspor: Staff_Extracted_Final.xlsx
4. Copy semua data dan paste ke Google Sheets
5. (Alternatif) Gunakan API untuk upload otomatis
```

#### 3.2 Struktur Data di Google Sheets
Pastikan kolom-kolom berikut tersedia:
```
A: Date (dari reviewTime)
B: Rating (1-5)
C: Reviewer Name
D: Review Text
E: Staff_Mentioned
F: Sentiment (positif/negatif - optional)
G: Service_Category (jenis layanan - optional)
```

#### 3.3 Buat Koneksi di Looker Studio
```
1. Buka Looker Studio (looker.com/studio)
2. Klik "Create" → "Report"
3. Klik "Data" (di sidebar) → "Add data source"
4. Pilih "Google Sheets"
5. Select spreadsheet yang berisi data Staff_Extracted_Final
6. Klik "Connect"
```

#### 3.4 Konfigurasi Data Types
Di Looker Studio Data Source:
```
Field Name          → Data Type
Date               → Date
Rating             → Number
Reviewer Name      → Text
Review Text        → Text (tidak perlu di-aggregasi)
Staff_Mentioned    → Dimension (Text)
Sentiment          → Dimension (Text)
Service_Category   → Dimension (Text)
```

---

### **Step 4: Building the KPI Dashboard**

#### 4.1 Dashboard Layout & Components

**A. Header Section (KPI Cards)**
```
┌─────────────────────────────────────┐
│ Total Reviews    │ Avg Rating │ Month  │
│      156         │   4.7      │ Jan'26 │
└─────────────────────────────────────┘
```
- Scorecard widgets untuk quick metrics
- Akurat month-over-month comparison

**B. Service Treatment Analysis**
```
Service Treatment Mentioned (Top 10)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
1. Diamond Microdermabrasi... [████████] 3
2. Detox Package            [██████] 2
3. Purifying Facial         [████] 1
4. Massage                  [████] 1
...
```
- Bar Chart menampilkan layanan apa yang paling sering didiskusikan
- Sorted descending by mention count

**C. Staff Leaderboard**
```
Staff Mentioned (Top 10)
━━━━━━━━━━━━━━━━━━━━━━━━━
Staff         │ Mention Count │ Avg Rating
Ayuk          │      9        │   5.0 ⭐
Kathleen      │      6        │   4.8 ⭐
Ria           │      5        │   5.0 ⭐
Weby          │      3        │   5.0 ⭐
...
```
- Table atau Bar Chart
- Primary KPI untuk evaluate staff performance
- Sortable by mention count atau rating

**D. Time Series Analysis**
```
Review Volume Over Time
━━━━━━━━━━━━━━━━━━━━━━━━━
   │     
 5 │  ╱╲
 4 │ ╱  ╲
 3 │╱    ╲  ╱╲
 2 │      ╲╱  ╲
 1 │           ╲
   └──────────────── 
    Dec        Jan
```
- Line/Area Chart
- Trend volume ulasan per bulan
- Identify seasonal patterns atau campaign effectiveness

**E. Rating Distribution**
```
Record Count by Rate
━━━━━━━━━━━━━━━━━━━
⭐⭐⭐⭐⭐  [████████████████] 12
⭐⭐⭐⭐   [████] 3
⭐⭐⭐    [██] 1
⭐⭐     [_] 0
⭐      [_] 0
```
- Pie Chart atau Bar Chart
- Berapa persen customers puas (5⭐)?

#### 4.2 Cara Membuat Widgets di Looker Studio

**Contoh: Staff Leaderboard (Table)**
```
1. Click "Insert" → "Table"
2. Drag ke area dashboard
3. Di Datasoure Properties (right panel):
   - Dimension: Staff_Mentioned
   - Metric: COUNT() as Mention_Count
   - Metric: AVG(Rating) as Avg_Rating
4. Sort by: Mention_Count descending
5. Click "Apply" untuk confirm
```

**Contoh: Time Series (Line Chart)**
```
1. Click "Insert" → "Time series"
2. Drag ke area dashboard
3. Di Data Properties:
   - Dimension: Date (X-axis)
   - Metric: COUNT() as Review_Count (Y-axis)
4. Click "Apply"
```

**Contoh: Rating Distribution (Pie Chart)**
```
1. Click "Insert" → "Pie chart"
2. Di Data Properties:
   - Dimension: Rating
   - Metric: COUNT()
3. Click "Apply"
```

#### 4.3 Interactivity & Filters
```
Tambahkan filters untuk dynamic analysis:

1. Click "Insert" → "Filter"
2. Pilih filter type:
   - Date Range (untuk time period selection)
   - Dropdown (untuk Staff_Mentioned)
   - Dropdown (untuk Service_Category)
3. Connect filter ke semua charts
```

---

## 📂 Struktur Proyek

```
Google Business Review Intelligence/
│
├── 📄 README.md                           # Documentation (file ini)
├── 🔧 Automation_Google_Review.ipynb      # Main Python engine
├── 📊 Looker_Studio_Dashboard_Link.txt    # Link akses dashboard
│
├── 📁 data/                               # Raw data dari Google Business
│   ├── reviews-ABHRLXUgZrOzYgQR3VZKiTe4hHDPDDULTgoejQ.json
│   ├── reviews-ABHRLXUHKxQGytfIYauicYZv1KotauVlepX9BH.json
│   ├── reviews-ABHRLXUNxGK6quVlsTH0Z_0zurQHi2ERCKL3GB.json
│   ├── reviews-ABHRLXV--srl83a0LRneaiQ6_axEfEjFCnzjoN.json
│   ├── reviews-ABHRLXV3iVDWk2N60Cy_ZD_2i90lOsPwlFlyN4.json
│   ├── reviews-ABHRLXVUfmSDlorY_OF7V_56q_lTa9OMgq_Wzd.json
│   ├── reviews-ABHRLXW3FJ6sSeXmSLq_4KUmNDXxN8jzJnr8qA.json
│   ├── reviews-ABHRLXW75b8nKGtPp6I-v6qBrT4bsSX0xQzyob.json
│   ├── reviews-ABHRLXWj8eh7W0SgYP4BCUFMj5EjzTU9cRl689.json
│   ├── reviews-ABHRLXWLXpvm9RMsFcu91MeK7hddHPrGjxM8Gv.json
│   ├── reviews-ABHRLXWZKEF8Za77Ua5-865LRg9HU82FSJ3Rcp.json
│   ├── reviews-ABHRLXX6R4dKzVOFI7WJ_360MDU2DZPtk-QofR.json
│   ├── reviews-ABHRLXXcru25os7ZuTMNIJPzyyZbabBCpnYyiD.json
│   ├── reviews-ABHRLXXJF-O_E9-jlQ6LQ_3GzumGKYWqVWsMZp.json
│   ├── reviews-ABHRLXXr0UG_dT-lNBEF7Kpk6q9DfIovn3g1zE.json
│   ├── reviews-ABHRLXXx0t8ZsqLQ0jeiYbkatkqhcDS-MTSxft.json
│   └── reviews.json                       # Combined data (jika ada)
│
└── 📁 output/                             # (Optional) Hasil output
    ├── Staff_Extracted_Final.xlsx
    └── Staff_Extracted_Final.csv
```

**Penjelasan File:**

| File | Deskripsi |
|------|-----------|
| `README.md` | Dokumentasi lengkap proyek (Anda sedang membacanya!) |
| `Automation_Google_Review.ipynb` | Jupyter Notebook berisi logic Python untuk ETL dan ekstraksi nama staf |
| `Looker_Studio_Dashboard_Link.txt` | File teks berisi link akses ke Looker Studio dashboard (untuk sharing) |
| `data/*.json` | File-file JSON raw dari Google Business Profile API/export |
| `output/Staff_Extracted_Final.xlsx` | File Excel hasil olahan - data siap untuk Looker Studio |

---

## 🎓 Cara Menggunakan Repository

### Untuk User Baru (Setup Awal)

#### 1️⃣ Clone atau Download Repository
```bash
# Jika menggunakan Git:
git clone https://github.com/[username]/Google-Business-Review-Intelligence.git
cd Google-Business-Review-Intelligence

# Atau download langsung sebagai ZIP
```

#### 2️⃣ Install Dependencies
```bash
# Pastikan Python 3.8+ terinstall
pip install pandas openpyxl requests

# Untuk Jupyter Notebook (jika belum ada):
pip install jupyter
```

#### 3️⃣ Prepare Raw Data
```bash
# Copy file-file reviews JSON ke folder:
# data/reviews-*.json

# Struktur folder harus seperti ini:
data/
├── reviews-ABHRLXUgZrOzYgQR3VZKiTe4hHDPDDULTgoejQ.json
├── reviews-ABHRLXUHKxQGytfIYauicYZv1KotauVlepX9BH.json
├── ... (file lainnya)
```

#### 4️⃣ Run Automation Script
```bash
# Buka Jupyter Notebook
jupyter notebook Automation_Google_Review.ipynb

# Atau jalankan cell-by-cell sesuai dokumentasi
# Output akan tersimpan di: output/Staff_Extracted_Final.xlsx
```

#### 5️⃣ Setup Looker Studio
```
1. Buka file output/Staff_Extracted_Final.xlsx
2. Copy semua data → Google Sheets
3. Di Looker Studio, create report baru
4. Connect data source ke Google Sheets tadi
5. Build dashboard sesuai Step 4 (di atas)
```

---

### Untuk User Lanjutan (Customization)

#### Edit Staff Names
Buka `Automation_Google_Review.ipynb`, cari section:
```python
STAFF_NAMES = {
    'Nama Staf': ['alias1', 'alias2'],
    # ... tambahkan atau edit sesuai kebutuhan
}
```

#### Adjust Regex Patterns
Jika ada nama yang tidak terdeteksi, Edit pattern matching:
```python
# Contoh: untuk detect "dr brill", "dr. brill", "dr brill"
'Dr. Brill': ['dr brill', 'dr. brill', 'brill', 'dr brill'],
```

#### Extend dengan Sentiment Analysis
```python
# Add library (TextBlob atau vader)
from textblob import TextBlob

def analyze_sentiment(text):
    """Convert text to sentiment score"""
    analysis = TextBlob(text)
    if analysis.sentiment.polarity > 0.1:
        return 'Positive'
    elif analysis.sentiment.polarity < -0.1:
        return 'Negative'
    else:
        return 'Neutral'

df['Sentiment'] = df['text'].apply(analyze_sentiment)
```

---

## 📦 Instalasi & Dependensi

### System Requirements
- **Python**: 3.8 atau lebih baru
- **Memory**: Minimal 2GB RAM (untuk processing data)
- **Disk Space**: 500MB (untuk data + dependencies)
- **Internet**: Diperlukan untuk akses Google Sheets & Looker Studio

### Package Dependencies

```bash
# Pandas - untuk data processing
pip install pandas>=1.3.0

# OpenPyXL - untuk baca/tulis Excel files
pip install openpyxl>=3.0.0

# Requests - untuk API calls (optional, jika menggunakan API)
pip install requests>=2.26.0

# Jupyter - untuk interactive notebook
pip install jupyter>=1.0.0

# (Optional) TextBlob untuk Sentiment Analysis
pip install textblob>=0.17.0

# Install semua sekaligus
pip install pandas openpyxl requests jupyter textblob
```

### Virtual Environment Setup (Recommended)

```bash
# Create virtual environment
python -m venv venv

# Activate (Windows)
venv\Scripts\activate

# Activate (Mac/Linux)
source venv/bin/activate

# Install dependencies
pip install pandas openpyxl requests jupyter

# Deactivate environment (when done)
deactivate
```

---

## 🔍 Penjelasan Script Python

### Main Processing Flow dalam Automation_Google_Review.ipynb

```
Step 1: Load JSON Files
    ↓
Step 2: Parse & Convert to DataFrame
    ↓
Step 3: Extract Staff Mentions (Regex)
    ↓
Step 4: Calculate KPI Metrics
    ↓
Step 5: Flatten Data (1 row = 1 review-staff pair)
    ↓
Step 6: Export to Excel
    ↓
Step 7: Upload to Google Sheets
    ↓
Step 8: Create/Update Looker Studio Dashboard
```

### Contoh Output Data

**Raw DataFrame (sebelum flatten):**
```
| Date       | Rating | Reviewer    | Text                           | Mentioned_Staff       |
|------------|--------|-------------|--------------------------------|----------------------|
| 2025-12-03 | 5      | V Moritz    | Great treatment! Dr Brill...   | ['Dr. Brill']         |
| 2025-12-04 | 5      | Herman      | Staff was prof. Kathleen...    | ['Kathleen', 'Amel']  |
| 2025-12-15 | 5      | Grace Kelly | Visited Cocoon. Amel, Ria...  | ['Amel', 'Ria']       |
```

**Flattened DataFrame (siap dashboard):**
```
| Date       | Rating | Reviewer    | Text                           | Staff_Mentioned |
|------------|--------|-------------|--------------------------------|-----------------|
| 2025-12-03 | 5      | V Moritz    | Great treatment! Dr Brill...   | Dr. Brill      |
| 2025-12-04 | 5      | Herman      | Staff was prof. Kathleen...    | Kathleen        |
| 2025-12-04 | 5      | Herman      | Staff was prof. Kathleen...    | Amel            |
| 2025-12-15 | 5      | Grace Kelly | Visited Cocoon. Amel, Ria...  | Amel            |
| 2025-12-15 | 5      | Grace Kelly | Visited Cocoon. Amel, Ria...  | Ria             |
```

**KPI Summary:**
```
Staff Member    | Mention Count | Avg Rating | % of Total Reviews
Ayuk            | 9             | 5.0        | 14.5%
Kathleen        | 5             | 5.0        | 8.1%
Ria             | 5             | 5.0        | 8.1%
Weby            | 3             | 5.0        | 4.8%
... dan seterusnya
```

---

## 🎨 Dashboard Preview

**Screenshot dari Looker Studio Dashboard:**

### Layout Komponen:
1. **Top Section**: KPI Cards (Total Reviews, Avg Rating, This Month)
2. **Left Column**: 
   - Service Treatment Mentioned (Bar Chart)
   - Staff Mentioned (Leaderboard Table)
3. **Right Column**:
   - Date-based metrics
   - Rating Distribution (Pie Chart)
4. **Bottom Section**:
   - Time Series (Review trends)
   - Detailed Review Table dengan staff mentions

---

## 🐛 Troubleshooting

### Q: Error "Module not found: pandas"
**A:** Install pandas dengan `pip install pandas`

### Q: Nama staf tidak terdeteksi
**A:** 
1. Check apakah nama sudah ditambahkan di STAFF_NAMES dictionary
2. Test regex pattern dengan contoh teks
3. Tambahkan lebih banyak aliases (variasi penulisan)

### Q: Data tidak muncul di Looker Studio
**A:**
1. Pastikan data sudah di-upload ke Google Sheets
2. Check struktur kolom di Google Sheets sesuai requirement
3. Refresh data source di Looker Studio
4. Periksa permission Google Sheets (harus accessible)

### Q: Review text mengandung special characters
**A:** Gunakan encoding UTF-8 saat load JSON:
```python
import json
with open('reviews.json', 'r', encoding='utf-8') as f:
    data = json.load(f)
```

---

## 🚀 Next Steps & Improvement Ideas

### Phase 2: Advanced Features
- [ ] **Sentiment Analysis**: Implementasi NLP untuk classify positif/negatif
- [ ] **Service Category Detection**: Auto-detect jenis layanan yang dibicarakan
- [ ] **Response Analysis**: Analyze owner responses dan response rate per review
- [ ] **Multi-Location Support**: Extend untuk analyze multiple branches

### Phase 3: Machine Learning
- [ ] **Predictive Analytics**: Predict rating based on review text
- [ ] **Anomaly Detection**: Flag unusual reviews (spam/fake reviews)
- [ ] **Topic Modeling**: Identify top topics/complaints automatically
- [ ] **Recommendation Engine**: Suggest improvements based on reviews

### Phase 4: Integration & Automation
- [ ] **API Integration**: Direct Google Business API integration (replace manual export)
- [ ] **Scheduled Refresh**: Automated daily/weekly data refresh
- [ ] **Email Reports**: Automatic KPI reports sent to management
- [ ] **Slack Notifications**: Alert saat ada review dengan mention

---

## 📞 Support & Documentation

### Additional Resources
- [Google My Business API Documentation](https://developers.google.com/my-business/content/overview)
- [Pandas Documentation](https://pandas.pydata.org/docs/)
- [Looker Studio Help Center](https://support.google.com/looker-studio)
- [Python Regex Tutorial](https://docs.python.org/3/library/re.html)

### Contributing
Untuk improvement atau issue, silakan buat issue atau pull request di repository ini.

### Version History
- **v1.0** (Jan 2026): Initial release dengan staff mention extraction & basic KPI dashboard

---

---

**Happy analyzing! 📊 Semoga dashboard ini membantu mengidentifikasi top performers dan improve customer satisfaction! 🎉**
