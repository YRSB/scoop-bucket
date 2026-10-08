# scoop-bucket

Personal [Scoop](https://scoop.sh) bucket with manifests I maintain myself.

## Install

```pwsh
scoop bucket add scoop-bucket https://github.com/YRSB/scoop-bucket
scoop install scoop-bucket/videometricslab
```

To try a manifest before it is pushed, point Scoop at a local clone instead:

```pwsh
scoop bucket add scoop-bucket C:\path\to\scoop-bucket
```

## Apps

| App | Description |
| --- | --- |
| [videometricslab](https://github.com/4KVCD/VideoMetricsLab) | Measure and compare video encode quality: VMAF, VMAF NEG, PSNR, SSIM, XPSNR, SSIMULACRA2, Butteraugli, ColorVideo VDP |

`videometricslab` needs FFmpeg 9 or newer with libvmaf at runtime. It is not
bundled, and it is not a hard dependency of the manifest, because the app asks
for the FFmpeg location on first launch when it is not on `PATH`.

## Add a manifest

1. Copy the shape from an existing file in [`bucket/`](bucket).
2. Keep the required fields: `version`, `description`, `homepage`, `license`,
   `url`, `hash`.
3. Add `checkver` and `autoupdate` so Excavator can bump the version on its own.
4. Verify locally before committing:

   ```pwsh
   scoop install .\bucket\<app>.json
   scoop uninstall <app>
   ```

## Workflows

- **Excavator** (`.github/workflows/excavator.yml`) runs every 4 hours, checks
  each manifest's `checkver` and opens a pull request with the new version and
  hash.
- **Pull Requests** (`.github/workflows/pull_request.yml`) validates every pull
  request: JSON format, required fields, download hash, `checkver` and
  `autoupdate`.

Both need `Settings > Actions > General > Workflow permissions` set to
`Read and write permissions`, otherwise Excavator cannot push branches.

## License

Manifests in this bucket are released into the public domain, see
[LICENSE](LICENSE). The apps themselves keep their own licenses; check the
upstream project before redistributing anything.