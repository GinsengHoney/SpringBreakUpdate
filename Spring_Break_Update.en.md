# Spring Break Update

**Algorithm / project:** NUREC (based on 3DGRUT)

**GitHub repository:** [nv-tlabs/3dgrut: Ray tracing and hybrid rasterization of Gaussian particles](https://github.com/nv-tlabs/3dgrut)

---

## Three sets of render results

### Official Garden dataset

- Training PNGs: about **11 MB per image**, **140 images** for this scene.

![image-20260323123737146](C:\Users\10305\AppData\Roaming\Typora\typora-user-images\image-20260323123737146.png)

![image-20260323123543889](C:\Users\10305\AppData\Roaming\Typora\typora-user-images\image-20260323123543889.png)

### My examples

**Example 1:**

![image-20260323130714838](C:\Users\10305\AppData\Roaming\Typora\typora-user-images\image-20260323130714838.png)

![image-20260323125515945](C:\Users\10305\AppData\Roaming\Typora\typora-user-images\image-20260323125515945.png)

![image-20260323130623564](C:\Users\10305\AppData\Roaming\Typora\typora-user-images\image-20260323130623564.png)

![image-20260323130855809](C:\Users\10305\AppData\Roaming\Typora\typora-user-images\image-20260323130855809.png)

**Example 2:**

![image-20260323130746578](C:\Users\10305\AppData\Roaming\Typora\typora-user-images\image-20260323130746578.png)

![image-20260323131714865](C:\Users\10305\AppData\Roaming\Typora\typora-user-images\image-20260323131714865.png)

### Renders from about 2–3 weeks ago

![image-20260323132155199](C:\Users\10305\AppData\Roaming\Typora\typora-user-images\image-20260323132155199.png)

---

## Optimization attempts

Reference: [How to Enhance 3D Gaussian Reconstruction Quality for Simulation (NVIDIA Blog)](https://developer.nvidia.com/blog/how-to-enhance-3d-gaussian-reconstruction-quality-for-simulation/)

![image-20260324170416653](C:\Users\10305\AppData\Roaming\Typora\typora-user-images\image-20260324170416653.png)

To mitigate **holes or blurry textures** where **camera coverage is limited**, I applied the blog’s **Fixer** approach to the dataset images; **PSNR improved**.

**NVIDIA blog screenshot:**

![image-20260323133358348](C:\Users\10305\AppData\Roaming\Typora\typora-user-images\image-20260323133358348.png)

Reference: [gsplat large-scale example](https://docs.gsplat.studio/main/examples/large_scale.html)

**Key points:**

- Scales Garden into a large grid (on the order of **30M Gaussians** in the example).
- Uses `radius_clip` in rasterization to discard distant Gaussians with a tiny projected radius.
- Keeps rendering fast by reducing the number of active splats.

Besides training and rendering with **3DGRUT**, I also tried **gsplat** and used the techniques from the blog above for rendering.

![image-20260323123543889](C:\Users\10305\AppData\Roaming\Typora\typora-user-images\image-20260323123543889.png)

---

## CARLA

**CARLA** is an open-source autonomous driving simulator that provides **controllable, reproducible** virtual urban environments.

### 1. `carla_trajectory_demo.py`

- **Purpose:** Trajectory-tracking demo in CARLA.
- **Includes:** Spawn a vehicle, build a route from map waypoints, Pure Pursuit steering + PID speed control, automatic braking at the end of the route, and cleanup of actors when finished.
- **Optional:** Follow-cam modes (`--follow-view`, `--view-mode chase|first_person`, `--chase-distance`, `--chase-height`, etc.).

### 2. `carla_aeb_pedestrian_test.py`

- **Purpose:** Pedestrian crossing + **Automatic Emergency Braking (AEB)** test.
- **Includes:** Construct a crossing scenario, trigger braking using **TTC**, use a collision sensor for real collision detection, and optional first-person / chase camera views.
- **Reported metrics:** e.g. `triggered`, `min_ttc`, `collision`, `final_speed`, `trigger_speed`, `stop_time`, `stop_distance`.

### 3. `carla_weather_robustness_test.py`

- **Purpose:** Batch-run trajectory tracking under **multiple weather conditions** for robustness testing.
- **Includes:** Export a CSV (fields such as `weather`, `success`, `elapsed_s`, `avg_speed_mps`, `max_cte_m`) and run in **synchronous step** mode.

![image-20260323134645745](C:\Users\10305\AppData\Roaming\Typora\typora-user-images\image-20260323134645745.png)

![image-20260324160055233](C:\Users\10305\AppData\Roaming\Typora\typora-user-images\image-20260324160055233.png)

![image-20260324160108746](C:\Users\10305\AppData\Roaming\Typora\typora-user-images\image-20260324160108746.png)

---

## Issues and troubleshooting

### (1) USDZ exported from 3DGRUT does not import correctly into Isaac Sim

- Converting to **USDA** still did not display.

![image-20260324161738100](C:\Users\10305\AppData\Roaming\Typora\typora-user-images\image-20260324161738100.png)

- Tried injecting a **triangle mesh** into the existing USDZ (`add_mesh_to_usdz` and the Garden-specific injection approach) so Isaac could at least show `UsdGeom.Mesh`: **still could not import**; likely **environment / Kit incompatibility** with the generated USDZ.

### (2) Large quality gap across datasets with 3DGRUT

- **Garden (official, recommended):** good quality.

![image-20260324162010066](C:\Users\10305\AppData\Roaming\Typora\typora-user-images\image-20260324162010066.png)

- **Custom capture:** much worse quality. Training settings and COLMAP preprocessing were double-checked; the **dataset itself** is the most likely bottleneck.

Reference: [How to Instantly Render Real-World Scenes in Interactive Simulation (NVIDIA Blog)](https://developer.nvidia.com/blog/how-to-instantly-render-real-world-scenes-in-interactive-simulation/)

COLMAP outputs were reviewed multiple times; no obvious fatal issues (but subtle differences may exist).

![image-20260324163116196](C:\Users\10305\AppData\Roaming\Typora\typora-user-images\image-20260324163116196.png)

Per 3DGRUT guidance, capture instructions are brief; I increased the number of images from about **40** to **120** (Garden official has about **140**) and verified continuous viewpoints. Capture methods included iPhone bursts and frame extraction from video + COLMAP.

![image-20260324163225972](C:\Users\10305\AppData\Roaming\Typora\typora-user-images\image-20260324163225972.png)

**Observation:** official images are large (~**13 MB per image**); my PNGs are about **1–2 MB**, JPGs only a few hundred KB.

![image-20260324170053347](C:\Users\10305\AppData\Roaming\Typora\typora-user-images\image-20260324170053347.png)

![image-20260324170116602](C:\Users\10305\AppData\Roaming\Typora\typora-user-images\image-20260324170116602.png)

![image-20260324170233120](C:\Users\10305\AppData\Roaming\Typora\typora-user-images\image-20260324170233120.png)

---

## Isaac Sim

**USDZ**, **PLY**, and converted **USDA** from 3DGRUT / gsplat **fail to import** into Isaac Sim (errors on open).

I therefore switched to **NVIDIA sample data** (driving and robotics, among others). I downloaded a **robotics scene USDZ** and it **imports successfully**.

![image-20260324171016652](C:\Users\10305\AppData\Roaming\Typora\typora-user-images\image-20260324171016652.png)

![image-20260324171022698](C:\Users\10305\AppData\Roaming\Typora\typora-user-images\image-20260324171022698.png)

For **automotive driving sensor** releases: data often records a vehicle driving from A to B on the road, sometimes **only one side of the street**—not a **complete, closed** scene. It is effectively **single-viewpoint** data, so I **did not use it for training**.

![image-20260324171524639](C:\Users\10305\AppData\Roaming\Typora\typora-user-images\image-20260324171524639.png)

![image-20260324171543455](C:\Users\10305\AppData\Roaming\Typora\typora-user-images\image-20260324171543455.png)
