# Installation
1. Follow tensorflow_installation.md to create a conda environment
2. Install model maker with a prebuilt pip package. (https://www.tensorflow.org/lite/models/modify/model_maker)
```
pip install tflite-model-maker
pip install tflite-support
pip install pycocotools
```
I tried to run teh example code:
```
from tflite_model_maker.config import QuantizationConfig
```
I encountered the following error, the issue seems to be the version of numpy: 
```
AttributeError: module 'numpy' has no attribute 'object'.
`np.object` was a deprecated alias for the builtin `object`. To avoid this error in existing code, use `object` by itself. Doing this will not modify any behavior and is safe. 
The aliases was originally deprecated in NumPy 1.20; for more details and guidance see the original release note at:
    https://numpy.org/devdocs/release/1.20.0-notes.html#deprecations
```
Solution:
```
pip install numpy==1.23.4
```
I tried to run 
```
train_data, validation_data, test_data = object_detector.DataLoader.from_csv(
    'gs://cloud-ml-data/img/openimage/csv/salads_ml_use.csv')
```
I encountered this error: 
```
All attempts to get a Google authentication bearer token failed, returning an empty token. Retrieving token from files failed with "NOT_FOUND: Could not locate the credentials file.". Retrieving token from GCE failed with "FAILED_PRECONDITION: Error executing an HTTP request: libcurl code 6 meaning 'Couldn't resolve host name', error details: Could not resolve host: metadata".
```

The issue seems to be that I cannot access to google cloud. 
#TODO: install gsutils: https://cloud.google.com/storage/docs/gsutil_install
#TODO: copy the csv to a local folder and try again
It was really hard to get csv from google cloud, finally, I prefered download a dataset myself. 

The code works:
```
from absl import logging
import numpy as np
import os

from tflite_model_maker.config import QuantizationConfig
from tflite_model_maker.config import ExportFormat
from tflite_model_maker import model_spec
from tflite_model_maker import object_detector

import tensorflow as tf
assert tf.__version__.startswith('2')

tf.get_logger().setLevel('ERROR')
logging.set_verbosity(logging.ERROR)


spec = model_spec.get('efficientdet_lite0')

train_data = object_detector.DataLoader.from_pascal_voc(
    'android_figurine/train',
    'android_figurine/train',
    ['android', 'pig_android']
)

val_data = object_detector.DataLoader.from_pascal_voc(
    'android_figurine/validate',
    'android_figurine/validate',
    ['android', 'pig_android']
)

model = object_detector.create(train_data, model_spec=spec, batch_size=4,
                               train_whole_model=True, epochs=20, validation_data=val_data)

model.evaluate(val_data)

model.export(export_dir='.', tflite_filename='android.tflite')
model.evaluate_tflite('android.tflite', val_data)

print("end")

```











# Testing existing classification code in ubuntu
I tried to run the following code in ubuntu:
```
```
I encountered error:
```
AttributeError: partially initialized module 'cv2' has no attribute 'gapi_wip_gst_GStreamerPipeline' (most likely due to a circular import)
```
Solution: 
```
pip install opencv-python==4.5.5.64
```
