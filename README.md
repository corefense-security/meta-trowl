# meta-trowl

Scan a Yocto image with [Trowl](https://github.com/corefense-security/trowl) once BitBake builds it. 

> Tested on Scarthgap (5.0) and Wrynose with `core-image-minimal`/`qemux86-64`.

See the [Yocto integration guide](https://trowl.dev/integrations/yocto/) for further details.

## Quick start

Initialize your Yocto build environment, then run from the build directory:

```sh
git clone https://github.com/corefense-security/meta-trowl.git ../meta-trowl
bitbake-layers add-layer ../meta-trowl
echo 'INHERIT += "trowl"' >> conf/local.conf
bitbake core-image-minimal
```

## Build behaviour

The class adds native scanner dependencies to each image's `do_build`, leaving
image-generation tasks and outputs unchanged. A `BuildCompleted` handler scans successful
builds and writes reports outside `TMPDIR`.

BitBake builds Trowl and the image, then scans it. Scanning requires `INHERIT += "trowl"`.

Reports go to `${TOPDIR}/trowl/<image>/`: `index.html`, `openvex.json`, `report.json` and `scan.log`.
The scan prints the HTML report path. Open it in a browser.

## Settings

Set these in `conf/local.conf`:

| Variable | Default | Purpose |
|---|---|---|
| `TROWL_SCAN` | `1` | `0` disables scanning and scanner build dependencies |
| `TROWL_REPORT_DIR` | `${TOPDIR}/trowl` | Report directory, with one subdirectory per image |
| `TROWL_ALWAYS` | `0` | `1` rescans after every successful build |
| `TROWL_FSTYPES` | ext4, squashfs, tar, cpio and wic variants | Image formats to try, in order |
| `TROWL_SBOM_NAMES` | [SBOM suffixes](https://trowl.dev/integrations/yocto/#input-discovery) | Ordered SBOM suffixes to try |
| `TROWL_KERNEL_TREE` | empty | Kernel source override |
| `TROWL_KERNEL_CONFIG` | empty | Kernel config override |
| `TROWL_LLM` | `0` | `1` builds `llama-server`, fetches a model and enables the AI summary |
| `TROWL_LLM_MODEL` | `qwen3.5-4b` | Model: `qwen3-4b`, `qwen3.5-4b` or `qwen3.5-9b` |

> A kernel restored from sstate has no source or config. Set them in `local.conf`:

```bitbake
TROWL_KERNEL_TREE = "/path/to/linux"
TROWL_KERNEL_CONFIG = "/path/to/.config"
```

Relative paths resolve from `${TOPDIR}`. Inputs must match the image's kernel, including its commit and patches. The supplied config is used only when the image carries none.

## Caching

`BB_NO_NETWORK = "1"` adds `--no-network` to scans.

> [!NOTE]
> Offline builds need previously cached BitBake downloads and `~/.cache/trowl` populated by an online scan.

## Local AI Summary (Optional)

Set `TROWL_LLM = "1"` to enable local LLM summaries. BitBake builds `trowl-llama-native`
and fetches the model weights into `DL_DIR` (checked against their **SHA-256**).

`llama-server` selects a CPU backend at runtime. Building all CPU variants allows
native builds to be shared through sstate across CPUs.
