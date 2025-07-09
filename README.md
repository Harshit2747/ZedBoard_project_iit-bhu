# Drone ditection using zedboard(Work in Progress).

This project aims to build a real-time drone detection system using the ZedBoard (Zynq-7000 SoC) platform. It focuses on capturing live video from a camera, processing the video using OpenCV on the ARM Cortex-A9 (PS), and detecting moving drone-like objects based on motion.

# Current progress.

The main objective of this project is to develop a real-time drone detection system using the ZedBoard (Zynq-7000 SoC) by integrating image processing and video interfacing techniques on both hardware (PL) and software (PS) sides. To achieve this, the project was approached incrementally, moving from basic ZedBoard familiarity to full-fledged video processing.

# The key objectives and development steps are as follows:
Understanding ZedBoard Architecture:

.Studied the architecture of the Zynq-7000 SoC, focusing on the interaction between the Processing System (ARM Cortex-A9) and the Programmable Logic (FPGA).

.Implemented basic PS-PL projects such as LED blinking, GPIO control, and simple data communication to understand the AXI interface and IP integration.

.Initial Video Interfacing Using PL Only:

.Explored the fundamentals of video timing, frame buffers, and VGA interfacing.

.Implemented a basic VGA video output system using only the PL part of the ZedBoard to test and display RGB color patterns.

.This phase helped in understanding synchronization signals like HSync, VSync, and pixel clock.

.Video Interfacing Using PS-PL Co-Design:

.Progressed to more advanced designs using AXI VDMA (Video Direct Memory Access) to transfer frames from DDR memory (PS) to the video output hardware (PL).

.Integrated the ADV7511 HDMI transmitter using AXI IIC to configure and output high-definition video to an HDMI monitor.

.Verified HDMI and VGA video paths by displaying RGB color patterns and test images stored in memory.

.Image Processing and Photo Display:

.Implemented image processing steps such as grayscale conversion, thresholding, and edge detection either on the PS or using custom IPs in the PL.

.Displayed static test images (e.g., drone photos) through the video interface to verify the complete image pipeline.

.This confirmed that the board could successfully process and display image data in real-time.

# Towards Drone Detection:

These progressive stages established a strong foundation for real-time video capture, processing, and display.

The final objective is to integrate a detection algorithm that can recognize and highlight drones in the video stream based on shape, color, or edge features.

# Video Interfacing on ZedBoard (Completed)
As part of the drone detection system, the video interfacing and frame buffer setup have been successfully implemented on the ZedBoard (Zynq-7000) using Vivado 2018.2.

We have completed the hardware design for capturing and displaying video frames using the AXI VDMA, video timing controller, and Zynq PS. This design allows us to stream image frames from DDR memory to the VGA output, forming the foundation for later software-based drone detection using OpenCV.

# Vivado Block Design Summary
The current block design includes the following major components:

    | IP Block                                             | Function                                                                |
    | ---------------------------------------------------- | ----------------------------------------------------------------------- |
    | **Zynq7 Processing System**                          | Handles ARM Cortex-A9 (PS), DDR, and AXI interfaces.                    |
    | **AXI VDMA**                                         | Transfers video frames between DDR and streaming video interface.       |
    | **Video Timing Controller (v\_tc\_0)**               | Generates timing signals for sync (VGA).                                |
    | **AXI4-Stream to Video Out (v\_axi4s\_vid\_out\_0)** | Converts video stream to RGB signals for VGA.                           |
    | **AXI SmartConnect**                                 | Manages AXI interconnect between PS, VDMA, and peripherals.             |
    | **AXI DMA**                                          | For optional memory transfers (currently supporting future extensions). |
    | **AXIS Switch and Width Converter**                  | Aligns data widths for stream compatibility.                            |
    | **Clock Wizard**                                     | Generates system clocks.                                                |
    | **Processor System Reset**                           | Ensures proper reset signal distribution.                               |


# What It Does
Streams frames from memory (written by the ARM processor).

Converts and outputs the video in VGA format.

Allows future integration of processed video frames (e.g., with bounding boxes or overlays from detection software).
