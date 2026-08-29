# Talon

Cross-platform acquisition agent for the [Citadel](https://github.com/sltcnb/citadel) pipeline. Collects Windows, Linux and macOS artifacts into one hash-verified ZIP.

Talon runs against a live host, an already-mounted volume, or a raw block device, and writes a triage ZIP. With `--bundle-manifest` it also emits a `manifest.json` describing every artifact it collected, which is what [Sluice](https://github.com/sltcnb/sluice) validates against `bundle_manifest.schema.json`. It can hand the result straight to a Citadel API, or leave it on disk.

## Install

```bash
pip install git+https://github.com/sltcnb/talon
```

Python 3.11 or newer. To build a standalone binary for a host without Python:

```bash
./build.sh     # Linux and macOS
build.bat      # Windows
```

## Collecting

Live host, everything in the default set:

```bash
talon --case-id IR-2026-014 --output ./triage.zip
```

Pick categories:

```bash
talon --case-id IR-2026-014 --collect evtx,registry,mft,prefetch --output ./triage.zip
```

Around 80 artifact categories exist across the three platforms; Windows alone declares 56, grouped in `capabilities.yaml` as Core System, Network and Devices, Security and Defense, Browsers, Email, Messaging, Cloud Sync, Remote Access, Applications and User Data, Infrastructure, Live Triage, and Heavy / Opt-in. Heavy categories (memory, PE, documents, printing) are opt-in.

Dead-box against a mounted image:

```bash
talon --path /mnt/evidence --collect evtx,mft --output ./triage.zip --skip-problematic
```

Linux can mount a raw device itself, with `ntfs-3g` or `dislocker` present:

```bash
talon --disk /dev/sdb1 --bitlocker-key <recovery-key> --output ./triage.zip
```

The BitLocker key stays local and is never written to `config.json`.

## Hunting for files by name

`--fetch` sweeps for files by glob or regex, repeatable and comma-separated:

```bash
talon --fetch 'mimikatz*' --fetch 're:\.(ps1|hta)$' --fetch-root C:/Users --output ./triage.zip
```

Capped by `--fetch-max-files` (200) and `--fetch-max-mb` (100 MB per file).

## Options

| Flag | Purpose |
|---|---|
| `--output`, `-o` | Output ZIP path, or a directory to write into |
| `--case-id` | Case this collection belongs to |
| `--api-url`, `--api-token` | Upload to a Citadel API instead of leaving it on disk |
| `--collect` | Comma-separated category keys |
| `--path` | Already-mounted filesystem root |
| `--disk` | Raw block device, Linux only |
| `--bitlocker-key` | BitLocker recovery key |
| `--fetch`, `--fetch-root`, `--fetch-max-files`, `--fetch-max-mb` | Filename sweep |
| `--bundle-manifest` | Also write a Citadel bundle `manifest.json` |
| `--skip-problematic` | Skip categories known to fail in dead-box mode |
| `--dry-run`, `--verbose` | |

CLI arguments override the embedded `config.json`.

## Remote agent

`collector_server.py` serves collection over gRPC with mutual TLS, so you can drive an endpoint without shipping a shell. Client stubs come from `./generate_stubs.sh`. `secure_upload.py` and `fo_uploader.py` handle chunked, resumable upload to S3 or MinIO; `crypto.py` handles signing.

## Tests

```bash
pip install pytest
pytest -q
```

`pip install -e .` does not currently work. The repo is a flat layout with seven top-level modules and `pyproject.toml` does not declare which to package, so setuptools refuses. Run from a checkout.

## License

[PolyForm Noncommercial 1.0.0](LICENSE). Run, modify and self-host it for any noncommercial purpose. Commercial use needs written authorization from the copyright holder; see [LICENSING.md](LICENSING.md).

This is a source-available license, not an OSI-approved open source license.

## Related

[Citadel](https://github.com/sltcnb/citadel) · [Sluice](https://github.com/sltcnb/sluice) ingests the output · [citadel-contracts](https://github.com/sltcnb/citadel-contracts)
