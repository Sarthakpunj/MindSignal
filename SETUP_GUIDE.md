# How to Upload to GitHub

## Option A: GitHub Web UI (Easiest)

1. Go to [github.com/new](https://github.com/new)
2. Name the repo: `MindSignal`
3. Set to **Public** (or Private if you prefer)
4. Check **"Add a README file"** — NO (we have our own)
5. Click **Create repository**
6. On the repo page, click **"uploading an existing file"**
7. Drag and drop ALL files from this folder maintaining the structure
8. Commit with message: `Initial commit — MindSignal PoC`

## Option B: Git CLI

```bash
# 1. Navigate to this folder
cd MindSignal

# 2. Initialise git
git init
git add .
git commit -m "Initial commit — MindSignal PoC"

# 3. Create repo on GitHub, then:
git remote add origin https://github.com/YOUR_USERNAME/MindSignal.git
git branch -M main
git push -u origin main
```

## Your Notebooks

**Important:** Your two Colab notebooks need to be saved as `.ipynb` files.

In Google Colab:
1. Open each notebook
2. Go to **File → Download → Download .ipynb**
3. Rename them:
   - Full pipeline → `1_multimodal_pipeline.ipynb`
   - RAVDESS classifier → `2_ravdess_classifier.ipynb`
4. Place them in the `notebooks/` folder

## Final Folder Structure

After uploading, your repo should look like:

```
MindSignal/
├── README.md
├── LICENSE
├── .gitignore
├── SETUP_GUIDE.md
├── notebooks/
│   ├── 1_multimodal_pipeline.ipynb
│   └── 2_ravdess_classifier.ipynb
├── docs/
│   ├── MindSignal_Whitepaper.docx
│   └── MindSignal_BusinessPlan.docx
└── assets/
    └── architecture.png
```

## Optional: Add Colab Badges

After pushing to GitHub, update the Colab badge links in README.md to point to your actual notebooks:

```
https://colab.research.google.com/github/YOUR_USERNAME/MindSignal/blob/main/notebooks/1_multimodal_pipeline.ipynb
```
