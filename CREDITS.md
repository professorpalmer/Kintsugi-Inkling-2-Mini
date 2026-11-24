# Credits

This release stands on a lot of other people's work. If we missed you, tell us and we will add you.

## The model

- **Thinking Machines**: Inkling 2 Mini, the model: its weights, tokenizer, chat template and the looped
  mixture-of-experts design (Apache-2.0; see their Model Acceptable Use Policy).
  Every tensor of our model file that is not a coded expert comes from their release.
- **turboderp (exllamav3)**: the EXL3 trellis codec and MCG codebook, the `quantize_exl3` quantizer that made our expert
  fits, and the CUDA GEMV kernels in our engine (MIT).
- **Albert Tseng, Qingyao Sun, David Hou, Christopher De Sa (QTIP, Cornell RelaxML)**: the trellis quantization method
  that EXL3 builds on.
- **Escha Labs (Zihao Tu, Yangzihao Wang, Jiaming Pan and the escha-mlx authors)**: the Escha W2 format (EXL3 code with
  Hadamard input / output scales) that our coded experts use, and the `escha-mlx` reference (Apache-2.0).
- **StillDeadcode (Qwen3.8-27B quant challenge)**: the public corpus whose calibration part gave our Hessians and whose
  public part is our golden set (CC BY 4.0); credit to the publishers of its sources (listed on the dataset card).
- **Unsloth (Daniel Han, Michael Han and the Unsloth team)**: their llama.cpp port of Inkling 2 Mini (MIT), which our
  engine code is ported from (below); and their Inkling GGUF (Apache-2.0), whose metadata, tokenizer and chat template
  our model file reuses, and which we used as a reference while we built ours.

## Inkling 2 Mini support in the engine

The Inkling code in our engine is ported from Unsloth's llama.cpp mix `b12588-mix-c0c3de8` (MIT, the ggml authors and
Unsloth) and from upstream llama.cpp. Each ported file names its source in its header.

Ported by us (the file headers or commits name them):

- **Unsloth (Daniel Han and the Unsloth team)**: the mix itself, from which the banded attention op, the Inkling graph,
  the K/V cache, hyperparameter and model-loading code are ported, and Unsloth's own Inkling 2 Mini loop layout (the
  GGUF keys `num_loops` / `loop.core_*`), which this engine and our model file use.
- **Piotr Wilkin (pwilkin / ilintar)**: Inkling 2 support (text and vision), ggml-org/llama.cpp
  [#32217](https://github.com/ggml-org/llama.cpp/pull/32217), including the per-head attention masks (commit
  `5c008e000`); and the DSpark drafter support, [#32218](https://github.com/ggml-org/llama.cpp/pull/32218) (via
  Unsloth's mix; not used by this release's launcher).
- **Aldehir Rojas (aldehir)**: the Inkling chat parser (tool calls and reasoning), with its JSON response-format and
  anchored-trigger grammar fixes, [#32381](https://github.com/ggml-org/llama.cpp/pull/32381).
- **Xuan-Son Nguyen (ngxson)**: `sort_keys` in the jinja engine, which Inkling's template needs,
  [#32366](https://github.com/ggml-org/llama.cpp/pull/32366).
- Upstream CUDA mixture-of-experts work, cherry-picked with its authors kept: **Johannes Gäßler** (MMQ with many
  experts, [#29941](https://github.com/ggml-org/llama.cpp/pull/29941)), **Joel Fernandes** (MMQ Stream-K blocks,
  [#30553](https://github.com/ggml-org/llama.cpp/pull/30553)), **Konrad Moren** (NVFP4 accumulation in MMQ,
  [#29857](https://github.com/ggml-org/llama.cpp/pull/29857)), **Aman Gupta** (top-k MoE,
  [#28432](https://github.com/ggml-org/llama.cpp/pull/28432); graph alloc dependencies,
  [#27301](https://github.com/ggml-org/llama.cpp/pull/27301)), **anujj** (fused MoE weighted expert reduction,
  [#25952](https://github.com/ggml-org/llama.cpp/pull/25952)).

In Unsloth's mix (the work our ported files build on; not ported by us as separate changes):

- **Daniel Han (danielhanchen, Unsloth)**: the banded flash attention for the relative bias
  ([unslothai/llama.cpp#386](https://github.com/unslothai/llama.cpp/pull/386)), the chunked relative-bias attention
  ([unslothai/llama.cpp#385](https://github.com/unslothai/llama.cpp/pull/385)), the loop-slot fix after the scale pass
  ([#32704](https://github.com/ggml-org/llama.cpp/pull/32704)) and the expert gate-up scales
  ([#32657](https://github.com/ggml-org/llama.cpp/pull/32657)).
- **Piotr Wilkin**: the attention-mask fixes ([#32429](https://github.com/ggml-org/llama.cpp/pull/32429)).

Related upstream work, not in this engine:

- **Pedro Cuenca (pcuenca)**: Inkling 2 Mini in upstream llama.cpp,
  [#32694](https://github.com/ggml-org/llama.cpp/pull/32694). It uses another GGUF format (arch `inkling2_loop`); this
  engine follows Unsloth's loop layout instead, so that code is not ported.

Speed work on Inkling 2 Mini:

- **Project Maya (MIT)**: [maya-inkling-mini](https://github.com/alesha-pro/maya-inkling-mini), the Inkling branch
  of [Project Maya](https://github.com/mw00/project-maya) v1.1.70 (upstream pull requests
  [#981](https://github.com/mw00/project-maya/pull/981), [#1079](https://github.com/mw00/project-maya/pull/1079),
  [#1095](https://github.com/mw00/project-maya/pull/1095)), and its engine maya-ik2:
  - **Peasant Smith (mw00) and the Project Maya contributors**, and **Niko1221 and the Strata contributors**: Project
    Maya and Strata, the engine family it grew out of.
  - **mrsirg97-rgb** and **OscarSheng**: the Inkling 2 engine that maya-ik2 started from (#981: the file format and a
    reference forward pass; #1079: maya-ik2, served by Maya's server).
  - **Alexey Fateev (alesha-pro)**: the Inkling 2 Mini work (#1095), in maya-inkling-mini (commit `c520fd1`): the
    measurement of llama.cpp's per-token launch overhead on Inkling 2 Mini, and the idea to fuse its small glue
    launches. Our fused ops (the L2 clip, the short convolution, the mix and the loop gate) are our own kernels, made
    after the launch fusions that maya-inkling-mini describes; they copy no Maya code.
  - **cobra91**: the Windows / CUDA 13 port of maya-inkling-mini.

  The licence of Project Maya is in `licenses/LICENSE-project-maya` in the bundles and in the engine source, for any
  file that takes code from it (such a file names Project Maya in its header).



## The engine

- **The ggml authors and every llama.cpp contributor** (ggml-org, MIT): the engine is a llama.cpp fork.
- **PrismML (Pasha Khosravi, Brian / bri-prism, Pooya Khosravi)**: the llama.cpp fork our engine descends from (MIT),
  including the speculative-decoding fixes and the GB10 kernel work.
- **sudoingX**: the batch-invariant mode (`GGML_CUDA_BATCH_INVARIANT`) and the planar activation layout; our identity
  checks depend on it.
- **Ajay9o9 (AjHomelab), llama.cpp-escha**: the Escha op for ggml (CPU and CUDA) and the tensor-core prefill in our
  engine base (MIT), with his authorship kept.
- **turboderp (exllamav3)**: the EXL3 trellis CUDA kernels in our engine base (MIT).
- **spiritbuun, buun-llama-cpp**: the ggml port of the exllamav3 kernels that showed the GEMV split without a
  cooperative launch (MIT).
- **Chirag Patnaik**: DFlash2 support in llama.cpp (local convolution + candidate selector).
- **alesha-pro**: the Mirai S port to ggml in our engine base, and the control-vector projection mode.
- **Mirai Labs**: the Mirai S trellis codec whose port is in the engine base (its kernels are left out of this build).
- **usmaneth**: DFlash fix for borrowed embeddings and head with Hadamard transforms; review fixes.
- **zhaoyilun**: Hadamard inverse on the MTP token-embedding lookup for qwen35.
- **cklxx**: made the CUDA ADD + RMS_NORM + MUL fusion reachable.
- **Empressia**: `input_image` in `function_call_output` for the server.
- **lenny76**: AVX-512 single-token flash attention, PQ2_0 VNNI GEMM, Hadamard tensors kept out of CPU repack.
- **abdulaziz**: PQ2_0 x86 AVX2 / AVX-VNNI kernels and fixes.
- **Scott Guyton**: SSE2 / SSSE3 and SYCL kernels for PTQ1_0 and PQ2_0.
- **Strike3D**: AVX2 / AVX-VNNI kernels for the Q1_0 repack path.
- **Ankit Khandelwal**: vectorized HIP PTQ1_0 dot product.
- **Mark Schatza**: native PQ2_0 unpacking in the HIP MMQ loader.
- **Jason Titus**: Metal PTQ1_0 mat-vec for two to four columns.
- **Tomer Weller**: Metal language 4.0 when the tensor API is there.
- **Magnitude (magnitudedev)**: mixed K / V cache types on the tensor-core path. (Credited for the idea; that option is
  not in this build.)

## Runtime pieces

- **NVIDIA**: the CUDA runtime and cuBLAS libraries in `bin` (NVIDIA CUDA EULA; not covered by our licences).
- **The authors of the npm packages in the built-in web chat page** (the llama.cpp web UI): each package, its licence
  and its copyright text are in `licenses/web-ui-THIRD-PARTY.txt` in the bundles.
- **Bytecode Alliance (wasmtime)** and **VMware Labs (WebAssembly Language Runtimes)**, with **CPython (PSF)**: the
  sandbox of the tool layer (the code interpreter is off in this build).

## Benchmarks we report

The public corpus of the Qwen3.8-27B quant challenge (StillDeadcode, CC BY 4.0) for the fidelity of the model file;
WikiText-2 (Merity et al., the wikitext-2-raw-v1 test split, CC BY-SA) for the engine's KL checks; our own needle,
code-copy and harness tests.

## Contributors

- **Elhussein Ahmed ([@emourad](https://github.com/emourad))**: the Linux launcher's `LLAMA_SERVER` override (the
  bundled engine stays the default) and the clean stop of the engine and the layer on every failed start (#2).

## This release

Cary Palmer ([professorpalmer](https://github.com/professorpalmer)): the reference forward pass, the Hessian capture
for the looped experts, the expert fits and the bit map, the port of Inkling 2 Mini to our engine, the trellis experts
in the engine, the quantized and tiered K/V cache for its global attention, the repetition guard, the two-slot mode
(the sequence check in the banded attention), the launcher, the saved-chat layer, and the measurements.

No person or company above endorses or sponsors this project.
