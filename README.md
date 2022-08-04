# OpenLane-Workshop
This repository contains all the steps made during the VSD-AIT ADVANCED PHYSICAL DESIGN USING OPENLANE/SKY130 Workshop. 
This will be described step by step and day by day.

# Day 1
## Exploring the files
To understand what is inside the PDKs, we will enter the folders
<img src="Day1/Files1.png">
Here we can see the libs.ref folder which contains the standard cells 
<img src="Day1/Files3.png">
This folders contain different types of information such as the verilog description, the layout information, power and timing of each cells and much more
<img src="Day1/Files2.png">
On the libs.tech folder, there are the information used by the tools such as magic, ngspice, xschem and others

## Using OpenLane
To understand more clearly what happens while we run OpenLane we run the docker image container and run the ./flow.tcl file in the interactive mode, this lets us go step by step.
First of all we need a design such as picorv32a which comes base with the tool, and prepare it using the -prep command
<img src="Day1/Prep.jfif">
We can see the results of this in the tmp folder
<img src="Day1/PrepResults.png">

Then we are going to run a synthesis that takes info from the design ./config.tcl file, overriding system default variables, like clock periods, paths and other information
<img src="Day1/Configtcl.png">
Using the run_synthesis command
<img src="Day1/Synthesis.jpeg">
Here we can see that it was a successful run, and we can analyze the reports on the folder of the same name.
<img src="Day1/ReportsF.png">
<img src="Day1/SynthReport1.png">
<img src="Day1/SynthReport2.png">
We can see how many cells we used (A total of 14876 of which 1613 are Flipflops and 1664 are buffers, giving us a Flop Rate of around 10.84%)
