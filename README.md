# Wedding Seating App

Guest-facing wedding lookup site with QR code + Admin dashboard for bulk Excel uploads, edit/delete, and download.

## Run locally

```bash
pip install -r requirements.txt
# Terminal 1 (guest site)
python app.py
# Terminal 2 (admin site)
python admin.py
```

- Guest site: http://127.0.0.1:5000/
- QR page:    http://127.0.0.1:5000/qr
- Admin site: http://127.0.0.1:5001/

## Data format
Excel must include a header row: `FirstName, LastName, Nickname, TableNumber`.


## Deploy to Render (no Docker needed)
1. Push this folder to a GitHub repo (e.g., `wedding-seating-app`).
2. On https://render.com, click **New +** → **Blueprint** and select your repo.
3. Render will detect `render.yaml` and create **two web services** (guest + admin).
4. Click **Apply**. After build, you'll get two URLs, e.g.:
   - Guest: https://wedding-guest-site.onrender.com/
   - Admin: https://wedding-admin-site.onrender.com/
5. Upload your `data.xlsx` via the Admin site or commit it to the repo.

## Deploy with Docker (optional)
Build and run locally:
```bash
docker build -f Dockerfile.guest -t wedding-guest .
docker run -p 5000:5000 -v %cd%/data.xlsx:/app/data.xlsx wedding-guest

docker build -f Dockerfile.admin -t wedding-admin .
docker run -p 5001:5001 -v %cd%/data.xlsx:/app/data.xlsx wedding-admin
```


## Shared data with S3
Both services read/write a single Excel file stored in **Amazon S3**.

### Required environment variables
- `S3_BUCKET` — your S3 bucket name
- `S3_KEY` — object key (e.g., `data.xlsx`)
- `AWS_REGION` — e.g., `us-east-1`
- `AWS_ACCESS_KEY_ID` / `AWS_SECRET_ACCESS_KEY` — IAM user/role with `s3:GetObject` & `s3:PutObject` on that key

On Render: add these in the **Environment** section for both services (the `render.yaml` already defines placeholders).

## Admin Basic Auth
Set:
- `ADMIN_USER`
- `ADMIN_PASS`

Access the admin site and your browser will prompt for these credentials.

## Notes
- Locally, if `S3_BUCKET` is not set, the apps fall back to the local `data.xlsx` for convenience.
- In production, set the S3 variables so both guest & admin read the same Excel file.


### Header validation
- Uploads must include an exact header row: `FirstName, LastName, Nickname, TableNumber` (case-insensitive match).
- Rows must include all four values; otherwise the upload is rejected with a 400 error.
