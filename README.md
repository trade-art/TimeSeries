# RedisTimeSeries Windows 模块构建（MSYS2/MinGW）

这个仓库只做一件事：在 GitHub Actions 的 Windows runner（MSYS2/MinGW64）上编译 `redistimeseries` 模块，并打包产物供 Windows Redis 加载使用。

Windows 版 Redis 可直接使用：`https://github.com/redis-windows/redis-windows`（需要支持 `MODULE LOAD`/`loadmodule`）。

## GitHub Actions

1. 进入 Actions，运行 workflow：`Build RedisTimeSeries Module (Windows)`
2. 下载 artifact（zip）：
   - `redistimeseries.so`（同时复制了一份 `redistimeseries.dll` 方便不同 loader/习惯）
   - 可能需要的 MSYS2 运行时 DLL（若链接为动态依赖）

## Windows 使用示例

1) 启动你的 Windows Redis（来自 `redis-windows/redis-windows` 或你自己的构建）

2) 加载模块（示例）：

```powershell
redis-cli.exe MODULE LOAD (Resolve-Path .\redistimeseries.so)
```

3) 验证 TS：

```powershell
redis-cli.exe TS.CREATE test:ts RETENTION 86400000
redis-cli.exe TS.ADD test:ts * 100
redis-cli.exe TS.GET test:ts
```

## 为什么需要补丁

RedisTimeSeries 构建会用到 `deps/readies/bin/platform` 来识别平台；在 Windows runner 上会走到 `deps/readies/paella/platform.py` 的 windows 分支，而该分支里有 `os.version()` 调用（Python 标准库 `os` 并没有这个 API），导致脚本报 `platform: cannot identify` 并中断构建。

workflow 里对该处做了最小替换：`os.version()` → `platform.version()`。

