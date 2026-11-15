# RPB2023

## Vision-Based Rotation Commands with ROS

Team 3 coursework repository for **Introduction to Mechanical System Design and Robot Programming**, Seoul National University, Spring 2023.

The main project classifies the background color of incoming images and publishes clockwise, counterclockwise, or stop commands through ROS. It extends a course-provided Python/ROS skeleton with color-decision logic for a propeller rotation task. The repository also contains separate exercises in model fitting, clustering, and ROS communication.

**Core tools:** Python, OpenCV, NumPy, ROS 1, `cv_bridge`  
**Additional exercises:** MATLAB, Matplotlib, RANSAC-style fitting, K-means

## Execution Experience

The team ran the image-classification and ROS command-publishing implementation during the Spring 2023 course. This repository records code that was used in the course environment, rather than a proposal or an unexecuted design.

The original execution is distinct from the current documentation update, which did not rerun the ROS environment. No numerical accuracy, latency, or RPM-control performance is claimed here.

## Project Context

The course introduced mechanical design alongside programming, image processing, and robot communication. For the vision task, the teaching materials supplied the ROS subscription/publication structure and defined the interface to the motor-side system. Teams were asked to determine the image background color and complete the decision logic.

This repository preserves the software portion of that coursework. The motor-side subscriber, driver, CAD files, demonstration footage, and evaluation results are not included.

## Main Pipeline

**ROS image → BGR conversion → 50 × 30 image → pixel-color counts → direction decision → ROS command**

In [`skeleton.py`](skeleton.py):

1. Subscribe to `/camera/color/image_raw` as `sensor_msgs/Image`.
2. Convert each image to OpenCV `bgr8` format using `CvBridge`.
3. Resize it to **50 × 30 pixels** for classification.
4. Count pixels in blue, red, green, white, yellow, magenta, and cyan categories using fixed channel thresholds of **150**.
5. Add cyan pixels to the blue count, then compare blue, red, green, yellow, magenta, and white counts.
6. Set `Header.frame_id` and publish the command on `/rotate_cmd`.

### Command Interface

| Direction | Topic | Message type | Purpose |
| --- | --- | --- | --- |
| Input | `/camera/color/image_raw` | `sensor_msgs/Image` | Image stream from a camera publisher or ROS bag |
| Output | `/rotate_cmd` | `std_msgs/Header` | Original image header with `frame_id` set to the command |

| Decision in `skeleton.py` | `frame_id` | Intended motor action |
| --- | --- | --- |
| Blue + cyan count equals the largest count | `"+1"` | Counterclockwise |
| Red count equals the largest count, after the blue check fails | `"-1"` | Clockwise |
| Neither condition above is met | `"0"` | Stop |

The incoming header timestamp is retained for the course's timestamp-based evaluation. Using `Header.frame_id` for commands is a **course-specific convention**.

**Decision details:** Comparisons use strict `>` and `<` thresholds, so channel values exactly equal to 150 are excluded from those categories. The blue branch takes precedence in ties. If all counts are zero, the current implementation also selects `"+1"`; an unrecognized or dark image is therefore not guaranteed to produce a stop command.

## Repository Guide

| File | Role | Notes |
| --- | --- | --- |
| [`skeleton.py`](skeleton.py) | Main course-task implementation | Fixed-threshold pixel counting and ROS direction-command publishing |
| [`skeleton2.py`](skeleton2.py) | Alternative color-classification experiment | Distances to six reference BGR colors; currently uses `np` without importing NumPy |
| [`RANSAC.py`](RANSAC.py) | Random-sample line-fitting exercise | Reads external `RANSAC_data.csv`; uses an inlier threshold, then selects eligible models by total squared distance |
| [`RANSAC_quad.py`](RANSAC_quad.py) | Quadratic RANSAC-style fitting exercise | Reads an external CSV from a hard-coded absolute path; selects by inlier count |
| [`my_kmeans.m`](my_kmeans.m) | MATLAB clustering exercise | Two clusters with city-block distance; requires external `data.txt` and Statistics and Machine Learning Toolbox |
| [`main.m`](main.m) | MATLAB data-loading/plotting exercise | Loads `X1` and `X2` but plots undefined `X`; requires correction and external data |
| [`calc.py`](calc.py), [`divide.py`](divide.py) | Python fundamentals | Division and zero-division handling exercises |
| [`ros/talker.py`](ros/talker.py), [`ros/listener.py`](ros/listener.py) | ROS publisher/subscriber exercises | `std_msgs/String` messages on `chatter`; separate from the vision pipeline |

The fitting and clustering exercises are **not connected to the color-classification node**. The main pipeline does not use K-means, RANSAC, or a trained model to classify images.

## Running the Main Node

### Requirements

- An existing **ROS 1** environment with Python 3, `rospy`, `sensor_msgs`, `std_msgs`, and `cv_bridge`.
- OpenCV (`cv2`) and NumPy available to the same Python interpreter.
- A desktop/display session for `cv2.imshow`.
- A camera publisher or course-provided ROS bag containing `/camera/color/image_raw`.

The original ROS distribution and package versions are not recorded in this repository. It does not include a ROS package manifest, launch file, or the course bag files. The sequence below follows the course workflow; full ROS/bag/hardware execution has not been revalidated for this documentation update.

### Playback Workflow

Clone the repository and enter its directory:

```bash
git clone https://github.com/MinjaeIdle/RPB2023.git
cd RPB2023
```

Source your ROS 1 environment in each terminal before running the following commands.

**Terminal 1 — Start the ROS master:**

```bash
roscore
```

**Terminal 2 — Run the classifier from the repository directory:**

```bash
python3 skeleton.py
```

**Terminal 3 — Observe published commands:**

```bash
rostopic echo /rotate_cmd
```

**Terminal 4 — Play an externally supplied bag:**

```bash
# Replace this path with the location of your course bag.
rosbag play /path/to/demo01.bag
```

Use `rostopic list` and `rostopic info /camera/color/image_raw` to check that the expected image topic is available. The bag filename comes from the course tutorial; the data must be obtained separately. A physical camera is optional for bag playback. Physical propeller operation additionally requires the external motor-side system.

## Scope and Limitations

- **Direction commands:** The published values encode rotation direction or stop. They do not specify a target RPM, PWM duty cycle, or thrust.
- **No RPM feedback loop in this repository:** There is no measured-speed input or speed-error controller in the main node.
- **Color robustness:** Fixed BGR thresholds and resizing are implemented, but no explicit denoising filter, illumination calibration, confidence threshold, or temporal smoothing is present in `skeleton.py`.
- **Evaluation:** Accuracy, latency, noise robustness, motor response, and competition ranking are not documented by the available repository files.
- **Reproducibility:** Bag files and fitting/clustering datasets are absent. The alternative scripts include the import, path, and variable issues noted above. The line-fitting exercise also needs handling for vertical samples and cases where no eligible fit is found.

## Teamwork and Attribution

This is **Team 3's course repository**. The ROS skeleton and command interface originated in the teaching materials; the completed classification logic is preserved here. The available materials do not establish each team member's individual responsibility, so this README describes repository-level work without assigning individual authorship.

Course references used to document the project include the Spring 2023 course introduction, the Week 5 programming overview, and **Python CV Practice** (`05-tur-python-cv.pdf`, dated April 25, 2023), especially its image-input, command-output, and hardware-check instructions. These references describe the assignment; they are not evidence of this team's measured performance.

## 한국어 요약

2023년 서울대학교 기계시스템설계 및 로봇프로그래밍 기초 수업의 3조 저장소입니다. 제공된 ROS 골격 코드를 바탕으로 영상의 배경색을 판별하고 `/rotate_cmd`에 회전 방향·정지 명령을 발행하며, 당시 수업 환경에서 실제로 실행한 구현입니다. 별도의 RANSAC·K-means·ROS 통신 실습 코드도 포함되어 있습니다. 공개 코드의 핵심 범위는 색상 판별과 방향 명령 발행이며, RPM 피드백 제어나 정량 성능 측정은 포함되어 있지 않습니다. 이번 README 정리 과정에서 ROS 실행을 새로 검증한 것은 아닙니다.
