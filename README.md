Many thanks to Dima Rekesh, PhD who is the original author of these builds and write-up.

### For initial setup instructions, visit the [wiki Home](https://github.com/open-horizon/cogwerx-jetson-tx2/wiki)

### Docker build instructions and files for deep learning container images
* Jetson TX2 (base)
* CUDA, CUDNN, OpenCV and supporting libs, full and lean variants
* Caffe deep learning framework
* Darknet deep learning framework with Yolo

These docker images are also available at the public bluehorizon docker hub [repo](https://hub.docker.com/u/openhorizon/) as part of the [Horizon](https://bluehorizon.network) project.



#### To build docker images (each successive image depends on the previous)

1. Clone this repo locally
2. Build jetson tx2 image. This base image is the prerequisite for later containers using CUDNN, OpenCV, and various deep learning frameworks

```
docker build -t openhorizon/jetson-tx2 .
```
3. Build additional interim images with CUDA, CUDNN, and OpenCV libs

#### Validating
To test the speed of AlexNet on the GPU:
```
docker run --privileged --rm openhorizon/caffe-tx2 /caffe/build/tools/caffe time --model=/caffe/models/bvlc_alexnet/deploy.prototxt --gpu=0
```
To do the same on the CPU:
```
docker run --privileged --rm openhorizon/caffe-tx2 /caffe/build/tools/caffe time --model=/caffe/models/bvlc_alexnet/deploy.prototxt
```

To run Digits:
```
docker run --privileged --name digits -p 5001:5001 -d openhorizon/digits-tx2
# Now you can connect to http://jetsonip:5001 using your browser 
```

To run Yolo:
```
# as root on host tx2 allow x, e.g. xhost +
# then, assuming you have a usb webcam hooked up:
# tiny yolo model against the VOC data set is the fastest (only 20 classes though)
docker run --privileged -e DISPLAY=$DISPLAY -v /tmp:/tmp --rm openhorizon/darknet-tx2 /darknet/darknet detector demo -c 1 cfg/voc.data cfg/tiny-yolo-voc.cfg tiny-yolo-voc.weights
# regular yolo model is much more accurate and against MS COCO 91 classes; however, 2x slower in my tests:
docker run --privileged -e DISPLAY=$DISPLAY -v /tmp:/tmp --rm openhorizon/darknet-tx2 /darknet/darknet detector demo -c 1 cfg/coco.data cfg/yolo.cfg yolo.weights
```

To run DustyNV's container:
```
# xhost + then
docker run --privileged -v /dev:/dev -e DISPLAY=$DISPLAY -v /tmp:/tmp --net=host --ipc=host --rm -ti openhorizon/dustyinference-tx2 bash -c "cd /jetson-inference/build/aarch64/bin && ./imagenet-camera"
```
