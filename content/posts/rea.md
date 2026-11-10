# REA（Reverse Engineer Anything）安装与使用指南

> 适用平台：Windows 10/11 x64（本文命令为 PowerShell）
> 实测版本：`rea-agents 6.8.0`（SDK 2.3.1，MCP 工具 229 个 / CLI 命令 175 个 / Prompt 6 个）
> 最后验证：2026-11-10

---

## 一、能力边界

REA 面向**已发布产物与运行时证据**的逆向分析，不替代仓库工具读源码。

| 能用 | 不适用 |
| --- | --- |
| 原生二进制（PE/ELF/Mach-O）静态解析 | PHP / Java / 前端源码工程的常规阅读 |
| 反编译、伪代码、交叉引用（需 Ghidra/IDA/Hopper） | 业务逻辑解读、代码重构 |
| 托管程序（.NET PE/CLI）元数据 | — |
| Electron / ASAR / 前端 JS bundle 分析 | — |
| 打包产物（APK/IPA/MSIX/DMG/PyInstaller）成员清单 | — |
| 固件镜像、信号录音、WASM、SQLite、流量抓包 | — |
| 浏览器运行时取证、HAR/SAZ 分析 | — |

两个前提：

1. **安装 skill 不会注册 MCP**，也不会安装分析引擎；
2. **REA 不下载、不安装任何引擎**，只读取环境变量指向的已有安装。

---

## 二、安装 REA

### 2.1 前置：Node.js

REA 需要 Node.js 24.x（SQLite 检查要求 ≥ 24.15）或 26+。

```powershell
node -v          # 实测 v24.22.1
```

低于要求时用官网 `.msi` 覆盖安装后重开终端。

### 2.2 全局安装

`--allow-scripts=frida` 用于放行 frida 的安装脚本（npm 默认拦截）。

```powershell
npm install -g --allow-scripts=frida rea-agents
rea --version    # 实测 6.8.0
```

### 2.3 注册到 Trae 的 MCP

```powershell
where.exe rea-agents      # 取以 .cmd 结尾的那一行
```

Trae → 设置 → MCP → 手动配置，粘贴（路径换成上一步结果，注意双反斜杠）：

```json
{
  "mcpServers": {
    "rea": {
      "command": "C:\\Users\\<用户名>\\AppData\\Roaming\\npm\\rea-agents.cmd",
      "args": ["mcp"]
    }
  }
}
```

### 2.4 重启并验证

完全退出 Trae 进程后重开，再执行任一验证：

```powershell
rea capabilities --format json      # provider 与 CLI 能力
rea doctor --json                   # 注册、引擎、环境总览
```

MCP 侧调用 `binary_session`（入参 `{}`），返回 `server_identity` 与服务端工具清单。

### 2.5 可选：官方交互式向导

```powershell
npx.cmd rea-agents@latest setup
```

用 ↑↓ 移动、空格选中 Trae、Enter 确认。**即使提示 Not detected 也没关系**，
按 2.3 手动填 JSON 更稳定。

---

## 三、可选分析引擎

按需安装，全部装进 `D:\program`（可替换为任意目录）。装完**必须重启 Trae / 重连 REA MCP**，
因为 MCP 进程只在启动时读取环境变量。

### 3.1 引擎与环境变量对照

| 引擎 | 启用能力 | Windows | 环境变量 |
| --- | --- | --- | --- |
| Ghidra 12.1.x | 反编译、伪代码、xref、函数分析 | ✅ | `GHIDRA_INSTALL_DIR` + `JAVA_HOME` |
| JDK 21+ | Ghidra 运行前提 | ✅ | `JAVA_HOME` |
| WABT 1.0.42 | WASM 校验 / 反汇编 | ✅ | `REA_WABT_BIN_DIRECTORY` |
| Wireshark 4.x | pcap 端点 / 协议分析 | ✅ | `REA_TSHARK_COMMAND` + `REA_CAPINFOS_COMMAND` |
| JADX | Android 代码级 | ✅ | `REA_JADX_MCP_JAR` |
| apktool | APK 资源解码 | ✅ | PATH |
| adb | Android 设备操作 | ✅ | PATH |
| JEB | Android 反编译 | ✅ | `REA_JEB_MCP_URL` |
| IDA | 反编译（商业授权） | ✅ | `REA_IDA_MCP_CONFIG` |
| Kuna | 反编译 | ✅ | `REA_KUNA_COMMAND` |
| Hopper | 反编译 | ❌ 仅 macOS | `HOPPER_LAUNCHER_PATH` |
| Binwalk / Unblob | 固件提取 | ❌ 仅 Linux | `REA_BINWALK_COMMAND` / `REA_UNBLOB_COMMAND` |
| sigrok-cli | 信号解码 | ❌ 仅 Linux | `REA_SIGROK_COMMAND` |
| pwntools | 二进制布局 / core dump | ❌ 仅 Linux x64 | `REA_PWNTOOLS_PYTHON` |
| Wakaru | JS 源码恢复 | ❌ 仅 Linux x64 | `REA_WAKARU_COMMAND` |

### 3.2 Ghidra（反编译 / 伪代码 / 交叉引用）

```powershell
# 1) 装 JDK 21（Ghidra 12.x 要求），解压到 D:\program\jdk-21
# 2) 解压 Ghidra 到 D:\program\ghidra_12.1.4_PUBLIC
# 3) 写环境变量
[Environment]::SetEnvironmentVariable('JAVA_HOME','D:\program\jdk-21\jdk-21.0.13+9','User')
[Environment]::SetEnvironmentVariable('GHIDRA_INSTALL_DIR','D:\program\ghidra_12.1.4_PUBLIC\ghidra_12.1.4_PUBLIC','User')
# 4) 重启 Trae，再验证
rea doctor --json
```

`rea doctor` 中以下检查应全部 `healthy`：
`ghidra`、`ghidra-installation`、`ghidra-version`、`ghidra-headless`、
`ghidra-native_decompiler`、`ghidra-java`、`ghidra-native_authority`。

> 版本要求是 **12.1.x**；装 12.2 及以上是否被接受未验证。

### 3.3 WABT（WASM）

```powershell
# 解压 wabt-1.0.42-windows-x64.tar.gz，可执行文件在 wabt-1.0.42\bin
[Environment]::SetEnvironmentVariable('REA_WABT_BIN_DIRECTORY','D:\program\wabt-1.0.42\wabt-1.0.42\bin','User')
rea inspect-wasm-artifact <文件.wasm> --format json
```

目录须含 `wasm-validate`、`wasm-objdump`、`wasm2wat`。
单次 WAT 输出上限 **32MB（固定预算，无法调高）**，大模块会报 `resource_constraint`。

### 3.4 Wireshark（pcap 分析）

```powershell
# 静默安装到指定目录（/D= 必须在最后且不加引号）
Start-Process -FilePath 'Wireshark-4.6.9-x64.exe' -ArgumentList '/S','/D=D:\program\Wireshark' -Wait
[Environment]::SetEnvironmentVariable('REA_TSHARK_COMMAND','D:\program\Wireshark\tshark.exe','User')
[Environment]::SetEnvironmentVariable('REA_CAPINFOS_COMMAND','D:\program\Wireshark\capinfos.exe','User')
rea inspect-pcap-capture <抓包.pcap> --format json
```

> **两个变量都必须给绝对路径**。只设 `REA_TSHARK_COMMAND` 会报
> `The selected tool capinfos cannot report its version ... Win32 87`。
>
> 注意：这只能启用 **pcap 离线分析**；字段级抓包 `inspect_packet_capture` /
> `inspect_packet_fields` 仍硬性要求 Linux 主机。

### 3.5 国内下载加速

GitHub 直连在实测网络中约 11KB/s，改用镜像：

| 组件 | 来源 |
| --- | --- |
| JDK 21 | `https://mirrors.tuna.tsinghua.edu.cn/Adoptium/21/jdk/x64/windows/` |
| Wireshark | `https://mirrors.tuna.tsinghua.edu.cn/wireshark/win64/` |
| WABT | `https://api.github.com/repos/WebAssembly/wabt/releases/assets/<asset_id>`（带 `Accept: application/octet-stream`） |
| Ghidra | `https://gh-proxy.com/` + 原始 GitHub Release 地址 |

---

## 四、本机实测结果（2026-11-10）

### 4.1 环境

| 项 | 结果 |
| --- | --- |
| 平台 | win32 / x64 |
| Node | v24.22.1 |
| REA | 6.8.0，MCP 已连接（229 工具 / 175 命令） |
| Chrome | 已安装（浏览器取证可用） |

### 4.2 已安装组件

| 组件 | 版本 | 路径 |
| --- | --- | --- |
| Temurin JDK | 21.0.13+9 LTS | `D:\program\jdk-21\jdk-21.0.13+9` |
| Ghidra | 12.1.4 PUBLIC | `D:\program\ghidra_12.1.4_PUBLIC\ghidra_12.1.4_PUBLIC` |
| WABT | 1.0.42 | `D:\program\wabt-1.0.42\wabt-1.0.42\bin` |
| Wireshark | 4.6.9 | `D:\program\Wireshark` |

用户级环境变量：`JAVA_HOME`、`GHIDRA_INSTALL_DIR`、`REA_WABT_BIN_DIRECTORY`、
`REA_TSHARK_COMMAND`、`REA_CAPINFOS_COMMAND`；`Path` 追加 `jdk-21\bin`、WABT `bin`、`D:\program\Wireshark`。

### 4.3 验证证据

| 验证项 | 结果 |
| --- | --- |
| `java -version` | `openjdk 21.0.13 2026-10-20 LTS`（Temurin） |
| `tshark -v` / `capinfos -v` | `4.6.9` |
| `wasm-objdump --version` | `1.0.42` |
| `rea capabilities` | `ghidra.availability.status = available` |
| `rea doctor` | Ghidra 全项 `healthy`；`hopper`/`ida`/`jadx`/`kuna` 为 `missing_analysis_engine` |
| `rea inspect-wasm-artifact MediaInfoModule.wasm` | 校验通过（3.7MB，WAT 全量输出超 32MB 预算） |
| `rea inspect-pcap-capture <测试.pcap>` | 通过（`packet_count: 1`，provider 为 TShark 4.6.9） |

### 4.4 生效条件

- **MCP**：环境变量只在服务进程启动时读取 → 装完引擎必须重启 Trae；
- **CLI**：新开的 PowerShell 自动继承用户级变量。

---

## 五、能力矩阵

### 5.1 已可用（零安装）

| 域 | 工具 |
| --- | --- |
| 会话与证据 | `open_binary`、`binary_session`、`compare_artifacts/bundles/functions`、`export/import_evidence_bundle`、`list_unknowns`、`inspect_analysis_settings` |
| 浏览器取证 | `list_browser_targets`、`inspect_web_page`、`capture_browser_scenario`、`capture_web_screenshot`、`observe_web_execution/session`、`inspect_web_event_listeners`、`compare_web_captures/screenshots` |
| 前端 JS | `analyze_web_bundle`、`export_web_scripts`、`trace_web_module_imports`、`trace_web_source_location`、`inspect_web_network_capture`（HAR/SAZ/mitmproxy/qlog） |
| JS / Electron | `analyze_javascript_application`、`list_javascript_runtime_targets`、`observe_javascript_runtime`、`list_electron_targets`、`inspect_electron_page`、`inspect_electron_fuses` |
| 静态二进制 | `inspect_pe_imports`、`inspect_pe_certificates`、`inspect_pe_resources`、`inspect_pe_load_config`、`inspect_elf_notes`、`inspect_native_exports`、`inspect_byte_entropy`、`inspect_with_objdump` |
| 格式专项 | `inspect_go_binary`、`inspect_rust_binary`、`inspect_python_pyc`、`inspect_erlang_beam`、`inspect_model_artifact`、`inspect_sqlite_database`、`inspect_desktop_application` |
| 抓包 | `inspect_pcap_capture`、`inspect_pcap_client`（需 tshark + capinfos） |
| 动态调试 | `list_frida_devices/processes`、`start_frida_session`、`start_gdb_session`、`execute_rizin_command`、`cutter_command` |

### 5.2 需先 `open_binary`

`inspect_artifact`、`extract_artifact`、`inspect_apk_signature`、`decode_interface_builder`、
`inspect_keyed_archive`、`trace_dylib_resolution`、`open_bundle_resources`。

### 5.3 本机仍不可用

| 能力 | 原因 | 替代方案 |
| --- | --- | --- |
| 字段级抓包 | 仅 Linux（TShark + prlimit） | Chrome 导出 HAR；或 pcap 离线分析 |
| `inspect_binary_layout` / `inspect_recorded_crash` | 仅 Linux x64（pwntools） | `inspect_with_objdump` + `inspect_native_exports` |
| `recover_javascript_sources` | 仅 Linux x64（Wakaru） | `npx js-beautify` / `prettier` |
| 固件 / 信号 / EVM | 仅 Linux 且无对应工具 | 不适用 |
| Hopper | 仅 macOS | Ghidra |
| IDA / Kuna | 商业授权 / 需自带可执行文件 | Ghidra |
| Android（JADX/apktool/adb/JEB） | 未安装，且本项目无 APK | 不适用 |
| macOS 原生工具组 | 仅 macOS | 不适用 |

---

## 六、与本项目（ThinkPHP Web）的适配

| 类别 | 项目内产物 | 结论 |
| --- | --- | --- |
| 浏览器运行时 / 前端 JS | `public/static/**`（Layui、Element、UEditor、forge.min.js） | 保留 |
| HTTP 流量 | Chrome HAR / pcap | 保留 |
| WASM | `public/static/resources/MediaInfoModule.wasm` | 保留（低优先） |
| 原生 PE | `UEditorSnapscreen.exe`、`hiddeninput.exe` | 保留（低优先） |
| Android / iOS / 固件 / 信号 / EVM | 无对应产物 | 排除 |

---

## 七、常用命令速查

```powershell
rea capabilities --format json                 # provider 与能力
rea doctor --json                              # 环境与引擎诊断
rea inspect-wasm-artifact <file.wasm> --format json
rea inspect-pcap-capture <file.pcap> --format json
rea analyze-javascript-application <目录或 asar> --json
rea capture-browser-scenario <scenario.json>   # 需 Chrome，可 launch/connect
```

MCP 常用入口：`binary_session`、`list_browser_targets`、`inspect_web_page`、
`capture_browser_scenario`、`inspect_web_network_capture`、`open_binary`。

---

## 八、故障排查

| 现象 | 原因与处理 |
| --- | --- |
| `npx` 报“禁止运行脚本” | 临时用 `npx.cmd`；或 `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned` 后重开终端 |
| 引擎装好但 REA 仍报 `not_configured` | 环境变量没进 MCP 进程 → 重启 Trae；用 `rea doctor --json` 确认读取值 |
| `capinfos ... Win32 87` | 只给了 tshark 路径 → 补 `REA_CAPINFOS_COMMAND` 绝对路径 |
| `WABT output exceeded 33554432-byte` | 模块过大，32MB 预算固定，改用 `wasm-objdump -h` 或分段分析 |
| GitHub 下载过慢 / 连接被重置 | 用第三章 3.5 的镜像与前缀 |
| MCP 工具列表为空 | 客户端未加载注册 → 检查 JSON、重启客户端；仍无则看客户端 MCP 启动日志 |

---

## 九、卸载

```powershell
npm uninstall -g rea-agents
# 按需删除 D:\program 下的 jdk-21 / ghidra_12.1.4_PUBLIC / wabt-1.0.42 / Wireshark
# 并清理 JAVA_HOME、GHIDRA_INSTALL_DIR、REA_WABT_BIN_DIRECTORY、
# REA_TSHARK_COMMAND、REA_CAPINFOS_COMMAND 及 Path 中的对应项
```
