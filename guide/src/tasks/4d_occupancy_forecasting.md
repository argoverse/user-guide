# 4D Occupancy Forecasting

## Overview

Understanding how an environment evolves with time is crucial for motion planning in autonomous systems. Classical methods may be lacking because they rely on costly human annotations in the form of semantic class labels, bounding boxes, and tracks or HD maps of cities to plan their motion — and thus are difficult to scale to large unlabeled datasets. Related tasks such as point cloud forecasting require algorithms to implicitly capture (1) vehicle extrinsics (i.e., the egomotion of the autonomous vehicle), (2) sensor intrinsics (i.e., the sampling pattern specific to the particular lidar sensor), and (3) the motion of other objects (things and stuff) in the scene. We argue from an autonomy perspective, the most useful and generic output is (3), which is directly evaluated by 4D spacetime occupancy forecasting. This allows for the possibility of training and evaluating occupancy algorithms across diverse datasets, sensors, and vehicles.

## Task Definition

Given an agent's observations of the world for the past n seconds, aligned to the LiDAR coordinate frame of the current timestep, predict the spacetime evolution of the world for the next n seconds. For this challenge, we take n as 1 and 3. We process and predict 5 timesteps each in this window. In other words,

Given 5 point clouds from the past and current timesteps (all aligned to the LiDAR coordinate frame of the current timestep), forecast the spacetime occupancy for the next 5 timesteps.
These 5 point clouds may span a horizon of 1s in the past and present and 1s in the future (at a frequency of 5Hz), or a horizon of 3s in the past and present and 3s in the future (at a frequency of 5/3Hz). For example, for 1s forecasting, you should process the timesteps (-0.8s, -0.6s ..., 0s) and output the timesteps (0.2s, 0.4s ..., 1s).
All input sequences (in the above example from -0.8s to 0s) and all output occupancies (in the above example from 0.2s to 1s) should be aligned to the LiDAR coordinate frame of the 0s point cloud.
Given a prediction of the future spacetime occupancy of the world, our objective is to scalably evaluate this prediction by querying rays into the occupancy volume. For this, we provide a set of query rays. For each query ray, we require an estimate of the expected distance travelled by this ray before hitting any surface. We will refer to this as the expected depth along the given ray origin and direction.

