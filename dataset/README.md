# Dataset for HabiCrowd
We document the HabiCrowd dataset as follows.

```
|- configs/       (Config for all baselines.)
    |- baseline_<name>.yaml (Specified config on hyperparameters of each algorithm)
    |- challenge_crowdnav.local.rgbd.yaml (Shared config about environment of all algorithms.)
|- crowd-nav
    |- bot_config_v1.0/             (Config for bots)
        |- train                    (Train split)
            |- <scene_name>.json
        |- val                      (Val split)
            |- <scene_name>.json
    |- crowdnav_hm3d/             (Our modified config for HabiCrowd as we only utilize 56 scenes)
        |- train/                   (Train split)
            |- content
                |- <scene_name>.json.gz
            
            |- train.json.gz    
        |- val/                     (Val split)
            |- content
                |- <scene_name>.json.gz
        |- val_mini/                (Demo split)
            |- content
                |- <scene_name>.json.gz

|- entities                 (Folder contains glb files of virtual humans)
    |- textures
    |- human_<i>.glb
    |- human_<i>.mtl
    |- human_<i>.object_config.json
```

For hyperparameters of baselines as well as simulator, see [configs](./configs/).

For virtual human parameters settings of each scene, see [crowd-nav/bot_configs_v1.0](crowd-nav/bot_config_v1.0). Each file specifies the number of virtual humans used, the initial position, rotation, and their desired linear, angular velocities.

For the navigation settings, see [crowd-nav/crowdnav_hm3d](crowd-nav/crowdnav_hm3d). The folder is adapted from [Objectnav_HM3D](https://dl.fbaipublicfiles.com/habitat/data/datasets/objectnav/hm3d/v1/objectnav_hm3d_v1.zip).

For the material of humans, see [entities](entities).

We will release weights of pre-trained model after finishing code instructions.

Note that, this is only the documentation of HabiCrowd. If you want to install as well as run code, we recommend to read the instructions from the [main site](https://github.com/habicrowd/HabiCrowd).
