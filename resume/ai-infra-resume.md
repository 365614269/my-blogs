# Leqian Wu (吴乐谦) — AI Infrastructure / LLM Serving

> Resume tailored for **AI Infra / LLM Inference** roles.
> Two versions below: **English** (big tech / lab / overseas) and **中文** (国内秋招 2027 届).
> Keep it to **one page** per version when exporting to PDF.

---

## English Version

**Leqian Wu**
(+86) 173-1650-6438 · mark.wu.23@ucl.ac.uk · GitHub: `<your-handle>` · Blog: `<link-to-technical-write-up>`

**Target:** AI Infrastructure / LLM Inference & Serving Engineer

### Summary
MEng (Mathematical Computation) student at UCL with hands-on **LLM serving** experience on
non-NVIDIA accelerators: source-level vLLM analysis, Intel XPU production stack (`intel/llm-scaler`),
latency decomposition benchmarking, and production image optimization. Strong systems + C++ HPC
background and a competitive-math profile (BMO / AMC).

### Education
**University College London (UCL)** — MEng Mathematical Computation (expected **Jun 2027**, class of 2027)
- First-class Honours (Year 1 & Year 2)
- Coursework focus: AI algorithms, advanced algorithm analysis, systems & software engineering

### Experience

**Intel Corporation — AI Frameworks Group (CCG), Shanghai**
*AI Framework Engineer Intern* · Jun 2025 – Sep 2025
- Evaluated porting vLLM's **Rust frontend** into Intel's downstream XPU serving stack
  (`intel/llm-scaler`): built a **frontend-vs-engine latency-decomposition benchmark** (client-side
  ITL vs. engine-side Prometheus histograms), showing **6–7× frontend CPU-cycle savings** but
  **≤10% end-to-end gain on GPU-bound workloads**; the data-driven "defer the port" recommendation
  was adopted by the team.
- Led triage of a user-reported vLLM **EngineCore crash** on Arc Pro B70 (`intel/llm-scaler#493`) as
  issue assignee: reproduced across B60/B70, ruled out image/GPU/OOM causes, and narrowed the root
  cause from a host-driver mismatch to a **structured-output-path regression** via differential
  analysis of driver/runtime stacks (NEO, Level Zero, torch-xpu); coordinated handoff to the fix owner.
- **Halved the production Docker image (25 GB → 12.2 GB, −51%)** via multi-stage builds, RUN-layer
  consolidation, and a slimmer base image, cutting pull time and registry footprint for every
  customer deployment.

**LEGO — Digital & Intelligence Center, Shanghai**
*AI Software Engineer Intern* · Jun 2025 – Sep 2025 *(overlaps above; keep only if timeline is clarified)*
- Built a computer-vision system identifying **1,500+ LEGO SKUs** from shelf images (**Top-1 > 90%**),
  fully automating planogram/shelf-compliance checks.
- Scaled the training set **8×** with advanced data augmentation, improving robustness under
  real-world retail conditions (lighting changes, occlusion).
- Designed a **FAISS**-based similarity-search pipeline with vector encoding for millisecond-level
  retrieval over a large product catalog.

**GMPT (Quantum Photonics), Shanghai**
*Algorithm / R&D Engineer Intern* · Jul 2024 – Aug 2024
- **HPC:** built a high-performance **finite-element (FEM) solver in C++** from scratch; low-level
  optimization delivered **~97× speedup vs. a SciPy sparse-solver baseline**.
- **Inverse design (EDA / photonics):** optimized an optical Y-branch splitter, raising transmission
  efficiency from **70% → 97.5%**.

**DaoCloud — Cloud-native unicorn, Shanghai**
*Product Development Intern (AI Ops)* · Jun 2023 – Aug 2023
- Built an automated ML training pipeline over **6,000+ product features** using GitHub Actions +
  shell scripting, automating model iteration (AutoML / CI-CD).

### Projects
**Streaming Content-Strategy App — UCL Computer Science** · Jan 2026 – May 2026
- Python (Flask) services over large recommender datasets (MovieLens, Personality-2018); tuned
  online inference and data-serving performance.
- Containerized with Docker for concurrent request handling.

### Skills
- **Languages:** C++, Python, Rust (working), SQL, Java, JavaScript, Haskell
- **LLM Serving:** vLLM (internals), `llm-scaler`, continuous batching, PagedAttention,
  quantization / FP8-KV, Prometheus
- **Systems & Tools:** Docker (multi-stage / image opt), Linux, Kubernetes, ZMQ, SYCL/oneAPI,
  Git / GitHub Actions, Redis
- **ML:** PyTorch, computer vision (YOLO, DINOv2), FAISS

### Awards
- **Gold**, American Computer Science League (ACSL) 2022 — top 5% US
- **Distinction**, British Mathematical Olympiad (BMO) 2021 — top 1% UK
- **Distinction**, American Mathematics Competitions (AMC) 2021 — top 5% global

---

## 中文版（国内秋招 2027 届）

**吴乐谦**
(+86) 173-1650-6438 · mark.wu.23@ucl.ac.uk · GitHub：`<你的-handle>` · 博客：`<技术长文链接>`

**求职方向：** AI Infra / LLM 推理与服务工程师

### 教育背景
**伦敦大学学院 (UCL)** — 数学计算工程硕士 (MEng)（**预计 2027 年 6 月毕业，2027 届**）
- 大一、大二成绩：一等荣誉 (First-class Honours)
- 核心方向：AI 算法、高级算法分析、系统与软件工程

### 实习经历

**英特尔 (Intel) — CCG AI Frameworks 组，上海**
*软件工程实习生* · 2025.06 – 2025.09
- 评估 vLLM 上游 **Rust 前端**向 Intel XPU 下游栈（`intel/llm-scaler`）的移植价值：设计
  **"前端 / 引擎延迟分解"基准**（客户端 ITL 对比引擎侧 Prometheus 直方图），证明 Rust 前端节省
  **6–7 倍前端 CPU 开销**、但在 **GPU 瓶颈负载下端到端收益 ≤10%**，"暂缓移植"的数据化结论被团队采纳。
- 作为 assignee 主导用户报障排查（`intel/llm-scaler#493`，Arc Pro B70 上触发 **EngineCore 崩溃**）：
  跨 B60/B70 系统性复现排除，通过驱动栈差分（NEO / Level Zero / torch-xpu）将根因从宿主机驱动收敛至
  **结构化输出路径回归**，并协调修复交接。
- 将生产 **Docker 镜像从 25 GB 压缩至 12.2 GB（−51%）**：多阶段构建、RUN 层合并、精简基础镜像，
  直接降低客户部署拉取时间与存储成本。

**乐高 (LEGO) — 数智化中心，上海**
*AI 软件工程师实习生* · 2025.06 – 2025.09 *（与上条时间重叠，请按真实起止时间二选一保留）*
- 开发基于计算机视觉的 AI 系统，识别货架图像中 **1,500+ 种乐高产品**（**Top-1 > 90%**），
  实现货架合规性全自动检测。
- 通过高级数据增强将训练集扩大 **8 倍**，显著提升模型在复杂零售环境（光照变化、遮挡）下的鲁棒性。
- 设计基于 **FAISS** 的相似性搜索管线，利用特征向量编码实现海量产品库的毫秒级检索。

**芯钬量子 (GMPT)，上海**
*算法 / 研发工程师实习生* · 2024.07 – 2024.08
- **高性能计算 (HPC)：** 使用 **C++ 从零构建高性能有限元 (FEM) 求解器**，底层优化使计算速度
  **较 SciPy 稀疏求解器基线提升约 97 倍**。
- **智能设计优化（EDA / 光子学）：** 通过逆向设计优化光学 Y 型分支器，将传输效率从 **70% 提升至 97.5%**。

**道客云 (DaoCloud) — 云原生独角兽，上海**
*产品开发实习生（AI Ops 方向）* · 2023.06 – 2023.08
- 结合 GitHub Actions 与 Shell 脚本，构建 **6,000+ 产品特征**的 AI 模型自动化训练流水线，
  实现模型迭代自动化（AutoML / CI-CD）。

### 项目经历
**流媒体内容策略应用 — UCL 计算机科学系** · 2026.01 – 2026.05
- 采用 Python (Flask) 处理 MovieLens、Personality-2018 等大规模推荐数据集，优化在线推理与数据服务性能。
- 使用 Docker 容器化部署以支撑并发调用。

### 技能清单
- **编程语言：** C++, Python, Rust (working), SQL, Java, JavaScript, Haskell
- **LLM 服务：** vLLM（源码级）、`llm-scaler`、continuous batching、PagedAttention、
  量化 / FP8-KV、Prometheus
- **系统与工具：** Docker（多阶段 / 镜像优化）、Linux、Kubernetes、ZMQ、SYCL/oneAPI、
  Git / GitHub Actions、Redis
- **机器学习：** PyTorch、计算机视觉 (YOLO, DINOv2)、FAISS

### 奖项荣誉
- **金奖**，美国计算机科学联赛 (ACSL) 2022（全美前 5%）
- **优秀奖 (Distinction)**，英国数学奥林匹克 (BMO) 2021（全英前 1%）
- **优秀奖 (Distinction)**，美国数学竞赛 (AMC) 2021（全球前 5%）

---

### Notes on this rewrite (why the changes)
1. **Intel experience is now the first entry** — it is the strongest AI-Infra signal (vLLM internals,
   Intel XPU stack, quantifiable results). Blog references: Rust-frontend evaluation and GPU-bound
   bottleneck conclusion (Jun 2026 posts), image slimming, and issue triage.
2. **Timeline conflict flagged:** the original resume lists both LEGO and Intel as *Jun–Sep 2025*, and
   the blog shows the Intel/vLLM work in *Jun 2026*. Correct the real dates so no two roles show
   overlapping "Present"/summer 2025 windows. Placeholders left where dates must be confirmed.
3. **`97×` now cites a baseline** (SciPy sparse solver) so the number is credible.
4. **Skills reordered for AI Infra** — vLLM / serving / Docker / systems first; dropped padding
   (AWS/Azure/TensorFlow) unless genuinely used.
5. **UCL streaming app compressed** to two plain bullets and moved to Projects; removed inflated
   "microservices ecosystem / seamless horizontal scaling" wording that invites failed follow-ups.
6. **Graduation set to Jun 2027 (class of 2027)** and Chinese phone number kept for domestic autumn
   recruiting; competition awards retained (recruiters still screen on them).
7. **Fill the placeholders:** GitHub handle (consider renaming to `leqianwu`/`markwu`), and direct
   links to clean technical write-ups (not the diary blog homepage). If a merged PR or a resolved
   `#493` lands, upgrade the matching bullet to "merged upstream / landed fix".
