|文件|是否保留|原因|
|---|---|---|
|`config_lgb.py`|保留|路径、字段映射、常量配置|
|`data_lgb.py`|保留|读取数据、中文列名转英文|
|`features_lgb.py`|保留|特征工程核心|
|`labels_lgb.py`|保留|构造 `future_return_5d`、排序标签|
|`dataset_lgb.py`|保留|训练数据集构建与缺失处理|
|`validation_lgb.py`|保留|评估函数，后续调试还会用|
|`train_lgb_ranker.py`|保留|Ranker 训练脚本|
|`predict_lgb_ranker_train_lastday.py`|保留|正确无泄漏预测脚本|
|`make_confidence_result.py`|保留|生成置信度加权结果|
|`finalize_result.py`|保留|最终安全切换 `result.csv`|
