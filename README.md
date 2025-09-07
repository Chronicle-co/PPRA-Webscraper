# **PPRA Tender Scraper**

A Python-based automation tool that scrapes tenders from the **Public Procurement Regulatory Authority (PPRA) Pakistan** website, downloads related PDFs, generates a structured Excel report, and sends it via **Gmail API**.  
It is fully automated using **GitHub Actions**.

---

## **Features**

- Scrapes tenders from **PPRA official website**.
- Downloads tender-related **PDF documents**.
- Renames PDFs using **Tender Numbers**.
- Generates a **well-formatted Excel report** using **pandas + openpyxl**.
- Detects tenders matching custom **keywords**.
- Sends **automated email** with attachments via **Gmail API**.
- Uses **GitHub Actions** to run automatically on a schedule.

---

## **Workflow Overview**

1. **Scraper Setup** → Configures Chrome in headless mode and sets download preferences.
2. **Navigate & Scrape** → Opens PPRA, selects desired sector, extracts data.
3. **PDF Downloading** → Downloads tender PDFs and renames them.
4. **Keyword Detection** → Highlights tenders matching keywords.
5. **Excel Report** → Generates a styled Excel report.
6. **Gmail API** → Sends emails with Excel + PDFs attached.
7. **Automation** → GitHub Actions runs the scraper automatically.

---

## **Gmail API Setup**

### **Step 1 — Enable Gmail API**

1. Go to [Google Cloud Console](https://console.cloud.google.com/).
2. Create a **new project** → e.g. `ppra-scraper`.
3. Navigate to **APIs & Services → Library** → Enable **Gmail API**.

### **Step 2 — Create OAuth 2.0 Credentials**

1. Go to **APIs & Services → Credentials**.
2. Click **Create Credentials** → Select **OAuth Client ID**.
3. Configure the **OAuth Consent Screen**:
   - App name → `PPRA Scraper`
   - User type → **External**
   - Test users → Add your Gmail address.
4. Choose **Desktop App** → Download `credentials.json`.

### **Step 3 — Generate Token Locally**

Run the scraper locally to generate `token.json`:

```bash
python scraper/main.py
```

### **Step 4 — Convert Credentials to Base64**

#### On Windows (PowerShell):

```powershell
certutil -encode credentials.json credentials_base64.txt
certutil -encode token.json token_base64.txt
```

#### On Linux / Mac:

```bash
base64 credentials.json > credentials_base64.txt
base64 token.json > token_base64.txt
```

### **Step 5 — Add GitHub Secrets**

Go to **GitHub → Settings → Secrets → Actions** and add:

| Secret Name        | Description                          |
| ------------------ | ------------------------------------ |
| `GMAIL_SENDER`     | Your Gmail address                   |
| `GMAIL_RECIPIENT`  | Recipient Gmail address              |
| `CREDENTIALS_JSON` | Base64 content of `credentials.json` |
| `TOKEN_JSON`       | Base64 content of `token.json`       |

---

## **GitHub Actions Setup**

### **Workflow File**

**Path:** `.github/workflows/scheduler.yml`

```yaml
name: Run PPRA Scraper

on:
  schedule:
    - cron: "0 * * * *"  # Runs every hour
  workflow_dispatch:

permissions:
  contents: write

jobs:
  run-scraper:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v3

      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: "3.10"

      - name: Install dependencies
        run: |
          pip install -r scraper/requirements.txt

      - name: Set up Chrome
        uses: browser-actions/setup-chrome@v1

      - name: Clean old downloads
        run: |
          rm -rf ppra_pdfs
          mkdir -p ppra_pdfs
          find . -name "*.xlsx" -type f -delete

      - name: Decode Gmail Credentials
        run: |
          echo "${{ secrets.CREDENTIALS_JSON }}" | base64 --decode > scraper/credentials.json
          echo "${{ secrets.TOKEN_JSON }}" | base64 --decode > scraper/token.json

      - name: Run Scraper
        env:
          GMAIL_SENDER: ${{ secrets.GMAIL_SENDER }}
          GMAIL_RECIPIENT: ${{ secrets.GMAIL_RECIPIENT }}
        run: |
          python3 scraper/main.py

      - name: Debug Cron
        run: echo "Workflow ran at $(date)"
```

---

## **Output**

- **Excel Report** → `ppra_info_comm_tech.xlsx`
- **PDFs** → Stored in `ppra_pdfs/`
- **Email** → Automatically sent with attachments

---

## **Troubleshooting**

| Problem                      | Solution                                         |
| ---------------------------- | ------------------------------------------------ |
| `credentials.json not found` | Ensure secrets are added and decoded correctly   |
| `token.json not found`       | Add Base64 token to secrets                      |
| Gmail API auth errors        | Delete `token.json` and rerun locally            |
| Selenium can't open site     | Use retries + DNS fallback (already implemented) |

---




## **Run Automatically via GitHub Actions**

1. Add Gmail API secrets.
2. Push your code to GitHub.
3. Workflow runs **hourly** (or manually via `workflow_dispatch`).


NOTE: Change github in github/workflows/ to .github to make yml files appear in Actions and run automatically.
      Also remember to add `GMAIL_SENDER`, `GMAIL_RECIPIENT` ,`CREDENTIALS_JSON` ,`TOKEN_JSON` in secrets in Settings or code will not work.
      Also make repo private if in use (added `CREDENTIALS_JSON` and `TOKEN_JSON` ) as it might be a security risk.
      Keywords can be changed in main.py's main function.
      
