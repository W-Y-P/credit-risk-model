# Credit Risk Model Skill

[![skills.sh](https://skills.sh/b/W-Y-P/credit-risk-model)](https://skills.sh/W-Y-P/credit-risk-model)

Reusable agent skill for credit-risk model development, validation, reporting, and strategy analysis.

This repository contains a Codex / skills.sh skill, not a standalone Python package. Install it into an agent environment, then ask the agent to use `$credit-risk-model` when developing scorecards, logistic regression models, LightGBM/XGBoost models, or other credit-risk models.

## 中文说明

### 这是什么

`credit-risk-model` 是一套面向信贷风控建模的 agent skill，用于把常见的信用风险建模流程沉淀成可复用的工作流。它适用于申请评分卡、行为评分卡、催收模型、反欺诈模型、客群筛选模型、额度/定价辅助模型等场景。

它会先明确建模需求，再按模型类型选择合适的建模路线。评分卡会走 WOE 分箱、IV、相关性、逐步回归、VIF、评分转换和报告验证流程；LightGBM、XGBoost、CatBoost、随机森林等树模型会走树模型应有的数据切分、早停、重要性/SHAP、过拟合诊断、调参和稳定性验证流程。

### Skill 亮点

- **先问清建模目标**：拒绝坏客户、挑选好客户、风险定价、额度、催收优先级、欺诈审核或其它目标。
- **支持多类模型**：评分卡、WOE 逻辑回归、普通逻辑/线性回归、LightGBM、XGBoost、CatBoost、随机森林，以及 champion/challenger 对比。
- **时间外验证优先**：OOT 默认选择最近的完整月份；验证集可以选择临近月，也可以从非 OOT 样本中分层抽样。
- **特征重要性更稳健**：重要特征按 LightGBM 重要性、原始 IV、分箱后 IV 加权综合排序，默认权重为 `0.4 / 0.3 / 0.3`。
- **评分卡流程贴近实战**：变量统计、分箱、WOE 趋势、IV、相关性、逐步回归、系数方向、VIF、评分卡分数、KS/AUC/PSI/lift、评分分布和策略建议。
- **树模型流程不过度套评分卡**：对 LightGBM/XGBoost 等模型使用树模型专属的缺失值处理、早停、特征重要性、过拟合/欠拟合诊断、正则化、采样比例、树深度和叶子数优化。
- **衍生变量覆盖面广**：差值、比值、均值、占比、相对差、窗口趋势、log/截尾、缺失标识、WOE 组合、2 到 3 层决策树组合特征、客群/渠道交互特征。
- **过强结果会查泄漏**：如果 KS/AUC/lift 异常好，会要求检查目标泄漏、时间泄漏、贷后字段、未来聚合、重复样本、训练/验证/OOT 污染等问题。
- **可选机构/渠道稳定性检查**：只有在存在机构或渠道字段时，才补充机构/渠道维度的样本、坏账率、缺失率、IV、PSI 或 OOS 验证。
- **异常月份只标注不默认过滤**：样本量过小、坏样本过少或月份异常时，会在报告中标注原因，默认不直接删除。
- **可输出策略候选**：建模完成后会询问是否需要策略。若需要，会列出召回率大于 1%、lift 大于 3 的高风险规则候选。
- **面向交付报告**：不是只输出 notebook 指标，而是要求形成模型开发验证报告、模型对比表、评分卡表、分箱表、lift 表、PSI 表和策略表。

### 安装

```bash
npx skills add W-Y-P/credit-risk-model
```

安装后，在 Codex 或支持 skills.sh 的 agent 中直接点名使用：

```text
使用 $credit-risk-model 开发一版申请评分卡，OOT 选择最近完整月，验证集从剩余样本分层抽样，输出模型开发验证报告。
```

### 怎么使用

你可以把它用于从零建模、复现旧评分卡、优化已有模型、对比评分卡与机器学习模型，或生成最终模型报告。

常用提示词示例：

```text
使用 $credit-risk-model 基于当前数据开发评分卡模型。目标是拒绝坏客户，Y=30天以上逾期，OOT 使用最近完整月份，评分卡参数 base_score=600、odds=0.05、pdo=60。
```

```text
使用 $credit-risk-model 训练 LightGBM 和评分卡做 champion/challenger 对比。第一版完成后请判断是否过拟合或欠拟合，再继续优化。
```

```text
使用 $credit-risk-model 参考已有模型开发验证报告，复现原评分卡，并尝试增加可部署的衍生变量。最终报告格式尽量和原报告一致。
```

```text
使用 $credit-risk-model 对这个模型做验证报告。请输出 KS、AUC、PSI、10等频分箱 lift、固定评分段表现、变量 PSI 和策略候选。
```

### 建模时会先明确的问题

skill 会在动手前尽量确认这些信息。如果你已经在任务里说明，它会直接沿用。

- 建模目标：拒绝坏客户、挑选好客户、风险定价、额度、催收、反欺诈或其它目标。
- 模型类型：评分卡、LightGBM、XGBoost、线性/逻辑回归、CatBoost、随机森林或其它模型。
- Y 值定义：如果已有明确目标就使用；如果没有，会根据业务目标、vintage、逾期滚动率和表现窗给建议。
- 样本切分：OOT 月份、验证集方式、训练集范围，是否需要机构/渠道 OOS。
- 参数调节：默认模型、轻量调参或更深的超参数搜索。
- 是否增加衍生特征：以及是否允许 WOE 组合、树组合特征，或只允许原始可部署公式。
- 报告模板和交付约束：是否有旧报告模板、固定评分段、强制变量、禁用变量、单调性、审批率目标或上线限制。
- 是否需要策略：建模完成后是否输出拒绝/复核/通过策略、cutoff 表或高 lift 规则。

### 最终输出文档会包含什么

如果用户提供报告模板，skill 会尽量保留原模板的 sheet 顺序、图形、标题和固定评分段；如果没有模板，会使用默认信用风险模型开发验证报告结构。

最终报告通常包含：

- **模型摘要**：建模目标、样本范围、Y 定义、模型类型、最终模型选择、核心指标、结论和建议。
- **样本定义与切分**：观察点、表现窗、坏样本定义、剔除规则、训练/验证/OOT 样本量、坏样本量、坏账率。
- **数据质量与变量清洗**：缺失率、同值率、异常月份标注、机构/渠道维度检查、变量删除原因。
- **特征筛选过程**：LightGBM 重要性、原始 IV、分箱后 IV、综合排序、相关性、PSI、Null Importance 可选结果。
- **变量剔除日志**：每轮剔除的阈值、变量、原因和保留依据。
- **衍生变量说明**：衍生公式、父变量、部署方式、单变量效果、入模原因。
- **评分卡分箱表**：每个变量的分箱、样本数、坏样本数、坏账率、WOE、IV、系数、分数。
- **模型参数与变量表**：逻辑回归系数、评分卡参数、树模型超参数、入模变量、重要性或解释性结果。
- **模型验证结果**：train/validation/OOT 的 KS、AUC、bad rate、PSI、lift、分数分布、稳定性对比。
- **样本表现统计**：10 等频分箱表现，或按模板要求的固定评分段表现，例如 `<200`、`200-240`、`240-280`。
- **PSI 与稳定性**：分数 PSI、变量 PSI、分箱 PSI、月份/机构/渠道稳定性。
- **泄漏与异常检查**：对过强模型结果、贷后字段、未来聚合、时间穿越、样本污染的检查说明。
- **策略建议**：评分段 cutoff、通过/复核/拒绝建议、审批率、坏样本捕获率、lift，以及召回率大于 1%、lift 大于 3 的高风险规则候选。
- **可复现产物清单**：`model_summary.json`、`model_comparison.csv`、`selected_features_*.csv`、`scorecard_bins_*.csv`、`scores_*.csv`、`feature_screening_audit.csv`、`variable_removal_log.csv` 和最终报告 workbook/document。

## English Overview

### What It Is

`credit-risk-model` is an agent skill for end-to-end credit-risk modeling. It helps an agent clarify the modeling request, choose the right workflow, train and validate models, compare candidates, produce scorecards, and generate model development and validation reports.

It is designed for application risk, behavior risk, collection prioritization, anti-fraud review, customer selection, limit/pricing support, and other credit-risk use cases.

### Key Highlights

- **Modeling intake first**: clarifies the business goal, model family, target definition, sample split, tuning depth, derived features, report template, and strategy needs.
- **Multiple model families**: supports scorecards, WOE logistic regression, logistic/linear regression, LightGBM, XGBoost, CatBoost, random forest, and champion/challenger comparison.
- **Time-aware validation**: defaults OOT to the nearest/latest complete month; validation can be a nearby month or a stratified random split from non-OOT data.
- **Robust feature ranking**: combines LightGBM importance, raw IV, and binned IV with default weights `0.4 / 0.3 / 0.3`.
- **Practical scorecard workflow**: variable statistics, WOE binning, IV, correlation filtering, stepwise logistic regression, coefficient direction checks, VIF, score scaling, validation, score distribution, and strategy suggestions.
- **Tree-model-specific workflow**: uses early stopping, feature importance/SHAP, calibration checks, overfitting/underfitting diagnosis, and targeted tuning for LightGBM/XGBoost-style models.
- **Broad derived-feature search**: differences, ratios, means, shares, relative gaps, trend features, transformations, missing indicators, WOE combinations, 2-to-3-level tree combination features, and group/channel interactions.
- **Leakage audit for overly strong results**: checks post-outcome fields, future aggregation, date leakage, duplicate keys, target-window overlap, and train/validation/OOT contamination.
- **Optional institution/channel checks**: only runs institution/channel sample, missingness, IV, PSI, or OOS analysis when such fields exist.
- **Report-oriented deliverables**: produces artifacts that can be audited, reproduced, and turned into a model development validation report.

### Install

```bash
npx skills add W-Y-P/credit-risk-model
```

Then call the skill explicitly in a supported agent:

```text
Use $credit-risk-model to train and validate an application scorecard. Use the latest complete month as OOT, stratified validation from the remaining sample, and generate a model development validation report.
```

### Usage Examples

```text
Use $credit-risk-model to build a scorecard for bad-customer rejection. Target is 30+ DPD, OOT is the latest complete month, and the score scale is base_score=600, odds=0.05, pdo=60.
```

```text
Use $credit-risk-model to train LightGBM and XGBoost challenger models against the existing scorecard. Diagnose overfitting or underfitting after the first baseline, then optimize.
```

```text
Use $credit-risk-model to reproduce the original scorecard from the existing validation report, add deployable derived features, compare lift, and keep the final report format aligned with the original template.
```

```text
Use $credit-risk-model to produce validation outputs: KS, AUC, PSI, decile lift, fixed score-band performance, variable PSI, and optional strategy candidates.
```

### Final Report Contents

When a template is provided, the skill tries to preserve sheet order, labels, charts, drawings, score bands, and report style. Without a template, it uses a default credit-risk model development validation report structure.

Typical final outputs include:

- **Executive summary**: modeling goal, target definition, model family, selected model, key metrics, conclusion, and recommendation.
- **Sample definition and split**: observation point, performance window, bad definition, exclusions, train/validation/OOT counts, bad counts, and bad rates.
- **Data quality and cleaning**: missing rates, dominant-value rates, abnormal-month marks, institution/channel checks when available, and removal reasons.
- **Feature screening**: LightGBM importance, raw IV, binned IV, composite ranking, correlation, PSI, and optional Null Importance.
- **Variable removal log**: each screening round, thresholds, removed variables, and reasons.
- **Derived features**: formulas, parent variables, deployability notes, univariate performance, and reason for inclusion.
- **Scorecard bins**: bin definitions, counts, bad counts, bad rates, WOE, IV contribution, coefficients, and points.
- **Model parameters and features**: logistic coefficients, score scale, tree-model hyperparameters, selected variables, feature importance, and explanation outputs.
- **Validation metrics**: train/validation/OOT KS, AUC, bad rate, PSI, lift, score distribution, and stability comparison.
- **Performance tables**: decile lift tables or fixed score-band tables such as `<200`, `200-240`, and `240-280` when required by the template.
- **PSI and stability**: score PSI, variable PSI, bin PSI, monthly stability, and institution/channel stability when available.
- **Leakage and exception checks**: documentation for unusually strong results, post-outcome fields, future aggregation, time leakage, and sample contamination.
- **Strategy suggestions**: cutoffs, approval/review/reject bands, approval rate, bad capture, lift, and high-lift rule candidates with recall greater than 1% and lift greater than 3 when strategy output is requested.
- **Reproducible artifacts**: `model_summary.json`, `model_comparison.csv`, `selected_features_*.csv`, `scorecard_bins_*.csv`, `scores_*.csv`, `feature_screening_audit.csv`, `variable_removal_log.csv`, and the final workbook or document.

## Skill File

See [SKILL.md](./SKILL.md).
