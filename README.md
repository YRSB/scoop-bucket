# scoop-bucket

Personal [Scoop](https://scoop.sh) bucket with manifests I maintain myself.

## Install

```pwsh
scoop bucket add scoop-bucket https://github.com/YRSB/scoop-bucket
scoop install scoop-bucket/videometricslab
scoop install scoop-bucket/opencode2api
```

Manifests are updated by the workflows below. Some Scoop builds have no
`scoop bucket update`, so refresh the clone by hand when a new manifest lands
(`scoop bucket list` shows the name it was added under):

```pwsh
git -C "$env:USERPROFILE\scoop\buckets\<bucketname>" pull
```

To try a manifest before it is pushed, point Scoop at a local clone instead
(`file:///` is required, a plain path is rejected as an invalid Git URL):

```pwsh
scoop bucket add scoop-bucket file:///C:/path/to/scoop-bucket
```

## Apps

| App | Description |
| --- | --- |
| [videometricslab](https://github.com/4KVCD/VideoMetricsLab) | Measure and compare video encode quality: VMAF, VMAF NEG, PSNR, SSIM, XPSNR, SSIMULACRA2, Butteraugli, ColorVideo VDP |
| [opencode2api](https://github.com/jasonxu114514/opencode2api) | Go gateway for OpenCode Zen with OpenAI and Anthropic API compatibility, key and proxy pooling, automatic routing and a built-in WebUI |

`videometricslab` needs FFmpeg 9 or newer with libvmaf at runtime. It is not
bundled, and it is not a hard dependency of the manifest, because the app asks
for the FFmpeg location on first launch when it is not on `PATH`.

`opencode2api` keeps its configuration in
`~/scoop/persist/opencode2api/config.json`, created from the bundled
`config.example.json` on first install. The shim passes `-config` for that
path, because the gateway reads `config.json` from the current directory and
exits when it is missing. The example listens for the API on `127.0.0.1:8080`
and for the WebUI on `0.0.0.0:8081`; change `server_keys` and `webui.password`
before putting it on a network.

## Add a manifest

1. Copy the shape from an existing file in [`bucket/`](bucket).
2. Keep the required fields: `version`, `description`, `homepage`, `license`,
   `url`, `hash`.
3. Add `checkver` and `autoupdate` so Excavator can bump the version on its own.
4. Mind the details Scoop does not guess for you:
   - Per-platform assets go in an `architecture` block, and `autoupdate` needs
     one block per platform too. There is no `$arch` substitution variable, so a
     single URL cannot be built from it.
   - Keep the extension in `bin` (`app.exe`, not `app`): the target is resolved
     with `Test-Path -PathType leaf`, which does not complete extensions.
   - Arguments use the `[target, name, args]` form and `args` is one string,
     for example
     `["opencode2api.exe", "opencode2api", "-config \"$persist_dir\\config.json\""]`.
   - A ZIP with a single top-level directory is not flattened, so either point
     `bin` at the nested path or move the contents up in an `installer` script.
   - Configuration the app edits belongs in `persist`, or every upgrade drops
     it.
5. Verify locally before committing:

   ```pwsh
   scoop install (Resolve-Path .\bucket\<app>.json)
   scoop uninstall <app>
   ```

   A relative path does not resolve, the manifest path has to be absolute.

## Workflows

- **Excavator** (`.github/workflows/excavator.yml`) runs every 4 hours, checks
  each manifest's `checkver` and opens a pull request with the new version and
  hash.
- **Pull Requests** (`.github/workflows/pull_request.yml`) validates every pull
  request: JSON format, required fields, download hash, `checkver` and
  `autoupdate`.

Workflow permissions are already set to `Read and write`. A fork needs the same
under `Settings > Actions > General > Workflow permissions`, otherwise
Excavator cannot push its update branches.

## License

Manifests in this bucket are released into the public domain, see
[LICENSE](LICENSE). The apps themselves keep their own licenses; check the
upstream project before redistributing anything.