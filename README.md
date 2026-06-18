# AntBO Reproduction

This README records current progress reproducing AntBO with Absolut on the antibody-antigen benchmark.

The current goal is:

```text
1 seed × 158 antigens × 200 BO iterations
```

This run is mainly used as a full-pipeline check. It verifies that the AntBO code, Absolut evaluator, antigen list, structure files, GPU environment, and result-saving process can work together. 

## 1. Paths

Main project directory:

```text
/mnt/data0/shared/AntBO
```

AntBO code:

```text
/mnt/data0/shared/AntBO/HEBO/AntBO
```

Absolut directory:

```text
/mnt/data0/shared/AntBO/Absolut
```

Main result directory:

```text
/mnt/data0/shared/AntBO/HEBO/AntBO/results/BO_transformed_overlap
```

Conda environment:

```bash
conda activate DGM
```

GPU used for the current run:

```bash
export CUDA_VISIBLE_DEVICES=2
```

## 2. Config

The main config file is:

```text
/mnt/data0/shared/AntBO/HEBO/AntBO/bo/config.yaml
```

Important settings:

```yaml
device: cuda
max_iters: 200
save_path: /mnt/data0/shared/AntBO/HEBO/AntBO/results
```


## 3. Antigen Lists

The original full antigen file is:

```text
/mnt/data0/shared/AntBO/HEBO/AntBO/dataloader/antigens_all.txt
```

This file contains duplicated antigen IDs. I created a cleaned list with 158 unique antigens:

```text
/mnt/data0/shared/AntBO/antigens_all_unique_158.txt
```

The 12 core antigens are listed in:

```text
/mnt/data0/shared/AntBO/HEBO/AntBO/dataloader/core_antigens.txt
```

The 12 core antigens are:

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
These 12 core antigens have already completed for seed 42.


## 4. Current Status

The run started from the cleaned 158-antigen list. The first two antigens were completed first:

```text
1ADQ_A
1FBI_X
```

Their log is:

```text
/mnt/data0/shared/AntBO/full_158_1seed_gpu.log
```

After that, the run was continued from `1H0D_C`. The continuation antigen list is:

```text
/mnt/data0/shared/AntBO/antigens_158_from_1H0D_C.txt
```

The continuation log is:

```text
/mnt/data0/shared/AntBO/full_158_1seed_from_1H0D_C.log
```
When continuing the run, use `tee -a` so the existing log is appended instead of overwritten.


## 5. Run Command

To continue from the current antigen list:

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

If starting over from the full cleaned antigen list, use:

```text
/mnt/data0/shared/AntBO/antigens_all_unique_158.txt
```

However, this is not recommended unless intentionally rerunning already completed antigens.



## 6. Results

Results are saved under:

```text
/mnt/data0/shared/AntBO/HEBO/AntBO/results/BO_transformed_overlap
```

Each completed antigen has its own folder. The folder name usually follows this pattern:

```text
antigen_<ANTIGEN_ID>_kernel_transformed_overlap_search-strat_local_seed_42_cdr_constraint_True_seqlen_11
```

The main output file is:

```text
results.csv
```

The folder also contains saved random states and optimizer state:

```text
torch_rd_state.pt
random_rd_state.pkl
np_rd_state.pkl
optim.pkl
```


## 7. Completed Antigens So Far

At the time of this handoff, the following 25 antigens have completed for seed 42:

```text
1ADQ_A
1FBI_X
1FNS_A
1FSK_A
1H0D_C
1NSN_S
1OB1_C
1OSP_O
1PKQ_J
1RJL_C
1S78_B
1TQB_A
1WEJ_F
1YJD_C
1ZTX_E
2B2X_A
2DD8_S
2HFG_R
2IH3_C
2JEL_P
2R29_A
2R56_A
2YPV_A
3RAJ_A
3VRL_C
```

The 12 core antigens are included in this completed set.


## 8. Example Antigen: `2R29_A`

For a single completed antigen, the related files are stored in several places.

### Antigen ID

```text
2R29_A
```

### Result folder

The result folder for this antigen should be found under the main result directory:

```text
/mnt/data0/shared/AntBO/HEBO/AntBO/results/BO_transformed_overlap/antigen_2R29_A_kernel_transformed_overlap_search-strat_local_seed_42_cdr_constraint_True_seqlen_11
```

### Main result file

Inside the antigen result folder, the main output file is:

```text
results.csv
```

This file records the sequences evaluated during BO and the corresponding scores.




## 9. Issues Encountered

### Relative `save_path`

Originally, `save_path` was relative. This caused results to be saved in the wrong place after Absolut was called.

It has been changed to:

```text
/mnt/data0/shared/AntBO/HEBO/AntBO/results
```

### Missing Absolut structure files

If the log repeatedly shows:

```text
(2, 'No such file or directory')
Starting Trial 1 for antigen ANTIGEN_ID
```

it usually means Absolut cannot find the required structure file.

To check which file is needed:

```bash
cd /mnt/data0/shared/AntBO/Absolut
./src/bin/Absolut info_filenames ANTIGEN_ID
```

If the file is missing, download it using the `curl` command printed by Absolut and unzip it.


## 10. Resource Notes

Observed runtime varies by antigen. For example:

```text
1ADQ_A: about 63.5 min
1FBI_X: about 36.1 min
```

A single AntBO process used relatively little GPU memory, around 1–2 GB in my run. GPU utilization was not very high, so the bottleneck is likely not pure GPU compute. Runtime likely also depends on Absolut evaluation, CPU/I/O, and the sequential BO loop.

If more speed is needed, the remaining antigen list can be split across multiple GPUs. In that case, each process should use a different antigen list and a different log file.
