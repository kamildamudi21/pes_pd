# Physical Design using OpenLANE/Sky130 

## OpenLANE Design Stages

OpenLANE flow consists of several stages. By default all flow steps are run in sequence. Each stage may consist of multiple sub-stages. OpenLANE can also be run interactively.

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

OpenLANE can be operated at 2 different modes ie., Automated flow and Interactive mode.

## To enter the automated flow, use these commands
```
cd OpenLane
make mount
./ flow.tcl -design openlane/<DESIGN_NAME>  -tag <TAG>
```

## To enter the Interactive mode, use these commands 
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
<summary>DAY 1 : Inception of opensource-EDA, Openlane and Skywater130</summary>
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

Standard Cell Libraries consist of cells with different functionality/drive strengths. These cells need to be characterized by liberty files to be used by synthesis tools to determine optimal circuit arrangement. The open-source software GUNA is used for characterization.
Characterization is a well-defined flow consisting of the following steps:

- Link Model File of CMOS containing property definitions
- Specify process corner(s) for the cell to be characterized
- Specify cell delay and slew thresholds percentages
- Specify timing and power tables
- Read the parasitic extracted netlist
- Apply input or stimulus
- Provide necessary simulation commands

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


## Modified Spice netlist


</details>


<details>
<summary>DAY 4 : Pre-Layout timing analysis and importance of good clock tree</summary>
<br>
    
## Extraction of LEF



### Setting grid values using above file info



## LEF Generation



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
