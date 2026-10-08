# AGENTS.md

本仓库是 Scoop bucket。下面的约定对改动的 agent 和人都适用，动 `bucket/` 里的
manifest 之前请读完。

## 仓库结构

| 路径 | 内容 |
| --- | --- |
| `bucket/<app>.json` | manifest，文件名必须与应用名一致 |
| `.github/workflows/excavator.yml` | 每 4 小时查 `checkver`，有新版开 PR |
| `.github/workflows/pull_request.yml` | PR 校验：格式、必填字段、hash、`checkver`、`autoupdate` |
| `README.md` | 面向使用者的说明；新增应用要在应用表里补一行 |
| `LICENSE` | Unlicense（manifest 本身） |

新增应用只需要动 `bucket/<app>.json` 和 `README.md` 两个文件。

## manifest 必填项

`version`、`description`、`homepage`、`license`、`url`、`hash`，另外必须有
`checkver` 和 `autoupdate`，否则 Excavator 不会帮你升版本。`description` 控制在
180 字符以内是 Main bucket 的习惯（scoop 的 schema 并不强制），图的是搜索结果和
README 的应用表里不换行。

## Scoop 不会替你猜的细节

每一条都在本机实测踩过，改 manifest 前对照检查：

| 事项 | 规则 | 原因 |
| --- | --- | --- |
| 分平台资源 | 放 `architecture` 块，`autoupdate` 里也每平台各写一块 | scoop 里**没有** `$arch` 这个替换变量，一条 URL 拼不出多架构 |
| `bin` 目标 | 必须带扩展名，如 `app.exe` | `create_shims` 用 `Test-Path -PathType leaf` 查找，不补扩展名；写成 `app` 会直接失败并报 `Can't shim 'app': File doesn't exist.` |
| `bin` 参数 | `[target, name, args]` 三元组，`args` 是**一整个字符串** | `shim_def` 按位置解构，第三项整串写进 shim；路径含空格要在字符串里自己加引号 |
| 只有一层目录的 ZIP | scoop 不会自动拍平 | 要么让 `bin` 指向带目录的路径，要么在 `installer` 脚本里把内容上移一层 |
| `installer` 脚本的路径 | 用 `$dir`，不要用 `.` 或相对路径 | 脚本通过 `Invoke-Command` 执行，工作目录是调用者的，不是 app 目录 |
| 用户会改的配置 | 放 `persist` | 否则每次升级都会丢 |
| 升级时重复初始化 | `persist` 里已有文件时，`installer` 里就别再复制 example | `persist_data` 会把新复制的文件改名成 `*.original` 堆在 app 目录里 |

## checkver：注意 tag 的编号体系

GitHub 模式的默认正则，遇到「tag 带前缀」就会失灵。以 `garbro` 的
`GARbro-Mod-1.0.2.2` 为例，实测结果：

| 模式 | 默认正则 | 结果 |
| --- | --- | --- |
| GitHub API（CI 里有 token，走这个） | `(?:v\|V)?([\d.-]+)` | 匹配成 `-`，把 `Mod` 前面的横杠当成了版本号 |
| GitHub HTML | `/releases/tag/(?:v\|V)?([\d.-]+)` | 空，`/releases/tag/` 后面紧跟字母，匹配不上 |

所以这类仓库要自己写 `url` + `regex`：

```json
"checkver": {
    "url": "https://github.com/<owner>/<repo>/releases/latest",
    "regex": "<tag 前缀>(?<version>[\\d.]+)"
}
```

选 tag 时还要分清两套编号：`garbro` 的 **GitHub Releases** 是
`GARbro-Mod-1.0.0.0` … `GARbro-Mod-1.0.2.2`（每个都带 zip），而仓库的 **git tag**
是另一套 `v1.5.x`（继承自上游、没有产物）。只认「每个 release 都带产物」的那套。

## 提交前验证

```pwsh
scoop info (Resolve-Path .\bucket\<app>.json)
scoop install (Resolve-Path .\bucket\<app>.json)
scoop uninstall <app>
```

manifest 路径必须绝对：`.\bucket\app.json` 会被拼成 `.bucketapp`，报
`Could not find manifest`。

装完顺手验证两件事：

- 应用真的能起来：通过 shim 启动一次，确认进程/窗口正常，然后收掉。
- 配置类应用要确认缓存和配置写在 `persist` 里，而不是 app 目录。

autoupdate 用 scoop 自己的 `Invoke-AutoUpdate` 干跑当前版本，`url` 和 `hash`（多架构
就每个架构各验一遍）应当原样复现。推送后在 Actions 里手动跑一次 Excavator，确认
仍然报 `Already up to date`。

## 提交与推送

`git add -A` → 提交（英文祈使句，如 `Add xxx manifest`）→ 推 `origin main`。
推送后可以 `scoop bucket add <name> https://github.com/YRSB/scoop-bucket` 从远程
路径再装一遍做终验。