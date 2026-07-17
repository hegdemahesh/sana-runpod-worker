# elabs / Sana 0.6B Text-to-Image

[![Run on RunPod](https://runpod.io/badge/runpod-hub)](https://runpod.io/console/hub)

Ultra-fast 1024×1024 text-to-image generation powered by NVIDIA's **Sana 0.6B**. Model weights are baked into the Docker image — no network volume, no cold-download delays.

![Sample output](https://pub-796a08821c1c483aaf5e274e0d03e350.r2.dev/studio-jobs/sana/b6afbdefd7b5/elabs-sana-demo-3-1780399611.png)

## Highlights
- **~1 second** per 1024×1024 image (18 steps, RTX 4090)
- **0.6B parameters** — small, fast, works on 4GB+ VRAM
- **Weights baked in** — no HF token, no gated access, no cold-download
- **Seed control**, negative prompt, configurable steps/guidance
- **Apache-2.0** license (NVIDIA Sana 0.6B)

## API

### Input
```json
{
  "input": {
    "prompt": "a neon cyberpunk cat on a rooftop at night, cinematic",
    "negative_prompt": "blurry, low quality",
    "height": 1024,
    "width": 1024,
    "num_inference_steps": 18,
    "guidance_scale": 5.0,
    "seed": -1
  }
}
```

### Output
```json
{
  "image_b64": "<base64 PNG>",
  "prompt": "a neon cyberpunk cat on a rooftop at night, cinematic",
  "seed": 374969113,
  "wall_time_s": 1.1
}
```

### Parameters
| Parameter | Type | Default | Description |
|---|---|---|---|
| `prompt` | string | **required** | Text prompt |
| `negative_prompt` | string | `""` | Negative prompt |
| `height` | int | `1024` | Output height (512–4096) |
| `width` | int | `1024` | Output width (512–4096) |
| `num_inference_steps` | int | `18` | Denoising steps (4 = ultra-fast, 30 = quality) |
| `guidance_scale` | float | `5.0` | CFG guidance scale |
| `pag_guidance_scale` | float | `2.0` | PAG scale |
| `seed` | int | `null` | Fixed seed for reproducibility (`-1` or `null` = random) |

## GPU Requirements
- **Recommended**: RTX 4090 / RTX 6000 Ada / L40S
- **Minimum**: Any GPU with ≥4GB VRAM (RTX 3080, A5000, L4, etc.)
- **CUDA**: 12.0+

## Benchmark
| GPU | Steps | Time |
|---|---|---|
| RTX 4090 | 18 | ~1.1s |
| RTX 4090 | 4 | ~0.3s |
| RTX A5000 | 18 | ~1.8s |

## Deploy to RunPod Serverless

This repository is already structured as a RunPod serverless worker:
- `handler.py` starts `runpod.serverless.start({"handler": handler})`
- `Dockerfile` builds a GPU image and launches the handler
- `.runpod/hub.json` contains RunPod Hub serverless metadata

### 1) Build and push your Docker image

Use any registry RunPod can pull from (Docker Hub, GHCR, ECR, etc).

```bash
# from this repository root
docker build -t <your-registry>/sana-runpod-worker:latest .
docker push <your-registry>/sana-runpod-worker:latest
```

### 2) Create a RunPod Serverless endpoint

In RunPod Console:

1. Go to **Serverless** → **New Endpoint**.
2. Choose **Custom Source**.
3. Set **Container Image** to your pushed image:
   - `<your-registry>/sana-runpod-worker:latest`
4. Set GPU and disk (recommended baseline):
   - GPU: 1x RTX 4090 / Ada / A5000 class
   - Container disk: 40 GB
5. Deploy the endpoint.

Notes:
- No Hugging Face token is required for this model.
- Model files are baked into the image at `/models/sana`.

### 3) Test the endpoint

After deploy, call your endpoint with:

```bash
curl -X POST "https://api.runpod.ai/v2/<ENDPOINT_ID>/run" \
  -H "Authorization: Bearer <RUNPOD_API_KEY>" \
  -H "Content-Type: application/json" \
  -d '{
    "input": {
      "prompt": "a neon cyberpunk cat on a rooftop at night, cinematic",
      "num_inference_steps": 18,
      "width": 1024,
      "height": 1024,
      "guidance_scale": 5.0
    }
  }'
```

If your endpoint runs in async mode, poll the returned job id using RunPod's status endpoint.

## GitHub Actions Docker Build

This repository now includes a workflow at `.github/workflows/docker-image.yml`.

What it does:
- Builds the Docker image on every push and pull request.
- Pushes image tags to GitHub Container Registry (GHCR) on push/tag/manual runs.
- Uses GitHub Actions cache to speed up rebuilds.

Published image format:

```text
ghcr.io/<github-username-or-org>/sana-runpod-worker:<tag>
```

Examples of tags:
- `main`
- `sha-<commit-sha>`
- `v1.0.0` (when pushing git tags like `v1.0.0`)
- `latest` (default branch)

### One-time repo setting

In GitHub repository settings:
- Ensure Actions has permission to write packages.
  - Settings -> Actions -> General -> Workflow permissions -> Read and write permissions

### Use image in RunPod

Set your RunPod endpoint image to:

```text
ghcr.io/<github-username-or-org>/sana-runpod-worker:latest
```

## License
Apache-2.0 — NVIDIA Sana 0.6B weights and diffusers implementation.
