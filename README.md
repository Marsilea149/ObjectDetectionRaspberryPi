# ObjectDetectionRaspberryPi
Using Raspberry Pi and a USB camera to perform object detection.

## Hardware 


## Software Installation
Make sure you get the latest updates:
```
sudo apt update
sudo apt upgrade
```
Reboot your raspberry pi.
Install necessaey package:
```
sudo apt-get install libcblas-dev
sudo apt-get install libhdf5-dev
sudo apt-get install libhdf5-serial-dev
sudo apt-get install libatlas-base-dev
sudo apt-get install libjasper-dev 
sudo apt-get install libqtgui4 
sudo apt-get install libqt4-test
```
Install pip to manage your virtual environments, it helps you for package management.
```
python3 -m pip install --user --upgrade pip
python3 -m pip --version
python3 -m pip install --user virtualenv

```
Create a new python virtual environment named 'tflite_env':
```
python3 -m venv tflite_env
```
Activate your virtual environment
```
source tflite_env/bin/activate
```
Double check the python interpreter that you are using:
```
which python
```
It should return: <Your_working_directory>/tflite_env/bin/python

### Take apicture with your camera 
fswebcam is a small and simple webcam app for *nix. It can capture images from a number of different sources and perform simple manipulation on the captured image. The image can be saved as one or more PNG or JPEG files.
```
sudo apt install fswebcam
```
The v4l-utils are a series of packages for handling media devices.
```
sudo apt-get install v4l-utils
```
See which device is your camera
```
v4l2-ctl --list-devices
```
In my case, the previous command returned:
```
USB 2.0 Camera: USB 2.0 Camera (usb-0000:01:00.0-1.2):
	/dev/video1
	/dev/video2
	/dev/media4
```
So, the camera device is: /dev/video1

Use fswebcam to take a picture and save the image
```
fswebcam -d /dev/video1 my_image.jpg
```
To find your camera's resolution:
```
v4l2-ctl --list-formats-ext -d /dev/video1
```
For me, it returned: 
```
ioctl: VIDIOC_ENUM_FMT
	Type: Video Capture

	[0]: 'YUYV' (YUYV 4:2:2)
		Size: Discrete 640x480
			Interval: Discrete 0.033s (30.000 fps)
		Size: Discrete 352x288
			Interval: Discrete 0.033s (30.000 fps)
		Size: Discrete 320x240
			Interval: Discrete 0.033s (30.000 fps)
		Size: Discrete 160x120
			Interval: Discrete 0.033s (30.000 fps)
		Size: Discrete 1280x720
			Interval: Discrete 0.033s (30.000 fps)
		Size: Discrete 1280x960
			Interval: Discrete 0.033s (30.000 fps)
	[1]: 'MJPG' (Motion-JPEG, compressed)
		Size: Discrete 640x480
			Interval: Discrete 0.033s (30.000 fps)
		Size: Discrete 352x288
			Interval: Discrete 0.033s (30.000 fps)
		Size: Discrete 320x240
			Interval: Discrete 0.033s (30.000 fps)
		Size: Discrete 160x120
			Interval: Discrete 0.033s (30.000 fps)
		Size: Discrete 1280x720
			Interval: Discrete 0.033s (30.000 fps)
		Size: Discrete 1280x960
			Interval: Discrete 0.033s (30.000 fps)
```
To chose the resolution of your image
```
fswebcam -d /dev/video1 -r 1280x720 my_image.jpg
```
### Streaming with your camera using python
First activate your python virtual environment
```
cd ~/projects
source tflite_env/bin/activate
cd ~/projects/ObjectDetectionRaspberryPi
```
Create a new pythonscript named 'retrieve_image.py', with this content:
```
import cv2

#index is the video device number of the camera 
index = 1
cam = cv2.VideoCapture(index)

while True:
	ret, image = cam.read()
	cv2.imshow('Imagetest',image)
	k = cv2.waitKey(1)
	if k != -1:
		break
cv2.imwrite('/home/pi/testimage.jpg', image)
cam.release()
cv2.destroyAllWindows()
```
Make this script executable:
```
chmod +x retrieve_image.py 
```
Run script:
```
python retrieve_image.py
```
### Run existing Tensorflow Lite object detection example

Go to your working directory, and clone the official github repository:
```
cd ~/projects
git clone https://github.com/tensorflow/examples --depth 1
```

```
cd ~/projects/examples/lite/examples/object_detection/raspberry_pi
```

Run this script to install the Python dependencies, and download the TFLite models. 
```
sh setup.sh
```
When directly running the example code, I encountered this error:

> ImportError: /lib/arm-linux-gnueabihf/libstdc++.so.6: version `GLIBCXX_3.4.29' not found (required by /home/pi/projects/tflite_env/lib/python3.9/site-packages/tensorflow_lite_support/metadata/cc/python/_pywrap_metadata_version.so)

To solve it, I installed tflite-support directly with pip:
```
python -m pip install --upgrade tflite-support==0.4.3
```

Run the example code, remeber to update the cameraId with your device index :
```
python3 detect.py \
  --model efficientdet_lite0.tflite --cameraId 1
```
You can run in debug mode with VSCode, you have to add a launch.json file:
```
{
    // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Python: detect.py",
            "type": "python",
            "request": "launch",
            // "program": "${file}",
            "program": "${workspaceFolder}/detect.py",
            "console": "integratedTerminal",
            "justMyCode": false,
            "args" : [
                "--cameraId", "1",
                "--model", "efficientdet_lite0.tflite"
            ]
        }
    ]
}
```
When running the script, you will see: ??? image ???

In this code, the detection result is retrieved with:
```
detection_result = detector.detect(input_tensor)
```
This is an example of detection result:
```
DetectionResult(detections=[Detection(bounding_box=BoundingBox(origin_x=9, origin_y=-1, width=577, height=483), categories=[Category(index=0, score=0.5, display_name='', category_name='person')]), Detection(bounding_box=BoundingBox(origin_x=3, origin_y=8, width=377, height=333), categories=[Category(index=0, score=0.3125, display_name='', category_name='person')])])
```

#TODO: extract detection result, extract the distance of the tennis ball using the bounding box.  