This is a repackaged version of FreeRTOS v9.0.0 using CMake 

Additionally this project contains:

* Texas Instruments TM4C12x controllers taken from TivaWare.

* FreeRTOS patch to allow thread aware debugging in OpenOCD.
  See: `src/FreeRTOS_openocd.c`
  If compiling in Debug configuration linker flags will be adjusted 
  accordingly. [openocd][http://openocd.org/doc/html/GDB-and-OpenOCD.html] 
  **NOTE** this is currently only valid for gcc as linker wrapper
