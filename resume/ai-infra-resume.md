# Leqian Wu (吴乐谦) — LLM Inference / AI Infrastructure Engineer

> **Single-focus resume: AI Infra / LLM Serving.** Everything not serving/systems/performance
> was cut. Two versions: **English** and **中文 (2027 届秋招)**. One page each in PDF.

---

## English Version

**Leqian Wu**
(+86) 173-1650-6438 · mark.wu.23@ucl.ac.uk · GitHub: `<your-handle>` · Write-ups: `<blog-link>`

**AI Infrastructure / LLM Inference & Serving Engineer**

### Summary
LLM-serving engineer on **non-NVIDIA accelerators**. Source-level vLLM contributor mindset:
latency decomposition, structured-output debugging on Intel XPU (`intel/llm-scaler`), and
production image / deployment optimization. C++ systems-performance background (97× HPC solver),
BMO top-1% math foundation.

### Education
**UCL** — MEng Mathematical Computation, expected **Jun 2027** · First-class Honours (Y1 & Y2)

### Experience

**Intel — AI Frameworks Group (CCG), Shanghai** · *AI Framework Engineer Intern* · Jun 2026 – Sep 2026
- Owned evaluation of porting vLLM's **Rust frontend** into Intel's XPU serving stack
  (`intel/llm-scaler`). Built a **frontend-vs-engine latency-decomposition harness** (client ITL vs.
  engine-side Prometheus histograms) proving **6–7× frontend CPU-cycle savings** but
  **≤10% end-to-end gain under GPU-bound serving**; my "defer the port" call was adopted by the team.
- Assignee on a production **vLLM EngineCore crash** on Arc Pro B70 (`intel/llm-scaler#493`):
  reproduced across B60/B70, eliminated image/GPU/OOM causes, and isolated the root cause to a
  **structured-output-path regression** via differential analysis of the driver/runtime stack
  (NEO, Level Zero, torch-xpu); drove handoff to the fix owner.
- **Cut the production serving image 25 GB → 12.2 GB (−51%)** with multi-stage builds, RUN-layer
  consolidation, and a slimmer base — faster pulls and lower registry cost on every customer deploy.

**LEGO — Digital & Intelligence Center, Shanghai** · *AI Software Engineer Intern* · Jun 2025 – Sep 2025
- Built a **FAISS** vector-retrieval serving pipeline (embedding index + similarity search) returning
  matches over a 1,500+ item catalog in **~2 s/frame**; deployed the full service on **AWS EC2** and
  broadcast it over the internal network. *(Model-training details omitted — not infra-relevant.)*

**GMPT, Shanghai** · *R&D Engineer Intern (HPC)* · Jul 2024 – Aug 2024
- Built a high-performance **FEM solver in C++** from scratch; low-level / memory-layout optimization
  delivered **~97× speedup vs. a SciPy sparse-solver baseline**.

**DaoCloud (cloud-native unicorn), Shanghai** · *AI Ops Intern* · Jun 2023 – Aug 2023
- Built a CI/CD-driven **automated model-training pipeline** (GitHub Actions + shell) over 6,000+
  features, automating model iteration in a Kubernetes-native environment.

### Skills
- **LLM Serving:** vLLM (internals), `llm-scaler`, continuous batching, PagedAttention,
  quantization / FP8-KV cache, structured/guided decoding, Prometheus metrics
- **Systems & Perf:** C++, Python, Rust (working), Docker (multi-stage / image opt), Linux,
  Kubernetes, SYCL / oneAPI, ZMQ, Git / GitHub Actions, profiling (top / xpu-smi)
- **Retrieval / ML:** FAISS, PyTorch

### Awards (math/quant signal)
Gold — ACSL 2022 (top 5% US) · Distinction — BMO 2021 (top 1% UK) · Distinction — AMC 2021 (top 5%)

---

## 中文版（2027 届秋招）

**吴乐谦**
(+86) 173-1650-6438 · mark.wu.23@ucl.ac.uk · GitHub：`<你的-handle>` · 技术长文：`<博客链接>`

**AI Infra / LLM 推理与服务工程师**

### 教育背景
**伦敦大学学院 (UCL)** — 数学计算工程硕士 (MEng)，**预计 2027 年 6 月毕业（2027 届）**
一等荣誉 (First-class Honours，大一、大二)

### 实习经历

**英特尔 (Intel) — CCG AI Frameworks 组，上海** · *软件工程实习生* · 2026.06 – 2026.09
- 主导评估 vLLM **Rust 前端**向 Intel XPU 服务栈（`intel/llm-scaler`）的移植：搭建
  **"前端 / 引擎延迟分解"测试台**（客户端 ITL 对比引擎侧 Prometheus 直方图），证明 Rust 前端节省
  **6–7 倍前端 CPU 开销**、但在 **GPU 瓶颈服务下端到端收益 ≤10%**，"暂缓移植"结论被团队采纳。
- 作为 assignee 主导生产环境 **vLLM EngineCore 崩溃**排查（`intel/llm-scaler#493`，Arc Pro B70）：
  跨 B60/B70 复现排除 image/GPU/OOM，通过驱动栈差分（NEO / Level Zero / torch-xpu）将根因锁定为
  **结构化输出路径回归**，并推动修复交接。
- 将生产**服务镜像从 25 GB 压缩至 12.2 GB（−51%）**：多阶段构建、RUN 层合并、精简基础镜像，
  降低每次客户部署的拉取时间与存储成本。

**乐高 (LEGO) — 数智化中心，上海** · *AI 软件工程师实习生* · 2025.06 – 2025.09
- 构建 **FAISS** 向量检索服务管线（embedding 索引 + 相似性搜索），在 1,500+ 产品库上做到
  **~2 秒/帧**返回匹配；在 **AWS EC2** 上完成整套服务部署并广播至内网。*（模型训练细节略——与 infra 无关。）*

**芯钬量子 (GMPT)，上海** · *研发工程师实习生（HPC）* · 2024.07 – 2024.08
- 使用 **C++ 从零构建高性能有限元 (FEM) 求解器**，底层 / 内存布局优化使计算速度
  **较 SciPy 稀疏求解器基线提升约 97 倍**。

**道客云 (DaoCloud，云原生独角兽)，上海** · *AI Ops 实习生* · 2023.06 – 2023.08
- 结合 GitHub Actions 与 Shell，在 Kubernetes 云原生环境中构建 **6,000+ 特征的自动化模型训练流水线**，
  实现模型迭代自动化（CI/CD）。

### 技能清单
- **LLM 服务：** vLLM（源码级）、`llm-scaler`、continuous batching、PagedAttention、
  量化 / FP8-KV cache、结构化 / guided decoding、Prometheus 指标
- **系统与性能：** C++, Python, Rust (working)、Docker（多阶段 / 镜像优化）、Linux、Kubernetes、
  SYCL / oneAPI、ZMQ、Git / GitHub Actions、性能剖析（top / xpu-smi）
- **检索 / ML：** FAISS, PyTorch

### 奖项（数学 / 量化背书）
金奖 ACSL 2022（全美前 5%）· Distinction BMO 2021（全英前 1%）· Distinction AMC 2021（全球前 5%）

---

### What was cut (and why)
- **Deleted:** LEGO CV model-training bullets (SAM2/YOLO/DINOv2 fine-tuning, data augmentation),
  GMPT photonics inverse-design bullet, the UCL streaming-app project, and Haskell / Java / JavaScript /
  TensorFlow / Azure from Skills — none of it sells AI Infra.
- **Kept but reframed to infra:** LEGO → FAISS retrieval-serving + EC2 deploy only; GMPT → C++ perf
  only; DaoCloud → CI/CD training pipeline on K8s.
- **Dates:** Intel corrected to **Jun–Sep 2026** (per blog); LEGO stays 2025. Confirm exact days.
- **Fill placeholders:** GitHub handle (rename `365614269` → `leqianwu`/`markwu`) + a link to a clean
  technical write-up, not the diary homepage. Upgrade the `#493` / Rust bullets to
  "merged upstream / landed fix" once a PR lands.
