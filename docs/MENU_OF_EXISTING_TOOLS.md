# 现有分析工具（配方）目录

> 这是项目内部的菜单说明，原样复制。里面的路径（如 `T/T803_...`）指向原项目，**你不需要打开它们**，只用来了解每个工具能做什么、有什么局限。

# 核心菜单：给一句课题思路，该调哪个配方

2026-11-07 起。总纲 v2（`CC_delivery/MASTER_PLAN.md`）第 3.1 节的 7 项核心菜单，每项都有一个“一条命令加一个 config”的配方。本页是入口：先选配方，再按各配方的 `README.md` 运行。
**八个配方均有主机试跑及对照记录。** T832 完整 B=200 已完成；T834 的 16 项旧表对照已补成全主机来源。T803 主机运行的 prepare、de、enrich、figures 步骤包含配对子集 DESeq2；T811 检查因保护历史报告而隔离执行。范围与证据见 `CC_delivery/gate1_host/`。原配方由同一个 AI 完成，没有独立人员复核，图为内部预览；主机试跑不等于跨 AI 的科学独立复核。

路径简写：`T` = `分工A_研究判断与常用分析/整合交付/local_release/tasks`；命令在项目根 `Z:/Bioinformatics`（主机 `D:/workflows/Bioinformatics`）下运行，虚拟机用 `py -3 -X utf8`，主机用已核验的 Python 完整路径；本机 `python` 可能指向商店占位程序。

## 一、接手的 AI 怎么用（按顺序）

1. **复述成一页理解**：目的层级（描述、关联、预测、机制支持）、证据缺口、竞争解释、不能回答的部分（格式见 `RESEARCH_JUDGMENT.md` 第 12 节，简版即可）。用户不在时不等回复，把问题停放并写默认假设（`START_HERE.md` §0.1）。
2. **选配方**：用下面第二、三节。可以多项组合。不在菜单里的需求看第四节，先如实告诉用户“没有配方”，不要假装有。
3. **确认数据**：默认数据是 TCGA-LIHC 和几个 GEO（清单与校验码见 `CC_delivery/data_store/DATA_STORE.csv`，核对用 `verify_data_store.py`）。用户自带数据看各配方的“换数据”一节，先审格式和设计（`CC_delivery/intake/review_data.py`）。取不到的数据按 `CC_delivery/intake/PUBLIC_DATA.md` 停下并写清缺口，不绕过 403。
4. **看结果前**把规则写进 config 的预登记（阈值、分组、过滤、种子、协变量），记时间；事后改动必须在报告里写明原因。
5. **运行**：长运行（超过 20 分钟）放后台、结果即时落盘、运行中不要改脚本（`LESSONS.md` L-20261107-01）；虚拟机上同时最多 2 个工作进程。
6. **核对**：跑各配方 `checks/` 里的独立复算和对照；**每张图打开看过**，按手册 §5 图面清单检查（不裁切、写 n 与阈值与比较方向）；出图前先清空旧预览，免得看到旧版（L-20261107-03）。
7. **交付**：结果表、图（连同 `captions.json`）、方法段（各配方的 `METHODS_en.md` 当模板，改成本次设置）、局限（各配方的 `REPORT.md` 当模板，不超过 2 页）、复现命令。
8. **协作**：登记写入范围、更新 `HANDOFF.md`、提交（`AGENTS.md`）。新课题的输出放新任务目录，或配方的 `out/<名字>/`（Git 忽略）；**不要原地改配方**，要改就复制出新目录或走总纲流程。

## 二、7 项菜单

### M1 差异表达加富集：`T/T803_hcc_unpaired_de`
- **回答**：癌对癌旁/正常哪些基因和通路变了；某处理对对照；用户自己的计数矩阵加样本表。
- **命令**：`py -3 -X utf8 "T/T803_hcc_unpaired_de/scripts/run_all.py" --config "<config 的绝对路径>"`
- **输入**：基因×样本计数矩阵（整数，或 `log2(count+1)` 会还原）、样本表（组、患者）。**换数据只改 `config.json`**。
- **输出**：差异表（limma-voom 为主、DESeq2 并列，两套都报告，不按“差异基因更多”挑方法）、GO/KEGG/Hallmark 的过表达和 GSEA、火山图、PCA、富集图、GSEA 曲线。有患者列时自带“配对子集”等敏感性。
- **耗时**：约 1 小时（主要是配对子集的 DESeq2）；只跑 `--steps` 里的几步可更快。
- **限制**：KEGG 是旧版快照（229 条，其中信号/代谢 177 条，不更新，用户 2026-11-13 决定）；**新基因集库已补（2026-11-13，可选）**：`configs/` 里有 MSigDB 2026.1 的 Hallmark、Reactome、WikiPathways、GO BP 示例配置，只改配置，用法见 T803 README“使用 MSigDB 新基因集库”（GO BP 7538 个集合实测不到 5 分钟、峰值不到 0.5 GB；各库集合高度重叠，q 值不是独立检验数；新旧 Hallmark 不得悄悄替换）；TCGA 的“正常”是癌旁实体组织。配对 RNA-seq 可用同一配方的配对敏感性；芯片数据用旧任务 `T802_hcc_array_de` 的脚本（不是配方）。

### M2 单基因/通路验证：`T/T830_single_gene_recipe`
- **回答**：基因 X 在肝癌里表达高不高、与临床和预后有没有关系、与谁共表达、与某通路评分是否相关。
- **命令**：`py -3 -X utf8 "T/T830_single_gene_recipe/scripts/run_recipe.py" --gene GPC3`
- **输入**：一个基因符号（重名基因会停下，要给 `ensembl_id`）。
- **输出**：肿瘤对正常表达（非配对加 50 对配对，可选 GSE14520 验证）、临床关联、KM 与 Cox（单因素、调整、每 1 SD、cox.zph）、共表达前后各 20 个基因、Hallmark 通路评分相关，共 8 张图。
- **耗时**：约 3.5 分钟。
- **限制**：通路评分可用 MSigDB 2026.1 新库（可选，示例在 `configs/`，集合名区分大小写，用法见 T830 README）；生存与临床只用 TCGA-LIHC；表达量是 log2(CPM+1)，换归一化结果会变（RBM39 例子里明显变弱）；共表达只给基因表，没有富集；第二队列只有 GPL3921 平台。

### M3 生存与预后：`T/T831_published_signature_validation` 与 `T/T832_prognostic_model_recipe`
- **T831（已发表签名外部验证，含复发终点）回答**：某个已发表的基因签名在另一队列有没有用。
  - **命令**：`py -3 -X utf8 "T/T831_published_signature_validation/scripts/run_recipe.py"`（默认是 T806 的 REO 签名）；换签名或队列用 `--config configs/…json`。
  - **输入**：签名定义文件（系数加权或基因对投票）、队列（TCGA-LIHC、GSE14520 的 GPL3921；终点 OS/DFS，**TCGA 的复发类终点 PFI/DFI 已补**，用 `configs/TCGA_CDR_PFI_DFI.json`）。
  - **输出**：评分、KM 与 log-rank、单因素与调整 Cox、C 指数（含区间）、时间依赖 AUC、cox.zph。约 2 分钟。
  - **复发终点（2026-11-13 已补，Gate 4 第 4.4 项）**：TCGA-CDR（Liu 2018 Cell）的 PFI（无进展间期）、DFI（无病间期）接成可选终点；默认配置和输出不变。**局限**：2018 年历史快照，随访到当时为止（中位随访约 2 年），时间起点是诊断；DFI 只对初治 R0 且无瘤者有定义；PFI 的事件含“因癌症死亡”；事件多为 100–200 个，区间仍宽。同一 AI 完成，无独立复核，无用户批准；详见 T831 `README.md` 与 `CC_delivery/task_records/gate4_item44_*`。
  - **注意**：**必须在 config 写明签名开发自哪个队列**（`signature.developed_in`），开发队列里的结果是表观的；时间依赖 AUC 依赖 `timeROC`，本机已安装在独立库，其他机器须自行核验。
- **T832（新建预后模型）回答**：能不能用候选基因建一个预后模型，内部和外部验证怎么样。
  - **命令**：`py -3 -X utf8 "T/T832_prognostic_model_recipe/scripts/run_recipe.py" --run 名字`（默认复现 T807）。
  - **输入**：候选基因表、训练和外部两个队列。
  - **输出**：单变量筛选、LASSO、逐步 Cox、风险评分、**嵌套 bootstrap 校正后的 C 指数**、外部检验、列线图与校准。
  - **耗时**：bootstrap B=200 约 1.5 小时（2 个进程），小候选池和 B=20 约 2 分钟；每次重复即时落盘，中断后重跑自动续做。
  - **注意**：训练集表观表现只作对照，**结论以嵌套校正和外部检验为准**；事件数少、候选基因多时逐步法不稳（默认例子就是“不支持”）。

### M4 免疫浸润/微环境：`T/T833_immune_infiltration_recipe`
- **回答**：免疫、基质评分怎样、与分期分级生存的关系；某基因与免疫评分的关系；免疫细胞标记基因集评分。
- **命令**：`py -3 -X utf8 "T/T833_immune_infiltration_recipe/scripts/run_recipe.py"`（默认 TCGA-LIHC）；基因模式加 `--gene GPC3`；自己的矩阵把 `input.mode` 改成 `matrix`（见 README）。
- **输出**：ESTIMATE 免疫/基质/综合评分、临床关联、KM 与 Cox、高低评分组差异基因、MCP-counter 标记基因 ssGSEA、基因与评分相关。约 3 分钟。
- **限制**：评分是推断的相对分数，**不是细胞比例、不是纯度**；ESTIMATE 签名基因会让高低组差异按构造显著，不是独立发现；**去卷积是可选步骤（2026-11-13 补，默认关闭）**：`deconvolution.enabled=true` 后在默认七步之后接 `deconv_prepare`、`deconv`、`deconv_figures`；`immunedeconv` 的 7 种方法（MCP-counter、ConsensusTME、ESTIMATE 移植版、EPIC、quanTIseq、ABIS、TIMER），TCGA-LIHC 肿瘤对癌旁（配对 Wilcoxon，病人为独立单位），报方法间一致性（中位 Spearman 0.49，13 对为负，不挑方法）和与 ESTIMATE 评分的对照；**输出都是评分，不是细胞比例**；xCell 缺依赖 `pracma` 没跑，CIBERSORT 需注册不用；用本机私有 R 库 `gate4_lib_4.6`；`geneset_scoring.gmt` 可指向 MSigDB 2026.1（C8、GO BP 示例已实测）；部分图没逐张目检。

### M5 单细胞：`T/T834_sc_gene_localization_recipe`
- **回答**：基因 X 主要在哪类细胞表达、癌与癌旁在某类细胞里不同吗、细胞组成是否不同。
- **命令**：`py -3 -X utf8 "T/T834_sc_gene_localization_recipe/scripts/run_recipe.py" --gene GPC3`；基因集用 `--geneset "名字=基因1,基因2,…"`。
- **输入**：默认 GSE149614（71,915 个细胞，10 位患者，PT/NTL/PVTT/MLN）；换数据只改 config（稀疏计数矩阵加细胞表，至少含患者、组织、细胞类型列）。
- **输出**：各细胞类型表达、患者层面 pseudobulk 比较（配对精确检验）、细胞组成比较（`unresolved` 不进分母和检验）。约 2 到 3 分钟，读矩阵峰值约 2 GB 内存，同一台机器同时只开一个。
- **限制**：**独立单位是患者**，配对可评价的只有 8 位；细胞类型标签来自标志基因审核，不是金标准；一个数据集，没有外部验证；细胞通讯和拟时序（旧任务 T808）论文专用性太强，保持“按需”，没有配方。

### M6 诊断/标志物 ROC：`T/T835_diagnostic_roc_recipe`
- **回答**：某个基因或公式能不能区分肿瘤与癌旁/正常，外部队列怎么样。
- **命令**：`py -3 -X utf8 "T/T835_diagnostic_roc_recipe/scripts/run_recipe.py"`（默认 T810 的 10 基因公式）；单基因 `--marker markers/single_GPC3.json`，换队列 `--cohorts cohorts/…json`。
- **输入**：标志物文件（基因与系数、方向）、队列清单（路径、SHA256、病例/对照标签、谁是训练队列）。
- **输出**：每队列 AUC（DeLong 与自助法区间）、训练队列 Youden 切点原样用于其他队列的灵敏度特异度、配对检验、不可评价的队列及原因、ROC 图。约 2 分钟（需科学 venv 的 numpy/scipy）。
- **限制**：**主要是组织样本的肿瘤对癌旁/正常分类；另有一组 10 例 HCC 对 10 例健康者的 PBMC 表达数据。这不是临床诊断或早筛验证。** 没有血清 AFP 基线、校准或患病率；共享 ROC 模板的图里没有队列名和标志物名，必须连同 `captions.json`；缺探针的队列按预设规则标“不可评价”，不放宽。

### M7 候选基因/网络：`T/T811_hcc_wgcna_network`
- **回答**：肝癌里的基因分成哪些共表达模块、哪些模块与分级生存有关、hub 候选是谁。
- **命令**：`py -3 "T/T811_hcc_wgcna_network/scripts/run_t811.py" --config "<config 的绝对路径>"`
- **输入**：表达矩阵（第一列基因符号，log2 尺度）、样本表（患者、性状列）；至少 20 个独立样本，高变基因 100 到 3000 个，不用差异表达预筛。
- **输出**：模块表、模块与性状、模块特征基因的 Cox、hub 基因、模块 GO 富集、子抽样稳健性、6 张图。约 14.5 分钟，内存峰值约 1.1 GB。**可选 STRING 子网络**（`config.json` 里 `string.enabled=true` 才运行，默认关闭，2026-11-13 补，第 4 关 4.2）：每个模块的 STRING 边数对随机基因集、模块内 STRING 度与 kME 的一致性，附敏感性、负对照和每模块 2 张图；约 2 分钟加出图约 1.5 分钟。
- **限制**：**探索性**，hub 不是关键因子或驱动基因；单队列、没有外部验证；模块划分本身没有独立核对；两个模块不稳定。**STRING 子网络（可选）**：STRING 的边含文献挖掘和共表达，与 WGCNA 共表达可能循环，STRING 度高的基因不是驱动因子；映射率约 70%；预登记背景的负对照未通过，已另加事后分析（结果页和 README 有说明）；同一 AI 完成，没有独立复核，没有用户批准。

## 三、几个常见问法怎么组合

| 用户的话 | 建议组合 |
|---|---|
| “基因 X 在肝癌里的作用” | M2（T830）为主，加 M4 基因模式（T833 `--gene X`）和 M5（T834 `--gene X`）；要全局背景再加 M1（T803）。共表达基因的富集还没有一键方式，需要时先和用户说明，再另写小脚本 |
| “某通路/过程在肝癌里是否激活” | M1（T803，GSEA）加 M2 的通路评分；免疫相关加 M4 |
| “我有一份自己的 RNA-seq 计数矩阵加样本表” | M1（T803，改 config）；免疫评分用 M4 矩阵模式；网络用 M7；先用 `review_data.py` 审设计 |
| “找预后标志物/建模型/验证已发表签名” | M3（T831、T832）；单基因看 M2 |
| “能不能做诊断标志物” | M6（T835），并如实说明各队列材料、组织分类及少量 PBMC 的局限 |
| “X 在哪类细胞表达” | M5（T834） |

## 四、没有配方的（先如实告诉用户）
- **去卷积**：已补为 T833 的可选步骤（见 M4）；仍没有 CIBERSORT（需注册）和 xCell（缺 `pracma`）。
- **MSigDB 新库已补（2026-11-13，msigdbr 2026.1，9 个集合导成 GMT，见 `CC_delivery/gate4/GENESETS_EXPORT.md`，数据仓 D58 到 D67；不含 KEGG 与 C7）；GTEx**：本机没有（**PPI/STRING 已补**：2026-11-13 起是 T811 的可选步骤 `string`，见 M7）。用户 2026-11-07 同意 MSigDB 和 GTEx“需要的话就下”，但官方下载要在浏览器注册并接受许可，需要用户操作，GTEx 超过 2 GB 先问；用户 2026-11-08 又同意 STRING 和外部验证队列“需要就下载”（仍遵守取数规则，单次超过 2 GB 先告诉用户）。
- **突变/拷贝数/甲基化/药敏/分子分型/免疫治疗响应/空间转录组/细胞通讯与拟时序**：只有旧任务或限定演示，不是通用配方，属于“按需”，要做先和用户商量。
- **原始 fastq、动物与湿实验方案**：不做。**非肝癌**：先单独核对数据和设计。

## 五、共同局限（读任何配方的结果前都要知道）
1. 八个配方均有主机试跑与对照；T803 已运行配对子集，T811 检查使用隔离副本，详见 `CC_delivery/gate1_host/`。换机器要用配置副本设置本机 Rscript、包库或科学 Python。
2. 同一 AI 完成、无独立人员复核；独立复算是另一种实现，覆盖不到的部分各报告里写了“未独立核对”。
3. 默认数据主要是 TCGA-LIHC、GSE14520、GSE149614：多数结果是单队列、没有外部验证。
4. 评分不是细胞比例，hub 不是关键因子，肿瘤对癌旁分类不是临床诊断。
5. 大样本的 Wilcoxon 在 R 4.6.1 里会把 P 错返回 0（z 为正且大于约 8.29，`LESSONS.md` L-20261107-02）；配方里已用自写正态近似或避开。Spearman 的 `cor.test` 一律 `exact = FALSE`。
6. 图的基因符号不是斜体、负号是短横线（共享绘图模板限制）。

## 六、与原来任务卡流程的关系
配方**不走** `research_task.py` 的 gate、plan、run、check、report 全链和长篇任务卡，用的是 config 预登记加配方自带的对照与独立复算。`research_task.py` 的设计闸门（伪重复、配对、混杂的机械检查）仍可手动调用来检查自拟的设计。不在菜单里的新任务按 [轻量任务模板](../LIGHTWEIGHT_TASK.md) 处理；旧任务全链保留供追溯和续跑，不再是新任务的必经流程。

## 七、位置索引
- 配方：`T/T803_hcc_unpaired_de`、`T811_hcc_wgcna_network`、`T830_single_gene_recipe`、`T831_published_signature_validation`、`T832_prognostic_model_recipe`、`T833_immune_infiltration_recipe`、`T834_sc_gene_localization_recipe`、`T835_diagnostic_roc_recipe`；每个目录有 `README.md`、`REPORT.md`、`METHODS_en.md`、`config.json`、`scripts/`、`checks/`。
- 数据清单与固化基因集：`CC_delivery/data_store/`。
- 经验：`LESSONS.md` L-20261107-01 至 07。
- 阶段报告：`handoff/sessions/phase0_*.md`、`phase2_*.md`；总纲：`CC_delivery/MASTER_PLAN.md`。
