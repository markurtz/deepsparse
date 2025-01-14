<!--
Copyright (c) 2021 - present / Neuralmagic, Inc. All Rights Reserved.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

   http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing,
software distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
-->

# Ultralytics/yolov3-DeepSparse Inference Examples
This directory contains examples of benchmarking, annotating, and serving inferences
of YOLOv3 models from the [ultralytics/yolov3](https://github.com/ultralytics/yolov5)
repository using the DeepSparse Engine. The DeepSparse Engine is able to achieve
[real-time inferencing of YOLOv3 on CPUs](https://neuralmagic.com/blog/benchmark-yolov3-on-cpus-with-deepsparse/)
by leveraging pruned and quantized YOLOv3 models. These examples can load pre-trained,
sparsified models from [SparseZoo](https://github.com/neuralmagic/sparsezoo) or you can
create your own using the 
[Sparseml-Ultralytics-yolov3 integration](https://github.com/neuralmagic/sparseml/blob/main/integrations/ultralytics-yolov3/README.md).

## Installation
The dependencies for this example can be installed using `pip`:
```bash
pip3 install deepsparse sparseml[torchvision] flask flask-cors
```

## Annotation Example
`annotate.py` is a script for using YOLOv3 sparsified (and not sparsified) YOLOv3 models
to run inferences on images, videos, or webcam streams. For a full list of options
`python annotate.py -h`.

To run pruned-quantized YOLOv3 on a local webcam run:
```bash
python annotate.py \
    zoo:cv/detection/yolo_v3-spp/pytorch/ultralytics/coco/pruned_quant-aggressive_94 \
    --source 0 \
    --quantized-inputs \
    --image-shape 416 416 \
    --no-save  # webcam only
```

In addition to webcam `--source` can take a path to a `.jpg` file, directory or glog pat
of `.jpg` files, or path to a `.mp4` video file.  If source is an integer and no
corresponding webcam is available, an exception will be raised.


## Benchmarking Example
`benchmarking.py` is a script for benchmarking sparsified and quantized YOLOv3
performance with DeepSparse.  For a full list of options run `python benchmarking.py -h`.

To run a benchmark run:
```bash
python benchmark.py \
    zoo:cv/detection/yolo_v3-spp/pytorch/ultralytics/coco/pruned_quant-aggressive_94 \
    --batch-size 1 \
    --quantized-inputs
```

Note for quantized performance, your CPU must support VNNI instructions.



# Example YOLO DeepSparse Flask Server

To illustrate how the DeepSparse Engine can be used for YOLO model deployments, this directory
contains a sample model server and client. 

The server uses Flask to create an app with the DeepSparse Engine hosting a
compiled YOLOv3 model.
The client can make requests into the server returning object detection results for given images.

### Server

First, start up the host `server.py` with your model of choice, SparseZoo stubs are
also supported.

Example command:
```bash
python server.py \
    zoo:cv/detection/yolo_v3-spp/pytorch/ultralytics/coco/pruned_quant-aggressive_94 \
    --quantized-inputs
```

You can leave that running as a detached process or in a spare terminal.

This starts a Flask app with the DeepSparse Engine as the inference backend, accessible at `http://0.0.0.0:5543` by default.

The app exposes HTTP endpoints at:
- `/info` to get information about the compiled model
- `/predict` to send images to the model and receive as detected in response.
    The number of images should match the compiled model's batch size.

For a full list of options, run `python server.py -h`.

Currently, the server is set to do pre-processing for the yolov3-spp
model; if other models are used, the image shape, output shapes, and
anchor grids should be updated. 

### Client

`client.py` provides a `YoloDetectionClient` object to make requests to the server easy.
The file is self-documented.  See example usage below:

```python
from client import YoloDetectionClient

remote_model = YoloDetectionClient()
image_path = "/PATH/TO/EXAMPLE/IMAGE.jpg"

model_outputs = remote_model.detect(image_path)
```
