# AntBO-reproduction
## Goal 
The reproduction is for running AntBO with Absolut on the antibody=antigen benchmark. 

Current Target:
```text
1 seed × 158 antigens × 200 BO iterations
```



## 2. 服务器与主要路径

服务器登录后，主要工作目录如下：

```text
总项目路径：
/mnt/data0/shared/AntBO

AntBO 路径：
/mnt/data0/shared/AntBO/HEBO/AntBO

Absolut 路径：
/mnt/data0/shared/AntBO/Absolut

目前的结果路径：
/mnt/data0/shared/AntBO/HEBO/AntBO/results/BO_transformed_overlap
```

Conda 环境：

```bash
conda activate DGM
```

GPU 设置：

```bash
export CUDA_VISIBLE_DEVICES=2
```


## 3. 重要配置文件

AntBO 配置文件：

```text
/mnt/data0/shared/AntBO/HEBO/AntBO/bo/config.yaml
```

当前关键配置应为：

```yaml
device: cuda
max_iters: 200
save_path: /mnt/data0/shared/AntBO/HEBO/AntBO/results
```

Absolut 路径应指向：

```yaml
path: /mnt/data0/shared/AntBO/Absolut
```

注意：`save_path` 必须使用绝对路径，否则程序可能把结果错误保存到 Absolut 目录下。

---

## 4. Antigen list

原始全量 antigen 文件为：

```text
/mnt/data0/shared/AntBO/HEBO/AntBO/dataloader/antigens_all.txt
```

该文件有重复项。已整理出 158 个唯一 antigen：

```text
/mnt/data0/shared/AntBO/antigens_all_unique_158.txt
```

12 个 core antigens 文件为：

```text
/mnt/data0/shared/AntBO/HEBO/AntBO/dataloader/core_antigens.txt
```

core antigens 包括：

```text
1ADQ_A
1FBI_X
1H0D_C
1NSN_S
1OB1_C
1WEJ_F
2YPV_A
3RAJ_A
3VRL_C
1S78_B
2JEL_P
2DD8_S
```

目前 12 个 core antigens 已经全部完成 seed 42 的运行。


## 5. 当前运行状态

已经完成并保存的前两个 antigen：

```text
1ADQ_A
1FBI_X
```

对应原始 log：

```text
/mnt/data0/shared/AntBO/full_158_1seed_gpu.log
```

后来从 `1H0D_C` 开始继续运行，当前继续运行的 list 为：

```text
/mnt/data0/shared/AntBO/antigens_158_from_1H0D_C.txt
```

当前继续运行的 log 为：

```text
/mnt/data0/shared/AntBO/full_158_1seed_from_1H0D_C.log
```

注意：后续继续跑时建议使用 `tee -a`，避免覆盖已有 log。

---

## 6. 正式运行命令

从当前 list 继续运行：

```bash
cd /mnt/data0/shared/AntBO/HEBO/AntBO
conda activate DGM
export CUDA_VISIBLE_DEVICES=2

python -u ./bo/main.py \
  --config ./bo/config.yaml \
  --n_trials 1 \
  --seed 42 \
  --antigens_file /mnt/data0/shared/AntBO/antigens_158_from_1H0D_C.txt \
  2>&1 | tee -a /mnt/data0/shared/AntBO/full_158_1seed_from_1H0D_C.log
```

如果重新从完整 158 antigen list 开始运行，则使用：

```bash
--antigens_file /mnt/data0/shared/AntBO/antigens_all_unique_158.txt
```

但目前不建议从头重跑，因为已有部分 antigen 已完成。

---



## 8. 输出结果

每个 antigen 的结果保存在：

```text
/mnt/data0/shared/AntBO/HEBO/AntBO/results/BO_transformed_overlap/
```

每个 antigen 有一个独立文件夹，例如：

```text
antigen_1ADQ_A_kernel_transformed_overlap_search-strat_local_seed_42_cdr_constraint_True_seqlen_11
```

每个文件夹中主要结果文件为：

```text
results.csv
```

同时还会保存随机状态和 optimizer 状态：

```text
torch_rd_state.pt
random_rd_state.pkl
np_rd_state.pkl
optim.pkl
```

快速查看已有结果文件：

```bash
find /mnt/data0/shared/AntBO/HEBO/AntBO/results/BO_transformed_overlap \
  -name results.csv | wc -l
```

查看某个 antigen 的结果：

```bash
head -20 /mnt/data0/shared/AntBO/HEBO/AntBO/results/BO_transformed_overlap/antigen_1ADQ_A_kernel_transformed_overlap_search-strat_local_seed_42_cdr_constraint_True_seqlen_11/results.csv
```

---

## 9. 已遇到并解决的问题

### 1. `save_path` 相对路径问题

原本 `save_path: ./results` 会导致结果保存到错误目录。已改成绝对路径：

```text
/mnt/data0/shared/AntBO/HEBO/AntBO/results
```

### 2. Absolut structure 文件找不到

错误形式：

```text
(2, 'No such file or directory')
Starting Trial 1 for antigen XXXX
```

原因通常是对应 antigen 的 structure 文件不在 Absolut 根目录。

解决方法：

```bash
cd /mnt/data0/shared/AntBO/Absolut

./src/bin/Absolut info_filenames ANTIGEN_ID
```

该命令会显示需要的 structure 文件名和下载链接。如果缺失，则下载并解压。

如果 structure 文件位于：

```text
/mnt/data0/shared/AntBO/Absolut/src/structures/
```

但程序在 Absolut 根目录找不到，可以建立软链接：

```bash
cd /mnt/data0/shared/AntBO/Absolut

for f in /mnt/data0/shared/AntBO/Absolut/src/structures/*Structures.txt
do
  ln -sf "$f" "$(basename "$f")"
done
```

目前 Absolut 根目录下已经能看到 222 个 structure 文件。

### 3. 已单独补过的 antigen

运行中曾因缺 structure 文件卡住过：

```text
1FBI_X
1H0D_C
2HFG_R
```

其中 `2HFG_R` 需要的文件为：

```text
SULDRUDURRUSULDDLRDRLRLRRL-10-11-0Structures.txt
```

已下载并解压成功。

---

## 10. 资源需求估计

当前运行规模：

```text
1 seed × 158 antigens × 200 iterations
```

单个 antigen 的运行时间差异较大。

```text
1ADQ_A：约 63.5 分钟
1FBI_X：约 36.1 分钟
```

GPU 使用情况：

```text
单进程显存占用较小，约 1–2 GB
GPU 利用率不高
```

主要耗时可能来自 Absolut black-box evaluation、




