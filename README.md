# Object-Detection-Tflite-QualcommRB5-CPU
This is a page that serves as an extra addition to [this tutorial by Mr. Trong](https://github.com/DoanNguyenTrong/object-detection-tflite-cpp) to make it easier to follow along. His tutorial deals with setting up an environment to deploy a neural network on the Qualcomm RB5, based off of Qualcomm's official tutorial [here](https://developer.qualcomm.com/project/object-detection-tensorflow-lite). Several updates have occurred to opencv and tensorflow though, and I believe it is prudent to note a couple of these changes and make the installation process more straightforward.  
  
Think of this as a supplementary note page to Mr. Trong's object-detection set-up. This is more of an additional guide or resource to assist some people in installing the setup onto the RB5 development kit, utilizing Mr. Trong's code, since there can be unexpected trouble popping up, which this can hopefully simplify. Extra note: This tutorial has not found out how to perfeclty replicate a live video feed, or have it run on the GPU or hexagon delegate, and any additional pointers, changes, or steps anyone can note would be appreciated!  

## Tensorflow Installation Notes  
As of branch r2.7, tensorflow no longer uses Make, but rather CMake. This workaround to get the object detection still utilizes the make file (following from the tutorial), and is still rather outdated as such. All these Qualcomm RB5 tutorials use older branches that involve the older versions of Make, and as such this tutorial will also be clarifying how to use the Makefile to build. However, if newer versions of tensorflow are needed or desired, consider looking towards another tutorial involving CMake (which will have to be updated as the RB5 uses Ubuntu 18.04, which has an incompatible version of CMake 3.10 to tensorflow) to build the tensorflow library (official instructions are [here](https://www.tensorflow.org/lite/guide/build_cmake)).  
  
Follow the steps on Mr. Trong's guide for tensorflow installation. One thing that should be noted is that the branch that should be checked out should be r2.5, as it is the most up to date one the guide works on. Minor testing revealed runtime errors that involved segmentation faults and messed up memory allocation checking out other versions (r2.4 and r2.6 for these errors respectively).  
That is to say, check out the r2.5 branch instead of r2.1 doing the following code below after cloning tensorflow:
```
git checkout r2.5
```

## OpenCV Installation Notes  
The latest versions of OpenCV have Wayland support built in. As such, if using Wayland, the old tutorial (listed [here](https://github.com/pfpacket/opencv-wayland.git) is not needed. The latest version of OpenCV should just be installed with necessary flags to build with Wayland support. As such, following the instructions to install OpenCV (using the zip packs) without worrying about open cv-wayland will suffice.   

Additional Note: Making OpenCV may use too much memory on the board, so feel free to use make instead of make -j7 to counter this (code referenced below).  
```
cmake -DCMAKE_INSTALL_PREFIX=/usr/local -DOPENCV_EXTRA_MODULES_PATH=../../opencv_contrib-master/modules -DWITH_IPP=OFF -DWITH_GSTREAMER=ON -DWITH_GTK=OFF -DWITH_WAYLAND=ON -DBUILD_opencv_python3=yes ..

# use make instead of make -j7 if board is unable to handle building in parallel
make -j7 && make install
```

## Building  
Although Mr. Trong mentions needing bazel to compile any missing libraries (which may be the case for other versions of tensorflow like r2.2, r2.3, etc.), if you have checked out the version r2.5 like above, no additional libraries should be needed to be compiled. Simply running the rest of the commands will suffice to build and run the application (shown below).
```
git clone https://github.com/DoanNguyenTrong/object-detection-tflite-cpp.git
cd object-detection-tflite-cpp/
./compile.sh
./object-detector 0 dev/video0
```

One thing of note should be running the object-detector app. The API is explained by Mr. Trong, but a general note to add is that following the guide with the above listed process will give some limited functionality. Although all delegates are installed, following the tutorial above exactly will mean these delegates will not run properly, with the Hexagon DSP crashing, and the GPU delegate failing certain features and falling back onto the CPU to run. There is also no displayed live feed, but the object-detection app will still record and run onto a video that can be pulled via adb to be examined. An additional point to note is that if using a USB webcam, it will usually connect to /dev/video2 (you can check via dmesg or looking at the /dev and seeing which device disappears and reappears when plugging/unplugging the camera), a consideration that should be had when passing parameters and running the app. Otherwise, the other onboard cameras can be accessed using /dev/video0 and /dev/video1 if those are the cameras being used.
