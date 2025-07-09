# Drone ditection using zedboard(Work in Progress).

This project aims to build a real-time drone detection system using the ZedBoard (Zynq-7000 SoC) platform. It focuses on capturing live video from a camera, processing the video using OpenCV on the ARM Cortex-A9 (PS), and detecting moving drone-like objects based on motion.
# Current progress.

# Video Interfacing on ZedBoard (Completed)
As part of the drone detection system, the video interfacing and frame buffer setup have been successfully implemented on the ZedBoard (Zynq-7000) using Vivado 2018.2.

We have completed the hardware design for capturing and displaying video frames using the AXI VDMA, video timing controller, and Zynq PS. This design allows us to stream image frames from DDR memory to the VGA output, forming the foundation for later software-based drone detection using OpenCV.
