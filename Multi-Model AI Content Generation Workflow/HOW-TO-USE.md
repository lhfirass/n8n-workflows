# xImage Studio — Multi-Model AI Content Generation Workflow

A single n8n workflow that routes one POST request to **9 different AI content generation pipelines** based on a `category` field. Images are saved automatically to Google Drive.

---

## Prerequisites

| Service | What you need |
|---|---|
| **n8n** | Self-hosted or cloud instance |
| **OpenAI** | API key with access to `gpt-image-2` |
| **OpenRouter** | API key (used for `gpt-4o-mini` via LangChain agent) |
| **KIE.ai / Higgsfield** | API key for image & video generation |
| **Google Drive** | OAuth2 credential connected in n8n |

---

## Setup

### 1. Import the workflow
In n8n, go to **Workflows → Import** and upload `xImage-studio.json`.

### 2. Add credentials
Replace all placeholder values in the workflow nodes:

| Placeholder | Where to add |
|---|---|
| `[ADD_YOUR_OPENAI_CREDENTIAL_ID]` | All `gen-img` / `gen-img2–6` nodes → Credentials |
| `[ADD_YOUR_OPENROUTER_CREDENTIAL_ID]` | `gpt-40-mini` node → Credentials |
| `[ADD_YOUR_HIGGSFIELD_API_KEY]` | All KIE.ai HTTP Request nodes → Authorization header |
| `[ADD_YOUR_GOOGLE_DRIVE_CREDENTIAL_ID]` | `Upload to Drive` node → Credentials |
| `[ADD_YOUR_GOOGLE_DRIVE_FOLDER_ID]` | `Upload to Drive` node → Folder ID |

### 3. Activate the workflow
Toggle the workflow to **Active** in n8n. This exposes the webhook endpoint.

---

## How to Trigger

Send a **POST** request to your webhook URL:

```
POST https://<your-n8n-domain>/webhook/xImage-gen
```

### Request body

```json
{
  "category": "<mode>",
  "prompt": "<your prompt or content>"
}
```

The `category` field routes to one of the 9 pipelines below.

---

## Available Modes (`category` values)

### 1. `img-scanner` — Image Style Analyzer
Analyzes a reference image and reverse-engineers its visual style into a reusable prompt template.
> Edit `Edit Fields6` node to set `img-url` to your reference image URL.

**Pipeline:** `img-url + system prompt → GPT-5.4 vision → style prompt → gen-img3 → Drive`

---

### 2. `x-yt-thumb` — YouTube Thumbnail (Direct)
Generates a YouTube thumbnail directly from your content text without an AI agent step.
> Send your content in the `prompt` field.

**Pipeline:** `content → gpt-image-2 → base64-to-png → Drive`

---

### 3. `yt-thumb-agent` — YouTube Thumbnail (Agent)
Uses a GPT-4o-mini agent to first write an optimized image prompt from your content script, then generates the thumbnail.
> Send your video script in the `prompt` field.

**Pipeline:** `content script → GPT-4o-mini agent → gpt-image-2 → base64-to-png → Drive`

---

### 4. `travel-map` — Travel Map / Prompt-to-Image (KIE.ai)
Generates an image from a text prompt using the `nano-banana-2` model on KIE.ai.
> Edit `Edit Fields1` to set your prompt, or pass it in the request body.

**Pipeline:** `prompt → KIE.ai createTask → poll status → image URL → Drive`

---

### 5. `portrait-img` — Cinematic Portrait Edit
Transforms a portrait photo into a cinematic Neo-Noir / Cyberpunk style using `gpt-image-2` edits.
> Edit `Edit Fields7` to replace `[ADD_YOUR_PORTRAIT_IMAGE_URL]` with your image URL.

**Pipeline:** `img-url + style prompt → gpt-image-2 edits → base64-to-png → Drive`

---

### 6. `learn-img` — Educational / Technical Illustration
Generates a hand-drawn ink-style technical diagram of any subject on notebook paper.
> The prompt in `gen-img1` contains `{s22-ultra-mobo}` as a placeholder — replace it with your subject.

**Pipeline:** `static prompt → gpt-image-2 → base64-to-png → Drive`

---

### 7. `omni` — Gemini Omni Video (Image-to-Video)
Generates a short cinematic video from an image using the `gemini-omni-video` model on KIE.ai.
> Edit `Edit Fields3` to replace `[ADD_YOUR_IMAGE_URL]` with your product/scene image URL.

**Pipeline:** `img-url + cinematic prompt → KIE.ai createTask → poll status → video URL → Drive`

---

### 8. `inter-design` — Interior Design Renderer
Transforms a raw/unfinished architectural space photo into a photorealistic finished interior.
> Edit `Edit Fields2` to replace `[ADD_YOUR_INTERIOR_IMAGE_URL]` with your space image URL.

**Pipeline:** `img-url + design prompt → KIE.ai nano-banana-2 → poll status → image URL → Drive`

---

### 9. `product` — Multi-Angle Product Photography Grid
Creates a professional 6-angle product photography grid from up to 4 product images.
> Edit `Edit Fields5` to replace the 4 `img-link-*` placeholders with your product image URLs.

**Pipeline:** `4x img-urls + studio prompt → KIE.ai nano-banana-2 → poll status → image URL → Drive`

---

## Output

- **GPT-image-2 pipelines** → image is converted from base64 → uploaded to Google Drive as `.png`
- **KIE.ai pipelines** → image/video URL is extracted → file is downloaded → uploaded to Google Drive
- Files are named: `YYYY-MM-DD_HH-mm-<category>.png`

---

## Notes

- KIE.ai jobs use a **polling loop** (Wait → check status → Switch). If the job is still `waiting`, the workflow loops back automatically. If it `fails`, the branch exits silently.
- The `omni` video pipeline uses a longer **30-second wait** before polling; image pipelines use **10 seconds**.
- The `x-imgs` pipeline (img-scanner → edit-prompt → gen-img3) uses a GPT-4o-mini agent to inject a real subject into the extracted style template before generating.
