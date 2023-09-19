# Physical Design using OpenLANE/Sky130 Tool

## OpenLANE


OpenLANE offers two distinct modes of operation: an Automated Flow and an Interactive Mode.

In the Automated Flow, all the essential steps of the chip design process are executed sequentially, ensuring a streamlined and automated progression from RTL synthesis to GDSII generation. Each stage within this flow consists of multiple sub-stages, with each step serving a specific purpose:

1. *Synthesis*
    1. `yosys` - Performs RTL synthesis
    2. `abc` - Performs technology mapping
    3. `OpenSTA` - Pefroms static timing analysis on the resulting netlist to generate timing reports
2. *Floorplan and PDN*
    1. `init_fp` - Defines the core area for the macro as well as the rows (used for placement) and the tracks (used for routing)
    2. `ioplacer` - Places the macro input and output ports
    3. `pdn` - Generates the power distribution network
    4. `tapcell` - Inserts welltap and decap cells in the floorplan
3. *Placement*
    1. `RePLace` - Performs global placement
    2. `Resizer` - Performs optional optimizations on the design
    3. `OpenPhySyn` - Performs timing optimizations on the design
    4. `OpenDP` - Perfroms detailed placement to legalize the globally placed components
4. *CTS*
    1. `TritonCTS` - Synthesizes the clock distribution network (the clock tree)
5. *Routing*
    1. `FastRoute` - Performs global routing to generate a guide file for the detailed router
    2. `CU-GR` - Another option for performing global routing.
    3. `TritonRoute` - Performs detailed routing
    4. `SPEF-Extractor` - Performs SPEF extraction
6. *GDSII Generation*
    1. `Magic` - Streams out the final GDSII layout file from the routed def
    2. `Klayout` - Streams out the final GDSII layout file from the routed def as a back-up
7. *Checks*
    1. `Magic` - Performs DRC Checks & Antenna Checks
    2. `Klayout` - Performs DRC Checks
    3. `Netgen` - Performs LVS Checks
    4. `CVC` - Performs Circuit Validity Checks

On the other hand, OpenLANE's Interactive Mode provides designers with a more flexible and hands-on approach to the chip design process. It allows for manual intervention and customization at various stages of the flow, empowering users to make real-time adjustments and optimizations as needed to meet specific design requirements.

## The automated flow, use these commands
```
cd OpenLane
make mount
./ flow.tcl -design openlane/<DESIGN_NAME>  -tag <TAG>
```

## The Interactive mode, use these commands 
```
cd OpenLane
make mount
./flow.tcl -interactive 
prep -design <path_to_your_design_folder> -tag <tag> -overwrite //overwrite is optional
```

**Interactive mode** offers us to learn all the steps present in automated flow step by step.
The steps are as follows : 

```
run_synthesis
run_floorplan
run_placement
run_cts
run_routing
write_powered_verilog followed by set_netlist $::env(lvs_result_file_tag).powered.v
run_magic
run_magic_spice_export
run_magic_drc
run_lvs
run_antenna_check
```

# Labs

<details>
<summary>DAY 1 : opensource-EDA, Openlane and Skywater130</summary>
<br>

## Skywater-130 PDK


## Invoking OpenLANE

![flowtcl](https://github.com/kamildamudi21/pes_pd/assets/141449459/5ae299c2-7d4d-41e6-95f7-983d567205a3)



flow.tcl is the file that contains the script to run the designs

## Importing package

Different software dependencies are needed to run OpenLANE. To import these into the OpenLANE tool we need to run: 
```package require openlane 0.9```


## Prepare the design for the flow 

```prep -design picorv32a```

![prepdesign](https://github.com/kamildamudi21/pes_pd/assets/141449459/cdcdfdce-4023-4183-8775-4fa50fbd72aa)


## Synthesis

```run_synthesis```

![synthesis](https://github.com/kamildamudi21/pes_pd/assets/141449459/7e116867-09fd-42ac-a903-e83ba6333c7f)

![flopratio](https://github.com/kamildamudi21/pes_pd/assets/141449459/07e7a767-d568-4986-b2f3-8be1a26e890d)

### Flop Ratio = (No. of D flip flops / Total number of cells) = 1613/14876 = 10.08%

</details>

<details>
<summary>DAY 2 : Good Floorplan vs Bad Floorplan and Introduction to library cells</summary>
<br>

## Floorplan

in OpenLANE, enter ```run_floorplan``` and the results will be updated in the runs folder

To view the layout of the floorplan, use the command ```magic -T /home/vsduser/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef def read picorv32a.floorplan.def &```


![1](https://github.com/kamildamudi21/pes_pd/assets/141449459/2a32ea3e-91f9-4840-87e1-de493012d365)


## Library Binding and Placement
### Placement

```run_placement```

![2](https://github.com/kamildamudi21/pes_pd/assets/141449459/ddbb2ceb-73f8-4e38-bf0e-40d4e63e5ed1)


To view the layout of the placement, use the command ```magic -T /home/vsduser/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef def read picorv32a.placement.def &```


![3](https://github.com/kamildamudi21/pes_pd/assets/141449459/c063cf00-b81a-4e56-9b4d-f7ad395335e5)


## Cell Design Flow

Cell design is done in 3 parts:

1. **Inputs** - PDKs (Process design kits), DRC & LVS rules, SPICE models, library & user-defined specs.
2. **Design Steps** - Design steps of cell design involves Circuit Design, Layout Design, Characterization. The software GUNA used for characterization. The characterization can be classified as Timing characterization, Power characterization and Noise characterization.
3. **Outputs** - Outputs of the Design are CDL (Circuit Description Language), GDSII, LEF, extracted Spice netlist (.cir), timing, noise, power.libs, function.

### Standard cell Charachterization Flow

Standard Cell Libraries encompass diverse cells featuring varying functionality and drive strengths. These cells must undergo characterization through liberty files to enable synthesis tools to establish the most advantageous circuit configuration. The open-source software GUNA serves as the means for conducting this characterization process.

Characterization follows a clearly defined sequence involving the following stages:

-Incorporate the Model File for CMOS, which contains property definitions.
-Define the process corner(s) to characterize the target cell.
-Set the threshold percentages for cell delay and slew.
-Specify tables for timing and power characteristics.
-Read the netlist with parasitic extraction results.
-Apply input signals or stimuli as required.
-Execute the necessary simulation commands.


### General Timing characterization parameters

#### Timing threshold definitions

- ```slew_low_rise_thr``` - 20% from bottom power supply when the signal is rising
- ```slew_high_rise_thr``` - 20% from top power supply when the signal is rising
- ```slew_low_fall_thr``` - 20% from bottom power supply when the signal is falling
- ```slew_high_fall_thr``` - 20% from top power supply when the signal is falling
- ```in_rise_thr``` - 50% point on the rising edge of input
- ```in_fall_thr``` - 50% point on the falling edge of input
- ```out_rise_thr``` - 50% point on the rising edge of ouput
- ```out_fall_thr``` - 50% point on the falling edge of ouput

These are the main parameters that we use to calculate factors such as propogation delay and transition time

- ```propogation delay ``` - time(out_*_thr) - time(in_*_thr)
- ```Transition time``` - time(slew_high_rise_thr) - time(slew_low_rise_thr)


</details>

<details>
<summary>DAY 3 :  Design library cell using Magic Layout and ngspice characterization  </summary>
<br>


## Inverter Layout using Magic

```
cd Desktop/work/tools/openlane_working_dir/openlane/vsdstdcelldesign
magic -T sky130A.tech sky130_inv.mag
```

## Exploring the Layout displayed by MAGIC

![layout1](https://github.com/kamildamudi21/pes_pd/assets/141449459/729be7eb-11db-4e95-abb1-5e2556981de5)


## Modified Spice netlist
![2spice](https://github.com/kamildamudi21/pes_pd/assets/141449459/494bbefb-7feb-484f-a780-51018685f96c)
![3](https://github.com/kamildamudi21/pes_pd/assets/141449459/9ccfa87b-64ab-4ee2-a861-6785a7b9b9f0)
![4](https://github.com/kamildamudi21/pes_pd/assets/141449459/f9f770d4-958e-44a1-950f-8d03c34f2408)


The results obtained from the graph are :
- Rise Transition : 0.0395ns
- Fall transition : 0.0282ns
- Cell Rise delay : 0.03598ns
- Cell fall delay : 0.0483ns


</details>


<details>
<summary>DAY 4 : Pre-Layout timing analysis and importance of good clock tree</summary>
<br>
    
## Extraction of LEF 


Track info can be found at :

``` ~/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/openlane/sky130fd_sc_hd/tracks.info```


![1](https://github.com/kamildamudi21/pes_pd/assets/141449459/c647cfe0-f55a-44c7-b7c9-9954fab64201)


- 1st value indicates the offset and 2nd value indicates the pitch along provided direction

### Setting grid values using above file info

![2](https://github.com/kamildamudi21/pes_pd/assets/141449459/572e9ec0-1eba-49f9-8c40-7a7c47aaa905)



- From the above pic, its confirmed that the pins A and Y are at the intersection of X and Y tracks. So the first condition is met.
- The PR boundary is taking 3 grids on width and 9 grids on height which says that the 2nd condition is also met

## LEF Generation

Since the layout is perfect, we can generate the lef file

#### 1. save the modified layout (with new grid)
   - In console, type ```save sky130_vsdinv.mag```
   - This saves the modified layout in current working directory

#### 2. Open the file and extract LEF
   - Open using ``` magic -T sky130A.tch sky130_vsdinv.mag```
   - in the console opened, type ```lef write``` and a lef file will be generated

![3](https://github.com/kamildamudi21/pes_pd/assets/141449459/0854237e-9670-41e3-86e8-ddaaba5f390d)


#### 4. Make sure the lef file is added

- Include the below command to include the additional lef into the flow:
      
          set lefs [glob $::env(DESIGN_DIR)/src/*.lef]
        
          add_lefs -src $lefs

![4](https://github.com/kamildamudi21/pes_pd/assets/141449459/1830ce59-7252-483c-bc4f-31c280f67143)


since there is slack, we have to reduce it

During the architectural design phase, VLSI engineers will acquire system specifications, which will establish the required operating frequency. To assess a circuit's timing performance, designers will employ static timing analysis tools (STA). When discussing pre-clock tree synthesis STA analysis, our primary focus lies in setup timing concerning the launch clock. STA will identify issues like the worst negative slack (WNS) and total negative slack (TNS), which pertain to the most critical and overall path delays within our setup timing constraint.

To address and rectify slack violations, STA analysis can be executed using OpenSTA, an integral component of the OpenLANE toolchain. To effectively communicate these constraints to tools and ensure their proper operation, two essential steps must be undertaken:

- Create design configuration files (.conf) - These files contain tool configuration settings specific to the designated design.

- Develop Synopsys design constraint (.sdc) files - These files adhere to industry standards and provide comprehensive constraints for the design, ensuring its accurate operation.
  
For the design to be complete, the worst negative slack needs to be above or equal to 0. If the slack is outside of this range we can do one of multiple things:

1. Review our synthesis strategy in OpenLANE
    - Enalbed CELL_SIZING
    - Enabled SYNTH_STRATEGY with parameter as "DELAY 1"
    - The synthesis result is :
      
![5](https://github.com/kamildamudi21/pes_pd/assets/141449459/68dcce2f-e9d3-4fc2-b455-6fd0f1a8f0f2)
![6](https://github.com/kamildamudi21/pes_pd/assets/141449459/7fa4024f-b1c7-46b1-93d8-23edb986ff98)





    The delay is high when the fanout is high. Therefore we can re-run synthesis by changing the value of ```SYNTH_MAX_FANOUT``` variable
    
2. Enable cell buffering 
3. Perform manual cell replacement on our WNS path with the OpenSTA tool

    - We can see which net is driving most outputs and replace the driver cell with larger form of its own kind

 
![7](https://github.com/kamildamudi21/pes_pd/assets/141449459/c5e6dc7f-6b72-4fa1-95c7-ead8dc58c05d)


4. Optimize the fanout value with OpenLANE tool

Since we have synthesised the core using our vsdinv cell too and as it got successfully synthesized, it should be visible in layout after ```run_placement``` stage which is followed after ```run_floorplan``` stage

![8](https://github.com/kamildamudi21/pes_pd/assets/141449459/e68b8588-bcd9-40ba-ad63-32b6dd9daa7b)

</details>

<details>
<summary>DAY 5 : Final steps for RTL2GDSII</summary>
<br>


## Power Distribution Network

After generating our clock tree network and verifying post routing STA checks we are ready to generate the power distribution network ```gen_pdn``` in OpenLANE:

The PDN feature within OpenLANE will create:

- Power ring global to the entire core
- Power halo local to any preplaced cells
- Power straps to bring power into the center of the chip
- Power rails for the standard cells

![1](https://github.com/kamildamudi21/pes_pd/assets/141449459/5ab085ae-8e65-437c-9af9-dd37ad913403)


Note: The pitch of the metal 1 power rails defines the height of the standard cells

## Global and Detailed Routing

OpenLANE uses TritonRoute as the routing engine ```run_routing``` for physical implementations of designs. Routing consists of two stages:

- Global Routing - Routing guides are generated for interconnects on our netlist defining what layers, and where on the chip each of the nets will be reputed
- Detailed Routing - Metal traces are iteratively laid across the routing guides to physically implement the routing guides

If DRC errors persist after routing the user has two options:

- Re-run routing with higher QoR settings
- Manually fix DRC errors specific in tritonRoute.drc file

## SPEF Extraction

After routing has been completed interconnect parasitics can be extracted to perform sign-off post-route STA analysis. The parasitics are extracted into a SPEF file. The SPEF extractor is not included within OpenLANE as of now.

```
cd ~/Desktop/work/tools/SPEFEXTRACTOR
python3 main.py <path to merged.lef in tmp> <path to def in routing>
```

The SPEF File will be generated in the location where def file is present





</details>

</details>
