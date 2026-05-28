# 代码说明

## 环境配置

本项目用于 2026 中国高校计算机大赛——大数据挑战赛“基于历史数据预测未来股价收益”赛题。

本方案主要依赖 Python 机器学习生态，核心模型为 LightGBM Ranker。

### Python 版本

```text
Python 3.10
```

### 主要依赖

```text
numpy
pandas
scikit-learn
lightgbm
joblib
```

如需重新安装环境，可根据项目中的依赖配置文件进行恢复。复现训练和预测阶段不需要联网。

### 运行入口

训练：

```bash
sh train.sh
```

推理：

```bash
sh test.sh
```

最终结果文件：

```text
output/result.csv
```

结果格式检查：

```bash
python tools/check_result.py --path output/result.csv
```

---

## 数据

### 数据来源

本方案使用公开可获取的沪深 300 成分股历史行情数据，字段包括：

```text
股票代码
日期
开盘
收盘
最高
最低
成交量
成交额
振幅
涨跌额
换手率
涨跌幅
```

数据文件存放于：

```text
data/train.csv
data/test.csv
```

其中：

- `train.csv`：用于训练模型、构造特征和标签；
    
- `test.csv`：用于当前预测阶段的输入数据；
    
- `output/result.csv`：模型最终生成的股票组合结果。
    

### 字段映射

程序内部会将中文字段统一映射为英文变量名：

```text
股票代码 -> stock_id
日期 -> date
开盘 -> open
收盘 -> close
最高 -> high
最低 -> low
成交量 -> volume
成交额 -> amount
振幅 -> amplitude
涨跌额 -> price_change
换手率 -> turnover
涨跌幅 -> pct_change
```

相关代码：

```text
code/src_lgb/config_lgb.py
code/src_lgb/data_lgb.py
```

### 数据处理说明

数据读取后会进行以下处理：

1. 将中文列名映射为英文内部字段；
    
2. 将日期字段转换为时间格式；
    
3. 将股票代码统一为 6 位字符串；
    
4. 按 `stock_id`、`date` 排序；
    
5. 构造量价特征、波动率特征和横截面排名特征；
    
6. 将无穷值替换为缺失值；
    
7. 对缺失特征使用训练集特征中位数填充。
    

---

## 预训练模型

本方案未使用任何外部预训练模型、embedding 或词典。

模型训练完全基于给定历史行情数据以及由行情数据构造出的特征和排序标签。

---

## 算法

### 整体思路介绍

本赛题的目标是从沪深 300 成分股中选出未来一周收益表现最好的股票组合。由于最终评价只关注所选股票组合的收益，而不是对所有股票收益率的精确回归，因此本方案将任务建模为 **横截面排序问题**。

具体思路是：

```text
历史行情数据
    ↓
量价特征工程
    ↓
构造未来收益标签
    ↓
转化为每日横截面排序标签
    ↓
训练 LightGBM Ranker
    ↓
预测股票排序分数
    ↓
选择 Top5 股票
    ↓
置信度加权生成 result.csv
```

模型不直接以预测收益率数值为最终目标，而是学习同一交易日内股票之间的相对排序关系。

---

### 方法的创新点

本方案的主要设计点包括：

1. **排序学习建模**  
    使用 LightGBM Ranker，将每日股票池作为一个 query group，直接学习横截面排序关系。
    
2. **横截面排名特征**  
    对关键量价特征按交易日进行横截面 percentile rank 处理，削弱市场整体涨跌、整体放量或缩量带来的非平稳影响。
    
3. **多类量价因子融合**  
    同时使用收益率、均线偏离、波动率、Parkinson 波动率、成交额、换手率等特征。
    
4. **Top5 置信度加权**  
    最终不是简单等权，而是根据 Ranker 排名对 Top5 股票进行置信度加权。
    

---

### 网络结构

本方案不使用深度神经网络结构。

核心模型为：

```text
LightGBM LGBMRanker
```

每个交易日为一个 query group，同一日内的股票作为该 group 内的排序样本。

主要模型参数：

```text
objective = lambdarank
metric = ndcg
eval_at = [5, 10]
learning_rate = 0.02
num_leaves = 31
n_estimators = 50
subsample = 0.8
colsample_bytree = 0.8
reg_alpha = 0.1
reg_lambda = 1.0
random_state = 2026
```

如环境中启用了确定性训练参数，则 LightGBM 会尽量保持训练结果稳定。

---

### 损失函数

模型使用 LightGBM 的 LambdaRank 排序目标：

```text
objective = lambdarank
```

训练标签为每日横截面收益分桶标签 `label_bucket`。

未来收益定义为：

```text
future_return_5d = open[T+5] / open[T+1] - 1
```

然后对同一交易日内的股票收益进行横截面排名，得到 `rank_label`，再构造分桶标签：

```text
rank_label >= 0.95 -> 4
rank_label >= 0.80 -> 3
rank_label >= 0.50 -> 2
rank_label >= 0.20 -> 1
其他 -> 0
```

该设计使模型更关注未来收益靠前的股票，而不是过度拟合具体收益率数值。

相关代码：

```text
code/src_lgb/labels_lgb.py
```

---

### 数据扩增

本方案未使用额外数据扩增方法。

模型主要依赖历史行情数据构造滚动窗口特征和横截面排名特征。

---

### 模型集成

当前最终提交方案不使用多模型集成。

开发阶段曾对比过：

```text
LightGBM Regressor
LightGBM Ranker
不同特征子集
不同后处理权重方案
```

最终选择：

```text
LightGBM Ranker + all_features + 置信度加权
```

---

### 算法的其他细节

#### 1. 特征工程

使用的主要特征包括：

基础价格特征：

```text
open_close_return
high_low_range
close_to_high
close_to_low
gap_return
```

多窗口收益率：

```text
return_1d
return_3d
return_5d
return_10d
return_20d
```

均线与偏离特征：

```text
ma_5
ma_10
ma_20
close_ma5_ratio
close_ma10_ratio
close_ma20_ratio
```

波动率特征：

```text
volatility_5d
volatility_10d
volatility_20d
```

Parkinson 波动率：

```text
parkinson_raw = log(high / low)^2
parkinson_vol_5d
parkinson_vol_10d
parkinson_vol_20d
```

横截面排名特征：

```text
cs_rank_return_5d
cs_rank_return_10d
cs_rank_return_20d
cs_rank_volatility_20d
cs_rank_high_low_range
cs_rank_turnover
cs_rank_amount
```

相关代码：

```text
code/src_lgb/features_lgb.py
```

#### 2. 组合构建

模型对预测截面内所有股票生成 `score_lgb_ranker`，按分数从高到低排序，选择前 5 只股票。

最终采用置信度加权：

```text
Top1 -> 0.4
Top2 -> 0.2
Top3 -> 0.2
Top4 -> 0.1
Top5 -> 0.1
```

最终输出权重和为 1.0，股票数为 5，满足提交要求。

相关代码：

```text
code/src_lgb/make_confidence_result.py
code/src_lgb/predict_lgb_ranker.py
```

#### 3. 复现性控制

本方案使用固定随机种子：

```text
SEED = 2026
```

训练和预测过程中按 `date`、`stock_id` 进行稳定排序。模型参数中设置固定 `random_state`，以提升复现稳定性。

---

## 训练流程

训练入口：

```bash
sh train.sh
```

`train.sh` 调用 LightGBM Ranker 训练流程，主要步骤如下：

1. 读取 `data/train.csv`；
    
2. 执行字段映射和基础数据处理；
    
3. 按股票代码和日期排序；
    
4. 构造未来收益标签 `future_return_5d`；
    
5. 构造横截面排序标签 `rank_label`；
    
6. 构造分桶标签 `label_bucket`；
    
7. 构造量价特征、波动率特征、横截面 rank 特征；
    
8. 删除无法构造未来收益标签的样本；
    
9. 将无穷值替换为缺失值；
    
10. 使用训练集特征中位数填充缺失值；
    
11. 按交易日构造 LightGBM Ranker 的 query group；
    
12. 训练 LightGBM Ranker；
    
13. 保存模型和特征列表到 `model/` 目录。
    

训练相关代码：

```text
code/src_lgb/train_lgb_ranker.py
code/src_lgb/dataset_lgb.py
code/src_lgb/features_lgb.py
code/src_lgb/labels_lgb.py
```

---

## 推理流程

推理入口：

```bash
sh test.sh
```

`test.sh` 执行最终预测流程，主要步骤如下：

1. 读取训练数据和预测数据；
    
2. 执行与训练阶段一致的字段映射和特征构造；
    
3. 使用训练阶段确定的特征列；
    
4. 训练或加载 LightGBM Ranker；
    
5. 对预测截面内所有候选股票生成排序分数；
    
6. 按分数从高到低排序；
    
7. 选取 Top5 股票；
    
8. 使用置信度权重 `[0.4, 0.2, 0.2, 0.1, 0.1]`；
    
9. 生成最终提交文件：
    

```text
output/result.csv
```

10. 调用结果检查脚本确认格式合法：
    

```bash
python tools/check_result.py --path output/result.csv
```

推理相关代码：

```text
code/src_lgb/predict_lgb_ranker.py
code/src_lgb/make_confidence_result.py
tools/check_result.py
```

---

## 文件结构说明

项目主要结构如下：

```text
app/
  code/
    src_lgb/
      config_lgb.py
      data_lgb.py
      features_lgb.py
      labels_lgb.py
      dataset_lgb.py
      train_lgb_ranker.py
      predict_lgb_ranker.py
      make_confidence_result.py
      finalize_result.py

      validation_lgb.py
      train_lgb_reg.py
      cv_lgb_ranker.py
      cv_lgb_ranker_strict.py
      cv_postprocess_lgb.py
      analyze_lgb_result.py
      summarize_candidates.py
      split_dataset_lgb.py

  data/
    train.csv
    test.csv

  model/
    lgb_ranker.pkl
    lgb_ranker_features.json

  output/
    result.csv

  temp/
    lgb_ranker_test_scores.csv

  tools/
    check_result.py

  init.sh
  train.sh
  test.sh
  readme.md
```

其中：

- `code/src_lgb/`：LightGBM Ranker 主流程和实验验证脚本；
    
- `data/`：训练和预测数据；
    
- `model/`：训练得到的模型及特征列表；
    
- `output/`：最终结果文件；
    
- `temp/`：中间预测分数；
    
- `tools/`：结果检查等辅助工具；
    
- `train.sh`：训练入口；
    
- `test.sh`：推理入口；
    
- `readme.md`：代码说明文件。
    

---

## 实验验证说明

开发阶段使用时间序列滚动验证，而非随机划分，避免未来信息泄露。

主要验证脚本：

```text
code/src_lgb/cv_lgb_ranker.py
code/src_lgb/cv_lgb_ranker_strict.py
code/src_lgb/cv_postprocess_lgb.py
```

验证指标包括：

```text
Top5 组合累计收益
Top5 平均收益
Top5 最差收益
Hit@10
Rank IC
NDCG@5
```

对比过的特征版本：

```text
A: all_features
B: no_raw_price
C: rank_only
```

最终选择 `all_features` 版本作为最终模型输入。

后处理阶段对比过：

```text
Top5 等权
置信度加权
高波动过滤
短期暴涨过滤
综合风险过滤
Top3 集中持仓
```

最终选择置信度加权方案。

---

## 其他注意事项

1. 本方案不使用外部预训练模型。
    
2. 本方案主要贡献为机器学习排序模型，包括模型训练和预测。
    
3. 训练和预测阶段不需要联网。
    
4. 若更新 `data/train.csv` 或 `data/test.csv`，需重新运行：
    

```bash
sh train.sh
sh test.sh
```

5. 若修改特征、标签或模型参数，建议重新运行滚动验证脚本。
    
6. 最终提交前应确认：
    

```bash
python tools/check_result.py --path output/result.csv
```

检查通过。

7. 可通过连续运行以下命令检查结果稳定性：
    

```bash
sh train.sh
sh test.sh
md5sum output/result.csv

sh train.sh
sh test.sh
md5sum output/result.csv
```

若两次 `md5sum` 一致，则说明当前环境下结果文件可稳定复现。

---

## 最终提交结果

最终提交文件为：

```text
output/result.csv
```

格式为：

```csv
stock_id,weight
xxxxxx,0.4
xxxxxx,0.2
xxxxxx,0.2
xxxxxx,0.1
xxxxxx,0.1
```

其中：

- `stock_id` 为 6 位股票代码；
    
- `weight` 为股票权重；
    
- 股票数量不超过 5；
    
- 权重和不超过 1.0。