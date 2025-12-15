# GitHub Actions Setup Guide for TikTok to YouTube Uploader

## 📋 Step-by-Step Setup

### 1. Create a GitHub Repository

1. Go to [GitHub](https://github.com) and create a new repository
2. Name it something like `tiktok-youtube-automation`
3. Make it **Private** (important for security)
4. Initialize with a README

### 2. Add Your Files to Repository

Upload these files to your repository:
- `tiktok_uploader.py` (the main Python script)
- Create folder `.github/workflows/`
- Add `tiktok-to-youtube.yml` (the workflow file) inside `.github/workflows/`

**Folder structure:**
```
tiktok-youtube-automation/
├── .github/
│   └── workflows/
│       └── tiktok-to-youtube.yml
├── tiktok_uploader.py
├── uploaded_videos.json (will be created automatically)
└── README.md
```

### 3. Prepare Your Credentials

#### A. Convert credentials.json to base64
Download your `credentials.json` from the Google Drive link, then:

**On Windows (PowerShell):**
```powershell
$content = Get-Content credentials.json -Raw
[Convert]::ToBase64String([Text.Encoding]::UTF8.GetBytes($content))
```

**On Linux/Mac:**
```bash
cat credentials.json | base64
```

Copy the output (long string).

#### B. Convert token.pickle to base64
Download your `token.pickle` from Google Drive, then:

**On Windows (PowerShell):**
```powershell
[Convert]::ToBase64String([IO.File]::ReadAllBytes("token.pickle"))
```

**On Linux/Mac:**
```bash
base64 token.pickle
```

Copy the output.

### 4. Add GitHub Secrets

Go to your repository → **Settings** → **Secrets and variables** → **Actions** → **New repository secret**

Add these secrets:

1. **GOOGLE_CREDENTIALS**
   - Value: The entire JSON content from `credentials.json` (NOT base64, just the raw JSON)

2. **GOOGLE_TOKEN**
   - Value: The base64 encoded content from `token.pickle`

3. **TIKTOK_USERNAME**
   - Value: `huongkimhoangngoc` (or your TikTok username)

4. **GH_PAT** (GitHub Personal Access Token)
   - Go to GitHub Settings → Developer settings → Personal access tokens → Tokens (classic)
   - Click "Generate new token (classic)"
   - Give it a name like "TikTok Uploader"
   - Select scopes: `repo` (full control)
   - Generate and copy the token
   - Add it as a secret named `GH_PAT`

### 5. Update Python Script for GitHub Actions

Modify your `tiktok_uploader.py` to read username from environment:

```python
# At the top, change this line:
TIKTOK_USERNAME = os.getenv('TIKTOK_USERNAME', 'your_tiktok_username')
```

Also change the main() call to upload one video per run:

```python
if __name__ == "__main__":
    # For GitHub Actions - upload one video per day
    main(upload_all=False)
```

### 6. Test the Workflow

1. Go to your repository → **Actions** tab
2. Click on "TikTok to YouTube Uploader" workflow
3. Click **Run workflow** → **Run workflow** (manual trigger)
4. Watch it run and check for any errors

### 7. Workflow Schedule

The workflow is set to run daily at **9:00 AM UTC**. To change the time:

Edit the cron expression in `.github/workflows/tiktok-to-youtube.yml`:
```yaml
- cron: '0 9 * * *'  # Change these numbers
```

**Cron format:** `minute hour day month day_of_week`

Examples:
- `0 9 * * *` = 9:00 AM UTC daily
- `0 0 * * *` = Midnight UTC daily
- `0 */6 * * *` = Every 6 hours
- `30 14 * * *` = 2:30 PM UTC daily

**Convert UTC to your timezone:**
- UTC 9:00 AM = 2:30 PM IST (India)
- UTC 0:00 AM = 5:30 AM IST

### 8. Monitor Your Workflow

- Check **Actions** tab to see workflow runs
- View logs for each run
- Check `uploaded_videos.json` in your repo to see progress

## ⚠️ Important Notes

1. **YouTube API Quota**: Free tier has limits (~10-50 uploads/day)
2. **GitHub Actions Minutes**: Free tier has 2,000 minutes/month
3. **Keep Repository Private**: Don't expose your credentials
4. **Backup uploaded_videos.json**: This tracks your progress

## 🔧 Troubleshooting

**If workflow fails:**
1. Check Actions logs for error messages
2. Verify all secrets are correctly added
3. Make sure credentials.json and token.pickle are valid
4. Check YouTube API quota in Google Cloud Console

**If authentication fails:**
- Re-download credentials and token from Google Drive
- Make sure base64 encoding is correct
- Verify secrets are properly set in GitHub

## 📊 What Happens

1. GitHub Actions runs daily at scheduled time
2. Downloads one TikTok video
3. Uploads to YouTube
4. Updates `uploaded_videos.json` with progress
5. Commits progress back to repository
6. Repeats next day with the next video

Your videos will be uploaded automatically, one per day! 🎉
