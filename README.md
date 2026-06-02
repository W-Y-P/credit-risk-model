# Credit Risk Model Skill

[![skills.sh](https://skills.sh/b/W-Y-P/credit-risk-model)](https://skills.sh/W-Y-P/credit-risk-model)

Reusable agent skill for end-to-end credit risk modeling, including scorecards, logistic regression, LightGBM, XGBoost, validation reports, strategy suggestions, and model diagnostics.

## Install

```bash
npx skills add W-Y-P/credit-risk-model
```

## What It Covers

- Modeling intake: objective, model family, target definition, split, tuning depth, derived features, report template, and strategy needs.
- Sample design: nearest complete month OOT by default, optional validation month or stratified validation, optional institution/channel OOS.
- Scorecard workflow: variable statistics, WOE binning, IV, correlation, stepwise logistic regression, VIF, score scaling, validation, score distribution, and strategy suggestions.
- Tree-model workflow: LightGBM/XGBoost/CatBoost style training, early stopping, feature importance/SHAP, calibration, overfitting/underfitting diagnosis, and optimization.
- Feature screening: weighted LightGBM importance + IV + binned IV, optional Null Importance, PSI, and variable removal logs.
- Derived features: raw arithmetic, window trends, WOE combinations, and 2-3-level decision-tree combination rules.
- Validation: KS, AUC, PSI, lift, score bands, OOT stability, leakage audit for unusually strong results, and report deliverables.

## Skill

See [SKILL.md](./SKILL.md).
