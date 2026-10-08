# scoop-bucket

个人自用的 [Scoop](https://scoop.sh) bucket，收录我自己维护的 manifest。

## 安装

```pwsh
scoop bucket add scoop-bucket https://github.com/YRSB/scoop-bucket
scoop install scoop-bucket/<应用名>
```

manifest 由下面的工作流自动更新。没有 `scoop bucket update` 的 Scoop 版本，用
`scoop bucket list` 查到本地的 bucket 名字后拉一次 clone 即可。

想先验证还没推送的 manifest，把 Scoop 指向本地 clone（必须写 `file:///`）。

## 应用

| 应用 | 说明 | 注意 |
| --- | --- | --- |
| [videometricslab](https://github.com/4KVCD/VideoMetricsLab) | 测量和对比视频编码质量：VMAF、VMAF NEG、PSNR、SSIM、XPSNR、SSIMULACRA2、Butteraugli、ColorVideo VDP | 需要 FFmpeg 9 及以上且带 libvmaf，没有写进 `depends`：`PATH` 里找不到时应用自己弹窗询问 |
| [opencode2api](https://github.com/jasonxu114514/opencode2api) | Go 写的 OpenCode Zen 网关，兼容 OpenAI 与 Anthropic API，带密钥与代理池、自动路由和内置 WebUI | 配置在 `~/scoop/persist/opencode2api/config.json`；示例里 API 监听 `127.0.0.1:8080`、WebUI 监听 `0.0.0.0:8081`，对外暴露前先改 `server_keys` 和 `webui.password` |
| [garbro](https://github.com/crskycode/GARbro) | 视觉小说资源浏览器：打开、解包各种游戏用的压缩包格式 | 跑 `GARbro.GUI.exe`，需要 .NET Framework 4.7.2 及以上（Win10/11 自带）；本 bucket 跟的是 crskycode 的 Mod 版，上游为 [morkt/GARbro](https://github.com/morkt/GARbro) |

## 工作流

- **Excavator**（`.github/workflows/excavator.yml`）每 4 小时查一次 `checkver`，有新版就带上新版本号和新 hash 开 PR。
- **Pull Requests**（`.github/workflows/pull_request.yml`）校验每个 PR：JSON 格式、必填字段、下载得到的 hash、`checkver` 与 `autoupdate`。

本仓库的 workflow 权限已设为 `Read and write`；fork 需要在
`Settings > Actions > General > Workflow permissions` 里设成一样。

## 许可

manifest 以公有领域方式发布，见 [LICENSE](LICENSE)。应用本身各有各的许可，二次分发之前请先看上游项目的许可。

新增或修改 manifest 前，请先读 [AGENTS.md](AGENTS.md)。