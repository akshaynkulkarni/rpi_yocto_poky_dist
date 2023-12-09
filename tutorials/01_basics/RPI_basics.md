# Bootfow of RPI
``` shell
Power On -> Broadcom ROM ->   GPU core                                     ARM core  --
                                  |                                         ^         |
                            bootcode.bin (enable sdram& load start.elf)     |      Uboot/kernel
                                  |                                         |
    (reads configs,cmdline)  ->  start.elf -------------------------------->|
                                
```
The Broadcom ROM loads the `bootcode.bin` (stage 2 BL) from the sd card (config specific, it could be USB, etc) to the L2 Cache.
This stage 2 BL enables the SDRAM and loads stage 3, i.e., start.elf. This start.elf is VideoCore OS (VCOS), of the GPU.
The start.elf reads config.txt, kernel commandline.txt Based on the configs, it triggers the ARM cores for the boot of kernel or U-boot, etc.
We can set different parameters like image file, RAM address, etc. 
