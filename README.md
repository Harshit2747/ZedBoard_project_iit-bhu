# Drone ditection using zedboard(Work in Progress).

This project aims to build a real-time drone detection system using the ZedBoard (Zynq-7000 SoC) platform. It focuses on capturing live video from a camera, processing the video using OpenCV on the ARM Cortex-A9 (PS), and detecting moving drone-like objects based on motion.
# Current progress.

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


