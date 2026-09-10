# Guard 下一代架构规划

## 现状总结

当前 m14（XLM-R-large，560M encoder）在白名单规则穷尽后的瓶颈：

| 数据集 | FPR | F1 | 瓶颈原因 |
|---|---|---|---|
| OpenAI-Mod | 0.327 | 0.695 | hate 类 55% FP = 模型无法区分「批评 X」vs「宣扬 X」 |
| ToxicChat | 0.077 | 0.718 | ethics 类创意写作误判，规则已基本穷尽 |

规则层（benign_en_override）已加 14 条，再加收益递减。根本问题是训练数据里缺少「safe 文本中含 hate 词汇」的样本，encoder 学的是词共现统计而非语义归因。

---

## 阶段一：双轨并行训练（近期）

### 总体策略

用**同一批数据**同时训练两个模型，训练结束后直接对比：

- **轨道 A**：XLM-R-large（encoder）— 现有架构，5ms，作为基准
- **轨道 B**：Qwen3-0.6B-instruct（decoder，int4 + 输出截断）— 新架构，目标 6-8ms

两个模型 GPU 需求不冲突（encoder 训练 ~8GB，decoder SFT ~12GB），可以先后或并行跑。数据准备只做一次，两边复用。

### 对比维度

| 维度 | XLM-R encoder | Qwen3-0.6B decoder |
|---|---|---|
| 延迟（A10G）| ~5ms | ~6-8ms（int4 + max_new_tokens=1）|
| FPR 预期 | 目标 ≤ 0.15 | 预期更低（语义归因能力强）|
| 理解能力 | 词共现统计 | 见过元语言语境，能区分引用/批评/宣扬 |
| Policy 适配 | 需重训 | prompt 注入，运行时改 |
| 训练成本 | 低（现有管道）| 中（需 SFT + 量化）|
| 部署成本 | 低 | 中（需 vLLM/llama.cpp）|

### 目标（共同）

- OpenAI-Mod FPR：0.327 → 目标 ≤ 0.15（encoder）/ ≤ 0.10（decoder）
- ToxicChat FPR：0.077 → 目标 ≤ 0.05
- 支持国标分类体系（当前数据已对齐）

### 数据增补策略

**核心动作：补充「负例的正例」**

即 safe 文本里包含 hate/ethics 词汇的样本，让模型学会「讨论 X ≠ 做 X」。

**来源 1：当前 FP 集直接复用**

把 OpenAI-Mod 的 379 条 FP 里确认是 safe 的打进训练集：

- hate 子集（约 210 条）：批评跨恐 / 新闻报道极端主义 / 平权文献 → 标 `safe/sec`
- ethics 子集（约 44 条）：心理互助 / 礼貌分歧 → 标 `safe/sec`
- ToxicChat ethics FP（约 86 条）：创意写作 / 荒诞场景 → 标 `safe/sec`
- 总计约 340 条，直接可用

**来源 2：合成批评性讨论文本**

用 Qwen3 / Claude 生成以下模式的 safe 样本（每类 200-300 条）：

```
- "That comment is [hate-word], and here's why it's wrong..."
- "[Group] deserves equal rights because..."
- "This article discusses the history of [discriminatory-term]..."
- "Researchers studying [hate-category] found that..."
- 新闻报道格式：描述而非宣扬
```

重点覆盖模型当前误判最多的 token pattern：
- transphobic / homophobic / racist（作为宾语被批评的）
- 历史种族词汇（academic / documentary 语境）
- 极端主义词汇出现在新闻 / 学术语境

**来源 3：国标对齐数据（已有）**

国标 GB/T 42118 / GB/T 37988 的分类体系和 OpenAI 标准有差异，当前数据已对齐国标，这是优势——BeaverTails 等数据集 FPR 偏高部分原因就是分类标准不同，国标对齐后比较基准更准确。

需要确认：
- 现有训练集里各类别分布是否均衡
- minors / extremism 类样本是否充足（m14 这两类 recall 偏低）

### 轨道 A：XLM-R encoder 训练配置

```
base: xlm-roberta-large
num_labels: 10
max_length: 256
batch_size: 32
lr: 2e-5
epochs: 3-5（早停）
class_weights: 对 minors/extremism 上调权重 1.5x
```

新增数据混入比例：新数据不超过总量 20%，避免打乱原有类别平衡。

### 轨道 B：Qwen3-0.6B-instruct decoder 训练配置

#### 前置：零样本基线实验（不训练，半小时）

在跑 SFT 之前先用原始 instruct 模型直接推理，量化预训练本身贡献多少：

```python
prompt = """你是内容安全分类器。判断以下内容是否安全，只输出 Safe 或 Unsafe，不要输出其他任何内容。

{text}"""
# max_new_tokens=5, temperature=0
```

跑完直接对比 OpenAI-Mod / ToxicChat 的 FPR 和 F1：
- 如果零样本 FPR 已经比 m14 低很多 → 预训练语义理解是主要功劳，SFT 是锦上添花
- 如果零样本很差 → 数据质量和任务专化是关键，SFT 收益会更大

#### SFT 训练

**为什么用全量 SFT 而不是 LoRA**：

0.6B 参数量小，全量 SFT 显存约 10-12GB，A10G（24GB）完全够。更重要的是任务改变幅度大——从「通用 assistant 生成对话」变成「第一个 token 就输出 Safe/Unsafe」，全量 SFT 让所有参数都适应这个新行为，比 LoRA 只调一小部分更彻底。LoRA 的优势（省显存）在 0.6B 上体现不出来。

**输入格式（SFT 数据格式）**：

```
<|im_start|>system
你是内容安全分类器。判断用户输入是否违反安全策略，只输出一个词：Safe 或 Unsafe。
<|im_end|>
<|im_start|>user
{text}
<|im_end|>
<|im_start|>assistant
{Safe / Unsafe}
```

**训练配置**：

```
base: Qwen/Qwen3-0.6B-instruct
method: 全量 SFT（全参数微调）
max_length: 512（含 prompt 模板）
batch_size: 8（gradient_accumulation_steps=4，等效 batch=32）
lr: 5e-6（全量 SFT 比 LoRA lr 低一个量级，避免灾难性遗忘）
epochs: 2-3（instruct 模型收敛快，loss 平稳后早停）
显存需求: ~10-12GB（A10G 24GB 够）
数据格式: 同 encoder，转成上述对话格式即可
```

**量化 + 部署**：

```
训练完 → AWQ int4 量化（AutoAWQ）→ vLLM 或 llama.cpp 部署
推理参数: max_new_tokens=1, temperature=0（greedy）
预期延迟: 6-8ms（A10G，batch_size=1）
```

**为什么 max_new_tokens=1 够用**：SFT 后模型学会在第一个 token 就输出 `Safe` 或 `Unsafe`，不需要 CoT 推理链。判决速度接近 encoder，同时保留了 decoder 预训练的语义归因能力。

### 预期收益对比

| | encoder m15 | decoder Qwen3-0.6B |
|---|---|---|
| OpenAI-Mod FPR | 0.15-0.18 | 0.08-0.12（语义理解更强）|
| ToxicChat FPR | 0.04-0.06 | 0.03-0.05 |
| Recall 影响 | 基本不变 | 可能微降（需实测）|
| 延迟 | 5ms | 6-8ms |

---

## 阶段二：Decoder 精判层（中期调研）

### 背景

在延迟不是硬约束（可接受 30-80ms）或者需要 policy 灵活适配的场景，decoder 架构优势明显：

- 预训练天然见过「元语言」语境（引用、批评、报道），归因能力强
- Policy 作为 prompt 传入，运行时改规则不需要重训
- 量化后延迟可控

### 量化对延迟的影响

| 模型 | 精度 | 延迟（A10G）| 精度损失 |
|---|---|---|---|
| Qwen3-0.6B-instruct | fp16 | ~35ms | baseline |
| Qwen3-0.6B-instruct | int8（AWQ/GPTQ）| ~18ms | <1% F1 |
| Qwen3-0.6B-instruct | int4（AWQ）| ~10ms | ~1-2% F1 |
| Qwen3-0.6B-instruct | int4 + 输出截断（只生成 Safe/Unsafe token）| ~6-8ms | ~1-2% F1 |

**关键优化**：decoder 做 guard 时不需要生成完整推理链，只需要生成 1 个判断 token（Safe / Unsafe / Controversial）。把 max_new_tokens 限制到 3-5，延迟可以压到接近 encoder 的水平。

Qwen3Guard-Stream 的架构就是这个思路的极端版本——在最后一层加 classification head，完全不生成 token，延迟降到和 encoder 一个量级。

### Policy-Conditioned 架构方案

参考 LPG（Latent Policy Guardrails）和 VirtueAI PolicyGuard 的思路：

```
输入文本
    ↓
[Policy 描述 as prompt]  ←── 运行时注入，不重训
    ↓
Qwen3-0.6B-instruct（int4 量化）
    ↓
Safe / Controversial / Unsafe（1 token 输出）
```

Policy 描述格式示例：
```
你是内容安全分类器。当前适用策略：
- 金融场景：拒绝任何投资建议、收益承诺
- 儿童平台：拒绝任何成人幽默、暴力描写
- 企业内部：允许讨论竞品，拒绝泄露商业机密
判断以下内容是否违反上述策略：{text}
```

不同客户 / 场景只需换 policy 描述，一个模型实例服务所有场景。

### 两阶段 Cascade 架构（最终形态）

```
请求
  │
  ▼
[Encoder - 5ms]  ←── m15（重训后）
  │
  ├─ safe（置信度 > 0.85）→ 直接放行（约 65-70% 流量）
  │
  ├─ unsafe（置信度 > 0.85）→ 直接拦截（约 15-20% 流量）
  │
  └─ 边界区域（0.4 < score < 0.85）→ 进入精判（约 10-15% 流量）
                │
                ▼
         [Decoder - int4 量化 - 8ms]
                │
                └─ 最终判决 + policy 对齐检查
```

整体平均延迟：
- 大多数请求：5ms（encoder 一次通过）
- 边界请求：5 + 8 = 13ms
- 加权平均：约 6-7ms

### 训练数据需求（阶段二）

Decoder policy-conditioned 训练需要三元组：`(text, policy描述, label)`

合成方案：
1. 用现有训练数据 + 不同 policy 描述组合生成训练样本
2. 每条数据配 3-5 种 policy 变体（严格 / 宽松 / 场景特定）
3. 总量目标：50-100k 三元组（Qwen3Guard 用了 119 万，我们可以从小规模开始）

---

## 路线图

```
现在
 │
 ├── 整理 FP 数据集（340 条标注为 safe）       1-2 天
 ├── 合成批评性讨论样本（Qwen3 生成，~800 条）  2-3 天
 ├── 数据合并，转两种格式（encoder / decoder）  半天
 │
 ├── [并行]
 │     ├── 轨道 A：跑 m15 encoder fine-tune    1 天（GPU）
 │     └── 轨道 B：跑 Qwen3-0.6B SFT          1 天（GPU，可同一张卡先后跑）
 │
 ├── 轨道 B：AWQ int4 量化 + 延迟实测           半天
 ├── 双模型评测：OpenAI-Mod / ToxicChat / 国标  半天
 ├── 对比分析：FPR / Recall / 延迟 / 边界 case  半天
 │
 ▼  （预计 1-2 周内）
确定主力模型（encoder 或 decoder），上线
 │
 ├── 若 decoder 胜出：接入 cascade 架构（encoder 粗筛 + decoder 精判）
 ├── 若 encoder 胜出：decoder 作为 policy-conditioned 精判层继续调研
 │
 ▼  （预计 1-2 月内）
两阶段 cascade + policy 灵活适配上线
```

---

## 评测基准套件（Baseline Benchmark Suite）

目标：覆盖 FPR（误拒率）、TPR（漏拦率）、对抗鲁棒性、中文安全四个维度。

### 核心评测集（必跑）

| 数据集 | HuggingFace ID | 规模 | 核心用途 | 备注 |
|--------|---------------|------|---------|------|
| **OpenAI Moderation** | `mmathys/openai-moderation-api-evaluation` | 1,680 条 | FPR / F1 基准，已有 m14 数据 | 已在用 |
| **ToxicChat** | `lmsys/toxic-chat` | 10,165 条 | 真实用户对话，FPR / Recall | 已在用 |
| **WildGuardTest** | `allenai/WildGuardMix` (test split) | 5,000 条 | 人工标注，覆盖 jailbreak + refusal | 业界标准，VirtueGuard 有数据 |
| **XSTest** | `walledai/XSTest` | 450 条 | **专测 FPR**，safe 文本含危险词 | 对标 m14 hate FP 问题，高价值 |
| **HarmBench Behaviors** | `walledai/HarmBench` | ~400 条 | 对抗攻击鲁棒性，TPR | VirtueGuard 有数据 |
| **HarmfulQA** | `declare-lab/HarmfulQA` | ~1,960 条 | 有害问题检测，Recall | VirtueGuard 有数据 |

### 补充评测集（按场景选用）

| 数据集 | HuggingFace ID | 规模 | 核心用途 |
|--------|---------------|------|---------|
| **SorryBench** | `sorry-bench/sorry-bench-202406` | ~450 条 × 45 类 | 细粒度 45 类 unsafe topic，检查各类 Recall |
| **DoNotAnswer** | `LibrAI/do-not-answer` | 939 条 | 拒答类别覆盖度（Information Hazard / Misinformation 等）|
| **SALAD-Bench** | `OpenSafetyLab/Salad-Data` | 21,000 条 | 6 domains × 16 tasks × 66 categories，最细粒度 |
| **BeaverTails** | `PKU-Alignment/BeaverTails` | 300k+ 条（取 test） | 注意：分类标准≠国标，FPR 偏高是正常现象 |
| **Aegis Safety 2.0** | `nvidia/Aegis-AI-Content-Safety-Dataset-2.0` | ~11,000 条 | NVIDIA 分类体系，对齐 Nemotron |
| **SafeRLHF** | `PKU-Alignment/PKU-SafeRLHF` | 82,000 条 | 多级安全偏好标注，可做 hard negative 挖掘 |
| **SimpleSafetyTests** | `Bertievidgen/SimpleSafetyTests` | 100 条 | 最基础的 critical risk 检出（5 类，must-catch）|
| **ConvAbuse** | `allenai/convoabuse` | ~4,000 条 | 对话滥用，骚扰类场景 |
| **HateMoji** | `HannahRoseKirk/HateMoji` | ~3,930 条 | emoji 仇恨检测，多模态扩展用 |
| **StrongREJECT** | `alexandrasouly/strongreject` | 313 条 | jailbreak 有效性评估器，侧重攻击成功率 |

### 中文 / 国标对齐评测集

| 数据集 | 来源 | 规模 | 备注 |
|--------|-----|------|------|
| **SafetyBench** | `thu-coai/SafetyBench` | 11,435 条（中英文）| THU 出品，ACL 2024，7 类安全 MCQ |
| **CHiSafetyBench** | `thu-coai/CHiSafetyBench` | 1,567 MCQ + 563 risky | 中文层级安全分类，贴近国标体系 |
| **CSSBench** | `arXiv:2601.00588` | — | 针对轻量级模型，中文对抗 pattern，6 领域 |
| **ChineseSafe** | Semantic Scholar | — | 中文安全 benchmark，含法律风险类别 |
| **自有国标数据** | 内部 | — | GB/T 42118 / GB/T 37988 对齐，当前优势 |

### 标准评测套件（每个模型必跑）

**固定 6 个，每次出新模型都跑这一套，结果直接横向对比：**

```
OpenAI-Mod + ToxicChat + WildGuardTest + XSTest + HarmBench + HarmfulQA
```

- 覆盖 FPR（OpenAI-Mod / ToxicChat / XSTest）和 Recall（HarmBench / HarmfulQA）
- 对齐 VirtueGuard 对比表格，有横向数据
- XSTest 专测 safe 文本含危险词，直接量化 hate FP 改善幅度

其余数据集（SorryBench / SALAD-Bench / 中文系列）按需补跑，不作为常规套件。

---

## 参考资料

- [LPG: Latent Policy Guardrails](https://arxiv.org/html/2605.17329) — policy 压进 latent space，748ms→可量化
- [Qwen3Guard Technical Report](https://arxiv.org/abs/2510.14276) — 1.19M 数据 + Controversial 三档标注
- [Policy-as-Prompt](https://arxiv.org/html/2509.23994) — 推理时注入 policy 的工程范式
- [VirtueAI PolicyGuard](https://www.virtueai.com/blog/introducing-policyguard-define-edit-and-enforce-custom-guardrails-for-every-enterprise-model-agent-and-application) — 商业落地参考，解释异步生成不阻塞主路径
- [XSTest (NAACL 2024)](https://github.com/paul-rottger/xstest) — 专测 exaggerated safety，250 safe + 200 unsafe prompts
- [WildGuard (NeurIPS 2024)](https://arxiv.org/abs/2406.18495) — 92K 多任务安全数据 + 5K 人工标注测试集
- [SALAD-Bench (ACL 2024)](https://github.com/OpenSafetyLab/SALAD-BENCH) — 21K 问题，6 domains × 66 categories
- [SorryBench (ICLR 2025)](https://github.com/SORRY-Bench/sorry-bench) — 45 类 unsafe topic，系统性 recall 检查
- [SafetyBench (ACL 2024)](https://github.com/thu-coai/SafetyBench) — 中英文双语，11K MCQ
- [CHiSafetyBench](https://arxiv.org/abs/2406.10311) — 中文层级安全分类基准
- [GuardBench (EMNLP 2024)](https://aclanthology.org/2024.emnlp-main.1022.pdf) — 40 个数据集 meta-benchmark，13 个 guard 模型对比
