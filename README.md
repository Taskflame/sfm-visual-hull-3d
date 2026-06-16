
# 📦 Blue Box 3D Reconstruction (SfM + Visual Hull)

Проект для восстановления **3D-модели синей коробки** из видео или набора кадров.
Используются методы **Structure-from-Motion (SfM)**, **маски по цвету** и **Visual Hull (space carving)**.

Подходит для тестирования на **разных устройствах**, с **разными видео**, без жёсткой привязки к одному окружению.

---

## 🧠 Идея пайплайна

1. 🎥 Видео → кадры
2. 🔵 Построение масок синей коробки
3. 📷 SfM (через COLMAP) — восстановление камер
4. 🧊 Visual Hull — воксельное пересечение силуэтов
5. 🧱 Mesh + сглаживание + окраска
6. 👀 Просмотр результата в Open3D

---

## 📁 Структура проекта

```text
Box_project_minor/
│
├── main_box.py                 # Быстрый SfM + point cloud из видео
├── make_masks_box.py           # Генерация бинарных масок коробки
├── visual_hull_from_masks.py   # Построение плотной 3D-модели (Visual Hull)
│
├── frames/                     # Кадры из видео (JPEG)
├── masks/                      # Маски коробки (PNG/JPEG)
│
├── colmap_images/
│   └── images/                 # Undistorted изображения из COLMAP
│
├── colmap_text/                # cameras.txt / images.txt / points3D.txt
│
├── output/
│   ├── box_sfm_color.ply
│   └── box_visual_hull.ply
│
└── README.md
```

---

## ⚙️ Требования

### Python

Рекомендуется **Python 3.9–3.11**

### Библиотеки

```bash
pip install numpy opencv-python open3d scikit-image
```

Если используешь ORB вместо SIFT — дополнительных шагов не требуется.

---

## 🎥 Работа с другим видео

### 1️⃣ Подготовь видео

Положи видео в корень проекта или укажи путь:

```python
VIDEO_PATH = "my_video.mp4"
```

Поддерживаются:

* `.mp4`
* `.mov`
* `.avi`

---

### 2️⃣ Запуск `main_box.py` (опционально)

```bash
python main_box.py
```

Что делает:

* читает видео
* ищет фичи (SIFT / ORB)
* триангулирует точки
* сохраняет облако точек:

```text
output/box_sfm_color.ply
```

> ⚠️ Этот шаг **не обязателен**, если используешь COLMAP отдельно.

---

## 📷 COLMAP (обязательно для Visual Hull)

Для корректной работы `visual_hull_from_masks.py` требуется результат COLMAP:

```text
colmap_text/
 ├── cameras.txt
 ├── images.txt
 └── points3D.txt
```

А также undistorted изображения:

```text
colmap_images/images/
```

Рекомендуемые шаги в COLMAP:

1. Feature Extraction
2. Feature Matching
3. Sparse Reconstruction
4. Image Undistortion
5. Export model as **TXT**

---

## 🔵 Генерация масок коробки

Перед запуском:

```text
frames/
 ├── frame_0001.jpg
 ├── frame_0002.jpg
 └── ...
```

Запуск:

```bash
python make_masks_box.py
```

Результат:

```text
masks/
 ├── frame_0001.jpg
 ├── frame_0002.jpg
 └── ...
```

Маски строятся:

* по HSV-цвету (синий)
* с заливкой контуров
* морфологическим сглаживанием

---

## 🧊 Построение 3D-модели (Visual Hull)

```bash
python visual_hull_from_masks.py
```

Скрипт:

* читает камеры COLMAP
* строит воксельную сетку
* выполняет **space carving**
* применяет **Marching Cubes**
* сглаживает mesh
* окрашивает вершины по кадрам

Результат:

```text
output/box_visual_hull.ply
```

---

## 👀 Просмотр результата

Открывается автоматически в окне Open3D
или вручную:

```bash
open3d output/box_visual_hull.ply
```

---

## 🛠 Полезные параметры

### `visual_hull_from_masks.py`

```python
VOX_RES = 640          # качество модели (256–640)
MIN_VIEWS = 3          # сколько камер должны видеть воксель
SMOOTH_ITERS = 5       # сглаживание
PAD_SCALE = 0.15       # запас вокруг объекта
```

### `main_box.py`

```python
FRAME_STRIDE = 2       # пропуск кадров
DOWNSCALE = 0.6        # ускорение
FOV_DEG = 70.0         # угол обзора камеры
```

---

## ⚠️ Частые проблемы

**Модель пустая**

* плохие маски
* несовпадение имён файлов
* мало камер в COLMAP

**Куб слишком грубый**

* увеличь `VOX_RES`
* добавь кадров

**Цвета серые**

* объект не попадает в маски
* увеличь `MAX_COLOR_VIEWS`

---

## 🚀 Для чего подходит

* Computer Vision / CV-проекты
* 3D-реконструкция объектов
* Исследование SfM + Visual Hull
* Прототипы для Digital Twin / BIM

---

Если хочешь, дальше могу:

* ✨ упростить пайплайн
* 📦 сделать Docker
* 📄 добавить схему архитектуры
* 🎯 адаптировать под **любой цвет / объект**

Просто скажи 👍
