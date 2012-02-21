  pcmsim
==========

A block device driver for Linux that simulates the presence of a Phase Change
Memory (PCM) in the system installed in one of the DIMM slots on the
motherboard. The simulator is implemented as a kernel module for Linux that
creates /dev/pcm0 when it is loaded -- a ramdisk-backed block devices with
latencies that of PCM. 

Original author: Peter Macko <pmacko@eecs.harvard.edu>
Project website: http://code.google.com/p/pcmsim/
Project license: BSD and GPL v. 3 (dual license)


  Compiling and Running pcmsim
--------------------------------

Build the project using the "make" command. Then load the generated kernel
module. This will create /dev/pcm0, which you can then format with your
favorite file system and use for benchmarking. Note that the data are backed
by a RAM disk, which means that all data that you put on the block device
will magically disappear upon reboot or module unload.

When the module is loaded, it will benchmark the performace of RAM on your
computer and compute the differences between the latencies of the RAM disk
and the simulated PCM. To edit the PCM parameters, modify pcm.c directly
(or fix the code and make it configurable). Also, please note that the RAM
benchmarking is not particularly reliable, but it works most of the time at
least for my test machines. Therefore, please make sure to run multiple trials
and then remove the outliers before using your results. Also, if you manage
to fix this, I'll appreciate if you send me a patch.

The pcmsim kernel module has been tested only on selected 2.6.x kernels; it
still needs to be ported to 3.0.x and 3.2.x kernels. If you mange to fix this,
please send me a patch :)



  Simulator Pseudo-Code
-------------------------

INIT:
  budget := 0
  dirty  := F


READ:
  budget := budget - overhead_time

  if not cached:
    # If the page was dirty, the processor has already performed
    # the write-back, so we can clear the dirty flag.
    budget := budget + read_time
    dirty  := F

  else:
    pass


WRITE:
  budget := budget - overhead_time

  if not cached and not dirty:
    # The processor should perform write-allocate.
    # Account for a write-back that would occur at a later time.
    budget := budget + write_time
    dirty  := T

  elif not cached and dirty:
    # The processor performed a write-back since the last time
    # we accessed the page, so in fact, the page is not dirty.
    # Account for a write-back that would occur at a later time.
    budget := budget + write_time
    dirty  := T
  
  elif cached and not dirty:
    # The data is in cache, but we never wrote since it was first read.
    # Account for a write-back that would occur at a later time.
    budget := budget + write_time
    dirty  := T

  elif cached and dirty:
    # The data has been loaded in the cache, and we've already written to it.
    # There was not write-back since the last time we accessed the page.
    # We have already accounted for the write-back.
    pass