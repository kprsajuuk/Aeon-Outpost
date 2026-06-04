# Linux installation options on Windows 11

## User

我新买了一台电脑，是个win11系统。我想在上面装linux，因为听说跑AI的话，最好用linux，所以我想先调研一下，有没有什么靠谱的方案，或者说应该怎么搞比较合适，是磁盘分个区然后把linux单独装个分区会好一点么，双系统的话会运行的慢么，另外有那种不需要重启切换系统的方案么，能否帮我调研一下

## Assistant

正在调研 Windows 11 上运行 Linux 做 AI 的主流方案，并整理对比与建议。

### Tool Web Search

```
{"searchTerm":"WSL2 CUDA GPU AI machine learning 2025 2026 Windows 11","toolCallId":"tool_32723a83-86ff-4bf2-bead-957f07edb7b"}
```

### Tool Web Search

```
{"searchTerm":"Windows 11 dual boot Linux AI development vs WSL2 performance 2025","toolCallId":"tool_07071219-0af9-4039-9ed8-0841b28492b"}
```

### Tool Web Search

```
{"searchTerm":"Windows 11 switch Linux without reboot WSL2 VM Hyper-V GPU passthrough","toolCallId":"tool_0041fe3b-86ec-40a5-9903-506362f2aab"}
```

### Tool Web Search

```
{"searchTerm":"Windows 11 dual boot Ubuntu 2025 secure boot fast startup BitLocker tips","toolCallId":"tool_5d88dd5c-ddca-4e01-a78c-8b488aef7b0"}
```

### Tool Web Search

```
{"searchTerm":"best Linux distro AI machine learning local LLM 2025 Ubuntu","toolCallId":"tool_201de6c4-a48d-4c6c-9be1-28faa362deb"}
```

## Assistant

你的需求可以拆成三个问题：**跑 AI 要不要 Linux**、**怎么装最靠谱**、**能不能不重启切换**。下面按实际场景整理。

---

## 先说结论：怎么选

| 你的目标 | 推荐方案 | 是否需要重启 |
|---------|---------|-------------|
| 先试试、日常开发、跑 Ollama/小模型 | **WSL2** | 否 |
| 认真做 AI（训练、大模型、vLLM 等） | **双系统（原生 Linux）** | 是 |
| 偶尔用 Linux 工具，不想动磁盘 | **WSL2** | 否 |
| 既要 Windows 游戏/办公，又要满血 AI | **双系统 + 平时用 WSL2 过渡** | 切换时重启 |

**一句话：** 大多数新手从 **WSL2** 开始最省心；确认要长期、重度跑 AI，再上 **双系统 Ubuntu**。

---

## 1. 跑 AI 真的需要 Linux 吗？

**不绝对，但 Linux 有明显优势：**

- **生态**：vLLM、TensorRT-LLM、Lambda Stack 等生产级工具以 Linux 为主；NVIDIA CUDA 文档和容器镜像也优先 Ubuntu。
- **性能**：原生 Linux 能直接访问 GPU，没有虚拟化层；WSL2 对 LLM 推理通常能到原生约 **90–100%**，但训练、大批量推理、磁盘 I/O 仍可能差 **5–30%** 甚至更多。
- **显存**：WSL2 会多占一些系统/显存开销（有测试说比原生 Linux 多约 1GB VRAM），大模型时会更明显。
- **Windows 也能跑**：Ollama、LM Studio 在 Windows 上可用，适合入门；要做更严肃的开发/部署，Linux 更合适。

所以：**入门、轻量推理 → Windows/WSL2 够用；长期、重度 AI → 原生 Linux 更值得。**

---

## 2. 四种主流方案对比

### 方案 A：WSL2（最推荐作为起点）

在 Windows 11 里跑真实 Linux 内核，**不用分区、不用重启**。

**优点：**
- 一条命令即可：`wsl --install`（PowerShell 管理员）
- **GPU 直通**：Windows 装 NVIDIA 驱动即可，WSL2 里 `nvidia-smi` 通常能直接用
- 和 Windows 文件互通，VS Code/Cursor 远程开发体验好
- 对 Ollama、llama.cpp、PyTorch 推理，性能接近原生

**缺点：**
- 不是 100% 原生 Linux（Hyper-V 虚拟化）
- 访问 `/mnt/c/` 等 Windows 盘会慢很多，**数据和模型应放在 WSL 的 ext4 里**（如 `~/`）
- 部分高级场景（完整 PCIe 直通、某些内核模块）不支持
- 不要在 WSL 里装 Linux 版 NVIDIA 驱动，会破坏 GPU 直通

**适合：** 学习、开发、本地推理、Docker 跑 AI 容器。

---

### 方案 B：双系统（磁盘分区装 Linux）

从 Windows 分区腾出空间，单独装 **Ubuntu 24.04 LTS**（AI 圈最常用）。

**优点：**
- **性能最好**：GPU、内存、磁盘无虚拟化损耗
- 完整 Linux 环境，所有 AI 工具都能用
- 和 Windows **互不干扰**——开机时选系统，**不会拖慢对方**

**关于「双系统会不会变慢」：**
- **不会。** 同一时刻只跑一个系统，Linux 和 Windows 不共享 CPU/GPU。
- 唯一「损失」是硬盘空间（建议给 Linux **至少 100GB**，重度 AI 建议 **200GB+**）和切换时要重启（约 1–2 分钟）。

**缺点 / 注意事项：**
- 安装前要处理 **BitLocker**（需先解密再分区）
- 关闭 **快速启动**（Fast Startup），否则可能影响 Linux 挂载 Windows 分区
- Ubuntu 24.04+ 一般可保留 **Secure Boot**
- 新电脑建议先 **备份**，用 U 盘安装，选「与 Windows 并存」

**适合：** 长期 AI 工作、训练、大模型、追求满血性能。

---

### 方案 C：虚拟机（Hyper-V / VMware / VirtualBox）

在 Windows 里跑完整 Linux 虚拟机。

**优点：** 不用重启，可随时开关  
**缺点：** GPU 支持差；Hyper-V 的 GPU 分区（GPU-PV）配置复杂，性能仍不如 WSL2 或双系统  
**适合：** 学 Linux、跑无 GPU 任务；**不推荐作为 AI 主力方案**

---

### 方案 D：Linux 为主 + 偶尔用 Windows

若 AI 是主用途，也可以反过来：主盘装 Ubuntu，需要时用虚拟机跑 Windows。这是进阶路线，新手一般不优先考虑。

---

## 3. 有没有「不重启切换系统」的方案？

**有，但都有取舍：**

| 方案 | 能否不重启 | GPU 性能 | 真实 Linux 程度 |
|-----|-----------|---------|----------------|
| **WSL2** | ✅ | 很好（~90–100% 推理） | 高（真内核，有虚拟化层） |
| **Hyper-V + GPU-PV** | ✅ | 中等，配置难 | 高，但折腾多 |
| **普通虚拟机** | ✅ | 差或无 GPU | 完整 |
| **双系统** | ❌ 需重启 | 最好 | 100% 原生 |

**结论：** 不想重启又想跑 AI，**WSL2 是目前最成熟、最省心的方案**。没有「完全不重启又和原生 Linux 一样快」的通用方案；Windows 下无法做真正的 PCIe GPU 直通到 Linux 虚拟机。

---

## 4. 针对新 Win11 电脑的具体建议

### 第一步：先确认硬件

- **NVIDIA 显卡**：AI 最省心（CUDA 生态最好）
- **AMD 显卡**：Linux 可用 ROCm，但体验和文档不如 NVIDIA
- **仅核显 / 无独显**：只能 CPU 推理，体验会差很多
- BIOS 里确认 **虚拟化（Intel VT-x / AMD-V）** 已开启（WSL2 需要）

### 第二步：推荐路径（由易到难）

```
阶段 1（今天就能做）
├── 安装 WSL2：wsl --install
├── Windows 装最新 NVIDIA 驱动
├── WSL 里装 Ubuntu 24.04
└── 试跑：Ollama / PyTorch + nvidia-smi

阶段 2（确认长期做 AI 后）
├── 备份重要数据
├── 关闭 BitLocker、快速启动
├── 磁盘管理里给 Linux 腾出 100–200GB
├── U 盘安装 Ubuntu 24.04 LTS（选「与 Windows 并存」）
└── 需要满血性能时重启进 Linux
```

### 第三步：发行版选择

- **首选：Ubuntu 24.04 LTS** — 驱动、CUDA、文档、社区支持最全
- 进阶可选：Pop!_OS（NVIDIA 版省心）、Fedora（较新驱动）
- 新手不建议一开始用 Arch/NixOS（学习成本高）

### 第四步：磁盘怎么分

- **只 WSL2**：不用单独分区，Linux 在虚拟磁盘里（默认约几十 GB，可在 `.wslconfig` 里调内存/CPU）
- **双系统**：在 Windows「磁盘管理」里 **压缩卷**，留出 **未分配空间** 给 Ubuntu 安装程序，不要自己先建 ext4 分区
- **空间建议**：
  - WSL2：模型和数据可能占很多，注意 WSL 虚拟盘大小
  - 双系统 Linux：**100GB 起步**，要存多个大模型建议 **200GB+**

---

## 5. 性能参考（心里有数即可）

不同测试结论会有差异，大致范围：

| 场景 | WSL2 vs 原生 Linux |
|-----|-------------------|
| Ollama / llama.cpp GPU 推理 | ~90–100% |
| 大模型训练 | 原生快约 10–30%+ |
| 磁盘读写（WSL 内 ext4） | ~80–95% |
| 磁盘读写（跨 `/mnt/c`） | 明显慢，约 30–50% |
| 显存占用 | WSL2 略多 |

**实践建议：** 在 WSL2 里把项目、数据集、模型都放在 Linux 家目录，不要放在 Windows 盘。

---

## 6. 我的综合建议

**如果你刚买电脑、还在调研：**

1. **先装 WSL2**，在 Windows 里把 AI 环境跑通（1–2 小时），确认显卡、驱动、常用框架都没问题。  
2. **用 1–2 周** 看是否经常需要 Linux、是否觉得 WSL2 性能或显存不够。  
3. 若确定长期做 AI，再 **加双系统 Ubuntu**，Windows 保留办公/游戏，AI 重度任务重启进 Linux。

这样风险小、不耽误用新电脑，也避免一上来就分区装双系统却发现自己其实 WSL2 就够用。

---

## 7. 常见坑（提前知道）

- WSL2 里执行 `sudo apt install nvidia-driver-*` → **会破坏 GPU**，只装 `cuda-toolkit`，驱动靠 Windows  
- 双系统前没关 BitLocker → 安装/分区可能失败  
- 没关快速启动 → Linux 可能读不了 Windows 分区  
- 模型放在 `/mnt/c/` → 加载慢很多  
- 新电脑若只有一块 SSD，双系统前 **务必备份**

---

如果你愿意补充这几项，我可以帮你缩成更具体的「安装清单」：

1. **显卡型号**（如 RTX 4060 / 核显）  
2. **内存和硬盘大小**（如 32GB + 1TB）  
3. **主要用途**（本地聊天模型 / 训练 / 开发部署）  
4. **是否还要在 Windows 上玩游戏**

有这些信息，可以直接给你定「只 WSL2」还是「WSL2 + 双系统」以及分区大概分多少。

## User

很好。只是有一点，我确实是个amd的显卡，我这次买的是amd的统一内存架构，128g。所以更需要linux折腾。在这里我还有两个更实际一点的问题
首先，你所推荐的那个乌班图，我怎么下载？我肯定需要个镜像吧无论如何第一步，去哪里是正规渠道
第二，我这台笔记本平时我想启动着放在那儿当个服务器，所以我想能很方便的远程他。就内网环境。所以我在想这个要怎么搞，windows环境下，我知道windows有个远程桌面，那个内网能用吧？我的常用的pc也是windows，所以如果可用的话也很方便；然后就是linux下，我现在还不知道有什么远程桌面的软件，所以也想你给我讲一下。我希望最好的情况是，都配置好了以后，可以所有操作都远程，除了开机，甚至连切换系统都远程操作，比如配置好下次启动自动切linux，然后重启，重启完我直接连进linux的远程控制方案

## Assistant

这两个问题都很实际。我先查 Ubuntu 官方下载渠道，以及 AMD 统一内存 + 内网远程（含双系统切换）的可行做法。

### Tool Web Search

```
{"searchTerm":"Ubuntu 24.04 LTS official download ISO site:ubuntu.com","toolCallId":"tool_81777d01-61cb-4d07-8f26-cb3474890aa"}
```

### Tool Web Search

```
{"searchTerm":"AMD Strix Halo unified memory 128GB Linux ROCm 2025 2026","toolCallId":"tool_35eaa486-debf-4acf-954f-23ade797925"}
```

### Tool Web Search

```
{"searchTerm":"remote reboot switch dual boot Windows Linux grub-reboot SSH","toolCallId":"tool_ba02f73f-388c-4772-875c-67da8035ce8"}
```

### Tool Web Search

```
{"searchTerm":"Windows 11 Home enable remote desktop RDP host incoming connection","toolCallId":"tool_6a53395e-2fc6-4eca-a267-5e21d1a9353"}
```

### Tool Web Search

```
{"searchTerm":"Linux remote desktop xrdp RustDesk Sunshine SSH headless server 2025","toolCallId":"tool_1ac61e15-ad3c-4b84-85e5-b04ce9d907d"}
```

## Assistant

你的 AMD 128GB 统一内存架构，选 Linux 是对的——这类机器在 Linux 上才能把大内存真正分给 GPU 跑大模型。下面分两部分说：**镜像怎么下**，以及**内网远程 + 远程切系统**怎么搞。

---

## 一、Ubuntu 镜像：正规下载渠道

### 官方入口（认准这两个域名）

| 用途 | 地址 |
|-----|------|
| 官网导航页 | https://ubuntu.com/download |
| 直接下载目录 | https://releases.ubuntu.com/noble/ |

**不要**从第三方「镜像站」「百度网盘」「不明论坛」下载，容易带篡改或捆绑。

### 你该下哪个文件？

你的笔记本是 x86_64（AMD64），常见选择：

| 镜像 | 大小 | 适合场景 |
|-----|------|---------|
| `ubuntu-24.04.x-desktop-amd64.iso` | ~6.2GB | 带图形界面，远程桌面、日常折腾 |
| `ubuntu-24.04.x-live-server-amd64.iso` | ~3.2GB | 纯服务器，无桌面，更轻 |

**建议：** 做 AI 服务器、又要远程桌面 → 选 **Desktop 版**。装好后可以当服务器用，GUI 远程也方便。

直接链接示例（版本号会随更新变，以目录页为准）：
- Desktop：https://releases.ubuntu.com/noble/ubuntu-24.04.4-desktop-amd64.iso
- Server：https://releases.ubuntu.com/noble/ubuntu-24.04.4-live-server-amd64.iso

### 下载后建议校验（可选但推荐）

同目录下有 `SHA256SUMS` 和 `SHA256SUMS.gpg`。在 Windows PowerShell 里：

```powershell
# 下载 ISO 后，在 ISO 所在目录执行
Get-FileHash .\ubuntu-24.04.4-desktop-amd64.iso -Algorithm SHA256
```

把算出来的哈希和 `SHA256SUMS` 里对应那一行对比，一致即可。

### 做成 U 盘启动盘

在 Windows 上常用 **Rufus**（https://rufus.ie/）：
1. 插入 8GB+ U 盘（会清空数据）
2. 选刚下的 ISO
3. 分区类型选 **GPT**，目标系统选 **UEFI**
4. 开始写入

---

## 二、AMD 128GB 统一内存：顺带提醒

你这类机器（多半是 Strix Halo / Ryzen AI Max 395 一类）在 Linux 上要额外注意：

- 需要较新内核（社区经验：**6.18.4+**）和匹配的 **ROCm 7.2+**
- 要在 GRUB 里加启动参数，才能把 128GB 里的大部分分给 iGPU，例如：
  ```
  iommu=pt amdgpu.gttsize=126976 ttm.pages_limit=32505856
  ```
- Ubuntu 24.04 默认内核可能偏旧，装完后很可能要升级 HWE 内核，或考虑 **Fedora 43** 这类内核更新的发行版

这和「下镜像」不冲突——**先装 Ubuntu 24.04 LTS 入门没问题**，但 AI 满血跑起来还要再折腾内核/ROCm。社区有现成指南，例如 [Framework Strix Halo LLM Setup](https://github.com/Gygeek/Framework-strix-halo-llm-setup) 和 [amd-strix-halo-toolboxes](https://github.com/kyuz0/amd-strix-halo-toolboxes)。

---

## 三、内网远程：Windows 端

### Windows 远程桌面（RDP）内网能用吗？

**能**，但有个前提：

| Windows 版本 | 能否被远程连接（当 Host） |
|-------------|-------------------------|
| **Win11 Pro / Enterprise / Education** | ✅ 原生支持 |
| **Win11 Home（家庭版）** | ❌ 不能当 RDP 服务端 |

很多笔记本预装的是 **Home 版**。如果是 Home：
- 你这台**不能**被别的电脑用「远程桌面连接（mstsc）」连进来
- 你仍然可以用 mstsc **去连别的 Pro 机器**
- 替代方案见下文

### 若你是 Win11 Pro：开启 RDP

1. 设置 → 系统 → 远程桌面 → 打开
2. 记下笔记本内网 IP（`ipconfig`，如 `192.168.1.100`）
3. 常用 PC 上：`Win + R` → `mstsc` → 输入 IP → 登录

内网一般**不用**端口转发；确保两台在同一局域网、防火墙允许即可。

### 若你是 Win11 Home：替代方案

| 方案 | 特点 | 是否推荐 |
|-----|------|---------|
| **RustDesk** | 开源、跨平台、内网 P2P、配置简单 | ⭐ 最省心 |
| **Parsec / Sunshine+Moonlight** | 低延迟，偏图形/GPU | 适合要流畅桌面时 |
| **升级到 Win11 Pro** | 原生 RDP，和常用 PC 体验一致 | 长期用值得考虑 |
| RDP Wrapper 等破解 | 非官方，更新易坏 | 不推荐 |

---

## 四、Linux 远程：有哪些方案？

按「AI 服务器」用途，建议**分层配置**，不要只依赖一种：

### 第一层：SSH（必装，AI 服务器核心）

```bash
sudo apt install openssh-server
sudo systemctl enable --now ssh
```

- 用途：命令行、跑模型、看日志、传文件（`scp`/`rsync`）
- 内网连接：`ssh 用户名@192.168.1.100`
- Windows 客户端：PowerShell 自带 `ssh`，或 **Windows Terminal**、**MobaXterm**

**做 AI 服务器，80% 的操作其实 SSH 就够了**（Ollama、Docker、训练脚本等）。

### 第二层：图形远程（需要看桌面时）

| 方案 | 客户端（你的 Win PC） | 特点 |
|-----|---------------------|------|
| **xrdp** | 自带「远程桌面连接 mstsc」 | 和 Windows RDP 同一套客户端，内网体验熟悉 |
| **GNOME Remote Desktop** | mstsc 或「远程桌面」应用 | Ubuntu 24.04 自带，Wayland 下较新 |
| **RustDesk** | RustDesk 客户端 | 跨 Win/Linux，Home 版 Windows 也能用 |
| **VNC（TigerVNC/TightVNC）** | VNC Viewer | 老牌，画质/性能一般 |
| **Sunshine + Moonlight** | Moonlight | 低延迟，配置稍复杂 |

**推荐组合（内网 + 你的 Win PC）：**

```
SSH        → 日常 AI 运维（必装）
xrdp       → 需要 Linux 图形界面时用 mstsc 连
RustDesk   → 跨系统统一方案，Home 版 Windows 也适用
```

xrdp 安装示例（Ubuntu Desktop）：

```bash
sudo apt install xrdp
sudo systemctl enable --now xrdp
# 防火墙若开启：sudo ufw allow 3389/tcp
```

然后从 Windows `mstsc` 连 `192.168.1.100:3389`。

### 第三层：无头服务器（合盖/不接显示器）

笔记本当 7×24 服务器时要处理：

1. **禁用睡眠/休眠**（Windows 和 Linux 都要设）
2. **合盖不休眠**：电源选项里改「合盖 → 不采取任何操作」
3. **固定内网 IP**：路由器里做 DHCP 保留，或手动设静态 IP
4. 无显示器时 Linux 远程桌面可能需要 **dummy display**（虚拟显示）或 **HDMI 欺骗器**（十几块钱），否则部分远程方案会黑屏

---

## 五、能否远程切换系统？——可以，但**必须重启**

没有「不重启就在 Win 和 Linux 之间热切换」的方案。能做到的是：

> **远程下发「下次启动进哪个系统」→ 远程重启 → 等 1～2 分钟 → 连对应系统的远程**

### 推荐整体架构

```
平时默认启动：Linux（AI 服务器）
     │
     ├─ SSH :22        → 命令行运维
     ├─ xrdp :3389     → 图形远程
     │
     ├─ 需要从 Linux 切到 Windows：
     │     SSH 执行 reboot-to-windows.sh → 重启 → mstsc/RustDesk 连 Windows
     │
     └─ 需要从 Windows 切到 Linux：
           PowerShell 执行 reboot-to-linux.ps1 → 重启 → SSH/xrdp 连 Linux
```

### 从 Linux 远程切到 Windows

SSH 进 Linux 后：

```bash
# /etc/default/grub 里需有 GRUB_DEFAULT=saved
sudo grub-reboot "Windows Boot Manager"   # 或菜单编号，如 sudo grub-reboot 2
sudo reboot
```

也可以写脚本 `reboot-to-windows.sh`，远程一条命令搞定。

### 从 Windows 远程切到 Linux

管理员 PowerShell：

```powershell
# 先看 UEFI 启动项
bcdedit /enum firmware

# 设置下次启动进 Ubuntu/GRUB（GUID 从上面输出里找）
bcdedit /set {fwbootmgr} bootsequence {xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx}
shutdown /r /t 0
```

同样可以保存为 `reboot-to-linux.ps1`，远程桌面里双击或计划任务执行。

### 更省心的策略

| 策略 | 说明 |
|-----|------|
| **默认永远进 Linux** | AI 服务器场景最合理，Windows 只是偶尔用 |
| **Linux 里一键切 Windows** | `grub-reboot` 只影响下一次启动，进 Windows 后再重启会回 Linux 默认项 |
| **Windows 里一键切 Linux** | `bcdedit bootsequence` 同理，只影响下一次 |
| **开机仍要物理/远程唤醒** | 笔记本可开 **Wake-on-LAN**，路由器或常用 PC 发魔术包唤醒 |

### 做不到的部分

- ❌ 不重启切换系统
- ❌ 远程按电源键开机（除非 BIOS 支持 WoL + 网卡一直供电）
- ❌ 在 GRUB 菜单界面远程「点选」（除非装 IPMI/iKVM，笔记本一般没有）

---

## 六、针对你场景的推荐配置清单

### 阶段 1：Windows 先跑起来（今天就能做）

1. 查 Windows 版本：`Win + Pause` 看是 Home 还是 Pro  
2. **Pro** → 开 RDP；**Home** → 装 RustDesk（服务端+客户端）  
3. 电源：合盖不休眠、禁止自动睡眠  
4. 路由器：给笔记本 IP 做 DHCP 绑定（如 `192.168.1.100`）

### 阶段 2：装 Ubuntu 双系统

1. 从 https://releases.ubuntu.com/noble/ 下 Desktop ISO  
2. Rufus 写 U 盘，安装时选「与 Windows 并存」  
3. 装完后在 Linux 里：
   - `openssh-server`（必装）
   - `xrdp` 或启用 GNOME Remote Desktop
   - 设 GRUB 默认进 Linux，`GRUB_DEFAULT=saved`

### 阶段 3：远程切换脚本

- Linux：`~/bin/reboot-to-windows.sh`  
- Windows：`C:\Scripts\reboot-to-linux.ps1`  
- 两边都测一遍：远程执行 → 重启 → 能否连上

### 阶段 4：AMD AI 专项（Linux 里）

- 升级内核、加 GRUB 内存参数  
- 装 ROCm / 用社区 toolbox 容器  
- 参考：https://github.com/Gygeek/Framework-strix-halo-llm-setup  

---

## 七、方案对比小结

| 需求 | 最佳方案 |
|-----|---------|
| 镜像下载 | https://releases.ubuntu.com/noble/ 官方 Desktop ISO |
| Win PC 远程连 Win 笔记本 | Pro 用 RDP；Home 用 RustDesk |
| Win PC 远程连 Linux | SSH（日常）+ xrdp/RustDesk（图形） |
| 7×24 当 AI 服务器 | 默认进 Linux + SSH + 禁睡眠 |
| 远程切系统 | `grub-reboot` / `bcdedit bootsequence` + 远程重启 |
| AMD 128GB 跑 AI | 原生 Linux + 新内核 + ROCm，比 WSL2 更合适 |

---

如果你愿意补充这三项，我可以把步骤再缩成一份「按你机器定制」的检查清单（包括该下 Desktop 还是 Server、Home 还是 Pro 走哪条远程路线）：

1. 笔记本具体型号（或 CPU 是否 Ryzen AI Max / Strix Halo）  
2. 预装 Windows 是 **Home 还是 Pro**  
3. 主要远程用途：**纯命令行跑模型**，还是也要**远程看 Linux 桌面**

## User

我还有两个问题，第一，我看ubuntu官网他现在最新的LTS是26.04，这个会更好吗？
第二，我希望的远程桌面是那种，即使断网也能用，就是纯内网环境不需要互联网就能用的方案，我知道win远程桌面可以，其他的方案可以吗？
最后补充问一下，我的好像确实是家庭版，我该怎么升级成旗舰版呢？

## Assistant

这三个问题都很具体，我先查 Ubuntu 26.04 LTS 的现状、纯内网远程方案，以及 Win11 家庭版升级 Pro 的官方途径。

### Tool Web Search

```
{"searchTerm":"Ubuntu 26.04 LTS release Noble vs 24.04 AMD ROCm Strix Halo 2026","toolCallId":"tool_9bce3256-fe18-48bc-b041-5dc8cfc6ea3"}
```

### Tool Web Search

```
{"searchTerm":"RustDesk xrdp LAN only offline no internet self-hosted 2025","toolCallId":"tool_488d9e4c-a370-4401-8383-9628b6d05b2"}
```

### Tool Web Search

```
{"searchTerm":"Windows 11 Home upgrade to Pro official Microsoft Store 2025 2026","toolCallId":"tool_005a150c-1102-45d6-9121-03ebdce4b1d"}
```

## Assistant

这三个问题可以分开看。另外先澄清一点：Windows 11 没有「旗舰版」，对应你说的应该是 **专业版（Pro）**——Remote Desktop 主机功能就在这一档。

---

## 一、Ubuntu 26.04 LTS 比 24.04 更好吗？

**对你这台 AMD 128GB 统一内存机器，26.04 更值得优先考虑。**

| 对比项 | Ubuntu 24.04 LTS | Ubuntu 26.04 LTS |
|-------|------------------|------------------|
| 支持到 | 2029 年 4 月 | 2031 年 4 月 |
| 默认内核 | 较旧，Strix Halo 需装 HWE 内核 | 较新，RDNA3.5 / gfx1151 支持更好 |
| ROCm | 需手动加 AMD 源装 7.2+ | 官方源自带 ROCm 7.1（`apt install rocm`） |
| Strix Halo (gfx1151) | 能跑，但要额外折腾内核 | Canonical CI 已纳入测试 |
| 社区教程 | 更多 | 还少一些，但对你这类新硬件更对口 |

### 关键细节：26.04 也不是「装完就能满血跑 AI」

Ubuntu 26.04 自带的是 **ROCm 7.1**，而 AMD 官方文档对 Strix Halo（gfx1151）的结论是：

- **ROCm 7.2+** → 稳定可用  
- **ROCm 7.1.x** → 对 RDNA3.5 APU **不算稳定组合**

所以即使用 26.04，你仍可能要：

1. 卸掉 Ubuntu 自带的 ROCm 7.1  
2. 按 AMD 官方文档手动装 **ROCm 7.2+**（目前文档主要写的是 24.04 的安装步骤，但社区反馈 26.04 也能用）  
3. 在 GRUB 里加统一内存参数（`amdgpu.gttsize=126976` 等）

### 实际建议

```
新机器 + AMD Strix Halo 128GB  →  优先装 Ubuntu 26.04 LTS Desktop
装完后                          →  升级/替换到 ROCm 7.2+
                                 →  配置 GRUB 内存参数
                                 →  参考社区 Strix Halo 指南
```

若你更在意「教程多、踩坑资料多」，24.04 + HWE 内核 + 手动 ROCm 7.2 也完全可以，只是初始折腾会多一点。

**下载地址（26.04）：**

- 官网：https://ubuntu.com/download/desktop  
- 直链目录：https://releases.ubuntu.com/26.04/（以页面实际文件名为准）

---

## 二、纯内网、断网也能用的远程桌面——哪些可以？

你的需求是：**只走局域网，不依赖互联网**。下面按是否满足来分。

### 完全满足（推荐）

| 方案 | 协议/端口 | 断网能否用 | 说明 |
|-----|----------|-----------|------|
| **Windows RDP**（需 Pro） | TCP 3389 | ✅ | 纯内网直连 IP，和互联网无关 |
| **SSH** | TCP 22 | ✅ | 命令行远程，AI 服务器主力 |
| **xrdp**（Linux） | TCP 3389 | ✅ | Windows 用 `mstsc` 直接连 Linux |
| **GNOME Remote Desktop**（Ubuntu 自带） | RDP | ✅ | 设置里打开「远程桌面」即可 |
| **VNC**（TigerVNC 等） | TCP 5900+ | ✅ | 老牌方案，画质一般 |
| **RustDesk 直连 IP 模式** | TCP 21118 | ✅ | 需手动开启「Direct IP Access」 |

### 默认不行，要改配置才行

| 方案 | 默认情况 | 纯内网用法 |
|-----|---------|-----------|
| **RustDesk 默认模式** | 走公网 ID/中继服务器 | 需开启 **Direct IP Access**，或在内网自建 hbbs/hbbr |
| **Parsec** | 要登录账号、走云端 | 不适合纯离线内网 |
| **TeamViewer / AnyDesk** | 通常要连厂商服务器 | 不适合纯离线内网 |
| **ToDesk / 向日葵** | 同上 | 不适合纯离线内网 |

### RustDesk 纯内网正确用法

RustDesk 官方 FAQ 明确说：**无互联网的内网可以用 Direct IP Access**。

配置要点：

1. **被控端**（笔记本）：设置 → 安全 → 开启 **Direct IP Access**，设固定密码  
2. **控制端**（常用 PC）：在 ID 框输入内网 IP，如 `192.168.1.100:21118`  
3. 注意：直连模式**默认不加密**；若在意安全，可在内网另一台机器自建 RustDesk 中继（仍不需要互联网）

### 针对你的最佳纯内网组合

```
┌─────────────────────────────────────────────────┐
│  常用 Windows PC                                 │
│    ├─ mstsc ──────→  Win11 Pro 笔记本 :3389     │  ← 升级 Pro 后
│    ├─ mstsc ──────→  Linux xrdp        :3389     │
│    ├─ ssh ────────→  Linux             :22       │  ← AI 日常运维
│    └─ RustDesk IP →  任意系统          :21118    │  ← Home 版 Win 备用
└─────────────────────────────────────────────────┘
         全部只走 192.168.x.x，不需要互联网
```

**结论：** 除了 Windows RDP（要 Pro），**xrdp、SSH、GNOME Remote Desktop、RustDesk 直连 IP** 都能纯内网用。选 1～2 种即可，不必全装。

---

## 三、Win11 家庭版怎么升级到专业版？

Windows 11 版本对应关系：

| 中文常见叫法 | 实际版本 | 能否当 RDP 被控端 |
|------------|---------|-----------------|
| 家庭版 | Home | ❌ |
| 专业版 | Pro | ✅ |
| 企业版 | Enterprise | ✅ |
| 旗舰版 | Win11 里**不存在** | — |

你要的应该是 **专业版（Pro）**。

### 方法一：Microsoft Store 在线升级（最省事）

1. `Win + I` → **系统** → **激活**  
2. 展开 **升级 Windows 版本**  
3. 点 **打开 Microsoft Store** / **升级到 Pro**  
4. 按提示购买并完成升级（约 **$99 美元**，中国区以商店显示为准）  
5. 重启后确认版本变为「Windows 11 专业版」

也可直接打开：https://www.microsoft.com/store/productId/9P3JFR0CLL3G

**特点：**

- 保留所有文件、软件、设置  
- 不需要重装系统  
- 需要联网、Microsoft 账户（Store 购买时）  
- 数字许可证绑定设备/账户

### 方法二：输入产品密钥（可离线激活）

若已有 **Win11 Pro 零售密钥**（25 位）：

1. **设置** → **系统** → **激活**  
2. **更改产品密钥**  
3. 输入密钥 → 下一步 → 重启  

这种方式**不要求 Microsoft 账户**，适合内网/离线环境。

密钥来源建议：

- ✅ Microsoft 官方 Store、授权经销商  
- ⚠️ 第三方「几块钱密钥」——来源不明，有封禁风险，不建议

### 方法三：若设备已有 Pro 数字许可证

同一 Microsoft 账户曾在别的机器买过 Pro 时：

1. **设置** → **系统** → **激活**  
2. 运行 **激活疑难解答**  
3. 登录账户，看能否识别已有 Pro 许可证  

### 升级前注意

1. **先确认 Home 已激活**（设置 → 激活里应显示「已激活」）  
2. **备份重要数据**（升级一般无损，但保险起见）  
3. 升级完成后：**设置 → 系统 → 远程桌面 → 开启**  
4. 防火墙放行 3389，内网用 `mstsc` 测一次

### 值不值得为 RDP 升级？

| 情况 | 建议 |
|-----|------|
| 主要远程 Linux 跑 AI | **不必**为 RDP 升级，SSH + xrdp 够用 |
| 也要远程 Windows 桌面，且想要原生 mstsc | **值得**升级 Pro |
| 不想花钱 | Linux 用 xrdp；Windows 用 RustDesk Direct IP（Home 也能当被控端） |

---

## 四、综合建议（针对你的三个问题）

1. **系统版本**：新装选 **Ubuntu 26.04 LTS**；装完再升到 ROCm 7.2+，并配置统一内存 GRUB 参数。  
2. **纯内网远程**：  
   - Linux：**SSH（必装）+ xrdp**  
   - Windows：升级 **Pro** 后用 RDP；不升级就用 **RustDesk Direct IP**  
   - 以上均不依赖互联网  
3. **Win 升级**：**设置 → 激活 → 升级到 Pro**，或输入正版 Pro 密钥；没有「旗舰版」这一档。

如果你愿意补充笔记本的**具体型号**和**预装 Win 是 Home 还是 Pro**，我可以按「26.04 安装 + 内网远程 + 是否值得升 Pro」给你列一份更短的执行清单（逐步勾选那种）。

## User

有个问题，我不太想用bitlocker，怎么关闭呢？

## Assistant

BitLocker 可以关，但**关闭 = 解密整个系统盘**，需要一些时间。若你打算装 Linux 双系统，务必等解密到 **100%** 再分区。

---

## 先确认：你开的是 BitLocker 还是「设备加密」？

Win11 **家庭版**通常没有完整 BitLocker 界面，但可能有 **设备加密（Device Encryption）**；**专业版**才是完整的 BitLocker。

### 方法一：设置里看

1. `Win + I` → **隐私和安全性** → **设备加密**（Home 常见）
2. 或 **隐私和安全性** → **Windows 安全中心** → **设备安全性** → **BitLocker 驱动器加密**（Pro 常见）

### 方法二：命令行看（最准确）

以**管理员**打开 PowerShell 或 CMD：

```powershell
manage-bde -status
```

关注 `C:` 那几行：

| 显示内容 | 含义 |
|---------|------|
| `转换状态: 完全加密` / `Fully Encrypted` | 还在加密/已加密，需解密 |
| `转换状态: 完全解密` / `Fully Decrypted` | 已关闭 |
| `保护状态: 保护已关闭` | 保护已关 |
| `已用空间加密百分比: 0.0%` | 适合继续装 Linux |

---

## 关闭 BitLocker / 设备加密

### 方式 A：图形界面（Pro 版 BitLocker）

1. `Win + I` → **隐私和安全性** → **Windows 安全中心**
2. **设备安全性** → **BitLocker 驱动器加密**
3. 点 **C:** → **关闭 BitLocker**
4. 选 **解密驱动器**
5. 等进度到 100%（可能几十分钟到几小时，视磁盘大小而定）

### 方式 B：Home 版「设备加密」

1. `Win + I` → **隐私和安全性** → **设备加密**
2. 关闭 **设备加密** 开关
3. 系统会自动解密，同样需要等完成

若设置里找不到，可在管理员 PowerShell 试：

```powershell
manage-bde -off C:
```

### 方式 C：命令行（Pro / 已启用 BitLocker 时通用）

管理员 PowerShell：

```powershell
# 关闭并解密 C 盘
manage-bde -off C:

# 查看进度（可反复执行）
manage-bde -status C:
```

解密过程中不要强制关机；笔记本建议插电。

---

## 关闭后建议再做的两件事

### 1. 确认真的解完了

```powershell
manage-bde -status C:
```

应类似：

- `转换状态: 完全解密`
- `保护状态: 保护已关闭`
- `已用空间加密百分比: 0.0%`

### 2. 若打算双系统，再关「快速启动」

`控制面板` → **电源选项** → **选择电源按钮的功能** → 取消勾选 **启用快速启动**

否则 Linux 可能读不了 Windows 分区，或安装程序识别分区异常。

---

## 常见问题

**Q：关 BitLocker 会丢数据吗？**  
一般不会。正常解密不删文件，但重要数据仍建议先备份。

**Q：解密很慢？**  
正常。128GB～1TB 的 SSD 可能 30 分钟到数小时。

**Q：Home 版没有 BitLocker 菜单？**  
看 **设备加密**；或用 `manage-bde -status` 确认。

**Q：以后还能再开吗？**  
可以。Pro 在 BitLocker 设置里重新开启；Home 在设备加密里再打开。

**Q：为了装 Linux 必须关吗？**  
**建议关，且必须等解密完成。** 加密状态下 Ubuntu 安装程序往往无法安全缩小 Windows 分区，还可能触发恢复密钥提示。

---

## 简要步骤

```
1. 管理员 PowerShell → manage-bde -status     （看当前状态）
2. manage-bde -off C:                          （开始解密）
3. 反复 manage-bde -status C: 直到 0.0%      （确认完成）
4. 关快速启动                                  （若要双系统）
5. 再去磁盘管理压缩卷 / 装 Ubuntu
```

如果你把 `manage-bde -status` 的输出贴出来，我可以帮你看是否已经关干净、能不能开始装 Linux。

## User

我如果当前是家庭版，能否先关了，然后再升级专业版？这样是不是就没有加密的过程了

## Assistant

可以，而且**这正是比较稳妥的顺序**：先在家庭版上关掉加密并等解密完成，再升级到专业版。

---

## 结论先说

| 问题 | 答案 |
|-----|------|
| 家庭版能先关加密再升专业版吗？ | ✅ 可以 |
| 升级专业版会重新加密吗？ | ❌ **不会自动加密** |
| 升级过程本身是在「加密/解密」吗？ | ❌ 不是，只是解锁 Pro 功能 |
| 这样能否避免以后反复折腾？ | ✅ 对，尤其你打算装 Linux 双系统 |

---

## 推荐顺序

```
1. 家庭版 → 关闭设备加密 / BitLocker
2. 等 C 盘完全解密（0.0%）
3. 升级到 Windows 11 专业版
4. 装 Ubuntu、分区、配远程
5. 双系统稳定后，再决定要不要开 BitLocker（可选）
```

---

## 为什么这样更好？

### 1. 家庭版关的是「设备加密」

Home 一般叫 **设备加密**，Pro 叫 **BitLocker**，底层往往都用 `manage-bde`，关法类似：

```powershell
# 管理员 PowerShell
manage-bde -status C:    # 先看状态
manage-bde -off C:       # 开始解密
```

或在 **设置 → 隐私和安全性 → 设备加密** 里关掉。

### 2. Home → Pro 升级 ≠ 重装系统

升级专业版是**原地升级**：

- 保留文件、软件、设置  
- **不会**自动重新加密  
- **不会**再跑一遍解密（前提是你已经解完）  
- 只是多了 Pro 功能（远程桌面、组策略、BitLocker 完整界面等）

### 3. 若先升级 Pro 再关 BitLocker

也能做，但对你这种要分区装 Linux 的场景：

- 多一步操作  
- 若 Pro 里开了 BitLocker，关的时候同样要等解密  
- 没实质好处  

所以：**先解密，再升级** 更干净。

---

## 升级 Pro 后会不会「又加密了」？

**正常情况不会自动开。**

但要注意：

| 情况 | 会不会重新加密 |
|-----|--------------|
| Store / 密钥升级 Pro | ❌ 不会 |
| 你手动去开 BitLocker | ✅ 会（你自己开的） |
| 某些企业/学校策略 | 可能被策略强制（家用机少见） |
| 出厂 OEM 特殊策略 | 极少见，一般升级后仍保持解密状态 |

升完 Pro 后可在 **设置 → 系统 → 激活** 确认版本，再用：

```powershell
manage-bde -status C:
```

确认仍是 **完全解密 / 0.0%**。

---

## 和你目标的对应关系

你要：**关 BitLocker + 可能升 Pro + 装 Linux + 内网远程**

```
✅ 家庭版先关设备加密，等解密完成
✅ 再升 Pro（若要 RDP）
✅ 关「快速启动」
✅ 磁盘管理里压缩卷，装 Ubuntu
❌ 双系统稳定前，别急着再开 BitLocker
```

**BitLocker 和双系统**可以共存，但常见问题包括：

- 安装 Ubuntu 前要解密（你已处理）  
- 从 GRUB 进 Windows 有时要输恢复密钥  
- 以后改分区可能再触发  

若笔记本当内网 AI 服务器、不常带出门，**长期不开 BitLocker 也合理**；若以后需要，等双系统都稳定再开也行。

---

## 实际操作 checklist

```
□ 1. 管理员 PowerShell：manage-bde -status C:
□ 2. 若已加密：manage-bde -off C:（或设置里关设备加密）
□ 3. 等到「完全解密 / 0.0%」（可能 30 分钟～数小时）
□ 4. 设置 → 系统 → 激活 → 升级到 Pro（可选）
□ 5. 再执行 manage-bde -status C: 确认仍解密
□ 6. 关快速启动
□ 7. 开始装 Ubuntu
```

---

**一句话：** 家庭版先关加密、等解密完，再升专业版——**不会**在升级时再走一遍加密；升级本身也不是加密过程。对你打算装 Linux 来说，这是更省事的做法。
