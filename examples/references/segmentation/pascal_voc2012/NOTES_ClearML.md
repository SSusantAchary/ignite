# Experiments tracking with ClearML

[Allegro ClearML](https://allegro.ai/clearml/docs/) is a full system open source ML / DL experiment manager and ML-Ops solution.
It is composed of a server, Python SDK and web UI. **Allegro ClearML** enables data scientists and data engineers
to effortlessly track, manage, compare and collaborate on their experiments as well as easily manage their
training workloads on remote machines.

## Install ClearML

Install [clearml](https://github.com/allegroai/clearml) by executing the following command:

```bash
pip install --upgrade clearml
```

## Install requirements

```bash
pip install -r requirements.txt
```

We need to also install Nvidia/APEX and libraries for opencv.
**Important**, please, check the content of `experiments/setup_opencv.sh` before running the script.

```bash
sh experiments/setup_apex.sh

sh experiments/setup_opencv.sh
```

## Download Pascal VOC2012 and SDB datasets

Download and extract the datasets:

```bash
python code/scripts/download_dataset.py /path/to/datasets
```

This script will download and extract the following datasets into `/path/to/datasets`

- The [Pascal VOC2012](http://host.robots.ox.ac.uk/pascal/VOC/voc2012/VOCtrainval_11-May-2012.tar) dataset
- Optionally, the [SBD](http://www.eecs.berkeley.edu/Research/Projects/CS/vision/grouping/semantic_contours/benchmark.tgz) evaluation dataset

## Setup the environment variables

### Setup the dataset path

Export the `DATASET_PATH` environment variable for the Pascal VOC2012 dataset.

```bash
export DATASET_PATH=/path/to/pascal_voc2012
# e.g. export DATASET_PATH=$PWD/input/ where VOCdevkit is located
```

### Setup the SBD dataset path

Export the `SBD_DATASET_PATH` environment variable for the SBD evaluation dataset.

```bash
export SBD_DATASET_PATH=/path/to/SBD/
# e.g. export SBD_DATASET_PATH=/data/SBD/  where "cls  img  inst  train.txt  train_noval.txt  val.txt" are located
```

## Run the experiment code

In **ClearML**, when you run the experiment code, `clearml` stores the experiment in [clearml-server](https://github.com/allegroai/clearml-server).

By default, `clearml` works with the demo **ClearML Server** ([https://demoapp.trains.allegro.ai/dashboard](https://demoapp.trains.allegro.ai/dashboard)),
which is open to anyone (although once a week it is refreshing and deleting all data). You can also set up your own [self-hosted](https://github.com/allegroai/clearml-server) **ClearML Server**.

After the experiment code runs once, you can [reproduce the experiment](#reproducing-the-experiment) using the
**ClearML Web-App (UI)**, which is part of `clearml-server`. You only need to run the code once to store it
in `clearml-server`.

### Setup

This setup is a specific for this code and is not required in general usage of ClearML.
We setup an output path as a local storage:

```bash
export CLEARML_OUTPUT_PATH=/path/to/output/clearml
# e.g export CLEARML_OUTPUT_PATH=$PWD/output/clearml
```

This environment variable helps to choose ClearML as experiment tracking system among all others.

### ClearML fileserver setup

The configuration to upload artifact must be done by modifying the `clearml` configuration file `~/clearml.conf`
generated by `clearml-init`. According to the
[documentation](https://allegro.ai/docs/examples/reporting/artifacts/), the `output_uri` argument can be
configured in `sdk.development.default_output_uri` to fileserver uri. If server is self-hosted, `ClearML` fileserver uri is
`http://localhost:8081`.

For more details, see https://allegro.ai/docs/examples/reporting/artifacts/

### Run the code

#### Training on single node and single GPU

Please, make sure to adapt training data loader batch size to your GPU type. For example, a single GPU with 11GB can have a batch size of 8-9.

Execute the following command:

```bash
export CLEARML_OUTPUT_PATH=/path/to/output/clearml
# e.g export CLEARML_OUTPUT_PATH=$PWD/output/clearml

python -m py_config_runner ./code/scripts/training.py ./configs/train/baseline_resnet101.py
```

#### Training on single node and multiple GPUs

For optimal devices usage, please, make sure to adapt training data loader batch size to your infrasture.
For example, a single GPU with 11GB can have a batch size of 8-9, thus, on N devices, we can set it as `N * 9`.

```bash
export CLEARML_OUTPUT_PATH=/path/to/output/clearml
# e.g export CLEARML_OUTPUT_PATH=$PWD/output/clearml

python -m torch.distributed.launch --nproc 2 --use_env -m py_config_runner ./code/scripts/training.py ./configs/train/baseline_resnet101.py
```

In **ClearML Web-App** a new project named _"Pascal-VOC12 Training"_ will be created,
with an experiment named _"baseline_resnet101"_ inside.

In your local environment, the console output includes the URL of the experiment's **RESULTS** page.

You can now view your experiment in **ClearML** by clicking the link or copying the URL into your browser.
It opens the results in the experiment's details pane, in the **ClearML Web-App (UI)**.

#### ClearML automatic Logging

When the experiment code runs, **ClearML** automatically logs your environment, code, and the outputs.
Which means that you don't need to change your code.

All you need is 2 lines of integration at the top of your main script

```python
from clearml import Task
Task.init("Pascal-VOC12 Training", "baseline_resnet101")
```

Once it's there, the following will be automatically logged by **ClearML**:

- **Resource Monitoring** CPU/GPU utilization, temperature, IO, network, etc
- **Development Environment** Python environment, Git (repo, branch, commit) including uncommitted changes
- **Configuration** Including configuration files, command line arguments (ArgParser), and general dictionaries
- Full **stdout** and **stderr** automatic logging
- Model snapshots, with optional automatic upload to central storage.
- Artifacts log & store, including shared folders, S3, GS, Azure, and Http/s
- Matplotlib / Seaborn / TensorBoard / TensorBoardX scalars, metrics, histograms, images, audio, video, etc

Additionally, **ClearML** supports explicit logging by adding calls to the **ClearML** Python client `Logger`
class methods in the code. For more information,
see [Explicit Reporting](https://allegro.ai/docs/examples/examples_explicit_reporting/) in the **ClearML** documentation.

## Track the experiment and visualize the results

In the **ClearML Web-App (UI)**, track the experiment and visualize results in the experiment's details pane,
which is organized in tabs and provides the following information;

- Source code, uncommitted changes, Python packages and versions, and other information, in the **EXECUTION** tab
- Hyperparameters in the **HYPERPARAMETERS** tab
- Input model, Configuration, Output model, and other artifacts in the **ARTIFACTS** tab
- Experiment Comments and General experiment information in the **INFO** tab
- Results in the **RESULTS** tab, including the log, scalar metric plots, plots of any data, and debug samples

## Reproducing the experiments

In **ClearML**, reproduce experiments using `clearml-agent` for remote execution. Rerun the same experiment,
by making an exact copy of it (a clone), and remotely execute the cloned experiment.

First, install `clearml-agent` and then configure it to work with your self-hosted **ClearML Server**.

Once `clearml-agent` is installed and configured, run `clearml-agent daemon`.
In **ClearML**, we call these _workers_, they pop experiments from a job execution queue and execute them.
Every machine with a _clearml-agent daemon_, becomes a registered _worker_ in your **clearml-server** cluster.

Using the **ClearML Web-App** you can easily send experiments to be remotely executed on one of these machines.

More details can be found on the _clearml-agent_ [github](https://github.com/allegroai/clearml-agent/)

### Install and configure clearml-agent

1.  Install `clearml-agent`

        pip install clearml-agent

1.  Configure `clearml-agent` by running the setup wizard

        clearml-agent init

### Remotely execute the experiment

1.  Start a **ClearML** worker. Run a `clearml-agent daemon` listening to a queue

    For example, run a `clearml-agent daemon` listening to the `default` queue and using multiple GPUs.

        clearml-agent daemon --gpus 0,1 --queue default

1.  Locate the experiment. In the **ClearML Web-App (UI)**, Projects page, click on the project card

1.  Make a copy of the experiment

    1. In the experiment table, right-click the experiment
    1. On the sub-menu, select **Clone**
    1. Select the project, type a name for the copy, and type a description, or accept the defaults
    1. Click the **CLONE** button

    The copy of the experiment is created. Its details pane opens.

1.  Send the experiment for remote execution, by enqueuing it in one of the job execution queues

    1. In the experiment table, right-click the experiment
    1. On the sub-menu, select **Enqueue**
    1. Select the _default_ queue
    1. Click the **ENQUEUE** button

    The experiment's status changes to Pending.

When the experiment reaches the top of the job execution queue, the `clearml-agent deamon` fetches it,
its status changes to Running, and `clearml-agent` executes it while logging and monitoring.
You can track the experiment while it is in progress, and anytime afterwards.
