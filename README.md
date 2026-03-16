# 🏛️ Parliamentary Intelligence Agent
### *All Protocol Observed*

> An always-on AI correspondent that monitors South African parliamentary activity and surfaces newsworthy developments to journalists in near-real time.

**MIT 808 · University of Pretoria · 2026**
**Team:** NeuroAvengers — Thabiso Msimango (`u25738497`) & Lungisani Khanyile (`u25743695`)
**Client:** Athandiwe Saba (investigative journalist, former *Mail & Guardian* editor)

---

## 📌 Project Overview

South African newsrooms have contracted sharply. Where 30 journalists once covered Parliament, as few as 2–3 now do. The Parliamentary Intelligence Agent (PIA) responds to this accountability gap by monitoring three high-value parliamentary data streams and alerting journalists when something newsworthy happens — so no significant development goes unreported.

The system ingests data from the [Parliamentary Monitoring Group (PMG)](https://pmg.org.za) REST API and processes it through an NLP pipeline to power an alerting engine and journalist-facing dashboard.

---

## 🗂️ Repository Structure

```
parliamentary-intelligence-agent/
│
├── data/                              # Local data store (gitignored)
│   ├── committee_meetings_filtered.csv
│   ├── active_bills_filtered.csv
│   └── questions_replies_filtered.csv
│
├── notebooks/
│   └── MIT808_NeuroAvengers_EDA.ipynb # Assignment 1 EDA notebook
│
├── reports/
│   └── EDA_Report_NeuroAvengers.pdf   # Assignment 1 EDA report (KDD format)
│
├── src/                               # Sprint 2+ pipeline code (coming)
│   ├── scraper/
│   ├── nlp/
│   ├── alerting/
│   └── dashboard/
│
├── .gitignore
└── README.md
```

> ⚠️ The `data/` folder is gitignored. Raw and processed CSV files contain parliamentary data and must not be committed to the repository.

---

## 📊 Data Sources

All data is fetched from the **PMG REST API** at `api.pmg.org.za`. The API is freely accessible for most content without authentication.

| Stream | Endpoint | Records | Date Range |
|--------|----------|---------|------------|
| Questions & Replies | `/committee-question/` | 5,000 | Jun 2025 – Mar 2026 |
| Committee Meetings | `/committee-meeting/` | 576 | Jan 2024 – Mar 2026 |
| Active Bills | `/bill/` | 119 | Jan 2024 – Mar 2026 |

### Target Committees
Health · Finance · Education · Justice · Police · Energy · Agriculture · Trade & Industry · **Public Accounts (SCOPA)** · Cooperative Governance

### Known API Limitations
- Combining `filter[committee_id]` and `filter[date__gte]` on `/committee-meeting/` triggers HTTP 500 — **date filtering must be done client-side in pandas**
- `/bill-status/` lookup endpoint returns empty in the free tier — bill stages are parsed inline from the `status` dict
- 21% of committee meeting records have `premium_content_excluded=True` — full body text requires a PMG authentication token (to be facilitated by the client)
- Q&R reply status (answered vs. unanswered) is unavailable in the free tier

---

## 🔑 Authentication

Some content requires a PMG user token:

```python
# In notebooks/MIT808_NeuroAvengers_EDA.ipynb — Cell 05
PMG_TOKEN = ""  # Paste token from api.pmg.org.za/user/ here
```

To obtain a token:
1. Register at [pmg.org.za](https://pmg.org.za)
2. Visit `https://api.pmg.org.za/user/` while logged in
3. Copy the `token` field from the JSON response
4. Paste into `PMG_TOKEN` in the notebook config cell

PMG credentials for university access are being facilitated by Athandiwe Saba.

---

## 🚀 Running the EDA Notebook

### On Google Colab (recommended)
1. Open [colab.research.google.com](https://colab.research.google.com)
2. Upload `notebooks/MIT808_NeuroAvengers_EDA.ipynb`
3. Run **Runtime → Run all**
4. The first cell installs all dependencies automatically

### Locally
```bash
pip install requests pandas numpy matplotlib seaborn scikit-learn pdfplumber pytesseract beautifulsoup4
jupyter notebook notebooks/MIT808_NeuroAvengers_EDA.ipynb
```

### Notebook Structure

| Section | Description |
|---------|-------------|
| **0 — Setup** | Install dependencies |
| **1A — Committees** | Fetch committee list, inspect real column names |
| **1B — Meetings** | Fetch meetings for all 10 target committees |
| **1C — Bills** | Fetch all bills, filter locally |
| **1D — Q&R** | Fetch questions via `/committee-question/` |
| **1E — Normalise** | Strip timezones, flatten nested dicts, extract ministry, clean stages |
| **2 — Confirm** | Assert all dataframes are populated; print shapes and quality metrics |
| **3 — Inventory** | Data stream inventory table + temporal coverage figures |
| **4 — Q&R Analysis** | Ministry distribution, word count, monthly volume |
| **5 — Meetings Analysis** | Per-committee counts, heatmap, content availability |
| **6 — Bills Analysis** | Stage distribution, bill age distribution |
| **7 — Topic Analysis** | TF-IDF keyword analysis across all three streams |
| **8 — Data Quality** | Missing values, premium content rate, hash-based change detection |
| **9 — Key Findings** | F1–F8 summary findings computed live from real data |

---

## 📋 Key EDA Findings

| # | Finding |
|---|---------|
| **F1** | 5,000 Q&Rs · 576 committee meetings · 119 bills — all real, current to March 2026 |
| **F2** | SCOPA leads with 105 meetings — highest-priority alerting target confirmed |
| **F3** | 21% of meeting body text is paywalled — PMG credentials will unlock this |
| **F4** | 30 bills under active NA consideration; 45% lack stage metadata |
| **F5** | 100% ministry attribution across Q&Rs; Sport, Arts & Culture leads (353 questions) |
| **F6** | Q&R reply status unavailable in free tier — highest-priority authentication need |
| **F7** | TF-IDF: Bills cluster around "appropriation/revenue/amendment"; Meetings around "audit/performance/energy" |
| **F8** | PMG token needed for: 21% meeting body text, Q&R reply status, bill stage lookup |

---

## 🛠️ System Architecture (Planned — Sprint 2+)

```
PMG API ──► Scraper ──► Raw Store
                           │
                    ┌──────▼──────┐
                    │  NLP Pipeline│
                    │  - spaCy NER │
                    │  - BART/T5   │
                    │  - TF-IDF    │
                    └──────┬──────┘
                           │
                    ┌──────▼──────────┐
                    │  Alerting Engine │
                    │  8 alert types   │
                    └──────┬──────────┘
                           │
                    ┌──────▼──────┐
                    │  Dashboard   │
                    │  (Streamlit) │
                    └─────────────┘
```

### 8 Alert Categories
1. Bill stage changes
2. Public comment periods opening
3. Ministerial reply publications
4. Committee outcomes and decisions
5. SCOPA investigations
6. Significant financial figures / procurement
7. Custom keyword matches (journalist-configurable)
8. Unanswered questions exceeding threshold

---

## 📅 Project Timeline

| Sprint | Dates | Deliverable |
|--------|-------|-------------|
| 1 | Feb–Mar 2026 | ✅ Project scoping + EDA (Assignment 1) |
| 2 | Mar–Apr 2026 | Data ingestion pipeline + NLP processing |
| 3 | Apr 2026 | Alerting engine (8 categories) |
| 4 | Apr–May 2026 | Streamlit dashboard |
| 5 | May 2026 | Pilot with client + evaluation |
| 6 | May–Jun 2026 | Final report + handover |

Client check-ins: fortnightly with Athandiwe Saba.

---

## 🔬 Technical Stack

| Layer | Technology |
|-------|-----------|
| Data collection | Python `requests`, PMG REST API |
| Data processing | `pandas`, `numpy` |
| NLP | `spaCy` (NER), `transformers` (BART/T5, FinBERT), `scikit-learn` (TF-IDF) |
| PDF extraction | `pdfplumber`, `pytesseract` (OCR fallback) |
| Change detection | MD5 hashing per document |
| Visualisation | `matplotlib`, `seaborn` |
| Dashboard | Streamlit (planned) |
| Scheduling | APScheduler / cron (planned) |

---

## 📄 Reports & Deliverables

| Item | Status | Location |
|------|--------|----------|
| Scoping worksheet | ✅ Submitted | `MIT_808_Scoping_Worksheet_NeuroAvengers.docx` |
| EDA notebook | ✅ Submitted | `notebooks/MIT808_NeuroAvengers_EDA.ipynb` |
| EDA report (KDD) | ✅ Submitted | `reports/EDA_Report_NeuroAvengers.pdf` |

---

## 👥 Team

| Name | Student # | Role |
|------|-----------|------|
| Thabiso Msimango | u25738497 | Developer |
| Lungisani Khanyile | u25743695 | Developer |

---

## 📜 Licence & Data Ethics

- All parliamentary data is sourced from PMG under their public access policy
- No personally identifiable information beyond publicly available parliamentary records is collected or stored
- Raw data is excluded from version control per MIT 808 data handling guidelines
- This system is designed to support journalistic accountability, not to facilitate surveillance or harassment

---

*Parliamentary Monitoring Group: "PMG aims to provide accurate, objective, and current information on all parliamentary committee proceedings."*
*[pmg.org.za](https://pmg.org.za) · [api.pmg.org.za](https://api.pmg.org.za)*
