# 这里是我的记录
## 1.使用SCCM的预处理
因为之前是在Swallow上做的，先转为swallow格式吧，预处理转化过程：
```
# 1.1 原始GTT23文件放入 pretrain_dataset/ 文件夹
# 1.2 使用GTT23的预处理文件pretrain_dataset\process_GTT.py 生成GTT23_train.hdf5放在在pretrain_dataset/ 文件夹里
(.venv) wrwut@DESKTOP-F6LBQVQ:/mnt/d/workspace/WF-SCSM/pretrain_dataset$ python process_GTT.py
原始记录数： 13900621
处理协议：首个 relay BEGIN 起点，保留后续 RELAY/RELAY_EARLY，少于 20 个单元的记录丢弃，时间归零
Converting GTT23: 100%|████████████████████████████████████████████████████████| 13900621/13900621 [1:08:37<00:00, 3375.78record/s, kept=7222474, no_begin=0, under_20=6678147]
保留 7222474 条；剔除无 BEGIN 记录 0 条；剔除少于 20 个单元的记录 6678147 条
转换完成，用时 1.14 小时
保存位置：./GTT23_train.hdf5
# 1.3 使用pretrain_process_dataset的处理文件读取上面第二步的生成文件来进一步处理，可选格式NetCLR,Swallow几种，我们选Swallow,其会生成pretrain_process_dataset/GTT23-Swallow-origin/文件夹（具体见对应yaml文件）
(.venv) wrwut@DESKTOP-F6LBQVQ:/mnt/d/workspace/WF-SCSM$ python pretrain_process_dataset/process_swallow.py --config pretrain_process_dataset/config_pretrain_dataset/GTT_swallow_origin.yaml
/mnt/d/workspace/WF-SCSM/.venv/lib/python3.10/site-packages/numpy/lib/format.py:934: UserWarning: metadata on a dtype is not saved to an npy/npz. Use another format (such as pickle) to store it.
  descr=dtype_to_descr(dtype),
Extracting Swallow CIF: 100%|█████████████████████████████████████████████████████████████████████████████████████████████████| 7222474/7222474 [1:03:01<00:00, 1910.18trace/s] 
Saved 7222474 traces with shape (7222474, 2, 1000) to /mnt/d/workspace/WF-SCSM/pretrain_process_dataset/GTT23-Swallow-origin
# 1.4 查看生成的数据文件notebook
# 1.5 择single,origin,muti3种方式不同的点在于slot_duration和slot_index，single 就是 origin 的"2.7 倍加密版"：duration 恰好是 origin 的 1/2.7，slot_count 是 2.7 倍，所以覆盖时长（20–80s）和触发 clamp 的 load_time 阈值完全一样，区别纯粹是网格更细 → 同一 trace 的包分布刻画更精确（也就是注释里"2700长度"对应 SCSM 的 max_matrix_length: 2700）。
multi 面向长流量：覆盖到 320s，超过 ≈320s 的包会被 slot_index >= slot_len 截断丢弃（对应 SCSM multi 的 max_matrix_length: 7200）。
# 1.6 如果要TAM矩阵，即用Swallow的就行，只需要方向序列，即用NetCLR的就行
```
