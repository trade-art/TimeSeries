# RedisTimeSeries for Windows (MSYS2/MinGW) - 独立构建仓库

这个仓库用于在 **GitHub Actions 的 Windows runner（MSYS2/MinGW64）** 上构建：

- `redis-server.exe`（Redis 源码构建）
- `redistimeseries` 模块（作为 **loadable module**，运行时通过 `MODULE LOAD` / `loadmodule` 加载）

目标：只构建 TimeSeries，避免触发 RedisJSON/RediSearch/RedisBloom 等额外模块带来的 Rust/CMake/依赖问题。

## 用法（GitHub Actions）

1. 新建一个空 GitHub 仓库，把本仓库内容推上去
2. 进入 Actions，运行 workflow：`Build Redis + TimeSeries (Windows)`
3. 下载 artifact：包含 `redis-server.exe`、`redis-cli.exe`、`redistimeseries.so` 及必要 DLL

推送示例（PowerShell）：

```powershell
cd "E:\win TimeSeries\redis-timeseries-windows"
git add .
git commit -m "build: windows redis + timeseries"
git branch -M main
git remote add origin https://github.com/<you>/<repo>.git
git push -u origin main
```

## 本地运行示例（Windows）

假设解压后目录里包含：
- `redis-server.exe`
- `redis-cli.exe`
- `redistimeseries.so`

1) 启动 Redis：

```powershell
.\redis-server.exe --port 6399
```

2) 加载模块：

```powershell
.\redis-cli.exe -p 6399 MODULE LOAD (Resolve-Path .\redistimeseries.so)
```

3) 验证 TS：

```powershell
.\redis-cli.exe -p 6399 TS.CREATE test:ts RETENTION 86400000
.\redis-cli.exe -p 6399 TS.ADD test:ts * 100
.\redis-cli.exe -p 6399 TS.GET test:ts
```

## 版本说明

- Redis 版本：默认 `8.0.0`（workflow 可输入覆盖）
- RedisTimeSeries 版本：默认 `v7.99.91`（workflow 可输入覆盖）

## 为什么要打这个补丁

RedisTimeSeries 的构建会用到 `deps/readies/bin/platform` 来识别平台；在 Windows runner 上该脚本走到 `deps/readies/paella/platform.py` 的 windows 分支，而该分支有 `os.version()` 调用（Python 标准库 `os` 没有这个 API），会导致脚本报 `platform: cannot identify` 并中断后续依赖/输出路径推导。

workflow 里对该处做了最小替换：`os.version()` → `platform.version()`。
