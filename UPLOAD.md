# Upload instructions

## Hugging Face

The repo already includes an upload helper:

```bash
npm run hf:upload-model -- \
  --repo <namespace>/julia-routing-strix-halo \
  --path models/julia-routing-strix-halo \
  --message "Upload Julia routing Strix Halo checkpoint"
```

Authenticate first if needed:

```bash
hf auth login
```

## GitHub

Recommended: publish this directory in a normal Git repo with Git LFS enabled for `*.safetensors`.

```bash
git lfs install
git lfs track "*.safetensors"
git add .gitattributes models/julia-routing-strix-halo
git commit -m "Add Julia routing Strix Halo checkpoint"
```

This package is intentionally generic. Do not include local harness-specific helper scripts, private paths, tokens, cache directories, or OpenCode-specific config in the public model repo.

## Runtime note

Mention in release posts/descriptions:

- CPU path is tested.
- GPU path should work through the Julia/PyTorch runtime but is not separately benchmarked in this package.
- NPU path is not validated yet. Do not label this release as NPU-ready until an AMD/XDNA NPU runtime is tested.

## Pre-upload checks

Run from the project root:

```bash
python3 scripts/check-model-release.py models/julia-routing-strix-halo
```
