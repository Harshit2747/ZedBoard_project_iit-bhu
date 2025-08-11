# Drone ditection using zedboard(Work in Progress).

This project aims to build a real-time drone detection system using the ZedBoard (Zynq-7000 SoC) platform. It focuses on capturing live video from a camera, processing the video on the ARM Cortex-A9 (PS), and detecting moving drone-like objects based on motion.

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

. Photo Display:

.Displayed static test images (e.g., drone photos) through the video interface to verify the complete image pipeline.

.This confirmed that the board could successfully process and display image data in real-time.

# Towards Drone Detection:

These progressive stages established a strong foundation for real-time video capture, processing, and display.

The final objective is to integrate a detection algorithm that can recognize and highlight drones in the video stream based on shape, color, or edge features.

# Video Interfacing on ZedBoard (Completed)
As part of the drone detection system, the video interfacing and frame buffer setup have been successfully implemented on the ZedBoard (Zynq-7000) using Vivado 2018.2.

We have completed the hardware design for capturing and displaying video frames using the AXI VDMA, video timing controller, and Zynq PS. This design allows us to stream image frames from DDR memory to the VGA output, forming the foundation for later software-based drone detection using OpenCV.

# Vivado Block Design
![WhatsApp Image 2025-07-02 at 14 42 46_dae373aa](https://github.com/user-attachments/assets/8aaa0a3f-c6da-4768-9e55-f08f948566e9)


# Vivado Block Design Summary
The current block design includes the following major components:

   | IP Block                                         | Function                                                                |
   | ------------------------------------------------ | ----------------------------------------------------------------------- |
   | Zynq7 Processing System                          | Handles ARM Cortex-A9 (PS), DDR, and AXI interfaces.                    |
   | AXI VDMA                                         | Transfers video frames between DDR and streaming video interface.       |
   | Video Timing Controller (v\_tc\_0)               | Generates timing signals for sync (VGA).                                |
   | AXI4-Stream to Video Out (v\_axi4s\_vid\_out\_0) | Converts video stream to RGB signals for VGA.                           |
   | **AXI SmartConnect**                             | Manages AXI interconnect between PS, VDMA, and peripherals.             |
   | AXI DMA                                          | For optional memory transfers (currently supporting future extensions). |
   | AXIS Switch and Width Converter**                | Aligns data widths for stream compatibility.                            |
   | Clock Wizard                                     | Generates system clocks.                                                |
   | Processor System Reset                           | Ensures proper reset signal distribution.                               |


# What It Does
Streams frames from memory (written by the ARM processor).

Converts and outputs the video in VGA format.

Allows future integration of processed video frames (e.g., with bounding boxes or overlays from detection software).


# PS Part (c code)
 
    #include "xparameters.h"
	    #include "xaxivdma.h"
	#include "xscugic.h"
	#include "sleep.h"
	#include <stdlib.h>
	#include "xil_cache.h"
	#include "xil_cache.h"
	#include "imageData.h"
	
	#define HSize 1920
	#define VSize 1080
	#define FrameSize HSize*VSize*3
	#define imgHSize 512
	#define imgVSize 512
	
	static XScuGic Intc;
	
	
	static int SetupIntrSystem(XAxiVdma *AxiVdmaPtr, u16 ReadIntrId);
	int drawImage(u32 displayHSize,u32 displayVSize,u32 imageHSize,u32 imageVSize,u32 hOffset, u32 vOffset,char *imagePointer);
	
	unsigned char Buffer[FrameSize];
	
	int main(){
		int status;
		int Index;
		u32 Addr;
		XAxiVdma myVDMA;
		XAxiVdma_Config *config = XAxiVdma_LookupConfig(XPAR_AXI_VDMA_0_DEVICE_ID);
		XAxiVdma_DmaSetup ReadCfg;
		status = XAxiVdma_CfgInitialize(&myVDMA, config, config->BaseAddress);
	    if(status != XST_SUCCESS){
	    	xil_printf("DMA Initialization failed");
	    }
	    ReadCfg.VertSizeInput = VSize;
	    ReadCfg.HoriSizeInput = HSize*3;
	    ReadCfg.Stride = HSize*3;
	    ReadCfg.FrameDelay = 0;
	    ReadCfg.EnableCircularBuf = 1;
	    ReadCfg.EnableSync = 1;
	    ReadCfg.PointNum = 0;
	    ReadCfg.EnableFrameCounter = 0;
	    ReadCfg.FixedFrameStoreAddr = 0;
	    status = XAxiVdma_DmaConfig(&myVDMA, XAXIVDMA_READ, &ReadCfg);
	    if (status != XST_SUCCESS) {
	    	xil_printf("Write channel config failed %d\r\n", status);
	    	return status;
	    }

    Addr = (u32)&(Buffer[0]);


	for(Index = 0; Index < myVDMA.MaxNumFrames; Index++) {
		ReadCfg.FrameStoreStartAddr[Index] = Addr;
		Addr +=  FrameSize;
	}

	status = XAxiVdma_DmaSetBufferAddr(&myVDMA, XAXIVDMA_READ,ReadCfg.FrameStoreStartAddr);
	if (status != XST_SUCCESS) {
		xil_printf("Read channel set buffer address failed %d\r\n", status);
		return XST_FAILURE;
	}

	XAxiVdma_IntrEnable(&myVDMA, XAXIVDMA_IXR_COMPLETION_MASK, XAXIVDMA_READ);

	SetupIntrSystem(&myVDMA, XPAR_FABRIC_AXI_VDMA_0_MM2S_INTROUT_INTR);

	//Fill the data
	/*for(int i=0;i<VSize;i++){
		for(int j=0;j<HSize*3;j=j+3){
			if(j>=0 && j<640*3){
				Buffer[(i*HSize*3)+j] = 0xff;
			    Buffer[(i*HSize*3)+j+1] = 0x00;
			    Buffer[(i*HSize*3)+j+2] = 0x00;
			}
			else if(j>=640*3 && j<1280*3){
				Buffer[(i*HSize*3)+j]   = 0x00;
			    Buffer[(i*HSize*3)+j+1] = 0xff;
			    Buffer[(i*HSize*3)+j+2] = 0x00;
			}
			else {
				Buffer[(i*HSize*3)+j]   = 0x00;
			    Buffer[(i*HSize*3)+j+1] = 0x00;
			    Buffer[(i*HSize*3)+j+2] = 0xff;
			}
		}
	}*/
	drawImage(HSize,VSize,imgHSize,imgVSize,(HSize-imgHSize)/2,(VSize-imgVSize)/2,imageData);


	status = XAxiVdma_DmaStart(&myVDMA,XAXIVDMA_READ);
	if (status != XST_SUCCESS) {
		if(status == XST_VDMA_MISMATCH_ERROR)
			xil_printf("DMA Mismatch Error\r\n");
		return XST_FAILURE;
	}

    while(1){
    }
    }


    /*/
       /* Call back function for read channel
    ******************************************************************************/

    static void ReadCallBack(void *CallbackRef, u32 Mask)
    {
	/* User can add his code in this call back function */
	xil_printf("Read Call back function is called\r\n");
	}
	
	/*/
	/*
	 * The user can put his code that should get executed when this
	 * call back happens.
	 *
	*
	******************************************************************************/
	static void ReadErrorCallBack(void *CallbackRef, u32 Mask)
	{
		/* User can add his code in this call back function */
		xil_printf("Read Call back Error function is called\r\n");
	
	     }
	
	
	    static int SetupIntrSystem(XAxiVdma *AxiVdmaPtr, u16 ReadIntrId)
	    {
		int Status;
		XScuGic *IntcInstancePtr =&Intc;
	
		/* Initialize the interrupt controller and connect the ISRs */
		XScuGic_Config *IntcConfig;
		IntcConfig = XScuGic_LookupConfig(XPAR_PS7_SCUGIC_0_DEVICE_ID);
		Status =  XScuGic_CfgInitialize(IntcInstancePtr, IntcConfig, IntcConfig->CpuBaseAddress);
		if(Status != XST_SUCCESS){
			xil_printf("Interrupt controller initialization failed..");
			return -1;
		}
	
		Status = XScuGic_Connect(IntcInstancePtr,ReadIntrId,(Xil_InterruptHandler)XAxiVdma_ReadIntrHandler,(void *)AxiVdmaPtr);
		if (Status != XST_SUCCESS) {
			xil_printf("Failed read channel connect intc %d\r\n", Status);
			return XST_FAILURE;
		}
	
		XScuGic_Enable(IntcInstancePtr,ReadIntrId);
	
		Xil_ExceptionInit();
		Xil_ExceptionRegisterHandler(XIL_EXCEPTION_ID_INT,(Xil_ExceptionHandler)XScuGic_InterruptHandler,(void *)IntcInstancePtr);
		Xil_ExceptionEnable();
	
		/* Register call-back functions
		 */
		XAxiVdma_SetCallBack(AxiVdmaPtr, XAXIVDMA_HANDLER_GENERAL, ReadCallBack, (void *)AxiVdmaPtr, XAXIVDMA_READ);
	
		XAxiVdma_SetCallBack(AxiVdmaPtr, XAXIVDMA_HANDLER_ERROR, ReadErrorCallBack, (void *)AxiVdmaPtr, XAXIVDMA_READ);
	
		return XST_SUCCESS;
	    }
	
	
	    int drawImage(u32 displayHSize,u32 displayVSize,u32 imageHSize,u32 imageVSize,u32 hOffset, u32 vOffset,char *imagePointer){
		for(int i=0;i<VSize;i++){
			for(int j=0;j<HSize;j++){
				if(i<vOffset || i >= vOffset+imageVSize){
					Buffer[(i*HSize*3)+(j*3)]   = 0x00;
				    Buffer[(i*HSize*3)+(j*3)+1] = 0x00;
				    Buffer[(i*HSize*3)+(j*3)+2] = 0x00;
				}
				else if(j<hOffset || j >= hOffset+imageHSize){
					Buffer[(i*HSize*3)+(j*3)]   = 0x00;
				    Buffer[(i*HSize*3)+(j*3)+1] = 0x00;
				    Buffer[(i*HSize*3)+(j*3)+2] = 0x00;
				}
				else {
					Buffer[(i*HSize*3)+j*3]     = *imagePointer/16;
				    Buffer[(i*HSize*3)+(j*3)+1] = *imagePointer/16;
				    Buffer[(i*HSize*3)+(j*3)+2] = *imagePointer/16;
				    imagePointer++;
				}
			}
		}
		Xil_DCacheFlush();
		return 0;
	    }
	

 
# RESULT 

We have successfully implemented an image display system on the ZedBoard using the designed Vivado hardware and PS-side software.
The current setup allows any custom image to be processed in MATLAB, uploaded to the ZedBoard via the Processing System (PS), and displayed in real time through the VGA output.

✅ Achievements
Hardware Design:

AXI VDMA configured to transfer frame data from DDR memory to VGA output.

Video Timing Controller and AXI4-Stream to Video Out generate VGA-compatible signals for the monitor.

Software Implementation (PS):

C program written in Xilinx SDK to read the processed image data from DDR and send it to VDMA.

Integrated MATLAB workflow to prepare the image pixel data in the required format.

Successfully displayed the uploaded image on the VGA monitor

# MATLAB Code – Image to Text for ZedBoard
	 i= imread('myimage.png'); 
	fid=fopen('Myfile.txt','w');
	for r =1:512
	for c =1:512
	fprintf(fid,"%d,",i(c,r));
	end
	end
	fclose(fid);

![WhatsApp Image 2025-07-02 at 13 41 17_6c0b8a98](https://github.com/user-attachments/assets/b6bcc186-8fd5-4f0b-a9b7-84086439eb07)


![WhatsApp Image 2025-07-29 at 22 24 33_0e2f9b8d](https://github.com/user-attachments/assets/d9923dfd-1583-47d6-b406-a4b644b0dc48)

