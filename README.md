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

# Day 2
## Floorplanning
First of all we will check what variables are used in the Floorplanning stage
<img src="Day2/VariablesFP.png">
Here are some optional variables to use during the floorplanning stage letting us configure things like the aspect ratio, the extension of the IO pins, the Die area and such. Then we can take these variables and use then in the corresponding .tcl file or let them be the default settings. Running the run_floorplan command we can see its effect
<img src="Day2/RunFP.png">
Then checking the log files we can see that the lef and def files were generated
<img src="Day2/IoLog.png">
We can also check the config.tcl used this run and see which variables were changed
<img src="Day2/Configtcl.png">
<img src="Day2/Configtc2.png">
Our changes to the metals were successful as well as the core size. Checking also the def and lef files using magic we get the following
<img src="Day2/ResultsFP.png">
We used a Die area of 660.685 um * 671.405 um = 443587.212 um^2
<img src="Day2/MagicFP.png">
<img src="Day2/PinsFP.png">
<img src="Day2/PinsFP2.png">


## Placement
First we are going to do the global placement which is a rough sketch of the placement, usign the run_placement command. This will iterate the position of the blocks until the timing is correct. And then we can legalized with the detailed placement
<img src="Day2/Placement.png">
<img src="Day2/PlacementMagic.png">
On magic we can check the changes as such. If we zoom in we can see the standard cells placed in the design
<img src="Day2/PlacementZoom.png">

# Day 3
## STD Cells SPICE
First of all we are not designing a standard cell from zero, we will use one from Nickson Jose (Thanks!) : git clone https://github.com/nickson-jose/vsdstdcelldesign.git.
<img src="Day3/Cloning.png">
Opening this file in magic shows the following
<img src="Day3/MagicLayout.png">
We make sure everything is connected according to our circuit and so we press S twice to check connectivity of each pin
<img src="Day3/PinCon.png">
Then to make sure this is the logic function we want, and to characterize the cell we have to use SPICE, to do that first extract the cell using the command extract all , and creating the .ext file then converting it to SPICE with ext2spice
<img src="Day3/ExtSpice.png">
<img src="Day3/SPICE.png">
Here we have to change the scale in which the parameters are used, for example magic's minimun unit is 0.01um so we change it to that. We also have to add references to the libs used. These are nshort.lib and pshort.lib
We are gonna test the circuit using a pulse wave source as its entrance, so we add a PULSE source, then to setup VDD and VSS we add DC voltage sources. And finally using the tran directive we run a transient simulation using ngspice
<img src="Day3/SPICEF.png">
<img src="Day3/Sim2.png">
<img src="Day3/Times.png">

<img src="Day3/Times2.png">

Here we can see the aproximate rise time (20% -> 80%) = 0.0663ns and the propagation delay = 0.06897ns if the output capacitance is 2.4f F

# Day 4
## Track Info
It is important to know how much space the standard cell occupies, and how much of it is correctly positioned. For this we will check the tracks.info file found inside the pdks folder, this will help us create a grid to position the layout correctly.


<img src="Day4/Tracks.png">


<img src="Day4/Grid.png">
There are a couple of rules that you have to abide to make a correct standard cell, such as the width of the cell being a multiple of the vertical pitch, and the pins falling in a cross section of the horizontal and vertical tracks. After correcting our cell we are going to extract its lef file. We do this with "save (name).mag" then "lef write"
<img src="Day4/SaveLEF.png">
<img src="Day4/LEF.png">
To use them in the design, we copy the newly generated LEF and the lib files into the source folder in the project. Then add the commands needed in the config.tcl file
<img src="Day4/FilesNeed.png">

<img src="Day4/Configtcl.png">
We can check if this worked by rerunning our synthesis and it shows as such:
<img src="Day4/SynthA.png">
Sadly this results in a timing violation so we got to fix it
<img src="Day4/Violation.png">

## Fixing Slack
We can change our Synth Strategy, enable cell buffering setting SYNTH_STRATEGY to DELAY 1 and SYNTH_SIZING to 1. We get
<img src="Day4/ViolationFixed.png">

## Floorplan and Placement
We run step by step using the commands
```
init_floorplan
place_io
global_placement_or
tap_decap_or
detailed_placement
gen_pdn
```
We can check each result going into the results folder
<img src="Day4/MagicPL.png">
<img src="Day4/NOT.png">
<img src="Day4/Expanded.png">

Now need to first create a .conf file and a .sdc file
```
vi pre_sta.conf
vi base.sdc
```
And then fill it as such
<img src="Day4/Conf.png">
<img src="Day4/SDC.png">
With this files we can run OpenSTA to check the slack using 
```
sta pre_sta.conf
```
We can see we still have a Violation of slack. To improve this we are going to reduce the fanout drive to 4 and rerun the synthesis 
Checking again the file. We have to change one high capacitance cell, in this case the Mux_2_1 was giving us problems so we replace it with Mux_2_4
<img src="Day4/FixedSlack.png">
This increases our area but fixes the delay violations. Now that we have the desired output, we use write_verilog to change our synthetized design with the corresponding changes.
And run floorplan and placement once again

## CTS
To start a CTS run we use run_cts command
<img src="Day4/CTS.png">
Now we just have to validate it
After running the following commands
```
read_lef designs/picorv32a/runs/06-08_15-30/tmp/merged.lef
read_def designs/picorv32a/runs/06-08_15-30/results/cts/picorv32a.cts.def
write_db pico_cts.db
read_db pico_cts.db
read_verilog designs/picorv32a/runs/09-07_05-15/results/synthesis/picorv32a.synthesis_cts.v
read_liberty -max designs/picorv32a/src/sky130_fd_sc_hd__slow.lib
read_liberty -min designs/picorv32a/src/sky130_fd_sc_hd__fast.lib
read_sdc designs/picorv32a/src/picorv32a.sdc
set_propagated_clock [all_clocks]
report_checks -path_delay min_max -format full_clock_expanded -digits 4
```
While using the typical.lib 

<img src="Day4/Random1.png">
<img src="Day4/Random2.png">


We set the wrong lib files so we change the lib_complete variable and we get
```
read_liberty designs/picorv32a/src/sky130_fd_sc_hd__typical.lib
echo $::env(CURRENT_DEF)
set ::env(CURRENT_DEF) placement/picorv32a.placement.def 
```
<img src="Day4/HoldV.png">
To fix this we change the Clock buffers, and the results are
```
set ::env(CTS_CLK_BUFFER_LIST) [lreplace $::env(CTS_CLK_BUFFER_LIST) 0 0] 
```
<img src="Day4/Good1.png">
<img src="Day4/Good2.png">
We are now respecting both Setup and Hold times so we are good to go

# Day 5
## PDN Generation
To create a PDN file we use the gen_pdn command
<img src="Day5/PDN1.png">

## Routing
Now we want to do rounting so we use the run_routing command. This will iterate the routes to make sure there are no DRC Violations.
<img src="Day5/0th.png">
<img src="Day5/5th.png">
<img src="Day5/7th.png">
Due to now having 0 DRC violations we can proceed to extract parasitics. We will do this by using the SPEF Extractor tool that comes with OpenLane
<img src="Day5/FilesPost.png">


# Conclusions
We have learned how to use the OpenLane design flow, how does it work with every step and which tools it uses, we also learned how to do a correct floorplan, how does it affect the timing, characterizing new cells and creating a standard cell from scratch. How to fix timing violations and remaking the verilog files with corrections. And finally doing the routing and having our full gds file. This took a lot of work and patiente to pull off but it was a very fulfilling experience!
