# 证据绑定风险场：合同审查数据集

[English](README.en.md)

本仓库公开论文《证据绑定风险场：在合同审查中分离证据判断与规则计算》所用的脱敏合同审查数据集，包括真实合同材料评测集、专业训练任务、两组实验的合成任务以及学生模型数据。仓库只包含数据与标注；模型、代码、模型输出与训练日志不公开。

## 目录结构

```text
evidence-bound-risk-field/
├── README.md / README.en.md
├── LICENSE.md, LICENSES/CC-BY-4.0.txt
├── SHA256SUMS.txt, MANIFEST_SHA256.json
├── 数据文件清单.csv            逐文件记录数、划分统计与 SHA256
├── 样本划分与编号.jsonl         每条记录的编号、划分与分组
├── 01_Professional_Training_Data/
├── 02_Experiment_1_Conditional_Judgment/
│   ├── condition_tasks/
│   └── new_expression_test/
├── 03_Experiment_2_Evidence_Binding/
├── 04_Real_Material_Evaluation/
└── 05_Student_Model_Data/
    └── earlier_round_tests/
```

## 数据组成

| 目录 | 文件与规模 | 在论文中的用途 |
|---|---|---|
| `01_Professional_Training_Data/` | `gold_train.jsonl`：77条专业训练任务，由24个经专家标注的原始条目拆分而来；`internal_dev.jsonl`：9条内部开发任务；`labor_auxiliary.jsonl`：1,657条劳动领域辅助训练任务，依据9份公开的劳动合同示范文本与《劳动合同法》编写。 | 专业监督微调；实验二训练时回放77条专业任务，并以9条开发任务作回归检查（第4.2节） |
| `02_Experiment_1_Conditional_Judgment/condition_tasks/` | `examples.jsonl`：1,213条记录，其中训练948条（512条合成条件样本、390条劳动合同证据任务、46条专业任务）、开发128条、原模板测试128条、回归9条；`pairs.jsonl`：576组成对关系（训练384、开发96、测试96）。 | 实验一的训练与原模板测试（第4.1节） |
| `02_Experiment_1_Conditional_Judgment/new_expression_test/` | `examples.jsonl`：256道新表述题，来自训练后另行构造的32个场景组，每组8个变体。 | 实验一的新表述测试（第4.1节，补充材料表S4） |
| `03_Experiment_2_Evidence_Binding/` | `scenes.jsonl`：672个合成场景，训练432、开发96、测试144，分别来自18、4、6个互斥场景组；测试集含42组成对事实变化，共498项标准引用。 | 实验二（第4.2节） |
| `04_Real_Material_Evaluation/` | `public_inputs.jsonl`：28道审查题的送审材料与问题；`gold_keys.jsonl`：87个评分项的参考答案与充分证据组。 | 真实材料评测（第4.3节） |
| `05_Student_Model_Data/` | `new_scenes.json`：1,512条（训练1,152、开发72，长句、短句、调整语序测试各96）；`historical.json`：以学生模型格式保存的实验二672个场景，其中144道测试题即论文中的144道历史题；段落与片段标注、关系对训练数据；`earlier_round_tests/`：此前各轮开发中使用的测试集，用于检查能力保持。 | 独立小模型（第4.4节） |

逐文件记录数、划分统计与SHA256见[数据文件清单](数据文件清单.csv)；每条记录的编号、划分与分组见[样本划分与编号](样本划分与编号.jsonl)。

## 真实材料评测集

一道审查题包含一组送审材料、一项审查任务和若干子问题，每个单独计分的子问题称为评分项。来自同一交易或密切相关材料的题目归为同一来源组（字段 `source_family`）。`gold_keys.jsonl` 中，`split` 为 `development` 的10道题是开发题（30个评分项）；`test_reserved` 的18道题包括论文的17道正式题（53个评分项，来源组SOURCE-GROUP-02、03、05、06、07）和1道单列的裁判文书阅读题（`task_type` 为 `judgment_reading`，4个评分项）。论文中的来源组G02至G07即数据中的SOURCE-GROUP-02至SOURCE-GROUP-07。

评测时请分别读取 `public_inputs.jsonl` 与 `gold_keys.jsonl`，不要把参考答案提供给模型。

## 合成任务

实验一新表述题的8个变体（字段 `variant`）为：`base` 基本表述、`paraphrase` 措辞改写、`template_only` 仅有承诺、`redacted_original` 原件脱敏、`wrong_actor` 主体错配、`wrong_time` 时间错配、`irrelevant` 无关材料、`conflict` 证据冲突。论文补充材料表S4中的场景组1–32依次对应数据中的SHIFT-000至SHIFT-031。

合成场景中的主体、事项与规则均为虚构，参考状态由程序按给定规则生成，不代表真实法律判断（字段 `label_authority` 注明了每条记录标签的来源）。

## 记录格式

JSONL文件每个非空行是一条记录；JSON文件为数组，或按内部集合组织的记录。`target`、`states`、标准 `bindings` 与参考答案用于监督或评分，测试时应与模型输入分开。同一批事实可能以多种表述出现；统计独立样本时，请依据 `pair_group`、`family` 等分组字段去重，并保留原有的训练、开发与测试划分。

记录编号与分组字段保留了数据构建时的编号（例如 `SHIFT-`、`V8-`、`R2-` 至 `R11-` 等前缀表示不同的构建批次），以保证各文件之间的对应关系不变。

## 脱敏

姓名、机构和来源使用稳定代号，同一主体在合同、题目与参考答案中的代号一致；直接联系方式、详细地址、银行账户、登记号码、合同标识和案号等已处理。判断所需的日期、金额、比例和逻辑关系予以保留。脱敏文本与原始实验输入存在文字差异，在这些文本上训练或推理应报告为新的实验。保留的金额、日期等信息仍可能支持来源关联；去除直接身份信息不等于不可逆匿名化。

## 不包含的内容

模型权重与适配器、特征缓存、模型输出、训练日志、代码、身份映射、原始合同附件与专家签认原件均不公开。

## 许可

数据与标注采用[CC BY 4.0](LICENSE.md)。第三方材料适用其原有许可与使用条件。

## 文件校验

在仓库根目录执行（macOS 使用 `shasum -a 256 -c SHA256SUMS.txt`）：

```bash
sha256sum -c SHA256SUMS.txt
```

[MANIFEST_SHA256.json](MANIFEST_SHA256.json) 以JSON格式记录同样的文件大小与SHA256（清单文件自身除外）。

## 引用

[作者补充：论文正式发表后的引用信息。双盲评审期间保持匿名。]
