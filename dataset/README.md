# Dataset for HabiCrowd
We document the HabiCrowd dataset as follows.

```
|- notebooks/       (Jupyter Notebooks for model evaluation, parameter selection and data analysis.)
|- utils/           (common utilities.)
|- TTS
    |- bin/             (folder for all the executables.)
      |- train*.py                  (train your target model.)
      |- distribute.py              (train your TTS model using Multiple GPUs.)
      |- compute_statistics.py      (compute dataset statistics for normalization.)
      |- convert*.py                (convert target torch model to TF.)
    |- tts/             (text to speech models)
        |- layers/          (model layer definitions)
        |- models/          (model definitions)
        |- tf/              (Tensorflow 2 utilities and model implementations)
        |- utils/           (model specific utilities.)
    |- speaker_encoder/ (Speaker Encoder models.)
        |- (same)
    |- vocoder/         (Vocoder models.)
        |- (same)
```
