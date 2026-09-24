# beat-this-onnx

The **Beat This!** `final0` model as ONNX, in the two builds the
[ParaTactus](https://github.com/CattyCathy/ParaTactus) library was developed and measured against. There is no code here:
the weights are Release assets, because 78 MB of them does not belong in git history.

| asset | size | SHA-256 |
| --- | --- | --- |
| `beat-this-final0.onnx` | 78.3 MB | `E5E37B7D1802895E42559C5A1CED7619B1601D257A5C8E167C9D45DE37C1B078` |
| `beat-this-final0-int8.onnx` | 20.9 MB | `58601CFB4517F6765929B7C169E9BC2ADE3A2C64498AB9D64A9BF0D01618F936` |

Check the checksum of anything you download against the row above: a weight file that differs by one byte is a different
model, and the numbers this repository quotes only describe these two files. Anything re-exported or re-quantised from
the checkpoint will have a different checksum, which is expected rather than a problem.

## What the files are

Both are format conversions of the `final0` checkpoint published by the Institute of Computational Perception, JKU Linz,
as part of Beat This! by Francesco Foscarin, Jan Schlüter and Gerhard Widmer (ISMIR 2024,
[arXiv:2407.21658](https://arxiv.org/abs/2407.21658)). `final0` is the checkpoint the library's every measurement uses.

The network takes a **log-mel spectrogram, not audio**, which is why the files contain only the model and why the
library computes the mel itself:

| | |
| --- | --- |
| input | `spectrogram`, `[1, frames, 128]`, float32, frame axis dynamic |
| outputs | `beat`, `downbeat`, `[batch, frames]`, float32 |
| opset | 17 |
| recorded producer | `pytorch 2.8.0`, `ir_version 8` (float); `onnx.quantize` (int8) |

The frontend that has to produce that input is documented in the library's `BeatThisBeatTracker` and has to be
reproduced exactly: 22050 Hz mono, `n_fft` 1024, hop 441 (50 frames per second), 128 mel bands on the Slaney frequency
scale with **un-normalised** triangular filters, and `log1p(1000 * mel)`.

## Which one to take

The int8 build is dynamic quantisation — weights to int8, activations quantised at run time — and is a third of the size
for a difference that was measured rather than assumed. On one track (Designant, 178.4 s), against the beatmap's own
grid:

| | float build | int8 build |
| --- | --- | --- |
| beats after the library's regulariser | 556 | 550 |
| of those, no partner within 20 ms | 56 | 50 |
| median residual to the grid | 29 ms | 29 ms |
| residual, p90 | 141 ms | 131 ms |
| residual, max | 385 ms | 372 ms |
| beats more than 60 ms from the grid | 200 of 556 (36.0%) | 194 of 550 (35.3%) |

So the two are **not interchangeable beat for beat** and the quantised one is **not worse** on the metric that matters
for animation. One track is not a corpus: take the float build if you want the reference artefact, the quantised one if
you want the size, and re-run the comparison on your own material with the library's `samples/ModelCompare` if the
choice matters to you. The library's `docs/model.md` carries this table in context.

## How they were produced

Both files are format conversions of the published `final0` checkpoint, and they are published here because they are the
exact files the measurements were taken with — the checksums above only mean something if the bytes are available.
Neither was exported by the maintainers of this repository, and the quantised one records only that its producer calls
itself `onnx.quantize`: its settings are not recoverable from the file it wrote, so its recipe cannot be stated from the
artefact.

For a file whose provenance is fully documented, export your own. The library's `tools/export_onnx.py` fetches nothing
itself, but given a checkpoint it writes both builds with the settings recorded in the code (`--int8-out` for the
quantised one), and `--verify` compares two ONNX files on the same input. A re-export will not reproduce these checksums
— different tool versions convert differently — so treat it as a new artefact and measure it rather than assuming these
numbers carry over.

## Using them

With ParaTactus the model is a path, not a bundled resource: pass it to `BeatGridProvider`, `BeatThisBeatTracker` or
`StreamingBeatTracker` and the library opens it with ONNX Runtime. Any other ONNX Runtime host can run the file directly
against the input contract above.

## Licence and citation

MIT, copyright (c) 2024 Institute of Computational Perception, JKU Linz, Austria. The full text is in
[`LICENSE`](LICENSE) — an exact copy of the notice that accompanies the checkpoint, and the licence the model files here
are distributed under. It is the authors' licence, not a claim of authorship by this repository; the notice has to be
reproduced with any redistribution, including a quantised build.

> Foscarin, F., Schlüter, J., and Widmer, G. (2024). *Beat This! Accurate Beat Tracking Without DBN Postprocessing*.
> Proceedings of the 25th International Society for Music Information Retrieval Conference (ISMIR).

Upstream project and checkpoint: [CPJKU/beat_this](https://github.com/CPJKU/beat_this).
