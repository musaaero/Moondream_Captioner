# 📸 Resumable Image Captioning with Moondream 2B

A Google Colab notebook that generates **detailed, natural-language descriptions** for every image in a folder — and saves them to a CSV file. Built for the free **Colab T4 GPU**, it's designed to be **crash-proof**: if the runtime dies halfway through, just re-run the cell and it picks up exactly where it left off.

---

## ✨ Features

- **🖼️ Recursive folder scanning** — finds all images in a folder and its subfolders (`.jpg`, `.jpeg`, `.png`, `.webp`, `.bmp`, `.gif`).
- **📄 Simple CSV output** — two columns only: `filename`, `description`.
- **🔁 Fully resumable** — the CSV is saved after *every single image*. If Colab crashes, just re-run and it continues from the first unfinished row.
- **➕ Auto-detects new files** — re-running with a folder that has new images will add them to the CSV automatically.
- **🎛️ GUI form** — enter your folder path, model choice, and prompt in a clean Colab form (no code editing needed).
- **⚡ Fast & lightweight** — uses Moondream 2B in `float16`, which fits comfortably in the 16 GB VRAM of a free T4 GPU.
- **🤖 Two model sizes** — choose Moondream 2B for quality or the lighter variant for speed.

---

## 📋 Requirements

- A Google account (for Google Drive + Colab)
- A **free Colab T4 GPU** runtime (`Runtime → Change runtime type → T4 GPU`)
- Your images uploaded to **Google Drive**

---

## 🚀 Quick Start

### 1. Open the notebook in Colab

Upload `image_captioning.ipynb` to [Google Colab](https://colab.research.google.com/), or open it directly from GitHub.

### 2. Set the runtime to GPU

`Runtime → Change runtime type → Hardware accelerator: T4 GPU → Save`

### 3. Run **Cell 1** (Setup)

```python
!pip install -q "transformers>=4.45.0,<5.0.0" accelerate einops pillow pandas
from google.colab import drive
drive.mount('/content/drive')
```

This installs dependencies, pins a compatible `transformers` version, and mounts your Drive.

> ⚠️ **You may see pip dependency warnings** about `gradio`, `diffusers`, and `huggingface-hub`. These are **harmless** and can be ignored — see [Troubleshooting](#-troubleshooting).

### 4. Restart the runtime (only needed the first time)

`Runtime → Restart runtime` — this ensures the newly installed `transformers` version is loaded.

### 5. Run **Cell 2** (Captioning)

A form will appear. Fill in:

| Field | Description |
| :--- | :--- |
| **FOLDER_PATH** | Path to your image folder in Drive, e.g. `/content/drive/MyDrive/MyPhotos` |
| **MODEL_CHOICE** | `Moondream 2B (recommended)` or `Moondream 0.5B (fastest)` |
| **CAPTION_PROMPT** | The prompt used for captioning (default: `Describe this image in detail.`) |

Click **Run**. The script will:

1. Scan the folder and find all images.
2. Create or resume `descriptions.csv` inside that folder.
3. Load the Moondream model onto the T4 GPU.
4. Caption each pending image one by one, **saving the CSV after every image**.
5. Show a progress bar for the remaining images.

### 6. Get your results

Your output lives at:

```
<your_folder>/descriptions.csv
```

with this format:

| filename | description |
| :--- | :--- |
| `IMG_0001.jpg` | A busy street market scene with vendors selling fresh produce... |
| `subfolder/photo.png` | A close-up of a tabby cat lying on a wooden floor, eyes half-closed... |
| `IMG_0002.jpg` | *(empty — will be filled on next run)* |

---

## 🔁 How Resumability Works

The script is built to survive runtime disconnections:

1. **First run** → creates `descriptions.csv` with every image filename and an empty `description` column.
2. **Each image processed** → the CSV is written to disk **immediately**.
3. **If Colab crashes** → the CSV already contains all completed descriptions.
4. **Re-run the cell** with the same folder path → the script reloads the CSV, finds the first empty `description`, and continues from there.
5. **New images added to the folder** → automatically appended to the CSV on the next run.

You can safely disconnect, close your laptop, and come back later. No work is lost.

---

## 🧠 Model Options

| Model | Size | Speed on T4 | Best For |
| :--- | :--- | :--- | :--- |
| **Moondream 2B** (default) | ~3.85 GB | ~1.4 s / short caption<br>~2.9 s / detailed caption | Best quality descriptions |
| **Moondream 0.5B** | ~1 GB | Faster, lower VRAM | High-throughput or small datasets |

Both are **open source** and run comfortably on a free Colab T4 with `float16` precision.

---

## 🛠️ Troubleshooting

### ❗ `AttributeError: 'HfMoondream' object has no attribute 'all_tied_weights_keys'`

This error means a **`transformers` 5.x** version was installed. The fix is baked into **Cell 1**, which pins:

```python
"transformers>=4.45.0,<5.0.0"
```

If you see this error, make sure you ran **Cell 1** as written, then **restarted the runtime**.

### ⚠️ Pip dependency warnings (`gradio`, `diffusers`, `huggingface-hub`)

You may see messages like:

```
gradio 6.30.0 requires huggingface-hub<3.0,>=1.16.0, but you have huggingface-hub 0.36.2
```

These are **expected and harmless**. The script doesn't use `gradio` or `diffusers`; it only needs `transformers` and its dependencies. Ignore the warnings and proceed.

### ❌ `The folder '...' does not exist`

Double-check the path in the form. A reliable way to get it:

1. In Google Drive, right-click the folder → **Get link** → copy the ID from the URL.
2. Or, in Colab's file browser (📁 icon), navigate to the folder and copy its path from the top bar.

Example valid paths:
- `/content/drive/MyDrive/Photos`
- `/content/drive/MyDrive/projects/dataset/images`

### 🐌 Captioning is slow

- Use **Moondream 0.5B** for faster throughput.
- Shorten the prompt (e.g. `"Describe this image briefly."`).
- Larger images take longer to encode — consider downscaling very high-resolution files beforehand.

### 💾 Want to regenerate all descriptions

Delete `descriptions.csv` from the folder and re-run. A new CSV will be created from scratch.

### ✏️ Want to re-caption a single image

Open `descriptions.csv`, clear that row's `description` cell, save, and re-run. The script will fill it in.

---

## 📂 Project Structure

```
.
├── README.md
├── image_captioning.ipynb   # The Colab notebook
└── (your images + descriptions.csv on Google Drive)
```

---

## 📜 License

This project is released under the **MIT License** — free for personal and commercial use.

The **Moondream** model is distributed under the **Apache 2.0 License** by its authors. See the [Moondream model card](https://huggingface.co/vikhyatk/moondream2) for details.

---

## 🙏 Credits

- **[Moondream](https://huggingface.co/vikhyatk/moondream2)** by Vikhyat Korrapati — the tiny, fast vision-language model powering captioning.
- **[Hugging Face Transformers](https://huggingface.co/docs/transformers)** — model loading and inference.
- **Google Colab** — free T4 GPU access.

---

## ⭐ Contributing

Issues and PRs are welcome. If you find a bug or want to add a feature (batch sizes > 1, multi-GPU, alternative models like BLIP-2 or LLaVA), feel free to open an issue.

---

**Built for anyone who has 200 images in a Drive folder and needs to caption them all — without babysitting a notebook.** 🚀
