# Kinect live processing for industrial safety applications &middot; [![GitHub license](https://img.shields.io/badge/license-MIT-blue.svg?style=flat-square)](https://github.com/SebastianPartarrieu/live-kinect/blob/master/LICENSE)
To help the industrial adoption of low-cost sensors for safety applications, this repository provides a proof-of-concept of person segmentation with depth estimation and point cloud rendering using the Kinect v2 sensor.

https://user-images.githubusercontent.com/55084624/215078439-82810ded-6bb9-43dc-a9d7-ad8161a8f7d0.mp4


The Kinect v2 provides RGB + IR information: we use the RGB to perform live person segmentation using either (i) the best performing models available on hugging face to benchmark or (ii) openvino lightweight implementations that can run on an intel CPU. Once the segmentation masks are acquired we can use these to extract the humans from the Kinect depth maps and then estimate the distance of each human to the sensor. Finally, after the depth estimation, we can render 3D point clouds of each visible human. This workflow could then be fed downstream to a navigating robot using the input data to update its trajectory.

[![Live person instance segmenation](https://img.youtube.com/vi/ZrdNq9Bg5i8/0.jpg)](https://www.youtube.com/watch?v=ZrdNq9Bg5i8)

> Disclaimer: There are both experimental and more mature scripts within the repository.

## Installation

### Getting started

Depending on what your use case is, there are different installation options available. Be sure to check the system requirements to see what was used to build this package and run the code. TLDR; As long as you're on linux, you should be fine to follow the instructions.

Install [openvino_notebooks](https://github.com/openvinotoolkit/openvino_notebooks/wiki/Conda#step-4-install-the-packages) by following the linked instructions. It will be easier to follow the rest if you create a conda environment as explained and install the correct packages using requirement_openvino.txt file.

If you intend to process the Kinectv2 data flow, you will need to install specific libraries (CAREFUL, the installation guide provided below works only with Linux in our experience):
- libfreenect2 : follow the instructions here : https://github.com/OpenKinect/libfreenect2
- freenect2-python : python wrapper for libfreenect2, allows a more pythonic way of processing the kinect v2 frames. See https://rjw57.github.io/freenect2-python/.
> Careful: for the freenect2-python installation, you don't need to reinstall libfreenect2, you just need to activate your favorite conda environment, set the system environment variable and then install the python wrapper.
```
conda activate your_openvino_convda_env
export PKG_CONFIG_PATH=$HOME/freenect2/lib/pkgconfig
pip install --user freenect2
```
You will probably still run into some issues even after following these installation guides when trying to run ```kinect_real_time.py``` or other scripts using libfreenect2. It seems there are some path issues that were [fixed](https://github.com/rjw57/freenect2-python/issues/6) by creating a symbolic link like so:
```
sudo ln -s $HOME/freenect2/lib/libfreenect2.so.0.2 /usr/lib/libfreenect2.so.0.2
```
And if you run into segmentation faults you will most likely need to go and modify the ```__init__.py``` file of freenect2 which is probably somewhere around ```/home/user/.local/lib/python3.8/site-packages/freenect2/__init__.py```, go to line 100 or so and change the ```__call__``` function of the QueueFrameListener class to the following
```
def __call__(self, frame_type, frame):
    if self.queue.qsize() >= 12:
        _ = self.get()
    self.queue.put_nowait((frame_type, frame))
```
This should ensure you don't get a segmentation fault. You can set the dynamic max queue size as you wish as long as it is under the max_queue_size define in the __init__ function of the class.

Finally, you will need the latest version of open3d. One way to install it is with pip :
```
pip install open3d>=0.16*
```

### Repository structure
```
.
├── docs
|   ├── Mareva mini-projet 2022-2023 slideshow.pdf
|   ├── Mareva mini-projet.pdf
├── models
|   ├── model-segmentation
|      ├── instance-segmentation-person-0007
|          ├── FP16
|             ├── instance-segmentation-person-0007.bin
|             └── instance-segmentation-person-0007.xml
|          └── FP32
|             ├── instance-segmentation-person-0007.bin
|             └── instance-segmentation-person-0007.xml
├── kinect_real_time.py
├── utils.py
├── point_cloud_to_video.py
├── live-segmentation-cpu.ipynb
├── swit-vit-segmentation-webcam.ipynb
```

### Details about files

- Mareva mini-projet 2022-2023 slideshow.pdf & Mareva mini-projet.pdf: details of the initial project ambitions, from Petr DOKLADAL who supervised the project.

- kinect_real_time.py : get real time data from Kinect v2 sensor, apply human instance segmentation framework (Openvino librairy), render human segmentation on the RGB frame, coupled with the distance between the camera and each human (evaluated with the depth channel). 3D point clouds are also rendered in real time, with only the segmented humans in them.

- utils.py : helper functions to process each RGB frame (apply model, calculate distances, show bounding boxes with confidence score on the detection).

- point_cloud_to_video.py : generate video based on .pcd files, saved after running kinect_real_time. With this script, you can retrieve the point cloud visualization that was not saved during real time processing (although shown on screen).

- live-segmentation-cpu.ipynb: notebook to use the openvino instance segmentation model on data coming from the webcam.

- swit-vit-segmentation-webcam.ipynb: notebook using swin vision transformers for instance segmentation on prerecorded RGB-D images as well as streamed data using the webcam.

### System requirements
- OS: elementary OS 5.1.7 Hera x86_64 (anything based on Ubuntu > 14.04 should work fine)
- CPU: Intel i5-9300H (8) @ 4.100GHz (as long as its Intel, you're good!)
- python 3.8 (see package requirements file)

## Developing

### Future directions
A number of future directions can be considered for this project.
- Adding better postprocessing to the pointclouds
- Continue optimizing the tradeoff between real-time rendering and accuracy in person segmentation
- Feed the pointclouds to an 'autonomous' robot or industrial application using the input data

### Configuration - What to change and where to change it
If you want to run this on your own kinect and make improvements, you will mainly want to work on the ```kinect_real_time.py script```. The code should be fairly self explanatory, however it does use lots of functions from opencv and open3d, so make sure you go through some of their tutorials to have an in-depth understanding of how we are using their functions here. If you want to save the visualizations, make sure to uncomment the code at the end of the script, future changes to this repository will make this more configurable.

## Authors
Emma Bou Hanna & Sebastian Partarrieu
