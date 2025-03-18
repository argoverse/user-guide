# Scenario Mining

## Table of Contents

<!-- toc -->

## Overview

Autonomous Vehicles (AVs) collect and pseudo-label terabytes of multi-modal data localized to HD maps during normal fleet tests. However, identifying interesting and safety critical scenarios from uncurated data streams is prohibitively time-consuming and error-prone. Although prior works have explored this problem in the context of structured queries and hand-crafted heuristics, we are hosting this challenge to solicit better end-to-end solutions to this important problem.

To this end, our benchmark includes 10,000 safety-critical natural language queries. Challenge participants can use all RGB frames, Lidar sweeps, HD Maps, and track annotations from the AV2 sensor dataset to find relevant actors in each log. Methods will be evaluated at three levels of spatial and temporal granularity. First, methods must determine if an action (defined by a natural language query) occurs in the log. Next, if the action occurs in the log, methods must temporally localize (e.g. find the start and end time) of the action. Lastly,  methods must detect and track all objects relevant to the text description. Our primary evaluation metric is HOTA-temporal, a tracking metric that jointly considers detection and association accuracy for referred objects.


## Baselines

- Referential Classification: [https://github.com/CainanD/refbot](https://github.com/CainanD/refbot)
- Tracking: [https://github.com/neeharperi/LT3D](https://github.com/neeharperi/LT3D/tree/main)

## Scenario Mining Taxonomy

| **Category** | **Description** |
|:-----------------|:----------------|
| `REFERRED_OBJECT` | The primary object or objects referenced by the prompt. |
| `RELATED_OBJECT` | All objects related to the prompt, but not the referred object. |
| `OTHER_OBJECT` | All other objects that are not referred or relevant to the prompt. |


### Submission Format 

The evaluation expects a dictionary of lists of dictionaries

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
                  "velocity_m_per_s": <velocity_m_per_s>,
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
- `velocity_m_per_s`: Object veloicty along the x,y,z axes.

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
      'velocity_m_per_s': array([[ 2.82435445e-03, -8.80148250e-04, -1.52388044e-04],
             [ 1.73744695e-01, -3.48345393e-01, -1.52417628e-02],
             [ 7.38469649e-02, -1.16846527e-01, -5.85577238e-03],
             ...,
             [-1.38887463e+00,  3.96778419e+00,  1.45435923e-01],
             [ 2.23189720e+00, -5.40360805e+00, -2.14317040e-01],
             [ 9.81130002e-02, -2.00860636e-01, -8.68975817e-03]]),
      'label': array([ 0, 0, ... 9,  0], dtype=int32),
      'name': array(['REGULAR_VEHICLE', ..., 'STOP_SIGN', 'REGULAR_VEHICLE'], dtype='<U31'),
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
```
- `track_predictions`: Track predictions
- `labels`: Ground truth annotations
- `objective_metric`: Metric to optimize per-class recall (e.g. HOTA, MOTA, default is HOTA)
- `ego_distance_threshold_m`: Filter for all detections outside of `ego_distance_threshold_m` (default is 50 meters).
- `dataset_dir`: Path to dataset directory (e.g. data/Sensor/val)
- `outputs_dir`: Path to output directory

