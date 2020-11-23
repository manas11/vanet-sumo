## Build and run simulation 

#### Tools required:
- [NS-3.30 (Network Simulator)](https://www.nsnam.org/releases/ns-3-30/)
- [SUMO (Simulation of Urban MObility)](https://sumo.dlr.de/docs/index.html)

## Setting up simulation

### 1. Installing SUMO
> Setup as per Ubuntu 20.04 LTS

Run following commands to install latest version of SUMO.

```
sudo add-apt-repository ppa:sumo/stable
sudo apt-get update
sudo apt-get install sumo sumo-tools sumo-doc
```

> Check your SUMO version using `sumo --version`

### 2. Downloading Map of area where simulation is to be run, and converting to ns-3 understandable format
There are 2 ways to export map of area where simulation is to be run:
> Step I is preferrable as it offers an interactive UI for setting up various parameters.
> Refer [SUMO Tutorials](https://sumo.dlr.de/docs/Tutorials.html) for more details about SUMO related steps

### I. Using osmWebWizard.py wizard
Once SUMO is installed using above steps, check the installation folder for it. Mostly it will be: **/usr/share/sumo/tools**
1. Enter Tools directory for SUMO
```
cd /usr/share/sumo/tools/
```
2. Run the OSM web wizard, an interactive UI will be opened in the default browser.
```
python osmWebWizard.py
```
3. Using options and configurations available in the right tab, set parameters for your map.
```
a. Select the area 
b. Set Duration
c. Click on Car Label and set number of Cars and other vehicles to simulate.
d. Set other desired parameters
```
4. Once all params are set, Click on Generate scenario button on top tight corner of the webpage.
> This will create some important files in the SUMO user files folder, which is by default **/home/$USERNAME/Sumo**
5. Enter Sumo user files directory
```
cd /home/$USERNAME/Sumo/

# You will be able to see the directory of scenario with timestamp, Enter that directory
cd 2020-11-18-21-19-33/ #(For example)

# Now, do ls to see all generated files
ls

# Sample output: 
build.bat         osm.net.xml              osm.poly.xml  run.bat*
osm_bbox.osm.xml  osm.passenger.trips.xml  osm.sumocfg   trace.xml
osm.netccfg       osm.polycfg              osm.view.xml  true
```
6. Now, generate **trace.xml** and **.tcl** files
```
# While in the scenario's timestamp directory, Run
sumo -c osm.sumocfg --fcd-output trace.xml # A trace.xml file will be generated in this directory

# Now, run traceExporter.py present in /usr/share/sumo/tools to export simulation into format that ns-3 undestands
python /usr/share/sumo/tools/traceExporter.py -i trace.xml --ns2mobility-output=/home/$USERNAME/$some_desired_output_path/mobility.tcl
```
7. We have **mobility.tcl** file which will be used when running vanet-routing-compare.cc.

### II. Using OpenStreetMap's website 
1. Visit [OpenStreetMap Website](https://www.openstreetmap.org).
2. Search for the area where you want to run simulation
3. Click on Export button at the top left, and download the .OSM file. Store this .osm file(assuming name as map.osm) in some directory(assuming it to be $map_dir)
4. Now run below mentioned steps:
```
# a. Enter map_dir 
cd $map_dir

# b. Run this command
netconvert --osm-files map.osm -o map.net.xml
```
5. Visit Recommended typemaps: [Recommended typemaps](https://sumo.dlr.de/docs/Networks/Import/OpenStreetMap.html#recommended_typemaps).
Create your typemap.xml file containing various configurations and parameters. Store typemap.xml in $map_dir

6. Import polygons taking help from [Adding polygons](https://sumo.dlr.de/docs/Networks/Import/OpenStreetMap.html#importing_additional_polygons_buildings_water_etc)
```V
polyconvert --net-file map.net.xml --osm-files map.osm --type-file typemap.xml -o map.poly.xml

# Run this to generate random trips, specify end time using -e
python /usr/share/sumo/tools/randomTrips.py -n map.net.xml -e 99 -l
python /usr/share/sumo/tools/randomTrips.py -n map.net.xml -r map.rou.xml -e 99 -l

# Create a map.sumo.cfg file and run following steps
sumo -c map.sumo.cfg
sumo -c map.sumo.cfg —-fcd-output maptrace.xml
```
7. Now, run traceExporter.py present in /usr/share/sumo/tools to export simulation into format that ns-3 undestands
```
python /usr/share/sumo/tools/traceExporter.py -i trace.xml --ns2mobility-output=/home/$USERNAME/$some_desired_output_path/mobility.tcl
```

### 3. Make changes in ns-3 vanet-routing-compare.cc and run program 

The following parameter are used/modified based on the need to run the simulations.

`m_traceFile`: The ns2 mobility file to run the simulation.

`m_nNodes`: Number of nodes/vehicles used to run the simulation.

`m_rate`: Data rate of the transmission.

The simulation can be run using the below command when the file is present in scratch directory.

```
./waf --run "scratch/vanet-routing-compare --scenario=2"
```

