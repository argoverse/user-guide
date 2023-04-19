# 4D Occupancy Forecasting
                                                
## Table of Contents                                                                            
                                                                                                
<!-- toc -->           
                                                
## Motivation                
                                                                                                
Understanding how an environment evolves with time is crucial for motion planning in autonomous systems. Classical methods may be lacking because they rely on costly human annotations in the fo
rm of semantic class labels, bounding boxes, and tracks or HD maps of cities to plan their motion — and thus are difficult to scale to large unlabeled datasets. Related tasks such as point clou
d forecasting require algorithms to implicitly capture (1) vehicle extrinsics (i.e., the egomotion of the autonomous vehicle), (2) sensor intrinsics (i.e., the sampling pattern specific to the 
particular lidar sensor), and (3) the motion of other objects (things and stuff) in the scene. We argue from an autonomy perspective, the most useful and generic output is (3), which is directl
y evaluated by 4D spacetime occupancy forecasting. This allows for the possibility of training and evaluating occupancy algorithms across diverse datasets, sensors, and vehicles.
                                                
                                                
## Problem Formulation
                                                                                                                                                                                                 
Given an agent's observations of the world for the past `n` seconds, aligned to the LiDAR coordinate frame of the current timestep, predict the spacetime evolution of the world for the next `n`
 seconds. Like other forecasting tasks, `n` can be taken as 1 or 3. We process and predict 5 timesteps each in this window. In other words,
                                                                                                                                                                                                 - Given 5 point clouds from the past and current timesteps (all aligned to the LiDAR coordinate frame of the current timestep), forecast the spacetime occupancy for the next 5 timesteps.
- These 5 point clouds span a horizon of say 3s in the past and present, and 3s in the future (at a frequency of 5/3Hz). For example, for 3s forecasting, you should process the timesteps (-2.4s
, -1.8s ..., 0s) and output the timesteps (0.6s, 1.2s ..., 3s).                                                                                                                                  - All input sequences (from -2.4s to 0s) and all output occupancies (from 0.6s to 3s) should be aligned to the LiDAR coordinate frame of the 0s point cloud.                               
- Given a prediction of the future spacetime occupancy of the world, our objective is to scalably evaluate this prediction by being agnostic to the choice of occupancy representation. We achiev
e this by querying rays into the occupancy volume. We provide a set of query rays, and for each query ray, we require an estimate of the expected distance travelled by this ray before hitting a
ny surface. We will refer to this as the expected depth along the given ray origin and direction.

                                                
## Evaluation Metrics            
                                                                                                
| **Metric** | **Description** |                                                                
|:-----------------|:----------------|
| L1 Error (L1) | The absolute L1 distance between the ground-truth expected depth along a given ray direction and the predicted expected depth along the same ray direction. |
| Absolute Relative Error (AbsRel) | The absolute L1 distance between the ground-truth expected depth along a given ray direction and the predicted expected depth along the same ray direction, divided by the ground-truth expected depth. This metric weighs the errors made close to the ego vehicle higher as compared to the same amount of error made far-away. |
| Near-field Chamfer Distance (NFCD) | A set of forecasted point clouds is created from the submitted expected depths along the given ray directions. Only the points within the near-field volume are retained. Near-field chamfer distance is the average bidirectional chamfer distance between this point cloud and the ground-truth point cloud for every future timestep, also truncated to the near-field volume. |
| Vanilla Chamfer Distance (CD) | A set of forecasted point clouds is created from the submitted expected depths along the given ray directions. Vanilla chamfer distance is the average bidirectional chamfer distance between this point cloud and the ground-truth point cloud, for every future timestep. |


## Getting Started

To get started with training your own models for this task on Argoverse 2.0, you can follow the instructions below:

- Download the Argoverse 2.0 [Sensor dataset](https://www.argoverse.org/av2.html#sensor-link) from our [website](https://www.argoverse.org/index.html). Technically the LiDAR dataset is the best suited for this self-supervised task as it is much larger than any of our other datasets and can be used to learn generic priors at scale but it can be too big to get started with.
- Check out an Argoverse dataloader implementation for this task provided [here](https://github.com/tarashakhurana/4d-occ-forecasting/blob/main/cvpr23-evalkit/data/av2.py). A sample script [here](https://github.com/tarashakhurana/4d-occ-forecasting/blob/main/cvpr23-evalkit/load_sequences.py) shows how to use this dataloader.
- Build your own 4D occupancy forecasting model with your choice of the occupancy representation! Some people like voxels, some even like point clouds (aka the line of work on point cloud forecasting), and some like NeRFs! Two reference baselines for this task are provided in a [recent work](https://github.com/tarashakhurana/4d-occ-forecasting).
- Evaluate your forecasts. See below for a script that does local evaluation.


### Note on Point Cloud Forecasting

One could also use point clouds as a representation of occupancy. Therefore, forecasting point clouds is also valid (therefore, this task also encapsulates the line of work on point cloud forecasting). Traditionally, point cloud forecasting approaches do not necessarily output the same number of points as in the input, or even points along the same ray directions as the input. If you are using a point cloud forecasting approach, you can get a set of expected depths along the input ray directions by computing an interpolated point cloud such as done [here](https://github.com/tarashakhurana/4d-occ-forecasting/blob/74549e2066d3f77aa2122f46fc30502b4d4bfbf9/utils/evaluation.py#L225). Pass `True` to `return_interpolated_pcd`.


## CVPR '23 Challenge

Once you have a working model, you can submit the results from your model in the first iteration of the [Argoverse 2.0 4D Occupancy challenge](https://eval.ai/web/challenges/challenge-page/1977/overview) being hosted as a part of the Workshop on Autonomous Driving at CVPR '23. First, generate the set of query rays from the [eval-kit](https://github.com/tarashakhurana/4d-occ-forecasting/tree/main/cvpr23-evalkit). For each query ray, you will submit an expected distance along that ray as defined above. The format of the JSON that will be created as query will look like the following:

```
{
    'queries': [
        {
            'horizon': '3s',
            'rays': {
                '<log_id>': {
                    '<frame_id>': List[List[List]],
                    '<frame_id>': List[List[List]]
                    ...
                },
                '<log_id>': {
                    '<frame_id>': List[List[List]],
                    '<frame_id>': List[List[List]]
                    ...
                }
            }
        }
    ]
}
```
- `horizon`: Temporal extent to which we want to forecast. For the purpose of the challenge, we only consider 3s.
- `<log_id>`: Identifier of the log being used. Each log can be used to create multiple sequences of 10 frames each. 5 of the frames will be the past (input) and 5 will be the future (output).
- `<frame_id>`: Identifier of the frame at t=0s in a sequence from the specified log.
- `List[List[List]]`: For every timestep in the future, there is a list of rays, where every ray has an origin and unit direction. If this were a numpy array, the shape would have been `5 x N x 6` but `N` can be variable for every timestep so these are instead stored as a list.

When making a submission to the Eval AI server, you will replace the last List of length 6, with a list of length 1 which will store the expected depth along this ray. You can also generate the groundtruth from the eval-kit and compute the metrics locally by using `evaluate.py`. Usage: ```python evaluate.py --annotations /path/to/annotations --submission /path/to/submission```.

## Citing
If you find any of this helpful and end up using it in your work, please consider citing:

```BibTeX
@INPROCEEDINGS {khurana2023point,
  title={Point Cloud Forecasting as a Proxy for 4D Occupancy Forecasting},
  author={Khurana, Tarasha and Hu, Peiyun and Held, David and Ramanan, Deva},
  journal={CVPR},
  year={2023},
}
```
