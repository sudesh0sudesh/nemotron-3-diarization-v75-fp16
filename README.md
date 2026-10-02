# Nemotron 3 Diarization — FP16 Hexagon V75 model

A **model-only** conversion of [NVIDIA Nemotron 3 Diarization](https://huggingface.co/nvidia/Nemotron-3-Diarization), compiled for the **Arduino VENTUNO Q / Qualcomm IQ-8275** NPU.

## Download

Download the model package and its archive checksum from the [v1.0.0 release](https://github.com/sudesh0sudesh/nemotron-3-diarization-v75-fp16/releases/tag/v1.0.0).

The release contains `nemotron-3-diarization-v75-fp16-model.zip`. The large compiled contexts are release assets rather than Git-tracked files. `SHA256SUMS` verifies individual extracted files; the separate `.sha256` asset verifies the archive.

## Target and model

| Property | Value |
|---|---|
| Base model | `nvidia/Nemotron-3-Diarization` |
| Base revision | `f667ed73aee57d40cc39428eb768b4fd87a0a29e` |
| Architecture | Approximately 100M parameters, 31 Transformer layers, hidden size 512 |
| Accelerator | **QNN HTP / Hexagon V75** |
| SoC model | **82**, IQ-8275 / QCS8275 |
| Tested board | **Arduino VENTUNO Q** |
| Precision | **FP16** neural execution |
| Compilation | Qualcomm AI Hub, QAIRT **2.45** |
| Device runtime | Tested with the existing QAIRT **2.46** HTP runtime |
| Input audio | Mono **16 kHz** |
| Speakers | Up to **eight**, anonymous arrival-order channels |
| Activity resolution | **10 ms**, with overlapping speakers allowed |

These are device/backend-specific QNN contexts, not general ONNX/GGUF models or GPU binaries. Other Hexagon architectures/SoCs require their own compilation.

## Package contents

| File | Purpose |
|---|---|
| `projection.bin` | Learned 8× feature-stacking projection, approximately 1.1 MB |
| `body.bin` | All 31 Transformer layers and learned high-resolution speaker head, approximately 224.2 MB |
| `mel_filterbank.f32` | 128 × 257 float32 mel filterbank |
| `silence.npy` | Learned float32 silence embedding for host-side speaker-cache handling |
| `model.json` | Target, profile, tensor contract, compilation provenance and checksums |
| `validation.json` | Validation protocol, complete-corpus metrics and per-recording differences |
| `LICENSE-WEIGHTS`, `NOTICE.md` | Model licence and notices of origin |
| `SHA256SUMS` | File checksums |

## Integration contract

This package is **not a complete diarization application**. It requires a compatible Qualcomm QNN HTP runtime and a host runner that loads the contexts and implements audio preprocessing, speaker-state management and segmentation.

1. Decode/resample audio to 16 kHz mono.
2. Generate the model's own log-mel features: pre-emphasis 0.97, 512-point FFT, non-periodic 400-sample Hann window, 160-sample hop, constant-zero STFT padding, Slaney mel filters, natural log with `2^-24` guard, no per-feature normalization.
3. Execute `projection.bin` on padded features; pack valid projected frames together with the recording's speaker cache/FIFO on the host.
4. Execute `body.bin` with independent sequence-validity and additive attention masks. Sequence capacities and actual QNN tensor metadata are recorded in `model.json`.
5. Extract only new valid audio predictions and update the **arrival-order speaker cache (AOSC)** and FIFO in FP32 on the host. Preserve first-compression logic, score boosts, learned silence slots and deterministic top-k handling.
6. Keep state across the complete recording, reset between recordings, defer lookahead, clamp EOF and permit simultaneous speaker activity. Do not replace eight independent probabilities with ArgMax.

### Recorded-file profile

All settings below are in **80 ms encoder frames**:

| Setting | Value |
|---|---:|
| Speaker cache | 264 |
| FIFO | 40 |
| Chunk advance | 340 |
| Right context | 40 |
| Configured cache update period | 300, constrained by overflow geometry |

New audio advances by **27.2 seconds**; chunk plus lookahead spans **30.4 seconds**. This is a large-chunk recorded-file profile, not a low-latency microphone configuration. Lengths and EOF masks must be respected even though graph capacities are static.

## Validation

Complete **AMI test SDM**: 16 recordings / 9.062 hours of UEM regions, Array1 channel 1, NVIDIA-linked forced-aligned reference RTTMs, **zero collar and overlap included**. Model activity was segmented using the declared 0.5 threshold.

| Backend | Corpus DER | Speaker-count accuracy |
|---|---:|---:|
| Matched FP32 ONNX CPU reference | 11.3657% | 87.5% |
| **FP16 HTP on Arduino** | **11.6833%** | **87.5%** |

Aggregate FP16 minus reference DER: **+0.3176 percentage points**. Independent NIST `md-eval-22.pl` scoring confirmed the corpus results. Recorded-file NPU diarization processed the full corpus at approximately **126.2× real time**, excluding download/scoring and cold model initialization.

Individual recordings vary: ES2004b had a +6.75-point DER increase dominated by missed speech, while some other meetings improved. Per-recording metrics are included in `validation.json`; this model is not claimed to be FP32-identical on every recording. The benchmark is not an exact reproduction of NVIDIA's BF16 model-card runtime. Speaker channels are anonymous, not biometric identities or real-world names.

## Compilation provenance

- Projection: https://workbench.aihub.qualcomm.com/jobs/jp2r8r74g/
- Transformer/head: https://workbench.aihub.qualcomm.com/jobs/jpe797415/
- Compile precision option: `default_graph_htp_precision=FLOAT16`.

## Licence

Model materials are governed by **OpenMDW 1.1**. See `LICENSE-WEIGHTS` and `NOTICE.md`; retain both with redistributions. The licence does not grant rights to proprietary Qualcomm runtime libraries, which are not included here.
