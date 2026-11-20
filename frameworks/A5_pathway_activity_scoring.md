# 说明书：A5 肝癌通路活性评分

## 0. 基本信息

- 清单编号：A5；类别：一；日期：2026-11-20。
- 状态：有待定项。本文是预分析说明书，不报告本项目新计算出的通路结果。
- 交付范围：补齐现有 T830/T803 通路分析中“固定精选基因集、患者级评分矩阵和统一可复核输出”。不重写配方，不从差异基因临时挑通路，不据此证明通路因果激活或代谢通量变化。
- 首版队列：TCGA-LIHC 原发肿瘤，以 A2 已审计的 cBioPortal Pan-Cancer Atlas RSEM 输入为主；GSE14520（GPL3921）仅作独立表达平台的敏感性复核，不合并两队列后打分。
- 主要问题：在患者层面估计预先指定通路转录程序的相对表达分数，并检验预先指定的组间/临床关联；一个患者是一个统计单位。

## 1. 要回答什么问题

通路活性评分把一个预先定义的基因集合压缩为每个样本一个分数，便于检查肿瘤与癌旁、临床分层或其他患者级指标之间是否伴随转录程序变化。首版覆盖清单指定的 EMT、缺氧、IL-6/STAT3、TGF-β、Hippo/YAP 相关信号、Notch、MYC 靶程序、糖酵解、胆固醇/脂肪酸合成、谷氨酰胺/氮代谢、甲硫氨酸代谢和铁死亡调控基因。

这些分数只表示所选基因集在转录组中的相对表达模式。它们不是蛋白磷酸化、酶活、代谢物浓度或代谢通量的直接测量；Hippo 核心组件表达也不等于 YAP/TAZ 转录活性。对“通路被激活”“导致肿瘤进展”等因果表述，必须有独立蛋白、功能或干预证据。

本说明书只处理预先选定的基因集评分与患者级比较/关联。全基因组差异表达后的富集仍用 T803；以通路分数做无监督亚型聚类属于单独的 B3 任务，不在此处默认执行。

## 2. 依据与文献

### 2.1 团队论文：主题依据，不冒称其都用了同一评分法

仓库 `docs/TEAM_PAPERS.md` 中与本清单相关的论文支持优先关注的生物学主题。阅读论文方法和结果时，需区分作者实际使用的证据类型（组织/IHC、细胞实验、测序、公开队列关联等）；没有证据表明这些团队研究都采用了统一的 MSigDB/ssGSEA 流程，因此本说明书不把主题相关性写成“团队已验证此评分方法”。

| 主题线索 | 论文与标识 | 对本分析的用途与边界 |
|---|---|---|
| 谷氨酰胺、β-catenin、mTORC1 | Adebayo 等，*Inhibiting Glutamine-Dependent mTORC1 Activation Ameliorates Liver Cancers Driven by β-Catenin Mutations*, PMID [30713111](https://pubmed.ncbi.nlm.nih.gov/30713111/), DOI [10.1016/j.cmet.2019.01.002](https://doi.org/10.1016/j.cmet.2019.01.002) | 论文结合小鼠肿瘤、细胞与人肿瘤相关证据研究谷氨酰胺合成酶/β-catenin/mTORC1；可支持把谷氨酰胺和 mTOR 作为主题，不能把其实验读数等同于本项目 RNA 分数。 |
| MYC 与 AKT/mTOR | Xu 等，*The mTORC2-Akt1 Cascade Is Crucial for c-Myc to Promote Hepatocarcinogenesis in Mice and Humans*, PMID [31062368](https://pubmed.ncbi.nlm.nih.gov/31062368/), DOI [10.1002/hep.30697](https://doi.org/10.1002/hep.30697) | 研究包含小鼠模型、人 HCC 组织及细胞实验；为 MYC/mTOR 主题提供团队相关性。单个 MYC 靶基因集分数仍不等于 MYC 蛋白活性。 |
| MYC、mTORC1/AKT1 | Zhou 等，*TSC/mTORC1 mediates mTORC2/AKT1 signaling in c-MYC-induced murine hepatocarcinogenesis via centromere protein M*, PMID [39325536](https://pubmed.ncbi.nlm.nih.gov/39325536/), DOI [10.1172/JCI174415](https://doi.org/10.1172/JCI174415) | 主题依据为 c-MYC 驱动肝癌中的 mTORC1/AKT1 机制。标题和模型明确含小鼠肝癌；不得据此声称公共人队列的通路分数已验证。 |
| EMT、Wnt/β-catenin | Li 等，*LOXL3-promoted hepatocellular carcinoma progression via promotion of Snail1/USP4-mediated epithelial-mesenchymal transition*, PMID [35841383](https://pubmed.ncbi.nlm.nih.gov/35841383/), DOI [10.1002/tox.23617](https://doi.org/10.1002/tox.23617) | 支持 EMT 是团队 HCC 主题之一；论文的组织和机制结果不替代固定 EMT 基因集评分。 |

### 2.2 方法出处、应用与局限

| 用途 | 文献（PMID / DOI） | 如何采用 |
|---|---|---|
| 基因集来源与 Hallmark 设计 | Liberzon 等，*The Molecular Signatures Database (MSigDB) hallmark gene set collection*, PMID [26771021](https://pubmed.ncbi.nlm.nih.gov/26771021/), DOI [10.1016/j.cels.2015.12.004](https://doi.org/10.1016/j.cels.2015.12.004) | Hallmark 集合由冗余较高的 founder signatures 精炼而来；按固定集合名和固定版本取基因，不在本项目里自编基因。 |
| 跨样本 GSVA | Hänzelmann 等，*GSVA: gene set variation analysis for microarray and RNA-seq data*, PMID [23323831](https://pubmed.ncbi.nlm.nih.gov/23323831/), DOI [10.1186/1471-2105-14-7](https://doi.org/10.1186/1471-2105-14-7) | 估计一个队列样本间的相对通路变化，可作为预先指定的敏感性方法；分数依赖该队列的表达分布，不能直接将不同队列的原始 GSVA 数值拼接比较。 |
| 单样本 ssGSEA | Barbie 等，*Systematic RNA interference reveals that oncogenic KRAS-driven cancers require TBK1*, PMID [19847166](https://pubmed.ncbi.nlm.nih.gov/19847166/), DOI [10.1038/nature08460](https://doi.org/10.1038/nature08460)；方法实现细节见 Bioconductor GSVA vignette | 每个样本内按基因表达排序，比较集合内外的累积分布；适合作为首版单患者分数。分数仍受输入矩阵、基因覆盖、集合定义和归一化选项影响。 |
| HCC 应用锚点 | *Comprehensive analysis of metabolic pathway activity subtypes derived prognostic signature in hepatocellular carcinoma*, PMID [35651292](https://pubmed.ncbi.nlm.nih.gov/35651292/), DOI [10.1002/cam4.4858](https://doi.org/10.1002/cam4.4858) | 报告在 TCGA 与 GSE14520 合并的 609 例训练队列中量化 41 个代谢通路并做分型。本文只借鉴“先评分再比较”的应用脉络，不复刻其数据合并、聚类或预后签名；其样本数不能当作本项目当前数据数。 |
| HCC 应用锚点/替代方法 | *Identification of two heterogeneous subtypes of hepatocellular carcinoma with distinct pathway activities and clinical outcomes based on gene set variation analysis*, PMID [39323867](https://pubmed.ncbi.nlm.nih.gov/39323867/), DOI [10.3389/fgene.2024.1441189](https://doi.org/10.3389/fgene.2024.1441189) | 使用 TCGA 与 ICGC 队列、GSVA 和 KEGG 通路做 HCC 分型。本文不照搬其数据驱动分型；作为方法应用参照，并强调外部队列应独立计算和独立报告。 |
| 铁死亡调控基因来源 | Zhou & Bao 等，*FerrDb V2: update of the manually curated database of ferroptosis regulators and ferroptosis-disease associations*, PMID [36305834](https://pubmed.ncbi.nlm.nih.gov/36305834/), DOI [10.1093/nar/gkac935](https://doi.org/10.1093/nar/gkac935) | 驱动因子、抑制因子、标志物是不同类别；不把全表合并为一个有方向的“铁死亡活性分数”。首版只分开描述 driver 与 suppressor 集合，marker 不纳入主分数。 |

### 2.3 版本冻结

- 项目现有流程使用 MSigDB 2026.1.Hs；为和仓库中的 T803/T830 示例保持一致，首版固定下载 [MSigDB 2026.1.Hs](https://docs.gsea-msigdb.org/MSigDB/Release_Notes/MSigDB_2026.1.Hs/)，使用 Hallmark（H）与 Reactome（C2:CP:REACTOME）/GO Biological Process（C5:BP）集合。记录 GMT 文件名、下载日期、SHA256、集合 ID、下载许可与基因映射版本。不得悄悄换成 MSigDB 2026.2 或其他版本；更新版本要作为新分析版本重算并并列报告。
- FerrDb 使用可下载的 FerrDb V2 人类基因表，记录获取日期、文件哈希、基因类别和条目版本；不能只写“FerrDb”而不留来源快照。若 V2 页面不可下载或类别字段不完整，铁死亡模块标“待定/不可评价”。
- 核心方法参数、集合列表、输入数据哈希和患者映射规则在看分数和结局前写入运行清单并冻结。

## 3. 数据与基因集

### 3.1 队列

| 队列 | 来源与数量锚点 | 用途 | 重要限制 |
|---|---|---|---|
| TCGA-LIHC 原发 HCC | cBioPortal Datahub Pan-Cancer Atlas 文件 `lihc_tcga_pan_can_atlas_2018/data_mrna_seq_v2_rsem.txt`；A2 审计记录为 366 名患者、366 个原发肿瘤样本 | 主队列：计算每名患者预设通路分数；有可靠配对的癌旁组织可作次要肿瘤-癌旁比较 | 执行时按 A2 锁定 SHA256 和 TCGA barcode 重核。不能把样本 aliquot 当患者；多样本选择沿用 A2 已审定规则。 |
| GSE14520 | GEO，GPL3921；A2 审计记录为 445 个阵列（225 肿瘤、220 癌旁），其中身份一致的肿瘤-癌旁配对 213 对 | 外部平台敏感性复核：独立计算分数，再检查预设关联方向是否相容 | 该队列是芯片，不与 RSEM 拼接；遇到缺探针/多探针要按预设映射规则登记，不以数据结果挑探针。平台覆盖差的集合可不可评价。 |

TCGA-CDR 临床字段可用于 A2 已有规则定义的次要 OS 关联；生存分析沿用 A2 的患者级 OS 定义、年龄范围、临床完整集及缺失登记。A5 主问题不是构建预后模型，不能因为某个分数显著就另建 cutpoint 或风险模型。若无临床分层变量，不妨碍先输出评分矩阵，但临床关联模块记为不可评价。

### 3.2 固定精选集合

主集合用 Hallmark，减少冗余和事后选集合。精确名称在目标版本 GMT 中查找，不接受模糊字符串或跨版本别名替换。下表中的系列是首版候选清单；运行前需由 GMT 核对精确 ID、基因数和映射后基因数，并把核对表随输出保存。

| 主题 | MSigDB 2026.1.Hs 集合名 | 主/次 | 解释边界 |
|---|---|---|---|
| EMT | `HALLMARK_EPITHELIAL_MESENCHYMAL_TRANSITION` | 主 | 反映 EMT 相关转录程序，不单独证明肿瘤细胞发生形态转化；可能含基质成分信号。 |
| 缺氧 | `HALLMARK_HYPOXIA` | 主 | 缺氧相关表达，不是组织氧分压测量。 |
| IL-6/STAT3 | `HALLMARK_IL6_JAK_STAT3_SIGNALING` | 主 | 细胞因子响应转录程序；不等于 p-STAT3。 |
| TGF-β | `HALLMARK_TGF_BETA_SIGNALING` | 主 | TGF-β 相关表达程序；需注意基质和免疫细胞来源。 |
| Notch | `HALLMARK_NOTCH_SIGNALING` | 主 | Hallmark 层面的转录集合。 |
| MYC targets | `HALLMARK_MYC_TARGETS_V1`、`HALLMARK_MYC_TARGETS_V2` | 主，分开报告 | 两集合分别计算，不合并；两者不是同一独立假设的重复样本。 |
| 糖酵解 | `HALLMARK_GLYCOLYSIS` | 主 | 代谢相关转录程序，不是糖酵解通量。 |
| 胆固醇 | `HALLMARK_CHOLESTEROL_HOMEOSTASIS` | 主 | 胆固醇稳态相关基因表达，不等于胆固醇含量。 |
| 脂肪酸代谢 | `HALLMARK_FATTY_ACID_METABOLISM` | 主 | 反映较宽的脂肪酸代谢程序，不解释为合成专属。 |
| 脂肪酸合成 | `GOBP_FATTY_ACID_BIOSYNTHETIC_PROCESS` | 次 | 更贴近生物合成定义；MSigDB 2026.1.Hs 中核对集合成员后独立评分。 |
| 固醇合成 | `GOBP_STEROID_BIOSYNTHETIC_PROCESS` | 次 | 固醇合成是胆固醇相关主题的过程集合，不与胆固醇稳态 Hallmark 混为同一指标。 |
| Hippo 核心信号 | `REACTOME_SIGNALING_BY_HIPPO` | 次 | 该集合覆盖信号通路成员，分数不直接等同 YAP/TAZ 核转位或转录输出；只有补充一份有来源、方向明确的 YAP/TAZ 靶基因签名后才可称“YAP/TAZ 输出分数”。 |
| 谷氨酸/谷氨酰胺 | `REACTOME_GLUTAMATE_AND_GLUTAMINE_METABOLISM` 与 `GOBP_L_GLUTAMINE_METABOLIC_PROCESS` | 次，分开报告 | Reactome 与 GO 集合有重叠；不可把两个分数作为独立证据相乘或叠加。 |
| 氮代谢 | MSigDB 2026.1.Hs 的 `GOBP_*NITROGEN*` 候选集合 | 待确认 | 先在固定 GMT 里筛出名称、定义及成员均确实对应“氮代谢”的集合；如存在多个，依据定义和基因映射预先选一个。没有明确集合时记“未找到”，不得用宽泛氨基酸集合代替。 |
| 甲硫氨酸 | `GOBP_L_METHIONINE_METABOLIC_PROCESS`；可将 `REACTOME_METHIONINE_SALVAGE_PATHWAY` 作为独立次要集合 | 次 | salvage 仅覆盖甲硫氨酸再利用，不能代替完整甲硫氨酸代谢；两个集合单独计算。 |
| 铁死亡 | FerrDb V2 `driver` 与 `suppressor` 两类人类蛋白编码基因 | 探索性 | 方向分开；抑制因子升高不能解释为“铁死亡增强”。排除 marker/unclassified 类作为主分数。 |

若主集合在映射后基因数少于 10 或超过 500，或队列中可测到的成员少于原集合 50%，标记 `not_evaluable` 并写原因；不得为了通过阈值临时加/删基因。集合映射和重叠审计至少报告：原集合成员数、输入矩阵覆盖数、有效唯一基因数、映射失败数、与其他集合的 Jaccard 重叠。

## 4. 具体操作步骤

### 4.1 输入冻结和整理

1. 建立 `A5_run_manifest.tsv`：run ID、数据集 accession/文件名、URL、获取日期、SHA256、平台、表达量单位、原始/已变换标记、患者条码规则、每名患者样本选择理由、基因集版本/文件哈希、评分算法与参数、R/Bioconductor/GSVA 版本、随机种子、分析目标和多重校正家族。
2. TCGA 主输入沿用 A2 的 RSEM 锁定文件；只在未对数的 RSEM 上一次性作 `log2(RSEM + 1)`。若矩阵已是 log 表达，则不重复变换。表达必须为有限数；负值、重复 gene symbol 且数值冲突、重复患者未解决时停止，而不是自动取最大表达。
3. 基因 ID 统一到冻结的 HGNC 官方符号映射表，记录下载日期和哈希；一对多映射、无法映射和多行冲突均输出审计表。每个样本保留唯一患者 ID；仅纳入 A2 审计规定的原发 HCC 样本类型。癌旁组织作为配对的次级分析，不能混入肿瘤主队列。
4. GSE14520 使用 GPL3921 平台已处理的标准化表达，先核实 GEO Series Matrix、平台注释与患者/组织标签；按 A2 的审核规则处理多探针/缺探针。该平台和 TCGA 的数据尺度不同，分别评分，不能在评分前合并或共同标准化。
5. 先输出样本数、患者数、肿瘤/癌旁数、可配对数、缺失标签、重复 ID 和基因集覆盖率。若核心元数据或文件哈希与锁定值不一致，停止沿用既有数量锚点并重新审计。

### 4.2 计算样本级通路分数

6. 主评分用 GSVA R 包的 `ssgsea` 方法，参数固定为 `alpha=0.25`、`normalize=TRUE`（按包版本核对实际参数名），对每个队列独立计算每个样本的集合分数。该选择是单样本评分的操作决定，不等于 ssGSEA 对所有数据都优于其他算法。
7. 一个基因集合只按冻结 GMT 的官方符号成员计算。对集合中的重复符号去重；仅当样本中可映射且表达有效的集合成员满足第 3.2 节的覆盖门槛时才评分。输出每个集合的分数，不先对集合做平均，不用结果挑“最佳算法”。
8. GSVA 算法作为预先指定的次要敏感性评分，仅在队列内使用同一固定集合、同一输入矩阵独立重算。GSVA 使用整队列样本表达分布，不能当作独立于队列分布的患者绝对标尺；不把其原始分数和 ssGSEA 原始分数合并。
9. 对每个队列内的评分计算中位数、IQR、范围、缺失数。用于连续回归时，在该队列内将评分中心化并按 1 SD 标准化；模型系数解释为该队列内每 1 SD 分数差异，不跨队列比较原始单位。外部队列方向复核要独立报告效应量与区间。
10. 不做聚类、不用“最佳截点”把分数二分。若后续 B3 需要聚类，另建预登记：固定纳入集合、距离、聚类数选择、重复性指标、训练和验证队列；不能将 A5 的分数图直接称为亚型。

### 4.3 预设比较与统计

11. 主要对比：TCGA-LIHC 肿瘤 vs 癌旁样本的通路分数；有同患者配对时以患者为单位作配对 Wilcoxon 符号秩检验，并报告配对数、配对中位差及区间。未配对样本另列，不和配对结果混成单一比较。若无足够配对患者，仅报告分布和效应区间，不以未配对检验替代主检验。
12. 次要临床关联仅限运行前指定的临床变量（如 A2 已冻结的年龄/分期字段）。连续变量使用相关或线性模型；分类变量按预先定义的两组/多组模型比较，具体字段编码和协变量沿用 A2 审计合同。缺失或组别样本不足时报告不可评价，不临时合并类别。生存如需分析，分数作为连续协变量纳入 A2 定义的 Cox 模型，不搜索 cutpoint；这属于探索性次级结果。
13. 首版主假设家族为 10 个预先指定的 Hallmark 集合（EMT、缺氧、IL6/JAK/STAT3、TGFβ、Notch、MYC V1、MYC V2、糖酵解、胆固醇稳态、脂肪酸代谢；按实际 GMT 核实清单后锁定）；对同一对比内所有可评价主集合的 P 值统一作 BH 校正，报告原始 P 和 q。建议校正阈值 q<0.05，但同时报告效应量及 95%CI；q 值不代表独立生物学证据。
14. Reactome、GO、FerrDb、其他临床变量及 GSVA 敏感性分析分别归为次要/探索性家族，先在 manifest 中列出每个家族的全部检验，再统一 BH。不能挑显著的集合改称“主要结果”，也不能因某组显著就省略同一预设家族中阴性结果。
15. 外部复核：在 GSE14520 内按同一 GMT 版本和 ssGSEA 参数独立打分；仅在技术映射和标签均通过时，检验预先指定关联方向是否一致。跨平台评分的绝对值和截点不可迁移。若核心集合可测基因不足，写 `not_evaluable`，不得通过缩短签名来换取可比性。

### 4.4 对照与质量检查

16. **集合身份检查：** 每个集合用官方 GMT 页面抽查 ID、来源描述和基因成员；锁定文件 SHA256，防止同名集合跨版本内容变化。
17. **阴性计算对照：** 以固定种子 `20261120`，为每个主集合生成 100 个表达丰度/集合长度匹配的随机集合，重算分数。若大量随机集合也与目标分组高度相关，应调查全局表达、增殖、肿瘤纯度或批次驱动，不把目标集合关联解释成特异通路证据。随机集合不是生物学阴性对照，不能证明因果。
18. **样本标签置换：** 仅用于评估流程在当前数据结构下的偶然关联；配对数据按患者成对置换标签，不逐个样本拆散配对。置换结果不能取代预定的效应估计和多重校正。
19. **敏感性检查：** 比较 ssGSEA 主分析与 GSVA 次分析；比较完整集合与平台覆盖阈值附近集合；配对分析与未配对描述分开；肿瘤纯度或免疫/基质比例可作已有 T833 指标的调整敏感性，但不能将同源/重叠基因造成的相关当作独立验证。方向改变须报告，不得择优保留。
20. **数值检查：** 确认同一表达矩阵重复运行结果一致；分数矩阵行列数与集合和样本清单匹配；无空集合、全常数集合、NA/Inf；样本 barcode 与 manifest 一对一。打开所有输出图检查名称、轴单位、患者数、配对数和阈值均可读。

## 5. 预先约定的结论规则

- 评分成功的最低条件：集合 ID 与 GMT 哈希可追溯；样本和患者映射通过；覆盖门槛通过；每个患者每个集合有一个有限分数；输出完整的集合成员/样本/失败审计表。
- 某条通路与临床分组“有统计学关联”需同时报告效应方向、95%CI、BH q 值和有效样本数；主家族以 q<0.05 作为统计阈值，不用未校正 P 值代替。即使达到阈值，仍写“该转录签名分数与……相关”，不写“通路被激活导致……”。
- 若 q≥0.05，报告效应及不确定区间并写“未获得预设统计证据”，不写“完全没有作用”。
- 若集合成员未覆盖、样本 ID/平台不明确、队列版本变化或对照失败，相关模块状态标记为 `not_evaluable` 或 `needs_review`，并说明原因；禁止临时换集合或调阈值使结果可报告。
- 不将一组集合中相互重叠的分数当作多份独立验证；对多个近义集合需列重叠系数，并在解释中合并成同一主题线索。

## 6. 必须交付的结果

| 文件/图 | 内容 |
|---|---|
| `A5_run_manifest.tsv` | 队列、文件哈希、版本、患者映射、基因集版本与哈希、参数、软件版本、分析和校正家族 |
| `A5_sample_audit.tsv` | sample_id、patient_id、组织类型、肿瘤/癌旁、配对关系、纳排原因、表达尺度 |
| `A5_geneset_audit.tsv` | 集合 ID、集合版本、来源、原始成员数、表达矩阵命中数、唯一命中数、覆盖率、与其他集合 Jaccard、状态 |
| `A5_ssgsea_scores.tsv` | 一行一患者一集合；cohort、patient_id、geneset_id、ssGSEA score、集合哈希、是否配对和临床字段可用性 |
| `A5_gsva_sensitivity.tsv` | GSVA 次要分数及其参数/版本；不得覆盖主表 |
| `A5_group_comparison.tsv` | 集合、队列、比较、n、配对 n、效应量/中位差、95%CI、P、BH q、检查结果 |
| `A5_random_set_control.tsv` | 随机集合生成种子、集合长度/表达匹配策略、随机分数与对照结果 |
| 分数分布与配对图 | 每个预设主通路按肿瘤/癌旁展示；显示患者点、配对连线、有效 n、效应方向和校正标记 |
| `A5_unresolved.tsv` | 暂缺氮代谢精确集合、Hippo/YAP 输出定义、FerrDb 版本文件或平台探针映射等事项，以及对结论的影响 |

## 7. 常见陷阱与替代方案

| 陷阱 | 处理 |
|---|---|
| 把一个通路分数称作真实生化活性或因果机制 | 使用“通路相关转录程序分数”；需要磷酸化、代谢物、功能实验才能提高机制证据。 |
| 看结果后挑选基因集/截点 | GMT 版本、集合名、主次分析和 BH 家族须在看分数/结局前冻结。 |
| 把不同平台的 ssGSEA/GSVA 原始分数拼一起 | 各队列单独计算、单独标准化、并列报告方向和区间；不把绝对值作跨平台比较。 |
| 以细胞/探针数充当患者数 | 先映射到 patient_id；患者是独立单位。重复组织样本保持配对或按预先规则聚合。 |
| gene set 重叠或包含驱动/抑制相反方向基因 | 报告 Jaccard；铁死亡 driver/suppressor 分开。Hallmark 与 Reactome/GO 不能被当作独立验证集。 |
| 把 TGFβ、EMT 或缺氧评分归因于肿瘤细胞 | bulk 信号可能来自基质和免疫细胞；用已有 T833/T834 作为辅助定位线索，不能仅凭 bulk 断言细胞来源。 |
| 集合覆盖不佳时删基因凑分数 | 按冻结的覆盖门槛标记不可评价；更换集合需新版本和新预登记。 |
| 借评分结果立刻做分型或预后模型 | 分型属于 B3；新建预后模型属于独立模型设计。A5 不根据同一队列的最优截点宣称预测性能。 |

### 替代评分方法

- **首选 ssGSEA**：产生每个样本的集合分数，适合固定集合逐患者比较；不足是分数依赖排序和归一化选项，绝对单位不是生物物理量。
- **敏感性 GSVA**：适合评估集合在队列中的相对变化；对队列组成和表达分布依赖更强，因此队列间分别计算，不能当作可迁移的绝对分数。
- **简单双向 z-score**：仅在有明确、预先指定的正/负向基因列表时作为备用；先在队列内对每个基因标准化，再按冻结的方向组合。对无方向的整套 Hallmark，不用未加方向的均值替代 ssGSEA。
- **富集检验（T803）**：若目标是病例-对照差异基因的通路过度代表或 GSEA，继续用 T803 并提供差异基因排名；不能把全样本评分误称为差异富集。

## 8. 待定项

1. **氮代谢集合待定。** 在锁定的 MSigDB 2026.1.Hs GMT 中先核对具体集合、定义及成员；若无足够具体且适用的“氮代谢”集合，本版只报告谷氨酰胺和甲硫氨酸相关签名，不用泛化 GO 项替代。
2. **YAP/TAZ 输出签名待定。** `REACTOME_SIGNALING_BY_HIPPO` 可用于 Hippo 组件线索；若需要“YAP/TAZ 转录活性”需独立来源的方向明确靶基因集、物种/平台映射和阳性对照。当前不自编目标列表。
3. **FerrDb 快照待定。** 写明 V2 当前下载链接、日期、文件哈希及分类字段。若最新版数据结构变化，先由团队决定沿用 V2 还是另立版本；不要把 driver、suppressor、marker 合并成一个得分。
4. **T830/T803 集成接口待定。** 现有配方可能使用 CPM、RSEM 或 MSigDB 2026.1 示例配置。实现前需确认项目负责人指定的标准输入文件、输出路径和依赖版本；本文不假设可直接替换现有配方配置。
5. **临床次级终点待定。** OS 如要纳入，固定协变量、缺失处理和多重校正家族以 A2 当前审计合同为准；在确认前只交付通路分数及肿瘤/癌旁比较。

## 9. 本版判断、来源核验与限制

- 队列数量和 A2 输入规则沿用仓库 `frameworks/A2_multi_gene_batch.md`，正式运行需再次校验 A2 哈希和 barcode。
- MSigDB 集合名称与版本须以官方 2026.1.Hs GMT/集合页为准；HALLMARK 名称、Hippo/谷氨酰胺/甲硫氨酸集合名在正式运行时仍需按该版 GMT 逐项核对。没有确认的氮代谢和 YAP/TAZ 靶签名明确保留“待定”。
- 文献中 HCC 的 GSVA/代谢分型结果只说明该类分析已有应用，不保证本说明书参数或当前队列能复现论文结论。论文反映的是各自研究问题和作者的选择，不等于最佳方法。
- 本次没有重新下载表达矩阵或运行评分，没有把论文结果写成仓库项目结果。分析结论待实际执行并通过输入审计、评分检查及患者级统计后再写。
