# Sweet Potato Leaf Disease Dataset

312 HEIC images manually classified into four plant disease categories using visual inspection in Replit.

---

## Folder Structure

```
Dataset/
├── Cercospora_Leaf_Spot/   (124 images)
├── Fusarium_Wilt/          (43 images)
├── Healthy/                (104 images)
├── Leaf_Curl_Virus/        (41 images)
├── unidentified/           (0 images)
└── picture to segregate/   (312 original source images, all intact)
```

All originals are preserved in `picture to segregate/`. The category folders contain **copies** made with `cp` (not `mv`).

---

## Classification Criteria

| Category | Visual Signs |
|---|---|
| **Cercospora Leaf Spot** | White/light circular spots, yellow halos, dark lesions, holes where lesions have dropped out |
| **Fusarium Wilt** | Severely wilted, yellow-brown or orange, dying/collapsed leaves, total desiccation |
| **Healthy** | Clean flat green leaves, no disease symptoms, minor insect holes are not disqualifying |
| **Leaf Curl Virus** | Curled/twisted/distorted/crumpled leaves, dark green crinkled texture, reddish-brown vein streaks |

---

## How This Was Done (Step-by-Step)

### Tools Used
- **Replit** — cloud workspace where all commands were run
- **ImageMagick** — converts HEIC photos to JPEG thumbnails for visual inspection
- **bash** — all file operations (copy, list, count)
- **GitHub API (curl)** — used to clean up files directly on GitHub

---

### Step 1 — Install ImageMagick (if not already available)

```bash
which convert || nix-env -iA nixpkgs.imagemagick
```

---

### Step 2 — Convert HEIC images to JPEG thumbnails for viewing

```bash
mkdir -p /tmp/thumbs
for f in "picture to segregate/"*.HEIC; do
  name=$(basename "$f" .HEIC)
  convert "$f" -resize 400x400 "/tmp/thumbs/${name}.jpg"
done
```

To convert a specific range (e.g. 6904 to 6923):

```bash
for f in "picture to segregate/IMG_E69"{04..23}".HEIC"; do
  name=$(basename "$f" .HEIC)
  convert "$f" -resize 400x400 "/tmp/thumbs/${name}.jpg"
done
```

---

### Step 3 — View thumbnails and classify

In Replit, use the **read tool** (or any image viewer) to open `/tmp/thumbs/IMG_EXXXX.jpg` and visually inspect each leaf.

Determine the category based on the criteria table above, then copy the original:

```bash
cp "picture to segregate/IMG_EXXXX.HEIC" Cercospora_Leaf_Spot/
cp "picture to segregate/IMG_EXXXX.HEIC" Fusarium_Wilt/
cp "picture to segregate/IMG_EXXXX.HEIC" Healthy/
cp "picture to segregate/IMG_EXXXX.HEIC" Leaf_Curl_Virus/
```

---

### Step 4 — Check counts at any time

```bash
echo "Cercospora: $(ls Cercospora_Leaf_Spot/ | wc -l)"
echo "Fusarium:   $(ls Fusarium_Wilt/ | wc -l)"
echo "Healthy:    $(ls Healthy/ | wc -l)"
echo "Leaf_Curl:  $(ls Leaf_Curl_Virus/ | wc -l)"
echo "Unidentified: $(ls unidentified/ | wc -l)"
echo "Source total: $(ls 'picture to segregate/' | wc -l)"
```

---

### Step 5 — Check for unclassified images

Find any source images not yet copied to any category folder:

```bash
for f in "picture to segregate/"*.HEIC; do
  name=$(basename "$f")
  found=0
  for dir in Cercospora_Leaf_Spot Fusarium_Wilt Healthy Leaf_Curl_Virus unidentified; do
    [ -f "$dir/$name" ] && found=1 && break
  done
  [ $found -eq 0 ] && echo "UNCLASSIFIED: $name"
done
```

---

### Step 6 — Push to GitHub

First, set your GitHub Personal Access Token as a Replit secret named `GITHUB_PERSONAL_ACCESS_TOKEN`.

Then push:

```bash
git add -A
git commit -m "Classify all images"
git push "https://YOUR_USERNAME:${GITHUB_PERSONAL_ACCESS_TOKEN}@github.com/YOUR_USERNAME/Dataset.git" main
```

---

### Step 7 — Remove unwanted files from GitHub (via API)

To delete a file directly on GitHub without needing git commit (e.g. `.replit`, `.gitkeep`):

```bash
SHA=$(curl -s -H "Authorization: token $GITHUB_PERSONAL_ACCESS_TOKEN" \
  "https://api.github.com/repos/YOUR_USERNAME/Dataset/contents/PATH/TO/FILE?ref=main" | \
  python3 -c "import sys,json; print(json.load(sys.stdin)['sha'])")

curl -s -X DELETE \
  -H "Authorization: token $GITHUB_PERSONAL_ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  "https://api.github.com/repos/YOUR_USERNAME/Dataset/contents/PATH/TO/FILE" \
  -d "{\"message\":\"Remove file\",\"sha\":\"$SHA\",\"branch\":\"main\"}"
```

---

## Final Dataset Counts

| Category | Count |
|---|---|
| Cercospora Leaf Spot | 124 |
| Fusarium Wilt | 43 |
| Healthy | 104 |
| Leaf Curl Virus | 41 |
| **Total** | **312** |
