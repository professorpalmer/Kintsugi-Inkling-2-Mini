# Kintsugi for Inkling 2 Mini

Thinking Machines' Inkling 2 Mini (a 26.4B looped mixture-of-experts model, Apache-2.0) in our own 2-3 bit model file,
on one 12 GB NVIDIA card with a **262,144-token context**. It comes as a model file and a Windows bundle and a Linux
bundle (engine, launcher, saved-chat layer).

Built on: **Inkling 2 Mini** by Thinking Machines; the **EXL3 trellis codec** by turboderp (exllamav3, after QTIP);
the **Escha W2 format** by Escha Labs; **Unsloth's llama.cpp port** of the model and the upstream llama.cpp work by
Piotr Wilkin, Daniel Han, Aldehir Rojas and Xuan-Son Nguyen. Full list: [CREDITS.md](CREDITS.md).

**What is special**

- **Our own model file**: every expert of Inkling 2 Mini in EXL3 trellis code at 2 or 3 bits (2.444 bits per
  weight for the experts, 9.07 GB in all), fitted by us on Thinking Machines' weights.
- **Close to the original**: mean KL 0.0310 against the official weights on a public golden set (the same top token
  93.62 %): 27 % lower than Unsloth's UD-Q2_K_XL (9.14 GB, about the same size), measured in the same engine on
  2026-11-21 (details and one limit: [benchmarks](benchmarks/README.md)).
- **Level with the official weights on AppWorld** (agent tasks): 595 vs 592 of 672 paired task-runs against the exact
  official weights (NVFP4 / MXFP8) in the same engine (release default, 4-bit cache; p = 0.78). Against Unsloth's setup (its file on its build;
  measured with the 8-bit cache): better, 298 vs 283 and 298 vs 281-283 of 336
  ([details and the setup caveat](benchmarks/README.md)).
- **262,144 tokens on a 12 GB card.** The 8 global attention passes of the model keep their K/V cache at 4 bits (q4_0:
  9 KiB per token, not 32 KiB at f16): on a free 12 GB card up to 250k tokens are in VRAM; with a desktop on the card,
  the rest goes to system RAM (the tiered K/V cache). With all of it in VRAM at f16, about 65,000 tokens fit on 12 GB.
- **Recall at depth**: 24 / 24 needles and 20 / 20 exact code copies at 64k, 117k and 245k tokens (q4_0 release default).
- **Speed** (RTX 4070): decode 64.5 tok/s at 4k, 57.9 at 32k, 42.4 at 117k and 29.75 at 245k tokens of context. At 4k
  and 32k that is 1.09x and 1.11x Unsloth's build with its UD-Q2_K_XL file on the same card, with prompt reads about 2x.
- **Saved chats**: a long chat continues after a restart without a full read of the prompt.
- **A repetition guard** for agent loops (on by default; `LLAMA_ARG_REPEAT_GUARD=0` turns it off): when the model
  repeats nearly the same tool call, the server adds one short note to that request. At two stuck points it broke the
  loop in 6 and 7 of 10 tries, against 0 of 10 without it.
- **Two slots (opt-in)**: `INKLING_SLOTS=2` serves two requests at the same time in one shared window (fewer cells in
  VRAM, saved chats off, not bit-equal to one slot, not faster for one user; see the
  [bundle guide](bundle/README.md)).
- **Works with coding harnesses**: Claude Code, Codex, Pi and OpenCode, 5 of 6 test tasks each, with no malformed tool
  calls, no loops and no server errors.
- **Runs on RTX 30, 40 and 50 cards** with 12 GB or more, and on H100 / H200 (RTX 50 and Hopper from v1.0.1; see
  "Hardware"); Windows and Linux.
- **Fully open**: Apache-2.0 for the model file and our files, MIT for the engine; the engine source is in every
  release. The NVIDIA CUDA libraries in the bundles are NVIDIA's (see [NOTICE](NOTICE)).

![Kintsugi for Inkling 2 Mini](inkling-card.png)


**How it was made:** the method, the evidence and the tests behind these numbers are in the paper "Kintsugi: Find and Repair Quantization Damage Where Tasks Break, Not Where Averages Look": https://github.com/professorpalmer/kintsugi-method (DOI [10.5281/zenodo.24455575](https://doi.org/10.5281/zenodo.24455575))

## Hardware

| Hardware | Status |
|---|---|
| RTX 40 (Ada) | Supported: every release check ran on an RTX 4070 (Windows and Linux). The v1.0.1 Windows engine gives the same output as v1.0 on it (KL check and 50 / 50 identity prompts token-identical). |
| RTX 30 (Ampere) | Supported: the v1.0 release engine (the same source as v1.0.1) was checked on an RTX 3080 Ti (Linux): it starts with the launcher defaults, 24 / 24 needles at 32k, the identity prompts answer, top-1 agreement 97.8 %. Its mean KL is 7.5 % lower than on the RTX 4070; without one near-tie token in the 4070 reference run the two cards differ by 0.6 % (the same kernels; only the order of sums differs). |
| RTX 50 (Blackwell, desktop) | Supported from v1.0.1. The Linux engine was checked on an RTX 5060 Ti 16 GB: it starts with the launcher defaults (all 262,144 cells in VRAM), mean KL 0.0397 and top-1 92.60 % on our 10-document check (RTX 4070: 0.0400 and 92.51 %), 24 / 24 needles at 32k, the 50 identity prompts answer, the same 50 answers with PDL on and off, 0 CUDA errors. The Windows engine has the same code for these cards, but we did not run it on an RTX 50 card. RTX 50 laptop GPUs and RTX PRO Blackwell cards: not checked. |
| H100 / H200 (Hopper) | Supported from v1.0.1. The Linux engine was checked on an H100 PCIe 80 GB: it starts with the launcher defaults (all 262,144 cells in VRAM), mean KL 0.0401 and top-1 92.57 % on the same check, 24 / 24 needles at 32k, the 50 identity prompts answer, the same 50 answers with PDL on and off, 0 CUDA errors. H200 has the same code (compute capability 9.0) but was not checked. The Windows engine has the code too, but we did not run it on a Hopper card. |
| Other Blackwell (B200 / GB200, GB10 / DGX Spark) | Not supported: this engine has no native code for them, and it does not run the portable code yet. The launcher stops with a message. |
| A100, V100 and older cards | Not supported: this engine has no code for them. The launcher stops with a message. |
| RTX 20 (Turing) | Not supported: it fails our exact-output checks and runs 6-14x slower than an RTX 4070. |
| AMD Radeon | Not yet: after Kintsugi-27B's AMD release (test builds exist). |
| Apple Silicon | Not yet: needs its own port; not scheduled. |
| Intel GPUs | Not supported: this engine has no code for them. |

You need 12 GB of VRAM or more, and an NVIDIA driver 580 or newer (Windows) or 570 or newer (Linux). 32 GB of system
RAM or more is best for very long chats.

## Quick start (Windows)

1. Download the zip from [Releases (latest)](https://github.com/professorpalmer/Kintsugi-Inkling-2-Mini/releases/latest)
   and unzip it, for example to `C:\Kintsugi-Inkling`.
2. Download `Kintsugi-Inkling-2-Mini-12G.gguf` from
   [Hugging Face](https://huggingface.co/CaryPalmer/Kintsugi-Inkling-2-Mini) into the `models` folder of the bundle.
   SHA-256 `7d93d5e8e61f6694776ec8b488751787a7dd3cb745d4f9eaee9f0b78b0e60cec`.
3. In PowerShell, in that folder: `powershell -ExecutionPolicy Bypass -File start-server.ps1 -Wait`. Wait for the
   line `ready  http://127.0.0.1:8080/v1`.
4. Open http://127.0.0.1:8080/ in a browser, or point an OpenAI-compatible app or a coding harness at
   `http://127.0.0.1:8080/v1`.

You need an NVIDIA driver 580 or newer. The full guide is in [bundle/README.md](bundle/README.md).

**Linux (x86_64):** download `Kintsugi-Inkling-2-Mini-v1.0.1-linux-x86_64.tar.gz` from the same Releases page, then
`tar -xzf` it, put the model in `models/`, and run `./start-server.sh --wait`. It needs NVIDIA driver 570 or newer and
glibc 2.34+ (Ubuntu 22.04 and 24.04). Full guide: [bundle-linux/README.md](bundle-linux/README.md).

## Index

| Folder | What is in it |
|---|---|
| [bundle/](bundle/README.md) | The user guide, the text files of the Windows zip (launcher, layer) and the changelog. |
| [bundle-linux/](bundle-linux/README.md) | The Linux guide and launcher scripts of the Linux tarball (the layer is the same as in bundle/). |
| [harnesses/](harnesses/README.md) | Settings for Claude Code, Codex, Pi and OpenCode, and what we tested. |
| [model/](model/README.md) | The model file: format, bits, how it was made, the model's shape (the loop, the K/V caches). |
| [engine/](engine/README.md) | The engine (a llama.cpp fork): what it adds, where the source is, how to build it, the driver needs. |
| [benchmarks/](benchmarks/README.md) | Each number on the card: the test, the settings, the card and the date. |

> **Important for multi-turn and agent use:** send the model's earlier thinking (`reasoning_content`) back with each
> earlier assistant turn. Users report much worse results without it (we did not measure that case). See [harnesses/](harnesses/README.md), step 4.

Licences: [LICENSE](LICENSE) (Apache-2.0) and [NOTICE](NOTICE). Use of the model is also subject to the Thinking
Machines [Model Acceptable Use Policy](https://thinkingmachines.ai/model-acceptable-use-policy). Credits:
[CREDITS.md](CREDITS.md). Problems, numbers from your card and fixes: please open an issue or a pull request.

No person or company named here endorses or sponsors this project.
