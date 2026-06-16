# SfM + Visual Hull — 3D-реконструкция объекта из видео

Пайплайн для построения **полноценной цветной 3D-модели** реального объекта из обычного видео (снятого, например, на iPhone). Реализованы два подхода: быстрый SfM-точечный облак прямо из OpenCV и плотный визуальный корпус (visual hull) через COLMAP + space carving + Marching Cubes.

Результат — `.ply`-файл с текстурированным мешем, который открывается в любом 3D-редакторе (MeshLab, Blender, Open3D).

---

## Как работает пайплайн

| Шаг | Инструмент | Что происходит |
|---|---|---|
| 1. Видео → кадры | OpenCV `VideoCapture` | извлечение кадров с заданным шагом |
| 2. Маски объекта | OpenCV HSV + морфология | бинарные силуэты объекта |
| 3. Восстановление камер | **COLMAP (SfM)** | нахождение поз всех камер и sparse point cloud |
| 4. Space carving | NumPy (реализация вручную) | воксельная сетка 640³, пересечение силуэтов из всех камер |
| 5. Извлечение меша | **Marching Cubes** (scikit-image) | iso-surface из воксельного occupancy grid |
| 6. Постобработка | **Open3D** | сглаживание Таубина, удаление дегенеративных граней, non-manifold рёбер |
| 7. Раскраска вершин | проекция по камерам | усреднение цвета по N ближайшим видам |
| 8. Просмотр | Open3D Visualizer | интерактивный 3D-вьювер |

---

## Структура репозитория

```
sfm-visual-hull-3d/
├── main_box.py                   — быстрый SfM: SIFT/ORB → триангуляция → sparse point cloud
├── make_masks_box.py             — генерация бинарных масок по HSV-цвету + морфологическое сглаживание
├── visual_hull_from_masks.py     — основной скрипт: space carving → Marching Cubes → окрашенный mesh
└── colmap_images/
    ├── run-colmap-geometric.sh   — dense reconstruction (geometric consistency)
    └── run-colmap-photometric.sh — dense reconstruction (photometric)
```

Рантайм-папки (не в репозитории, создаются при запуске):

```
frames/               — кадры из видео
masks/                — бинарные маски объекта
colmap_text/          — cameras.txt / images.txt / points3D.txt
colmap_images/images/ — undistorted кадры из COLMAP
output/               — box_sfm_color.ply, box_visual_hull.ply
```

---

## Требования

```bash
pip install numpy opencv-python open3d scikit-image
```

- Python 3.9–3.11
- [COLMAP](https://colmap.github.io/) (обязательно для `visual_hull_from_masks.py`)

---

## Запуск

### 1. Маски объекта

Положи кадры в `frames/`, запусти:

```bash
python make_masks_box.py
```

### 2. SfM через COLMAP

1. Feature Extraction → Feature Matching → Sparse Reconstruction
2. Image Undistortion → Export model as TXT
3. Результат: `colmap_text/cameras.txt`, `images.txt`, `points3D.txt`

### 3. Быстрый SfM (опционально, без COLMAP)

```bash
python main_box.py   # VIDEO_PATH = "my_video.mp4" в начале скрипта
```

Выход: `output/box_sfm_color.ply`

### 4. Visual Hull (основной сценарий)

```bash
python visual_hull_from_masks.py
```

Выход: `output/box_visual_hull.ply` — открывается автоматически в Open3D.

---

## Ключевые параметры

```python
# visual_hull_from_masks.py
VOX_RES      = 640   # разрешение воксельной сетки (выше → точнее, медленнее)
MIN_VIEWS    = 3     # минимум камер, чтобы воксель считался занятым
SMOOTH_ITERS = 5     # итерации сглаживания меша

# main_box.py
FRAME_STRIDE = 1     # шаг извлечения кадров (1 = каждый кадр)
DOWNSCALE    = 0.6   # уменьшение разрешения для ускорения
FOV_DEG      = 70.0  # угол обзора (iPhone ≈ 70°)
```

---

## Стек

- **Python**, **NumPy**
- **OpenCV** — извлечение кадров, SIFT/ORB, HSV-сегментация, морфологические операции
- **COLMAP** — Structure-from-Motion, восстановление поз камер и sparse point cloud
- **scikit-image** — Marching Cubes (извлечение iso-surface из воксельного поля)
- **Open3D** — постобработка меша (сглаживание Таубина), раскраска вершин, визуализация
