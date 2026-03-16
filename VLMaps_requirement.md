# VLMaps 部分最终输出要求（给下游 VLFM / A* 使用）

## 一、请你最终输出给我的文件目录结构

scene_export/
├─ occupancy_map.npy
├─ occupancy_map.png
├─ semantic_objects.json
├─ semantic_points.json
├─ topdown_rgb_map.png
├─ map_metadata.json
└─ export_readme.txt

---

## 二、每个输出文件的具体要求

### 1. occupancy_map.npy
**用途：**  
给 A* 和后续 VLFM 使用的二维占据栅格地图。

**要求：**
- 2D numpy 数组
- 建议 dtype 为 `uint8`
- 建议编码方式：
  - `0 = 障碍物`
  - `1 = 可通行区域`
  - `2 = 未知区域`
- 如果只做二值地图，也请在 `map_metadata.json` 中写清楚定义

---

### 2. occupancy_map.png
**用途：**  
占据栅格地图的可视化版本，方便人工检查。

**要求：**
- 与 `occupancy_map.npy` 完全对应
- 建议可视化方式：
  - 黑色 = obstacle
  - 白色 = free
  - 灰色 = unknown

---

### 3. semantic_objects.json
**用途：**  
给我做语言目标检索与导航目标选择。

**要求：**
- 请输出“对象级”语义结果，而不是只有可视化图片
- 每个对象至少包含以下字段

**示例格式：**
{
  "objects": [
    {
      "object_id": 0,
      "label": "chair",
      "score": 0.91,
      "grid_centroid": [142, 87],
      "world_centroid": [3.42, 0.00, -1.85],
      "bbox_grid": [138, 82, 146, 91],
      "grid_pixels": [[141, 87], [142, 87], [143, 87]]
    }
  ]
}

**字段说明：**
- `object_id`：对象编号
- `label`：语义类别，例如 `chair` / `door` / `desk` / `bookshelf`
- `score`：该对象类别置信度
- `grid_centroid`：该对象在二维栅格地图中的中心点
- `world_centroid`：该对象在世界坐标系中的中心点
- `bbox_grid`：对象在二维地图中的边界框
- `grid_pixels`：属于该对象的栅格点集合（可选，但强烈建议保留）

---

### 4. semantic_points.json
**用途：**  
如果对象级聚类结果不完整，至少给点级语义结果作为保底。

**要求：**
- 每个语义点至少包含以下字段

**示例格式：**
{
  "points": [
    {
      "grid": [142, 87],
      "world": [3.42, 0.00, -1.85],
      "label": "chair",
      "score": 0.91
    }
  ]
}

**说明：**
- 如果 `semantic_objects.json` 已经比较完整，这个文件可以作为补充
- 如果对象级结果不好做，至少一定要给这个文件

---

### 5. topdown_rgb_map.png
**用途：**  
给最终展示、调试和路径可视化使用。

**要求：**
- 俯视彩色地图
- 最好与 occupancy map 使用同一坐标系、同一分辨率
- 方便后续叠加：
  - 当前位姿
  - 目标点
  - A* 路径

---

### 6. map_metadata.json
**用途：**  
用于统一坐标系、栅格分辨率和地图解释规则。

**要求：**
至少包含以下内容：

**示例格式：**
{
  "scene_name": "library_scene_01",
  "map_height": 400,
  "map_width": 400,
  "cell_size_m": 0.05,
  "origin_world": [0.0, 0.0, 0.0],
  "grid_to_world": {
    "row_axis": "z",
    "col_axis": "x"
  },
  "occupancy_label_def": {
    "0": "obstacle",
    "1": "free",
    "2": "unknown"
  }
}

**字段说明：**
- `scene_name`：场景名称
- `map_height`, `map_width`：地图尺寸
- `cell_size_m`：每个栅格对应的实际米数
- `origin_world`：地图原点在世界坐标系中的位置
- `grid_to_world`：grid 和 world 的坐标轴对应关系
- `occupancy_label_def`：occupancy map 中各数字的含义

---

### 7. export_readme.txt
**用途：**  
补充说明，避免后续接口理解错误。

**请至少说明：**
- 实际使用了哪些 rosbag topic
- RGB / depth 是如何和轨迹时间戳对齐的
- 使用的是 keyframe 还是全部帧
- depth 单位是什么（米 / 毫米）
- 地图坐标系是如何定义的
- semantic label 的类别集合有哪些

---

## 三、最低交付要求（最少必须先给我这四个）
如果时间不够，至少先给我下面四个文件：

- `occupancy_map.npy`
- `semantic_objects.json`
- `map_metadata.json`
- `topdown_rgb_map.png`

这样我就可以先开始做后续的 VLFM 与 A* 对接。