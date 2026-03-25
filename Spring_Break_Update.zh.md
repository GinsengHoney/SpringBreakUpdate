# 春假进展更新

**算法 / 项目：** NUREC（基于 3DGRUT）

**GitHub 仓库：** [nv-tlabs/3dgrut：高斯粒子的光线追踪与混合光栅化](https://github.com/nv-tlabs/3dgrut)

---

## 三组渲染结果

### 官方数据集 Garden

- 用于训练的 PNG：每张约 **11 MB**，该场景共 **140 张** 图像。

![image-20260323123737146](C:\Users\10305\AppData\Roaming\Typora\typora-user-images\image-20260323123737146.png)

![image-20260323123543889](C:\Users\10305\AppData\Roaming\Typora\typora-user-images\image-20260323123543889.png)

### 我的示例

**示例 1：**

![image-20260323130714838](C:\Users\10305\AppData\Roaming\Typora\typora-user-images\image-20260323130714838.png)

![image-20260323125515945](C:\Users\10305\AppData\Roaming\Typora\typora-user-images\image-20260323125515945.png)

![image-20260323130623564](C:\Users\10305\AppData\Roaming\Typora\typora-user-images\image-20260323130623564.png)

![image-20260323130855809](C:\Users\10305\AppData\Roaming\Typora\typora-user-images\image-20260323130855809.png)

**示例 2：**

![image-20260323130746578](C:\Users\10305\AppData\Roaming\Typora\typora-user-images\image-20260323130746578.png)

![image-20260323131714865](C:\Users\10305\AppData\Roaming\Typora\typora-user-images\image-20260323131714865.png)

### 约 2–3 周前的渲染结果

![image-20260323132155199](C:\Users\10305\AppData\Roaming\Typora\typora-user-images\image-20260323132155199.png)

---

## 优化尝试

参考：[How to Enhance 3D Gaussian Reconstruction Quality for Simulation（NVIDIA 博客）](https://developer.nvidia.com/blog/how-to-enhance-3d-gaussian-reconstruction-quality-for-simulation/)

![image-20260324170416653](C:\Users\10305\AppData\Roaming\Typora\typora-user-images\image-20260324170416653.png)

针对**相机覆盖有限**导致部分表面**孔洞或纹理模糊**的问题，按文中 **Fixer** 思路处理数据集图像，**PSNR 有所提升**。

**NVIDIA 博客截图：**

![image-20260323133358348](C:\Users\10305\AppData\Roaming\Typora\typora-user-images\image-20260323133358348.png)

参考：[gsplat 大规模场景示例](https://docs.gsplat.studio/main/examples/large_scale.html)

**要点：**

- 将 Garden 扩展为大规模网格（示例中约 **3000 万**高斯）。
- 光栅化中使用 `radius_clip`，丢弃投影半径极小的远处高斯。
- 通过减少有效 splat 数量保持渲染速度。

除用 **3DGRUT** 训练与渲染外，期间也尝试了 **gsplat**，并结合上述博客中的技巧进行渲染。

![image-20260323123543889](C:\Users\10305\AppData\Roaming\Typora\typora-user-images\image-20260323123543889.png)

---

## CARLA

**CARLA** 是开源自动驾驶仿真器，提供**可控、可复现**的虚拟城市场景。

### 1. `carla_trajectory_demo.py`

- **作用：** 在 CARLA 中做轨迹跟随演示。
- **包含：** 生成车辆、由地图路点建路、Pure Pursuit 转向 + PID 速度控制、终点自动刹停、结束后清理 actor。
- **可选：** 跟随视角（`--follow-view`、`--view-mode chase|first_person`、`--chase-distance`、`--chase-height` 等）。

### 2. `carla_aeb_pedestrian_test.py`

- **作用：** 行人横穿 + 自动紧急制动（AEB）测试。
- **包含：** 构造横穿场景、按 TTC 触发制动、用碰撞传感器做真实碰撞判定、可选第一人称/跟车视角。
- **输出指标：** 如 `triggered`、`min_ttc`、`collision`、`final_speed`、`trigger_speed`、`stop_time`、`stop_distance`。

### 3. `carla_weather_robustness_test.py`

- **作用：** 在多种天气下批量跑轨迹跟随，做鲁棒性测试。
- **包含：** 导出 CSV（如 `weather`、`success`、`elapsed_s`、`avg_speed_mps`、`max_cte_m` 等字段），同步步进执行。

![image-20260323134645745](C:\Users\10305\AppData\Roaming\Typora\typora-user-images\image-20260323134645745.png)

![image-20260324160055233](C:\Users\10305\AppData\Roaming\Typora\typora-user-images\image-20260324160055233.png)

![image-20260324160108746](C:\Users\10305\AppData\Roaming\Typora\typora-user-images\image-20260324160108746.png)

---

## 问题与排查

### （一）3DGRUT 导出的 USDZ 无法正常导入 Isaac Sim

- 尝试转为 **USDA** 仍无法显示。

![image-20260324161738100](C:\Users\10305\AppData\Roaming\Typora\typora-user-images\image-20260324161738100.png)

- 尝试在现有 USDZ 中**注入三角网格**（`add_mesh_to_usdz` 及 Garden 专用注入脚本思路），使 Isaac 至少能显示 `UsdGeom.Mesh`：**仍无法导入**；判断为**运行环境与生成 USDZ 不兼容**。

### （二）不同数据集下 3DGRUT 场景质量差异大

- **Garden（官方推荐）**：生成质量较好。

![image-20260324162010066](C:\Users\10305\AppData\Roaming\Typora\typora-user-images\image-20260324162010066.png)

- **自建数据**：质量很差。已核对训练参数与 COLMAP 预处理流程一致，**问题更可能出在数据集本身**。

参考 NVIDIA 博客：[How to Instantly Render Real-World Scenes in Interactive Simulation](https://developer.nvidia.com/blog/how-to-instantly-render-real-world-scenes-in-interactive-simulation/)

多次检查 COLMAP 输出，未发现明显硬伤（但可能存在细微差异）。

![image-20260324163116196](C:\Users\10305\AppData\Roaming\Typora\typora-user-images\image-20260324163116196.png)

按 3DGRUT 相关指导，数据采集说明较短；已将图像数量从约 **40 张**增至 **120 张**（Garden 官方约 **140 张**），并确认视角连续。采集方式包括：iPhone 连续拍照，以及从视频抽帧 + COLMAP。

![image-20260324163225972](C:\Users\10305\AppData\Roaming\Typora\typora-user-images\image-20260324163225972.png)

**数据对比观察：** 官方每张图体积大（约 **13 MB/张**）；自采 PNG 约 **1–2 MB**，JPG 仅数百 KB。

![image-20260324170053347](C:\Users\10305\AppData\Roaming\Typora\typora-user-images\image-20260324170053347.png)

![image-20260324170116602](C:\Users\10305\AppData\Roaming\Typora\typora-user-images\image-20260324170116602.png)

![image-20260324170233120](C:\Users\10305\AppData\Roaming\Typora\typora-user-images\image-20260324170233120.png)

---

## Isaac Sim

3DGRUT / gsplat 生成的 **USDZ、PLY** 及转换后的 **USDA** 在 Isaac Sim 中打开**报错**，无法正常导入。

因此改用 **NVIDIA 官方示例数据**（含驾驶与机器人等）。已下载并测试**机器人场景 USDZ**，可正常导入。

![image-20260324171016652](C:\Users\10305\AppData\Roaming\Typora\typora-user-images\image-20260324171016652.png)

![image-20260324171022698](C:\Users\10305\AppData\Roaming\Typora\typora-user-images\image-20260324171022698.png)

关于**汽车驾驶传感器类数据**：多为车辆在路面上从 A 到 B 的行程，且常**仅覆盖单侧街景**，不像**完整、封闭**场景，属于**单视角**数据，**未用于训练**。

![image-20260324171524639](C:\Users\10305\AppData\Roaming\Typora\typora-user-images\image-20260324171524639.png)

![image-20260324171543455](C:\Users\10305\AppData\Roaming\Typora\typora-user-images\image-20260324171543455.png)
