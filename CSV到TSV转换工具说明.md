# CSV到TSV转换工具

这个工具可以将`progress.csv`格式的训练日志转换为`produce_results.py`需要的TSV格式，以便生成可视化结果。

## 功能

- 将简化的CSV格式转换为详细的TSV格式
- 自动生成所需的配置文件
- 支持MetaWorld任务环境的数据格式

## 使用方法

基本用法：

```bash
python convert_csv_to_tsv.py --input your_progress.csv --output saved_logs/cl/cl_custom
```

参数说明：

- `--input`: 输入的CSV文件路径（必须）
- `--output`: 输出目录路径（默认：saved_logs/cl/cl_custom）
- `--method`: 方法名称，用于配置文件（默认：cotasp）

## 示例

假设你有一个名为`progress.csv`的文件，格式类似于：

```
0-hammer-v1/return,0-hammer-v1/success,1-push-wall-v1/return,...
-29.87,0.0,-29.34,0.0,...
```

执行以下命令：

```bash
python convert_csv_to_tsv.py --input progress.csv --output saved_logs/cl/my_experiment
```

转换后，你可以使用`produce_results.py`来生成可视化结果：

```bash
python produce_results.py
```

## 转换映射说明

- CSV中的`{n}-{task}/return` → TSV中的`test/stochastic/{n}/{task}/return/avg`等
- CSV中的`{n}-{task}/success` → TSV中的`test/stochastic/{n}/{task}/success`
- 其他缺失的指标会填充默认值

## 输出格式

输出目录将包含以下文件：

1. `progress.tsv`: 转换后的TSV格式数据文件
2. `config.json`: 配置文件，用于`produce_results.py`

## 表头对应关系

下面是CSV和TSV文件中的表头对应关系：

| CSV格式 | TSV格式 |
|---------|---------|
| `0-hammer-v1/return` | `test/stochastic/0/hammer-v1/return/avg` |
| `0-hammer-v1/success` | `test/stochastic/0/hammer-v1/success` |
| `1-push-wall-v1/return` | `test/stochastic/1/push-wall-v1/return/avg` |
| ... | ... |
| `avg_return` | `train/return/avg` |
| `x` | `total_env_steps` |
| `steps_per_task` | `current_task_steps` |

TSV格式中的其他统计信息（如`return/std`, `return/max`, `return/min`, `ep_length`等）会根据CSV中的值进行估算或填充默认值。 