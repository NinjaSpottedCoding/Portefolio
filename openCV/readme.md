---
title: "OpenCV"
permalink: "/opencv"
layout: default
---


## OpenCV

OpenCV is an open-source computer vision and machine learning software library, aiming at providing a common infrastructure for accelerating the design of CV applications. OpenCV library provide basic tools for building CV applications, building on over 2500 optimized algorithms, including state-of-the-art computer vision and machine learning algorithms for a panoply of applications. Detecting and recognizing faces, classifying human actions in videos, track camera movements and moving objects, extracting 3D models of objects, producing 3D point clouds from stereo cameras, removing red eyes photos, following eye movements are just a few examples.

### Introduction

This document aims at introducing the concept of Computer Vision and some basic functionality of the OpenCV – Open Source Computer Vision Library, through a few illustrative examples building on its C++ libraries.  
In broad terms, Computer Vision comprises data acquisition (involving still images or video stream, sometimes mixed with audio), data compression, processing and classification, towards a (computer-based) decision or a new representation of the data. Note that Computer Vision relies on a binary representation of the data.  
\
OpenCV has C++, Python, Java and MATLAB interfaces and supports Windows, Linux, Android and Mac OS. In this case, C++ was opted to create the colour histogram of an image.

### Instalation

The installation is based on Debian 9 (Stretch), but should work in any other Debian-based OS. Firstly, do:

        sudo apt update
        sudo apt install build-essential
        sudo apt install cmake git libgtk2.0-dev pkg-config libavcodec-dev libavformat-dev libswscale-dev
        sudo apt install libopencv-dev python3-opencv
        
We also need to compile the library from source:  
`wget -O opencv.zip https://github.com/opencv/opencv/archive/4.x.zip`  
Then unzip it and create a build directory, and inside do:
        
        mkdir build
        cd build
        cmake -D OPENCV_GENERATE_PKGCONFIG=YES "../opencv-4.x/"
        make -j$(nproc)
        sudo make install
        sudo ldconfig

When compiling the C++ programs use: 

        g++ test.cpp -o teste `pkg-config --cflags --libs opencv4`

### Example 1: Loading an image file and visualizing it on screen

For this the program loads an image through a path and opens a window named `program1`, outputting the image:

        #include “./opencv-4.x/include/opencv2/opencv.hpp”
        #include <iostream>
        
        using namespace cv;
        
        int main(int argc, char ** argv) {
          Mat img = imread(argv[1], -1);
          if (img.empty()) return -1;
          namedWindow("Program1", cv::WINDOW_AUTOSIZE);
          imshow("Program1", img);
          waitKey(0);
          destroyWindow("Program1");
          return 0;
        }
        
The image is loaded for the "img" Mat-type structure, using the `imread()` function, on the line `Mat img = imread(argv[1],-1)`. This is a high-level function that automatically assigns the file type, according to the file extension (the file name is input as the first argument (argv[1]) and allocates the required memory space (for the image data structure). The second argument enables to define the image format: "-1" for
unchanged (as is); "0" for grayscale; "1" for colour). In this case ("-1"), the input image is not modified. The `imread()` function returns a pointer that will be used for manipulating the image and the corresponding data.  
The `NamedWindow()` function opens a new window with the name on first argument and defines the properties of the window: `cv::WINDOW_AUTOSIZE` will open the window with the same dimentions as the image.  
The `waitKey()` function pauses the execution of the program, until the user presses a key. The argument defines the "waiting deadline" (ms), that since it is "0", it waits indefinitely.  
Finally, the `destroyWindow()` function closes the window and frees the memory.

### Example 2: Loading a video file and visualizing it on screen

Loading a video is very similar to loading an image, but we need a loop to read each frame of the stream. The example builds on the "highgui" and "imgproc" libraries:

        #include "./opencv-4.x/modules/highgui/include/opencv2/highgui.hpp"  
        #include "./opencv-4.x/modules/imgproc/include/opencv2/imgproc.hpp"

        int main(int argc, char ** argv) {
                cv::namedWindow("Program2", cv::WINDOW_AUTOSIZE);
                cv::VideoCapture cap;
                cap.open(argv[1]);
                cv::Mat frame;
                for (;;) {
                        cap >> frame;
                        if (frame.empty()) break;
                        cv::imshow("Program2", frame);
                        if ((char)cv::waitKey(33) >= 0) break;
                }
        return 0;
        }
        
The `cv::VideoCapture cap` creates an instance (`cap` object) of the `cv::VideoCapture` class, which will store all  data from the input file.  
The `for` loop enables the storage of every fream in the structure and outputs to the screen. The `cap >> frame` instruction stores the current "reading" status of `cvCapture`.  
\
After each frame, there is a waiting time of 33 milliseconds, for a 30 fps rate.

### Example 3: Selecting a specific video frame and visualizing it on screen

For this we will have a slider that will control the frame output on the screen:

        #include “opencv2/highgui/highgui.hpp"
        #include “opencv2/imgproc/imgproc.hpp"
        #include <iostream>
        #include <fstream>
        using namespace std;
        
        int g_slider_position = 0;
        int g_run = 1; // = 1 is “Single Step” mode, 1 frame at a time
        int g_dontset = 0;
        cv::VideoCapture g_cap;
        
        void onTrackbarSlide(int pos, void * ) {
                g_cap.set(cv::CAP_PROP_POS_FRAMES, pos);
                if (!g_dontset) g_run = 1;
                g_dontset = 0;
        }
        
        int main(int argc, char ** argv) {
                cv::namedWindow("Program3", cv::WINDOW_AUTOSIZE);
                g_cap.open(argv[1]);
                int frames = (int) g_cap.get(cv::CAP_PROP_FRAME_COUNT);
                int tmpw = (int) g_cap.get(cv::CAP_PROP_FRAME_WIDTH);
                int tmph = (int) g_cap.get(cv::CAP_PROP_FRAME_HEIGHT);
                cout << "Video has " << frames << " frames of dimensions(" << tmpw << ", " << tmph << ")." << endl;
                cv::createTrackbar("Position", "Program3", & g_slider_position, frames, onTrackbarSlide);
                cv::Mat frame;
                for (;;) {
                        if (g_run != 0) {
                                g_cap >> frame;
                                if (frame.empty()) break;
                                int current_pos = (int) g_cap.get(cv::CAP_PROP_POS_FRAMES);
                                g_dontset = 1;
                                cv::setTrackbarPos("Position", "Program3", current_pos);
                                cv::imshow("Program3", frame);
                                g_run -= 1;
                        }
                        char c = (char) cv::waitKey(10);
                        if (c == 's') {
                                g_run = 1;
                                cout << "Single step, run = " << g_run << endl;
                        }
                        if (c == 'r') {
                                g_run = -1;
                                cout << "Run mode, run = " << g_run << endl;
                        }
                        if (c == 27) break;
                }
                
        return(0);
        }