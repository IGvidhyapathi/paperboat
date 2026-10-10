# Paperboat – project notes (handoff)

## Goal
Live wallpaper app. Two parts:
1. **Engine (internal)**: image -> seamless looping wallpaper video that feels *alive* (depth parallax + particles + motion).
2. **Wallpaper app (public)**: only plays the finished videos (catalog, favorites, battery handling). No generation code, we don't disclose how videos are made.
Build from existing GitHub repos, not from scratch.

## Decisions so far
- Core engine: BrokenSource/DepthFlow (https://github.com/BrokenSource/DepthFlow). Licence **AGPL-3.0**: fine while we only ship output videos and never distribute/network-expose the engine. If users upload photos to a server running it, revisit (swap in a permissive renderer, e.g. own shader + Depth Anything).
- Start with a pre-generated catalog; user uploads maybe later.
- Players to reuse: macOS Wallps / Muro; Android UndeadWallpaper / Zero / pallax-android; cross-platform ScreenPlay.

## Findings (tested on MacBook, M4, macOS 27, Python 3.12 via uv)
- `uv venv --python 3.12 .venv && uv pip install --python .venv/bin/python -e .` installs in ~40s.
- Render: `depthflow input -i img.jpg main -o out.mp4 -t 3 -f 30 -w 1920 -h 1080` (headless is the default for file output; there is no `--backend` flag on the CLI). Needs `ffmpeg` on PATH.
- Works on Apple GPU (OpenGL renderer "Apple M4"), ~98 fps at 1080p, depth model Depth Anything V2 Small.
- **DepthFlow is camera parallax ONLY.** Nothing inside the image moves, no particles. That is why it does not look "live".
- Video encoders available: h264, h265, av1-svt/rav1e, and NVENC (h264/h265/av1-nvenc) for NVIDIA.

## Laptop
Windows, Ryzen 5, RTX 3060 **6GB** VRAM. Needs CUDA torch build; AI video models must be low-VRAM (fp16/fp8, CPU offload, small res), expect slow.

## Next steps
1. (Laptop, RTX 3060) install DepthFlow, render same test image, try NVENC and bigger depth models, try 4K.
2. Try AI image-to-video for real motion on the 3060: Stable Video Diffusion (Stability-AI/generative-models), DynamiCrafter, PIA. Compare quality/VRAM (3060 is 12GB or 6GB laptop variant, check).
3. Build a depth-aware particle layer (dust/snow/fireflies/mist) as a GPU shader (moderngl, same stack as DepthFlow). No ready-made repo exists.
4. Procedural motion (water/cloud/grass) via masked shader warps (see glb99/cinemagraph, Text2Cinemagraph).
5. Wrap into `paperboat-engine` script: image -> loop-perfect HEVC wallpaper video.
6. Then build/adapt the player app.

## Layout
- `DepthFlow/` – cloned upstream repo (don't copy its `.venv` between machines; recreate it).
- `test/` – throwaway sample image and output.

## Vendored code
`DepthFlow/` is a copy of upstream BrokenSource/DepthFlow at commit bb85e20b0700df17151b4cae109da98b6cd2afce (AGPL-3.0, see DepthFlow/license.txt), included as plain files so one clone gets everything. Create the venv inside it per machine (never commit `.venv`).
