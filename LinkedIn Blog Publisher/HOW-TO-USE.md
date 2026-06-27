# LinkedIn Blog Publisher

An n8n workflow triggered by a built-in form. Submit your post text and target audience, and the workflow automatically generates a cinematic AI image to go along with your LinkedIn post.

---

## Prerequisites

| Service | What you need |
|---|---|
| **n8n** | Self-hosted or cloud instance |
| **OpenRouter** | API key — used to run the prompt-refinement agent (`gpt-4o-mini` or similar) |
| **OpenAI** | API key with access to `gpt-image-1.5` |

---

## Setup

### 1. Import the workflow
In n8n go to **Workflows → Import** and upload `linkedin-blog-publisher.json`.

### 2. Add credentials

In the imported workflow, open these nodes and connect your credentials:

| Node | Credential needed |
|---|---|
| `OpenRouter Chat Model` | OpenRouter API key |
| `Generate Image` | OpenAI API key |

### 3. Activate the workflow
Toggle the workflow to **Active**. This makes the form publicly accessible.

---

## How to Use

### Option A — Use the built-in n8n Form

1. Open the **`On form submission`** node and copy the **Test URL** (for testing) or **Production URL** (when workflow is active).
2. Open the URL in any browser — you'll see a form titled **Blog-Publisher**.
3. Fill in:
   - **text** *(required)* — your LinkedIn post content
   - **target-audience** *(required)* — who the post is aimed at (e.g. "freelancers", "startup founders")
   - **Image** *(optional)* — upload a photo of yourself to use as the visual base
4. Submit the form.

### Option B — POST request (API)

```
POST https://<your-n8n-domain>/webhook/<webhook-id>
Content-Type: multipart/form-data

text=<your post text>
target-audience=<your audience>
Image=<optional file>
```

---

## What Happens After Submission

```
Form submission
    ↓
[refine-text agent]  ←  OpenRouter (gpt-4o-mini)
    Reads your text + target audience
    Fills in a cinematic image prompt template
    ↓
[Merge]
    Combines the original form data with the refined prompt
    ↓
[Generate Image]
    Sends the refined prompt to OpenAI gpt-image-1.5
    Returns a base64-encoded image
    ↓
[base64 to File]
    Converts the image to a binary file
    (ready to be sent to LinkedIn, saved to Drive, etc.)
```

---

## Extending the Workflow

The `base64 to File` node currently has no outgoing connection. You can extend it to:

- **Post to LinkedIn** — add a LinkedIn node and connect your OAuth credential
- **Save to Google Drive** — add a Google Drive upload node
- **Send by email** — add a Gmail or SMTP node
- **Save locally** — add a Write Binary File node

---

## Notes

- The image prompt is written in **English only**, regardless of the language of your post text — this is by design to maximize image generation quality.
- The `Generate Image` node uses `gpt-image-1.5` at `1024x1024` high quality. You can change the `size` parameter to `1792x1024` (landscape) or `1024x1792` (portrait) as needed.
- The optional **Image** field is collected by the form but not yet wired into the generation pipeline. If you want the AI to edit your photo instead of generating from scratch, replace the `Generate Image` node with an OpenAI image-edits call (`/v1/images/edits`) and pass the uploaded file.
