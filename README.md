# CropVision AI

CropVision AI is a web application for exploring crop leaf disease predictions. Upload a leaf image to see a predicted class, symptom and treatment information, and a Grad-CAM heatmap showing image regions used by the model.

The application supports **17 classes** across **tomato, potato, and corn**, including healthy leaves. It is a project prototype; predictions and treatment suggestions should be independently checked before use in the field.

## Features

- **Leaf image upload:** drag and drop or choose an image in the browser.
- **Image classification:** PyTorch EfficientNet-B3 inference with test-time image transformations.
- **Visual explanation:** Grad-CAM heatmap for each prediction.
- **Class information:** symptoms, organic treatments, chemical treatments, and danger levels from `class_symptoms.json`.
- **Optional AI advice:** language-specific advice through Groq's API.
- **Optional audio:** Sarvam AI text-to-speech, with a browser speech fallback.
- **PDF report:** client-side export using jsPDF and html2canvas.

## Tech stack

| Layer | Technologies |
| --- | --- |
| Frontend | HTML, CSS, vanilla JavaScript |
| API | Python, FastAPI, Uvicorn |
| Computer vision | PyTorch, Torchvision, EfficientNet-B3, OpenCV, Grad-CAM |
| Optional services | Groq, Sarvam AI, Google Gemini |
| Report export | jsPDF, html2canvas |

## Getting started

### 1. Clone and install

```bash
git clone https://github.com/harshbhatia04/Crop_Disease_Detection.git
cd Crop_Disease_Detection
python -m venv .venv
```

Activate the environment:

```bash
# Windows PowerShell
.venv\Scripts\Activate.ps1

# macOS / Linux
source .venv/bin/activate
```

Install the application dependencies:

```bash
pip install -r requirements.txt
```

### 2. Add model weights

Place your trained EfficientNet-B3 checkpoint at the repository root as:

```text
best_model_vision_autosave.pth
```

Model weights and datasets are excluded from Git by `.gitignore`. **The repository does not include a trained checkpoint.** Although the application may start without the file, predictions are not meaningful until valid weights are loaded. The checkpoint must match the 17 classes in `class_symptoms.json`.

### 3. Configure optional API features

Create a `.env` file in the repository root if you want to use the external services:

```dotenv
GROQ_API_KEY=your_groq_key
SARVAM_API_KEY=your_sarvam_key
GEMINI_API_KEY=your_gemini_key
```

- `GROQ_API_KEY` enables the **Get Expert Solution** response.
- `SARVAM_API_KEY` enables Sarvam audio; the frontend can fall back to browser speech.
- `GEMINI_API_KEY` is used by an optional image-label override in `/predict`.

Keep `.env` private; it is ignored by Git.

### 4. Run the app

```bash
uvicorn app:app --host 127.0.0.1 --port 8090
```

Open [http://127.0.0.1:8090](http://127.0.0.1:8090). FastAPI's interactive API documentation is available at [http://127.0.0.1:8090/docs](http://127.0.0.1:8090/docs).

## API

| Route | Method | Purpose |
| --- | --- | --- |
| `/` | GET | Serve the web interface |
| `/predict` | POST | Accept a leaf image as multipart form data under the `file` field |
| `/ask_ai` | POST | Return advice for a crop, disease, and language |
| `/tts` | POST | Return speech audio for text and language |

Example prediction request:

```bash
curl -X POST -F "file=@path/to/leaf.jpg" http://127.0.0.1:8090/predict
```

The prediction response includes the crop, disease, confidence string, symptoms, treatments, danger level, and a Grad-CAM image path.

## Project structure

```text
app.py                         FastAPI routes and static-file serving
inference.py                   EfficientNet-B3 inference and Grad-CAM
class_symptoms.json            Class labels, symptoms, and treatments
model_training_vision_only.py  Vision-model training script
CropVision_*Training.ipynb     Training notebooks
static/                        Web interface and generated heatmap
requirements.txt               Application dependencies
```

The training scripts expect an ImageFolder-style dataset with `train/` and `val/` directories. The notebooks use Google Colab and Google Drive paths; adjust those paths for your environment.

## Current limitations

- No trained weights or dataset are published in this repository, and no reproducible accuracy figure is provided here.
- `app.py` includes an optional Gemini label override that can replace the model's predicted label and displayed confidence. Do not treat that displayed value as a calibrated model probability.
- The Grad-CAM image is written to a shared path in `static/`; concurrent requests can overwrite it.
- A machine-specific test-image path in `app.py` is skipped when that directory does not exist.
