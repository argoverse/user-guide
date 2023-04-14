# End-to-End (E2E) Forecasting

## Table of Contents

<!-- toc -->

## Overview

Object detection and forecasting are fundamental components of embodied perception. These problems, however, are largely studied in isolation. We propose a joint detection, tracking, and multi-agent forecasting benchmark from sensor data. Although prior works have studied end-to-end perception, no large scale dataset or challenge exists to facilitate standardized evaluation for this problem. In addition, self-driving benchmarks have historically focused on evaluating a few common classes such as cars, pedestrians and bicycles, and neglect many rare classes in-the-tail. However, in the real open world, self-driving vehicles must still detect rare classes to ensure safe operation.

To this end, our proposed benchmark will be the first to evaluate end-to-end perception on 26 classes defined by the AV2 ontology. Specifically, we will repurpose the AV2 sensor dataset, which has track annotations for 26 object categories, for end-to-end perception: for each timestep in a given sensor sequence, algorithms will have access to all prior frames and must produce tracks for all past sensor sweeps, detections for the current timestep, and forecasted trajectories for the next 3 s. This challenge is different from the Motion Forecasting challenge because we do not provide ground truth tracks as input, requiring algorithms to process raw sensor data. Our primary evaluation metric is Forecasting Average Precision, a joint detection and forecasting metric that computes performance averaged over static, linear, and nonlinearly moving cohorts. Unlike standard motion forecasting evaluation, end-to-end perception must consider both true positive and false positive predictions.

## Baselines

- [https://github.com/neeharperi/LT3D](https://github.com/neeharperi/LT3D)

## End-to-End Forecasting Taxonomy

| **Category** | **Description** |
|:-----------------|:----------------|
| `REGULAR_VEHICLE` | Any conventionally sized passenger vehicle used for the transportation of people and cargo. This includes Cars, vans, pickup trucks, SUVs, etc. |
| `PEDESTRIAN` | Person that is not driving or riding in/on a vehicle. They can be walking, standing, sitting, prone, etc. |
| `BOLLARD` | Bollards are short, sturdy posts installed in the roadway or sidewalk to control the flow of traffic. These may be temporary or permanent and are sometimes decorative. |
| `CONSTRUCTION_CONE` | Movable traffic cone that is used to alert drivers to a hazard.  These will typically be orange and white striped and may or may not have a blinking light attached to the top. |
| `CONSTRUCTION_BARREL` | Movable traffic barrel that is used to alert drivers to a hazard.  These will typically be orange and white striped and may or may not have a blinking light attached to the top. |
| `STOP_SIGN` | Red octagonal traffic sign displaying the word STOP used to notify drivers that they must come to a complete stop and make sure no other road users are coming before proceeding. |
| `BICYCLE` | Non-motorized vehicle that typically has two wheels and is propelled by human power pushing pedals in a circular motion. |
| `LARGE_VEHICLE` | Large motorized vehicles (four wheels or more) which do not fit into any more specific subclass. Examples include extended passenger vans, fire trucks, RVs, etc. |
| `WHEELED_DEVICE` | Objects involved in the transportation of a person and do not fit a more specific class. Examples range from skateboards, non-motorized scooters, segways, to golf-carts. |
| `BUS` | Standard city buses designed to carry a large number of people. |
| `BOX_TRUCK` | Chassis cab truck with an enclosed cube shaped cargo area. It should be noted that the cargo area is rigidly attached to the cab, and they do not articulate. |
| `SIGN` | Official road signs placed by the Department of Transportation (DOT signs) which are of interest to us. This includes yield signs, speed limit signs, directional control signs, construction signs, and other signs that provide required traffic control information. Note that Stop Sign is captured separately and informative signs such as street signs, parking signs, bus stop signs, etc. are not included in this class. |
| `TRUCK` | Vehicles that are clearly defined as a truck but does not fit into the subclasses of Box Truck or Truck Cab. Examples include common delivery vehicles (UPS, FedEx), mail trucks, garbage trucks, utility trucks, ambulances, dump trucks, etc. |
| `MOTORCYCLE` | Motorized vehicle with two wheels where the rider straddles the engine.  These are capable of high speeds similar to a car. |
| `BICYCLIST` | Person actively riding a bicycle, non-pedaling passengers included. |
| `VEHICULAR_TRAILER` | Non-motorized, wheeled vehicle towed behind a motorized vehicle.
| `TRUCK_CAB` | Heavy truck commonly known as “Semi cab”, “Tractor”, or “Lorry”. This refers to only the front of part of an articulated tractor trailer. |
| `MOTORCYCLIST` | Person actively riding a motorcycle or a moped, including passengers. |
| `DOG` | Any member of the canine family. |
| `SCHOOL_BUS` | Bus that primarily holds school children (typically yellow) and can control the flow of traffic via the use of an articulating stop sign and loading/unloading flasher lights. |
| `WHEELED_RIDER` | Person actively riding or being carried by a wheeled device. |
| `STROLLER` | Push-cart with wheels meant to hold a baby or toddler. |
| `ARTICULATED_BUS` | Articulated buses perform the same function as a standard city bus, but are able to bend (articulate) towards the center. These will also have a third set of wheels not present on a typical bus. |
| `MESSAGE_BOARD_TRAILER` | Trailer carrying a large, mounted, electronic sign to display messages. Often found around construction sites or large events. |
| `MOBILE_PEDESTRIAN_SIGN` | Movable sign designating an area where pedestrians may cross the road. |
| `WHEELCHAIR` | Chair fitted with wheels for use as a means of transport by a person who is unable to walk as a result of illness, injury, or disability. This includes both motorized and non-motorized wheelchairs as well as low-speed seated scooters not intended for use on the roadway. |

## Tracking

### Submission Format 

The evaluation expects a dictionary of lists of dictionaries

```python
{
      <log_id>: [
            {
                  "timestamp_ns": <timestamp_ns>,
                  "track_id": <track_id>
                  "score": <score>,
                  "label": <label>,
                  "name": <name>,
                  "translation_m": <translation_m>,
                  "size": <size>,
                  "yaw": <yaw>,
                  "velocity": <velocity>,
            }
      ]
}
```

- `log_id`: Log id associated with the track, also called `seq_id`.
- `timestamp_ns`: Timestamp associated with the detections.
- `track_id`: Unique id assigned to each track, this is produced by your tracker.
- `score`: Track confidence.
- `label`: Integer index of the object class.
- `name`: Object class name.
- `translation_m`: xyz-components of the object translation in the city reference frame, in meters.
- `size`: Object extent along the x,y,z axes in meters.
- `yaw`: Object heading rotation along the z axis.
- `velocity`: Object veloicty along the x,y,z axes.

An example looks like this:

~~~admonish example

```python

# (1). Example tracks.
example_tracks = {
  '02678d04-cc9f-3148-9f95-1ba66347dff9': [
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
      'velocity': array([[ 2.82435445e-03, -8.80148250e-04, -1.52388044e-04],
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
| $\text{HOTA}$ | Explicitly balances the effect of performing accurate detection, association, and localization into a single unified metric. It is shown to better align with human visual evaluation of tracking performance [^1]. |
| $\text{AMOTA}$ | Similar to $\text{MOTA}$, but averaged over all recall thresholds to consider the confidence of predicted tracks [^2]. |

[^1]: [_HOTA: A Higher Order Metric for Evaluating Multi-Object Tracking. Jonathon Luiten, Aljosa Osep, Patrick Dendorfer, Philip Torr, Andreas Geiger, Laura Leal-Taixe, Bastian Leibe. IJCV 2020_](https://arxiv.org/abs/2009.07736).

[^2]: [_3D Multi-Object Tracking: A Baseline and New Evaluation Metrics. Xinshuo Weng, Jianren Wang, David Held, Kris Kitani. IROS 2020_](https://arxiv.org/abs/1907.03961).

We can run tracking evaluation using the following code snippet. 
```bash 
from av2.evaluation.tracking.eval import evaluate
res =  evaluate(track_predictions, labels, objective_metric, ego_distance_threshold_m, dataset_dir, outputs_dir)
```
- `track_predictions`: Track predictions
- `labels`: Ground truth annotations
- `objective_metric`: Metric to optimize per-class recall (e.g. HOTA, MOTA, default is HOTA)
- `ego_distance_threshold_m`: Filter for all detections outside of `ego_distance_threshold_m` (default is 50 meters).
- `dataset_dir`: Path to dataset directory (e.g. data/Sensor/val)
- `outputs_dir`: Path to output directory

## Forecasting

### Submission Format

The evaluation expects a dictionary of dictionaries of lists of dictionaries

```python
{
  <log_id>: {
    <timestamp_ns>: [
      {
         "prediction": <prediction>
         "score": <score>
         "detection_score": <detection_score>,
         "instance_id": <instance_id>
         "current_translation_m": <current_translation_m>,
         "label": <label>,
         "name": <name>,
         "size": <size>,
      }, ...
    ], ...
  }, ...
}
```

- `log_id`: Log id associated with the forecast, also called `seq_id`.
- `timestamp_ns`: Timestamp associated with the detections.
- `prediction`: K translation forecasts 3 seconds into the future.
- `score`: Forecast confidence.
- `detection_score`: Detection confidence.
- `instance_id`: Unique id assigned to each object.
- `current_translation_m`: xyz-components of the object translation in the city reference frame at the current timestamp, in meters.
- `label`: Integer index of the object class.
- `name`: Object class name.
- `size`: Object extent along the x,y,z axes in meters.

~~~admonish example

```python
# (1). Example forecasts.
example_forecasts = {
  '02678d04-cc9f-3148-9f95-1ba66347dff9': {
    315969904359876000: [
      {'timestep_ns': 315969905359854000,
      'current_translation_m': array([6759.4230302 , 1596.38016309]),
      'detection_score': 0.54183,
      'size': array([4.4779487, 1.7388916, 1.6963532], dtype=float32),
      'label': 0,
      'name': 'REGULAR_VEHICLE',
      'prediction': array([[[6759.4230302 , 1596.38016309],
              [6759.42134062, 1596.38361481],
              [6759.41965104, 1596.38706653],
              [6759.41796145, 1596.39051825],
              [6759.41627187, 1596.39396997],
              [6759.41458229, 1596.39742169]],
 
              [[6759.4230302 , 1596.38016309],
              [6759.4210027 , 1596.38430516],
              [6759.4189752 , 1596.38844722],
              [6759.4169477 , 1596.39258928],
              [6759.4149202 , 1596.39673134],
              [6759.41289271, 1596.40087341]],
 
              [[6759.4230302 , 1596.38016309],
              [6759.42066479, 1596.3849955 ],
              [6759.41829937, 1596.38982791],
              [6759.41593395, 1596.39466031],
              [6759.41356854, 1596.39949272],
      ...
              [6759.41998895, 1596.38637619],
              [6759.4189752 , 1596.38844722],
              [6759.41796145, 1596.39051825]]]),
      'score': [0.54183, 0.54183, 0.54183, 0.54183, 0.54183],
      'instance_id': 0},
      ...
    ]
    ...
  }
}

# (2). Prepare for submission.
import pickle

with open("forecast_predictions.pkl", "wb") as f:
       pickle.dump(example_forecasts, f)
```
~~~

### Evaluation

| **Metric** | **Description**|
|:-----------|:---------------|
| $\text{mAP}_{\text{forecasting}}$ | This is similar to $\text{mAP}$, but we define a true positive with reference to the current frame $T$ if there is a positive match in both the current timestamp $T$ and the future (final) timestep $T + N$ . Importantly, unlike $\text{ADE}$ and $\text{FDE}$, this metric considers both true positive and false positive trajectories. We average $\text{AP}_\text{forecasting}$ over static, linear, and non-linearly moving cohorts. |
| $\text{ADE}$                      | The average $\ell_2$ distance between the best forecasted trajectory and the ground truth. The best here refers to the trajectory that has the minimum endpoint error. We average $\text{ADE}$ over static, linear, and non-linearly moving cohorts. |
| $\text{FDE}$                      | The $\ell_2$ distance between the endpoint of the best forecasted trajectory and the ground truth. The best here refers to the trajectory that has the minimum endpoint error. We average $\text{FDE}$ over static, linear, and non-linearly moving cohorts. |

```admonish info
For additional information, please see:

[_Forecasting from LiDAR via Future Object Detection. Neehar Peri, Jonothan Luitein, Mengtian Li, Aljosa Osep, Laura Leal-Taixe, Deva Ramanan. CVPR 2022_](https://arxiv.org/abs/2203.16297)
```

We show how to run the forecasting evaluation below:

```python 
from av2.evaluation.forecasting.eval import evaluate

res = evaluate(forecasts, labels, top_k, ego_distance_threshold_m, dataset_dir)
```
- `forecasts`: Forecast predictions
- `labels`: Ground truth annotations
- `top_k`: Top K evaluation of multi-future forecasts (default is 5)
- `ego_distance_threshold_m`: Filter for all detections outside of `ego_distance_threshold_m` (default is 50 meters).
- `dataset_dir`: Path to dataset directory (e.g. data/Sensor/val)

## Supporting Publications

If you participate in this challenge, please consider citing:

```BibTeX
@INPROCEEDINGS {peri2022futuredet,
  title={Forecasting from LiDAR via Future Object Detection},
  author={Peri, Neehar and Luiten, Jonathon and Li, Mengtian and Osep, Aljosa and Leal-Taixe, Laura and Ramanan, Deva},
  journal={CVPR},
  year={2022},
}
```

```BibTeX 
@INPROCEEDINGS {peri2022towards,
  title={Towards Long Tailed 3D Detection},
  author={Peri, Neehar and Dave, Achal and Ramanan, Deva, and Kong, Shu},
  journal={CoRL},
  year={2022},
}
```
