<div align="center">

<img src="https://readme-typing-svg.demolab.com?font=Orbitron&weight=900&size=42&duration=3000&pause=1000&color=00FFFF&center=true&vCenter=true&width=700&height=80&lines=SANDI+RIDWAN" alt="Sandi Ridwan" />

<img src="https://readme-typing-svg.demolab.com?font=Fira+Code&size=18&duration=2000&pause=500&color=00FF88&center=true&vCenter=true&width=700&lines=Senior+Scraping+Engineer;Data+Extraction+Architect;Web+Intelligence+Specialist" alt="titles" />

<br/>

[![Python](https://img.shields.io/badge/Python-3.10+-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://python.org)
[![Playwright](https://img.shields.io/badge/Playwright-Latest-45BA4B?style=for-the-badge&logo=playwright&logoColor=white)](https://playwright.dev)
[![Pandas](https://img.shields.io/badge/Pandas-Data_Engineering-150458?style=for-the-badge&logo=pandas&logoColor=white)](https://pandas.pydata.org)
[![Status](https://img.shields.io/badge/Status-Production_Ready-00FF88?style=for-the-badge)](.)
[![Records](https://img.shields.io/badge/Records_Extracted-293+-FF6B35?style=for-the-badge)](.)

</div>

---

<div align="center">

```
╔══════════════════════════════════════════════════════════════════╗
║                                                                  ║
║   ██╗   ██╗███████╗██████╗ ███╗   ███╗ ██████╗ ███╗   ██╗████████╗  ║
║   ██║   ██║██╔════╝██╔══██╗████╗ ████║██╔═══██╗████╗  ██║╚══██╔══╝  ║
║   ██║   ██║█████╗  ██████╔╝██╔████╔██║██║   ██║██╔██╗ ██║   ██║     ║
║   ╚██╗ ██╔╝██╔══╝  ██╔══██╗██║╚██╔╝██║██║   ██║██║╚██╗██║   ██║     ║
║    ╚████╔╝ ███████╗██║  ██║██║ ╚═╝ ██║╚██████╔╝██║ ╚████║   ██║     ║
║     ╚═══╝  ╚══════╝╚═╝  ╚═╝╚═╝     ╚═╝ ╚═════╝ ╚═╝  ╚═══╝   ╚═╝     ║
║                                                                  ║
║              V E T E R I N A R I A N S   D A T A               ║
║                    Vermont, USA — 293+ Records                  ║
╚══════════════════════════════════════════════════════════════════╝
```

</div>

---

## 📌 Overview

**Vermont Veterinarians Data Pipeline** adalah proyek end-to-end data extraction yang mengumpulkan, membersihkan, dan menggabungkan data veterinarian publik dari State of Vermont. Project ini mendemonstrasikan kemampuan reverse engineering API tersembunyi, multi-source data fusion, dan intelligent deduplication.

> 🎯 **Goal**: Mengumpulkan 400+ records veterinarian Vermont lengkap dengan nama, phone, alamat, email, dan website dari sumber-sumber publik.

---

## 🏗️ Architecture & Pipeline

```
┌─────────────────────────────────────────────────────────────────┐
│                    DATA PIPELINE OVERVIEW                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  SOURCE 1                    SOURCE 2                          │
│  ┌──────────────────┐        ┌──────────────────┐              │
│  │  VVMA Directory  │        │  Google Maps     │              │
│  │  vtvets.org      │        │  44 City Queries │              │
│  │                  │        │                  │              │
│  │  Angular SPA     │        │  Playwright +    │              │
│  │  MemberClicks    │        │  Stealth Plugin  │              │
│  │  API Intercept   │        │  Auto Scroll     │              │
│  │                  │        │                  │              │
│  │  236 records     │        │  ~300 records    │              │
│  └────────┬─────────┘        └────────┬─────────┘              │
│           │                           │                        │
│           └──────────┬────────────────┘                        │
│                      ▼                                          │
│           ┌─────────────────────┐                              │
│           │   MERGE ENGINE      │                              │
│           │                     │                              │
│           │  • Dedup by phone   │                              │
│           │  • Score by fields  │                              │
│           │  • VT filter        │                              │
│           │  • Name parser      │                              │
│           └──────────┬──────────┘                              │
│                      ▼                                          │
│           ┌─────────────────────┐                              │
│           │   OUTPUT            │                              │
│           │  293 unique records │                              │
│           │  CSV + Excel        │                              │
│           └─────────────────────┘                              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🔬 Technical Deep-Dive: 7-Variable Audit

Sebelum satu baris kode ditulis, dilakukan audit mendalam terhadap setiap target:

| Variable | VVMA (vtvets.org) | Google Maps |
|---|---|---|
| **Rendering** | CSR — Angular SPA | CSR — React |
| **Pagination** | MemberClicks API, 24 pages × 10 | Infinite scroll |
| **Anti-Bot** | Session-based, CORS restricted | Rate limiting |
| **Auth** | Public directory, no login | Public search |
| **Hidden API** | ✅ `/ui-directory-search/v2/` | ❌ No JSON API |
| **Stack** | Playwright intercept | Playwright + stealth |
| **Scale** | 236 records | ~300 records |

---

## 🕵️ VVMA API Reverse Engineering

### Discovery

Target `vtvets.org/find-a-vet-search` adalah Angular SPA berbasis **MemberClicks CMS**. Static `requests` hanya mengembalikan shell HTML kosong:

```html
<mc-root>Loading...</mc-root>
<script src="/ui-directory-search/v2/public-v2/dist/main.js"></script>
```

### API Endpoints Ditemukan

```
POST https://vvma.memberclicks.net/ui-directory-search/v2/search-directory/
GET  http://service-router.prod01.memberclicks.io/search-results/v2/results/{search_id}
```

### Request Payload

```json
{
  "directory_search_id": 2002471,
  "page_size": 50,
  "search_terms": []
}
```

### Response Structure

```json
{
  "total_count": 236,
  "total_page_count": 24,
  "search_id": "uuid-v4",
  "data_url": "http://service-router.prod01.memberclicks.io/...",
  "results": [
    {
      "title": "Dr. Rebecca Allen",
      "right": [{"html": "<strong>Phone</strong> (802) 922-7754"}],
      "left":  [{"html": "<strong>City/Town</strong> Vergennes"}],
      "top":   [{"html": "<strong>Address</strong> 2204 Jersey St."}]
    }
  ]
}
```

### Why Requests Failed — Why Playwright Works

```
❌ requests.get(service-router.io)  → DNS resolution OK, but CORS blocked
❌ page.evaluate("fetch(url)")      → Cross-origin CORS blocked by browser
✅ page.on("response", handler)     → Intercept Angular's OWN requests → no CORS
```

---

## 📁 Project Structure

```
vermont-vets-scraper/
│
├── 📄 starter_v2.py              # Google Maps scraper (Playwright + Stealth)
│   ├── 44 city queries coverage
│   ├── Auto-scroll infinite pagination
│   ├── Email extraction from websites
│   └── Auto-backup every 5 queries
│
├── 📄 vvma_scraper_v4.py         # VVMA API interceptor (FINAL)
│   ├── Playwright response interceptor
│   ├── Angular pagination trigger
│   ├── GMaps data cleaner
│   └── Smart merge + dedup engine
│
├── 📊 vermont_vets_comprehensive.csv    # Raw Google Maps output
├── 📊 vermont_vets_VVMA.csv            # Raw VVMA output
├── 📊 vermont_vets_merged_final.xlsx   # Final merged output ✅
└── 📄 README.md
```

---

## 🚀 Quick Start

### Prerequisites

```bash
pip install playwright pandas beautifulsoup4 openpyxl
playwright install chromium
```

### Step 1 — Google Maps Scrape (Optional, sudah ada hasilnya)

```bash
pip install playwright-stealth
python starter_v2.py
# Runtime: ~2-4 jam | Output: vermont_vets_comprehensive.csv
```

### Step 2 — VVMA Scrape + Merge

```bash
python vvma_scraper_v4.py
# Runtime: ~5-8 menit | Output: vermont_vets_merged_final.xlsx
```

---

## 📊 Results

<div align="center">

| Metric | Value |
|:---|:---:|
| 📋 Total Unique Records | **293** |
| 📞 Records with Phone | **267** (91%) |
| 📧 Records with Email | **99** (34%) |
| 🌐 Records with Website | **180** (61%) |
| 👤 Records with Doctor Name | **150** (51%) |
| 🔄 Dedup Rate | ~25% |
| ⏱️ Total Runtime | ~3 hours |

</div>

### Output Fields

```
clinic_name | first_name | last_name | email | phone | address | website | source_query
```

---

## 🔧 Key Technical Decisions

### 1. Playwright over Requests

`requests` tidak bisa hit `service-router.prod01.memberclicks.io` karena:
- Host internal MemberClicks infrastructure
- Session token bersifat ephemeral dan terikat browser context
- CORS policy memblokir cross-origin fetch

**Solution**: `page.on("response")` intercepts responses dari Angular app sendiri — tidak ada external request, tidak ada CORS issue.

### 2. Smart Deduplication

```python
# Priority: records dengan lebih banyak data menang
score = email_filled * 3 + phone_filled * 2 + website_filled * 1

# Dedup strategy:
# - Has phone → dedup by 10-digit phone number
# - No phone  → dedup by clinic name prefix (20 chars)
```

### 3. Name Parser

```python
# Handle: "Dr. John Smith DVM, PhD, DACVIM"
# Output: first="John", last="Smith"
name = re.sub(r"^(Dr\.|Dr)\s*", "", title)
name = re.sub(r"\b(DVM|VMD|PhD|MS|DACVIM|...)\b", "", name)
```

---

## ⚠️ Legal & Ethical Notes

- ✅ Semua data bersumber dari **direktori publik** yang dapat diakses tanpa autentikasi
- ✅ Tidak ada bypass login, CAPTCHA solving, atau akses ke data private
- ✅ Rate limiting diterapkan (0.8–2.8 detik antar request)
- ✅ Data digunakan untuk keperluan **lead generation B2B yang sah**
- ✅ Sesuai dengan `robots.txt` masing-masing situs

---

## 🛠️ Skills Demonstrated

```
✦ Web Scraping Architecture    ✦ API Reverse Engineering
✦ Playwright Browser Automation ✦ Cross-Origin Problem Solving
✦ Data Pipeline Engineering    ✦ Multi-Source Data Fusion
✦ Deduplication Algorithms     ✦ Regex & Text Parsing
✦ Python Data Engineering      ✦ Excel/CSV Export Automation
```

---

<div align="center">

**Built with 🔥 by [Sandi Ridwan](https://github.com/sandiridwan)**

*"Don't scrape what you can't reverse engineer."*

<img src="https://readme-typing-svg.demolab.com?font=Fira+Code&size=14&duration=3000&pause=1000&color=00FF88&center=true&vCenter=true&width=500&lines=Data+is+the+new+oil.+Extract+it+right.;Built+for+scale.+Engineered+for+precision." alt="footer" />

</div>
