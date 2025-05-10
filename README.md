# Continual World

Continual World is a benchmark for continual reinforcement learning. It contains realistic robotic tasks which come from
[MetaWorld](https://github.com/rlworkgroup/metaworld).

The core of our benchmark is CW20 sequence, in which 20 tasks are run, each with budget of 1M steps.

We provide the complete source code for the benchmark together with the tested algorithms implementations and code for
producing result tables and plots.

See also the [paper](https://arxiv.org/abs/2105.10919) and
the [website](https://sites.google.com/view/continualworld/home).

![CW20 sequence](./assets/images/cw20.png)

# Installation

You can either install directly in Python environment
(like virtualenv or conda), or build containers -- Docker or Singularity.

## Standard installation (directly in environment)

First, you'll need [MuJoCo](http://www.mujoco.org/) simulator. Please follow
the [instructions](https://github.com/openai/mujoco-py#install-mujoco)
from `mujoco_py` package. As MuJoCo has been made freely available, you can obtain a free
license [here](https://www.roboti.us/license.html).

Next, go to the main directory of this repo and run

`pip install .`

Alternatively, if you want to install in editable mode, run

`pip install -e .`

## Docker image

- To build the image with `continualworld` package installed inside, run
  `docker build . -f assets/Dockerfile -t continualworld`

- To build the image WITHOUT the `continualworld` package but with all the dependencies installed, run
  `docker build . -f assets/Dockerfile -t continualworld --build-arg INSTALL_CW_PACKAGE=false`

When the image is ready, you can run

`docker run -it continualworld bash`

to get inside the image.

## Singularity image

- To build the image with `continualworld` package installed inside, run
  `singularity build continualworld.sif assets/singularity.def`

- To build the image WITHOUT the `continualworld` package but with all the dependencies installed, run
  `singularity build continualworld.sif assets/singularity_only_deps.def`

When the image is ready, you can run

`singularity shell continualworld.sif`

to get inside the image.

# Running

You can run single task, continual learning or multi-task learning experiments with `run_single.py`, `run_cl.py`
, `run_mt.py` scripts, respectively.

To see available script arguments, run with `--help` option, e.g.

`python3 run_single.py --help`

## Examples

Below are given example commands that will run experiments with a very limited scale.

### Single task

`python3 run_single.py --seed 0 --steps 2e3 --log_every 250 --task hammer-v1 --logger_output tsv tensorboard`

### Continual learning

`python3 run_cl.py --seed 0 --steps_per_task 2e3 --log_every 250 --tasks CW20 --cl_method ewc --cl_reg_coef 1e4 --logger_output tsv tensorboard`

### Multi-task learning

`python3 run_mt.py --seed 0 --steps_per_task 2e3 --log_every 250 --tasks CW10 --use_popart True --logger_output tsv tensorboard`

## Reproducing the results from the paper

Commands to run experiments that reproduce main results from the paper can be found
in `examples/paper_cl_experiments.sh`,
`examples/paper_mt_experiments.sh` and `examples/paper_single_experiments.sh`. Because of number of different runs that
these files contain, it is infeasible to just run it in sequential manner. We hope though that these files will be
helpful because they precisely specify what needs to be run.

After the logs from runs are gathered, you can produce tables and plots - see the section below.

# Producing result tables and plots

After you've run experiments and you have saved logs, you can run the script to produce result tables and plots:

`python produce_results.py --cl_logs examples/logs/cl --mtl_logs examples/logs/mtl --baseline_logs examples/logs/baseline`

In this command, respective arguments should be replaced for paths to directories containing logs from continual
learning experiments, multi-task experiments and baseline (single-task) experiments. Each of these should be a directory
inside which there are multiple experiments, for different methods and/or seeds. You can see the directory structure in
the example logs included in the command above.

Results will be produced and saved on default to the `results` directory.

Alternatively, check out `nb_produce_results.ipynb` notebook to see plots and tables in the notebook.

## Download our saved logs and produce results

You can download logs of experiments to reproduce paper's results from
[here](https://e.pcloud.link/publink/show?code=XZXgkuZfJQeQRtMMSpHiSCmWIgV8VhnkkKX). Then unzip the file and run

`python produce_results.py --cl_logs saved_logs/cl --mtl_logs saved_logs/mt --baseline_logs saved_logs/single`

to produce tables and plots.

As a result, a csv file with results will be produced, as well as the plots, like this one (and more!):

![average performance](./examples/results/report_2021_12_15__20_06_56/average_performance.png)

Full output can be found [here](./examples/results/report_2021_12_15__20_06_56/).

# Acknowledgements

Continual World heavily relies on [MetaWorld](https://github.com/rlworkgroup/metaworld).

The implementation of SAC used in our code comes from [Spinning Up in Deep RL](https://github.com/openai/spinningup).

Our research was supported by the [PLGrid](https://plgrid.pl/) infrastructure.

Our experiments were managed using [Neptune](https://neptune.ai).

# CSV转TSV转换工具

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
