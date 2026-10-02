# Model origin and conversion notices

Nemotron 3 Diarization is by **NVIDIA Corporation**.

Original model: https://huggingface.co/nvidia/Nemotron-3-Diarization

Pinned source revision: `f667ed73aee57d40cc39428eb768b4fd87a0a29e`.

This distribution is a conversion of that model for **Qualcomm Hexagon V75 / SoC 82**, compiled in Qualcomm AI Hub using QAIRT 2.45 with FP16 HTP precision. It is not an official NVIDIA or Qualcomm release.

The original model architecture, learned parameters and learned silence embedding remain attributable to NVIDIA. The accompanying mel filterbank was obtained from the OpenMDW-licensed conversion at https://huggingface.co/beshkenadze/nemotron-3-diarization-onnx, revision `1e1e5e2993f9e9558c10a8a72337c769f0ec0516`.

Model materials and this conversion are distributed under **OpenMDW License Agreement 1.1**, included as `LICENSE-WEIGHTS`. Preserve that agreement and these notices of origin with redistributed model materials.

Qualcomm runtime libraries, native loading code, API/service code, datasets, recordings and credentials are not included in this model-only package.
