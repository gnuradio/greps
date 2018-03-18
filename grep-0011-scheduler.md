# GREP 0001 -- Create new scheduler to increase performance 

- Original Author: David Hofstee <gnuradio@c001.nl>
- Champion: David Hofstee <gnuradio@c001.nl>
- Status: Draft 

History:
- 15-Feb-2018: Initial Draft

## Abstract

Recent presentations at GRCon showed that performance of Gnuradio can
be enhanced by simple scheduling improvements. We propose to write a new scheduler
(framework) in Gnuradio in order to get better overal performance. This scheduler
will be based on the Criticality function[1] or a derivative. This function is 
suitable to analyze performance of Kahn Process Network applications like Gnuradio 
and their scheduling in both homogeneous and heterogeneous processing environments.

This scheduling framework should provide profiling (analysis), scheduling
for Gnuradio and provide feedback to the user to show scheduling hot spots (in
processing, bandwidth and memory usage) to allow for optimization.

Another goal is to document, present and explain the effort and results to the
Gnuradio community. 

## Copyright / License

This GREP is licensed as CC-BY-ND.
Copyright 2018 David Hofstee

## Motivation

### Background in scheduling
The scheduler from Gnuradio can be enhanced. Most schedulers work with a priority
of a process and schedule based on that. Although this can work on a 
homogeneous processing platform (like a single CPU, multi core CPU or NUMA 
architecture), it is not sufficient to describe what choices need to be made on 
a heterogeneous system (such as a CPU combined with GPU or FPGA) where processors 
have different execution speeds. 

One example that proves this is by imagining a processing architecture with a fast, 
a medium fast and a slow processor. If a scheduler had to preempt a thread for 
another, which thread would it need to pause? And why? Priority scheduling does 
not answer that question. Because it does not take the execution speed into account. 

Another important aspect of scheduling a KPN is in the fact that memory access and 
data transfers are usually not taken into account. But their behavior can be 
detrimental in the performance of the KPN due to cache misses, interrupts, delays, 
etc. 

Some background on traditional scheduling can be found in [6] [7] [8] [9]

### The role of the Criticality function
Contrary to priority schedulers, the Criticality function takes the differences 
in execution speed in heterogeneous environments into account. It allows the 
scheduler to analyze which KPN node must be scheduled for optimal performance.

I stress the use of heterogeneous processing environments because the 
homogeneous environment is a simplified version of it. Also, many GR users do have 
options to execute the KPN on other processors next to their main CPU. Furthermore
it helps software engineers focus their effort in optimisations (using SIMD, 
other languages or better compilers). 

No prior implementation of a Criticality-type scheduler, a scheduler using the 
Criticality function, exists. So it has to be made from scratch. All tooling, 
needed to analyze the performance of the Gnuradio application blocks need to be 
created too. This is necessary to calculate the Criticality function for the
scheduler. 

After creating the Criticality function for a KPN, other (sometimes conflicting) 
goals exist, such as having the scheduler:
- Optimize for low memory usage
- Optimize for cache hit rate at processors
- Optimize for latency
- Optimize for data throughput (at various data busses)
- Optimize for execution time jitter

If a user wants to accelerate the Gnuradio application, optimalisation needs to
be done. It would be useful if the scheduler can analyze the following for a user:
- What processes can be optimized and explain the required speedup needed to
  attain maximum performance. 
- Predict performance (bottlenecks) on multiple architectures

At some point, the GR scheduler must cooperate with the scheduler of the operating
system. Some consideration must also be taken into account for the differences 
of optimization that can be done. E.g. a RT kernel has different options for
high performance than a normal kernel. Other workloads of a system come into play
as well (e.g. a browser from a user). 
  
## Description

### Origins
There are a number of papers on GRCon17 describing that enhancements in scheduling
can be worthwile [2][3][4][5]. Some information on this topic was shared at FOSDEM18
with the remark of Marcus Müller that scheduling is insufficient at the moment.
Coincidentally, I was there and I wrote a thesis on 'Exploring criticality
numbers for Kahn Process Networks' and proposed a Criticality function as described
in [1]. This describes how Kahn Process Networks can be analyzed to show what block
is critical and what block is not (and by what factor). 


### Current status
Currently, no scheduler exists that uses the Criticality function. 

Todo:
- Create profiling tooling
- Calculate criticality function (and formulate derivatives to take other aspects into account)
- Create scheduler
- Create user interface and provide feedback to user
- Document.... 
- .... more .... 


[1] http://ce-publications.et.tudelft.nl/publications/1009_exploring_criticality_numbers_for_kahn_process_networks.pd
[2] https://www.gnuradio.org/wp-content/uploads/2017/09/Michael-Piscopo-Gnuradio-Opencl-Enabled-Blocks-GRCon-2017v7.pdf
[3] https://www.gnuradio.org/wp-content/uploads/2017/12/Johnathan-Corgan-Technical-Update-.pdf
[4] https://www.gnuradio.org/wp-content/uploads/2017/12/simpleXecutive-Optimizing-Your-Flowgraphs-for-Hardware.pdf
[5] https://www.gnuradio.org/wp-content/uploads/2017/12/David-Pocratsky-OpenCPI-_-GNU-Radio-Integration.pdf

[6] https://en.wikipedia.org/wiki/Scheduling_(computing)
[7] https://en.wikipedia.org/wiki/Flow_shop_scheduling
[8] https://www.cs.uic.edu/~jbell/CourseNotes/OperatingSystems/5_CPU_Scheduling.html
[9] http://www.engr.iupui.edu/~dskim/Classes/ESW5004/RTSys%20Lecture%20Note%20-%20ch03%20Overview%20of%20Real-Time%20Scheduling.pdf
	
