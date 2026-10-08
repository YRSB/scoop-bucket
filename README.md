# scoop-bucket

个人自用的 [Scoop](https://scoop.sh) bucket，收录我自己维护的 manifest。

## 安装

```pwsh
scoop bucket add scoop-bucket https://github.com/YRSB/scoop-bucket
scoop install scoop-bucket/videometricslab
scoop install scoop-bucket/opencode2api
```

manifest 由下面的工作流自动更新。部分 Scoop 版本没有 `scoop bucket update`，新
manifest 推送之后要手动刷新本地 clone（bucket 在本机叫什么名字，用
`scoop bucket list` 查）：

```pwsh
git -C "$env:USERPROFILE\scoop\buckets\<bucketname>" pull
```

想先在本地验证还没推送的 manifest，把 Scoop 指向本地 clone 即可（必须写成
`file:///` 形式，直接给路径会被当成非法 Git URL）：

```pwsh
scoop bucket add scoop-bucket file:///C:/path/to/scoop-bucket
```

## 应用

| 应用 | 说明 |
| --- | --- |
| [videometricslab](https://github.com/4KVCD/VideoMetricsLab) | 测量和对比视频编码质量：VMAF、VMAF NEG、PSNR、SSIM、XPSNR、SSIMULACRA2、Butteraugli、ColorVideo VDP |
| [opencode2api](https://github.com/jasonxu114514/opencode2api) | Go 写的 OpenCode Zen 网关，兼容 OpenAI 与 Anthropic API，带密钥/代理池、自动路由和内置 WebUI |
| [garbro](https://github.com/crskycode/GARbro) | 视觉小说资源浏览器：打开、解包各种游戏用的压缩包格式 |

`videometricslab` 运行时需要 FFmpeg 9 及以上、且带 libvmaf。它没有写进 manifest
的 `depends`：应用启动时如果 `PATH` 里找不到 FFmpeg，会自己弹窗询问路径。

`opencode2api` 的配置放在 `~/scoop/persist/opencode2api/config.json`，首次安装时由
包内的 `config.example.json` 生成。shim 固定带 `-config` 指向该路径，因为这个网关
默认读**当前目录**下的 `config.json`，读不到就直接退出。示例配置里 API 监听
`127.0.0.1:8080`、WebUI 监听 `0.0.0.0:8081`；对外暴露之前先改掉 `server_keys`
和 `webui.password`。

`garbro` 跑的是 `GARbro.GUI.exe`，需要 .NET Framework 4.7.2 及以上，Windows 10/11
自带，不用装。这里跟的是 crskycode 发布的 Mod 版本（tag 形如
`GARbro-Mod-1.0.2.2`），上游项目是 [morkt/GARbro](https://github.com/morkt/GARbro)，
因为 tag 里带前缀，`checkver` 才没有用 GitHub 模式的默认正则（那个正则会把
`GARbro-Mod-` 里的 `-` 当成版本号）。

## 添加 manifest

1. 照着 [`bucket/`](bucket) 里现有文件的结构写。
2. 必填字段：`version`、`description`、`homepage`、`license`、`url`、`hash`。
3. 加上 `checkver` 和 `autoupdate`，Excavator 才能自己升版本。
4. 注意 Scoop 不会替你猜的细节：
   - 分平台的资源放 `architecture` 块里，`autoupdate` 也要每个平台各写一块。
     Scoop 没有 `$arch` 这个替换变量，别指望用一条 URL 拼出来。
   - `bin` 里要带扩展名（`app.exe` 而不是 `app`）：scoop 用
     `Test-Path -PathType leaf` 找目标，不会自动补扩展名。
   - 带参数用 `[target, name, args]` 三元组，而且 `args` 是**一整个字符串**，
     例如 `["opencode2api.exe", "opencode2api", "-config \"$persist_dir\\config.json\""]`。
   - ZIP 里只有一层目录时 scoop 不会自动拍平：要么让 `bin` 指向带目录的路径，
     要么在 `installer` 脚本里把内容上移一层。
   - 应用会改的配置要放进 `persist`，否则每次升级都会丢。
5. 提交前先本地验证：

   ```pwsh
   scoop install (Resolve-Path .\bucket\<app>.json)
   scoop uninstall <app>
   ```

   相对路径不生效，manifest 路径必须是绝对路径。

## 工作流

- **Excavator**（`.github/workflows/excavator.yml`）每 4 小时跑一次，检查每个
  manifest 的 `checkver`，有新版就带上新版本号和新 hash 开 PR。
- **Pull Requests**（`.github/workflows/pull_request.yml`）校验每个 PR：JSON 格式、
  必填字段、下载得到的 hash、`checkver` 与 `autoupdate`。

本仓库的 workflow 权限已经设成 `Read and write`。fork 的话需要在
`Settings > Actions > General > Workflow permissions` 里设成一样，否则 Excavator
推不了分支。

## 许可

bucket 里的 manifest 以公有领域方式发布，见 [LICENSE](LICENSE)。应用本身各有各的
许可，二次分发之前请先看上游项目的许可。