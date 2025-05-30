# Scenario Mining

## Table of Contents

<!-- toc -->

## Overview

Autonomous Vehicles (AVs) collect and pseudo-label terabytes of multi-modal data localized to HD maps during normal fleet tests. However, identifying interesting and safety critical scenarios from uncurated data streams is prohibitively time-consuming and error-prone. Retrieving and processing specific scenarios for ego-behavior evaluation, safety testing, or active learning at scale remains a major challenge. While prior works have explored this problem in the context of structured queries and hand-crafted heuristics, we are hosting this challenge to solicit better end-to-end solutions to this important problem.

Our benchmark includes 10,000 planning-centric natural language queries. Challenge participants can use all RGB frames, Lidar sweeps, HD Maps, and track annotations from the AV2 sensor dataset to find relevant actors in each log. Methods will be evaluated at three levels of spatial and temporal granularity. First, methods must determine if a scenario (defined by a natural language query) occurs in the log. A scenario is a set of objects, actions, map elements, and/or interactions that occur over a specified timeframe. If the scenario occurs in the log, methods must temporally localize (e.g. find the start and end time) the scenario. Lastly, methods must detect and track all objects relevant to the text description. Our primary evaluation metric is HOTA-Temporal, a spatial tracking metric that only considers the scenario objects during the timeframe when the scenario is occuring.

## Downloading Scenario Mining Annotations
We recommend using the open-source [s5cmd](https://github.com/peak/s5cmd) tool to transfer the data to your local filesystem.
```
conda install s5cmd -c conda-forge
export TARGET_DIR="$HOME/data/datasets/av2_scenario_mining"
s5cmd --no-sign-request cp "s3://argoverse/tasks/scenario_mining/*" $TARGET_DIR
```

## Baselines

- Referential Classification: [https://github.com/CainanD/RefAV](https://github.com/CainanD/RefAV) 
- Tracking: [https://github.com/neeharperi/LT3D](https://github.com/neeharperi/LT3D/tree/main)

## Scenario Mining Categories

| **Category** | **Description** |
|:-----------------|:----------------|
| `REFERRED_OBJECT` | The primary object or objects referenced by the prompt. |
| `RELATED_OBJECT` | All objects related to the prompt, but not the referred object. |
| `OTHER_OBJECT` | All other objects that are not referred or relevant to the prompt. |


### Submission Format 


The submission site and leaderboard are hosted at [EvalAI](https://eval.ai/web/challenges/challenge-page/2469/overview). You are required to submit a .pkl file. The evaluation expects a dictionary of lists of dictionaries

```python
{
      (<log_id>, <prompt>): [
            {
                  "timestamp_ns": <timestamp_ns>,
                  "track_id": <track_id>
                  "score": <score>,
                  "label": <label>,
                  "name": <name>,
                  "translation_m": <translation_m>,
                  "size": <size>,
                  "yaw": <yaw>,
            }
      ]
}
```

- `log_id`: Log id associated with the track, also called `seq_id`.
- `prompt` : natural language text prompt.
- `timestamp_ns`: Timestamp associated with the detections.
- `track_id`: Unique id assigned to each track, this is produced by your tracker.
- `score`: Track confidence.
- `label`: Integer index of the object class.
- `name`: Object class name.
- `translation_m`: xyz-components of the object translation in the city reference frame, in meters.
- `size`: Object extent along the x,y,z axes in meters.
- `yaw`: Object heading rotation along the z axis.

An example looks like this:

~~~admonish example

```python

# (1). Example tracks.
example_tracks = {
  ('02678d04-cc9f-3148-9f95-1ba66347dff9', 'a car turning left'): [
    {
       'timestamp_ns': 315969904359876000,
       'translation_m': array([[6759.51786422, 1596.42662849,   57.90987307],
             [6757.01580393, 1601.80434654,   58.06088218],
             [6761.8232099 , 1591.6432147 ,   57.66341136],
             ...,
             [6735.5776378 , 1626.72694938,   59.12224152],
             [6790.59603472, 1558.0159741 ,   55.68706682],
             [6774.78130127, 1547.73853494,   56.55294184]]),
       'size': array([[4.315736  , 1.7214599 , 1.4757565 ],
             [4.3870926 , 1.7566483 , 1.4416479 ],
             [4.4788623 , 1.7604711 , 1.4735452 ],
             ...,
             [1.6218852 , 0.82648355, 1.6104599 ],
             [1.4323177 , 0.79862624, 1.5229694 ],
             [0.7979312 , 0.6317313 , 1.4602867 ]], dtype=float32),
      'yaw': array([-1.1205611 , ... , -1.1305285 , -1.1272993], dtype=float32),
      'label': array([ 0, 0, ... 1,  0], dtype=int32),
      'name': array(['REFERRED_OBJECT', ..., 'RELATED_OBJECT', 'REFERRED_OBJECT'], dtype='<U31'),
      'score': array([0.54183, ..., 0.47720736, 0.4853499], dtype=float32),
      'track_id': array([0, ... , 11, 12], dtype=int32),
    },
    ...
  ],
  ...
}

# (2). Prepare for submission.
import pickle

with open("track_predictions.pkl", "wb") as f:
       pickle.dump(example_tracks, f)
```
~~~

If the file is over 400MB, it must be uploaded to the EvalAI server via command line. This can be done with

```bash
pip install evalai
evalai set_token <EvalAI_account_token>
evalai challenge 2469 phase 4899 submit --file output/evaluation/val/track_predictions.pkl --large
```

### Evaluation Metrics

| **Metric** | **Description** |
|:-----------|:----------------|
| $\text{HOTA-Temporal}$ | HOTA on temporally localized tracks. |
| $\text{HOTA}$ | HOTA on the full length of a track |
| $\text{Timestamp F1}$ | Timestamp level classificaiton metric |
| $\text{Scenario F1}$ | Scenario level classification metric. |

HOTA explicitly balances the effect of performing accurate detection, association, and localization into a single unified metric. It is shown to better align with human visual evaluation of tracking performance. For more information, please check out HOTA: A Higher Order Metric for Evaluating Multi-Object Tracking. Jonathon Luiten, Aljosa Osep, Patrick Dendorfer, Philip Torr, Andreas Geiger, Laura Leal-Taixe, Bastian Leibe. IJCV 2020

We can run tracking evaluation using the following code snippet. 
```bash 
from av2.evaluation.scenario_mining.eval import evaluate
res =  evaluate(track_predictions, labels, objective_metric, ego_distance_threshold_m, dataset_dir, outputs_dir)

- `track_predictions`: Track predictions
- `labels`: Ground truth annotations
- `objective_metric`: Metric to optimize per-class recall (e.g. HOTA, MOTA, default is HOTA)
- `ego_distance_threshold_m`: Filter for all detections outside of `ego_distance_threshold_m` (default is 50 meters).
- `dataset_dir`: Path to dataset directory (e.g. data/Sensor/val)
- `outputs_dir`: Path to output directory
```

## Supporting Publications

If you participate in this challenge, please consider citing:

```BibTeX 
@article{davidson2025refav,
  title={RefAV: Towards Planning-Centric Scenario Mining},
  author={Davidson, Cainan and Ramanan, Deva and Peri, Neehar},
  journal={arXiv preprint arXiv:2505.20981},
  year={2025}
}
```

 
