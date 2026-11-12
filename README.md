# 📍 Localize — LiDAR Map Localization (early version)

ROS 2 Humble workspace for **localizing the KMUTT autonomous golf cart on a prebuilt 3D point cloud map**
using **NDT (OpenMP)** scan-to-map matching with an Ouster LiDAR.

> ℹ️ This is an **early version** of the localization work.
> The current, maintained version is **[Localize_get_path](https://github.com/Pakgard007/Localize_get_path)**
> (with path following and CBF-QP control), and maps are made with **[Mapping](https://github.com/Pakgard007/Mapping)**.

## 📦 Contents

| Path | Description |
|---|---|
| `src/lidar_localization_ros2-main/` | Point cloud map localization node (NDT / GICP) → `/pcl_pose`, `/path` |
| `src/ndt_omp_ros2-humble/` | OpenMP-accelerated NDT and GICP registration |
| `src/reMap*.pcd` | Map set **reMap** (default — `reMapcloudMerged.pcd`) |
| `src/Makro*.pcd` | Map set **Makro** |
| `src/Tester*.pcd` | Map set **Tester** |

Each map set is LIO-SAM output: `cloudCorner`, `cloudSurf`, `cloudGlobal` / `cloudMerged`
(full map), `trajectory` and `transformations` (keyframe poses).

## 🔌 Inputs / Outputs

| Topic | Type | |
|---|---|---|
| `/ouster/points` | `sensor_msgs/PointCloud2` | LiDAR scan (remapped from `/cloud`) |
| `/odometry` | `nav_msgs/Odometry` | Wheel odometry prior (`use_odom: true`) |
| `/initialpose` | `geometry_msgs/PoseWithCovarianceStamped` | Set the start pose from RViz |
| `/pcl_pose` | `geometry_msgs/PoseWithCovarianceStamped` | **Estimated pose** |
| `/path` | `nav_msgs/Path` | Traveled path |

## 🚀 Build & Run

```bash
mkdir -p ~/1Localize && cd ~/1Localize
git clone https://github.com/Pakgard007/Localize.git .
source /opt/ros/humble/setup.bash
rosdep install --from-paths src --ignore-src -r -y
colcon build --symlink-install --cmake-args -DCMAKE_BUILD_TYPE=Release
source install/setup.bash

ros2 launch lidar_localization_ros2 lidar_localization.launch.py
# use another map:
ros2 launch lidar_localization_ros2 lidar_localization.launch.py map_path:=$HOME/1Localize/src/MakrocloudMerged.pcd
```

Then set the initial pose in RViz with **2D Pose Estimate** if the cart does not start at the map origin.

## ⚙️ Configuration

`src/lidar_localization_ros2-main/param/localization.yaml`

| Parameter | Value | Note |
|---|---|---|
| `map_path` | launch argument, default `~/1Localize/src/reMapcloudMerged.pcd` | override with `map_path:=...` |
| `registration_method` | `NDT_OMP` | or `GICP_OMP` |
| `ndt_resolution` | `2.0` | m |
| `voxel_leaf_size` | `0.5` | scan downsampling, m |
| `scan_min_range` / `scan_max_range` | `1.5` / `150.0` | m |
| `use_odom` / `use_imu` | `true` / `false` | |
| `set_initial_pose` | `true` (origin) | |

## 🙏 Credits

- [lidar_localization_ros2](https://github.com/rsasaki0109/lidar_localization_ros2) — rsasaki0109
- [ndt_omp](https://github.com/koide3/ndt_omp) — koide3
- Maps built with [LIO-SAM](https://github.com/TixiaoShan/LIO-SAM)
