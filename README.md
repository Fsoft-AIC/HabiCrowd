<p align="center">
  <img width = "50%" src='res/img/habicrowd.png' />
  </p>

--------------------------------------------------------------------------------
# HabiCrowd
## NOTE: Both code and dataset are underdeveloped. Everything will be ready soon.
This repository contains the code for running HabiCrowd.

## Task: ObjectNav

In ObjectNav, an agent is placed at a random starting point and orientation in an unknown environment and instructed to go to an instance of an object category (*'find a chair'*). There is no map of the world provided, thus the agent must rely only on its sensory input to navigate.

The agent has an RGB-D camera as well as a (noiseless) GPS+Compass sensor. The GPS+Compass sensor determines the agent's current location and orientation in relation to the beginning of the episode. In simulation, we try to match the camera specifications (field of view, resolution) to the Azure Kinect camera, although this work does not include any injected sensor noise.

### Dataset
We use 56 scenes in the [Habitat-Matterport3D (HM3D)](https://aihabitat.org/datasets/hm3d/)[2] dataset with train/val/test splits on 36/8/12. Following Chaplot et al.[3], we use 6 object goal categories: chair, couch, potted plant, bed, toilet and tv.

### Starter
We have added a config in `configs/ddppo_objectnav.yaml` that includes a baseline using DD-PPO from Habitat-Lab.

1. Install the [Habitat-Sim](https://github.com/facebookresearch/habitat-sim/) and [Habitat-Lab](https://github.com/facebookresearch/habitat-lab/) packages. You can install Habitat-Sim using our custom Conda package for habitat challenge 2022 with: ```conda install -c aihabitat habitat-sim-challenge-2022```. For Habitat-Lab, we have created the `habitat-challenge-2022` tag in our Github repo, which can be cloned using: ```git clone --branch challenge-2022 https://github.com/facebookresearch/habitat-lab.git``` . Also ensure that habitat-baselines is installed when installing Habitat-Lab by using ```python setup.py develop --all``` . You will find further information for installation in the Github repositories. 

1. Download the HM3D dataset following the instructions [here](https://matterport.com/partners/facebook). After downloading extract the dataset to folder `habitat-challenge/habitat-challenge-data/data/scene_datasets/hm3d/` folder (this folder should contain the `.glb` files from HM3D). Note that the `habitat-lab` folder is the [habitat-lab](https://github.com/facebookresearch/habitat-lab/) repository folder. The data also needs to be in the habitat-challenge-data/ in this repository.

1. **Objectnav**: Download the episodes dataset for HM3D ObjectNav from [link](https://dl.fbaipublicfiles.com/habitat/data/datasets/objectnav/hm3d/v1/objectnav_hm3d_v1.zip) and place it in the folder `habitat-challenge/habitat-challenge-data/data/datasets/objectnav/hm3d`. If placed correctly, you should have the train and val splits at `habitat-challenge/habitat-challenge-data/data/datasets/objectnav/hm3d/v1/train/` and `habitat-challenge/habitat-challenge-data/data/datasets/objectnav/hm3d/v1/val/` respectively. Place HM3D scenes downloaded in step-4 of local-evaluation under the `habitat-challenge/habitat-challenge-data/data/scene_datasets` folder. If you have already downloaded thes files for the habitat-lab repo, you may simply symlink them using `ln -s $PATH_TO_SCENE_DATASETS habitat-challenge-data/data/scene_datasets` (if on OSX or Linux).

1. An example on how to train DD-PPO model can be found in [habitat-lab/habitat_baselines/rl/ddppo](https://github.com/facebookresearch/habitat-lab/tree/main/habitat_baselines/rl/ddppo). See the corresponding README in habitat-lab for how to adjust the various hyperparameters, save locations, visual encoders and other features.

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
            --exp-config ../habitat-challenge/configs/ddppo_objectnav.yaml \
            --run-type train \
            TASK_CONFIG.DATASET.SPLIT 'train'
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
            --exp-config ../habitat-challenge/configs/ddppo_objectnav.yaml \
            --run-type train \
            TASK_CONFIG.DATASET.SPLIT 'train'
        ```

    1. The preceding two scripts are based off ones found in the [habitat_baselines/ddppo](https://github.com/facebookresearch/habitat-lab/tree/main/habitat_baselines/rl/ddppo).

1. The checkpoint specified by ```$PATH_TO_CHECKPOINT ``` can evaluated by SPL and other measurements by running the following command:

    ```bash
    python -u -m habitat_baselines.run \
        --exp-config ../habitat-challenge/configs/ddppo_objectnav.yaml \
        --run-type eval \
        EVAL_CKPT_PATH_DIR $PATH_TO_CHECKPOINT \
        TASK_CONFIG.DATASET.SPLIT val
    ```
    The weights used for our DD-PPO Objectnav baseline for the Habitat-2022 challenge can be downloaded with the following command:
    ```bash
    wget https://dl.fbaipublicfiles.com/habitat/data/baselines/v1/ddppo_objectnav_habitat2022_challenge_baseline_v1.pth
    ```

## Acknowledgments

We thank the team behind [Habitat-Matterport3D](https://aihabitat.org/datasets/hm3d/) datasets and [Habitat-Challenge-2022](https://aihabitat.org/challenge/2022/)

## References

[1] [Habitat: A Platform for Embodied AI Research](https://arxiv.org/abs/1904.01201). Manolis Savva\*, Abhishek Kadian\*, Oleksandr Maksymets\*, Yili Zhao, Erik Wijmans, Bhavana Jain, Julian Straub, Jia Liu, Vladlen Koltun, Jitendra Malik, Devi Parikh, Dhruv Batra. IEEE/CVF International Conference on Computer Vision (ICCV), 2019.

[2] [Habitat-Matterport 3D Dataset (HM3D): 1000 Large-scale 3D Environments for Embodied AI](https://arxiv.org/abs/2109.08238) Santhosh K. Ramakrishnan, Aaron Gokaslan, Erik Wijmans, Oleksandr Maksymets, Alex Clegg, John Turner, Eric Undersander, Wojciech Galuba, Andrew Westbury, Angel X. Chang, Manolis Savva, Yili Zhao, Dhruv Batra. arXiv:2109.08238, 2021.

[3] [Object Goal Navigation using Goal-Oriented Semantic Exploration](https://arxiv.org/abs/2007.00643) Devendra Singh Chaplot, Dhiraj Gandhi, Abhinav Gupta, Ruslan Salakhutdinov. NeurIPS, 2020.

[4] [On evaluation of embodied navigation agents](https://arxiv.org/abs/1807.06757). Peter Anderson, Angel Chang, Devendra Singh Chaplot, Alexey Dosovitskiy, Saurabh Gupta, Vladlen Koltun, Jana Kosecka, Jitendra Malik, Roozbeh Mottaghi, Manolis Savva, Amir R. Zamir. arXiv:1807.06757, 2018.
