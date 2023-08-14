## Update
- As of 7th August 2023, we have included 424 more scenes to HabiCrowd, increasing the number of configured scenes to 480.
- As of 10th August 2023, we introduced ImageNav to HabiCrowd.

<p align="center">
  <img width = "50%" src='res/img/habicrowd.png' />
  </p>
<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License</a>. For more information, see <a rel="license" href="https://matterport.com/matterport-end-user-license-agreement-academic-use-model-data">HM3D license</a> and <a rel="license" href="https://aihabitat.org/terms-of-use/">Habitat Terms of Use</a>.

--------------------------------------------------------------------------------
## Table of contents
   1. [Overview](#overview)
   1. [ObjectNav](#objectnav)
   1. [PointNav](#pointnav)
   1. [ImageNav](#imagenav)

# HabiCrowd
This repository contains the code for running HabiCrowd.

## Overview
   
## ObjectNav

In ObjectNav, an agent is placed at a random starting point and orientation in an unknown environment and instructed to go to an instance of an object category (*'find a chair'*). There is no map of the world provided, thus the agent must rely only on its sensory input to navigate.

The agent has an RGB-D camera as well as a (noiseless) GPS+Compass sensor. The GPS+Compass sensor determines the agent's current location and orientation in relation to the beginning of the episode. In simulation, we try to match the camera specifications (field of view, resolution) to the Azure Kinect camera, although this work does not include any injected sensor noise.

### Dataset
We use 56 scenes in the [Habitat-Matterport3D (HM3D)](https://aihabitat.org/datasets/hm3d/) dataset with train/val/test splits on 36/8/12. We use 6 object goal categories: chair, couch, potted plant, bed, toilet and tv as traditional ObjectNav in Habitat simulator.

### Starter
To begin with, install the [Habitat-Sim](https://github.com/facebookresearch/habitat-sim/). Install [our forked version Habitat-Lab](https://github.com/habicrowd/habitat-lab), where we have developed our baselines as well as human dynamics. You can install Habitat-Sim using the custom Conda package for habitat challenge 2022 with:
```
conda install -c aihabitat habitat-sim-challenge-2022
```
Also ensure that habitat-baselines is installed when installing Habitat-Lab by using:
```
python setup.py develop --all
```
You will find further information for installation in the the forked Github repositories. 
2. Download the HM3D dataset following the instructions [here](https://matterport.com/partners/facebook). After downloading extract the dataset to folder `habitat-challenge/habitat-challenge-data/data/scene_datasets/hm3d/` folder (this folder should contain the `.glb` files from HM3D). Note that the `habitat-lab` folder is the [habitat-lab](https://github.com/habicrowd/habitat-lab) repository folder. The data also needs to be in the HabiCrowd/ in this repository. Move the downloaded folder to [dataset folder](dataset/).

1. An example on how to train a simple DD-PPO model (for other model, you should use appropriate baseline name) can be found in [habitat-lab/habitat_baselines/rl/ddppo](https://github.com/habicrowd/habitat-lab/tree/main/habitat_baselines/rl/ddppo). See the corresponding README in habitat-lab for how to adjust the various hyperparameters, save locations, visual encoders and other features. 
    1. First, navigate to our forked Habitat-Lab version. We expect the structure folder as follows:
        ```
        |- HabiCrowd
        |- habitat-lab
        ```

    1. To run on a single machine use the following script from `habitat-lab` directory:
        ```bash
        #/bin/bash

        export GLOG_minloglevel=2
        export MAGNUM_LOG=quiet

        set -x
        python -u -m torch.distributed.launch \
            --use_env \
            --nproc_per_node 1 \
            habitat_baselines/run.py \
            --exp-config ../HabiCrowd/dataset/configs/baseline_<name>.yaml \
            --run-type train \
            BASE_TASK_CONFIG_PATH ../HabiCrowd/dataset/configs/challenge_crowdnav.local.rgbd.yaml \
            TASK_CONFIG.DATASET.SCENES_DIR ../HabiCrowd/dataset/crowd-nav-data/data/scene_datasets/ \
            TASK_CONFIG.DATASET.SPLIT 'train' \
            TENSORBOARD_DIR ./tb \
            CHECKPOINT_FOLDER ./checkpoints \
            LOG_FILE ./train.log
        ```
    1. There is also an example of running the code distributed on a cluster with SLURM. While this is not necessary, if you have access to a cluster, it can significantly speed up training. To run on multiple machines in a SLURM cluster run the following script: change ```#SBATCH --nodes $NUM_OF_MACHINES``` to the number of machines and ```#SBATCH --ntasks-per-node $NUM_OF_GPUS``` and ```$SBATCH --gres $NUM_OF_GPUS``` to specify the number of GPUS to use per requested machine.
        ```bash
        #!/bin/bash
        #SBATCH --job-name=ddppo
        #SBATCH --output=logs.ddppo.out
        #SBATCH --error=logs.ddppo.err
        #SBATCH --gres gpu:1
        #SBATCH --nodes 1
        #SBATCH --cpus-per-task 10
        #SBATCH --ntasks-per-node 1
        #SBATCH --mem=60GB
        #SBATCH --time=12:00
        #SBATCH --signal=USR1@600
        #SBATCH --partition=dev

        export GLOG_minloglevel=2
        export MAGNUM_LOG=quiet

        export MASTER_ADDR=$(srun --ntasks=1 hostname 2>&1 | tail -n1)

        set -x
        srun python -u -m habitat_baselines.run \
            --exp-config ../HabiCrowd/dataset/configs/baseline_<name>.yaml \
            --run-type train \
            BASE_TASK_CONFIG_PATH ../HabiCrowd/dataset/configs/challenge_crowdnav.local.rgbd.yaml \
            TASK_CONFIG.DATASET.SCENES_DIR ../HabiCrowd/dataset/crowd-nav-data/data/scene_datasets/ \
            TASK_CONFIG.DATASET.SPLIT 'train' \
            TENSORBOARD_DIR ./tb \
            CHECKPOINT_FOLDER ./checkpoints \
            LOG_FILE ./train.log
        ```

    1. The preceding two scripts are based off ones found in the [habitat_baselines/ddppo](https://github.com/facebookresearch/habitat-lab/tree/main/habitat_baselines/rl/ddppo).

1. The checkpoint specified by ```$PATH_TO_CHECKPOINT ``` can evaluated by SPL and other measurements by running the following command:

    ```bash
    python -u -m habitat_baselines.run \
        --exp-config ../habitat-challenge/configs/ddppo_objectnav.yaml \
        --run-type eval \
        BASE_TASK_CONFIG_PATH ../HabiCrowd/dataset/configs/challenge_crowdnav.local.rgbd.yaml \
        TASK_CONFIG.DATASET.DATA_PATH ../HabiCrowd/dataset/crowd-nav/crowdnav_hm3d_v2.1/{split}/{split}.json.gz 
        TASK_CONFIG.DATASET.SCENES_DIR ../HabiCrowd/dataset/crowd-nav-data/data/scene_datasets/ \
        EVAL_CKPT_PATH_DIR $PATH_TO_CHECKPOINT \
        TASK_CONFIG.DATASET.SPLIT val
    ```
## PointNav
Follow the instructions from [Habitat-Lab](https://github.com/facebookresearch/habitat-lab/tree/main/habitat-baselines). First, you need to acquire HM3D PointNav dataset in the [link](https://dl.fbaipublicfiles.com/habitat/data/datasets/pointnav/hm3d/v1/pointnav_hm3d_v1.zip).

We still use [the forked version Habitat-Lab](https://github.com/habicrowd/habitat-lab). To run on a single machine use the following script from `habitat-lab` directory:
  ```
      python -u -m habitat_baselines.run \
      --config-name=pointnav/baseline_<name>.yaml
  ```
To test on a single machine use the following script from `habitat-lab` directory:
  ```
      python -u -m habitat_baselines.run \
      --config-name=pointnav/baseline_<name>.yaml \
      habitat_baselines.evaluate=True
  ```

## ImageNav
Follow the instructions from [Habitat-Lab](https://github.com/facebookresearch/habitat-lab/tree/main/habitat-baselines). First, you need to acquire HM3D_v0.2 Instance image goal navigation dataset in the [link](https://dl.fbaipublicfiles.com/habitat/data/datasets/imagenav/hm3d/v2/instance_imagenav_hm3d_v2.zip). Note that, you need to download [HM3D_v0.2](https://github.com/facebookresearch/habitat-sim/blob/main/DATASETS.md#habitat-matterport-3d-research-dataset-hm3d) for ImageNav benchmark.

Similar to the above task, we just need the change the config to instance_imagenav:
  ```
      python -u -m habitat_baselines.run \
      --config-name=instance_imagenav/baseline_<name>.yaml
  ```
To test on a single machine use the following script from `habitat-lab` directory:
  ```
      python -u -m habitat_baselines.run \
      --config-name=instance_imagenav/baseline_<name>.yaml \
      habitat_baselines.evaluate=True
  ```


## Acknowledgments

We thank the teams behind [Habitat-Matterport3D](https://aihabitat.org/datasets/hm3d/) datasets, [Habitat-Challenge-2022](https://aihabitat.org/challenge/2022/), and [Habitat-Lab](https://github.com/facebookresearch/habitat-lab).
