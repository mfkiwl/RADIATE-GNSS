
# RADIATE

RADIATE is a ray-tracing software written in Fortran. Ray-traced delays can be calculated for observations in the microwave as well as in the optical frequency range. The computation is based on numerical weather models and the output files contain several tropospheric parameters


## LICENSE

VieVS Ray-tracer (RADIATE)

Copyright (C) 2018 Daniel Landskron

This program is free software: you can redistribute it and/or modify it under the terms of the GNU General Public License as published by the Free Software Foundation, either version 3 of the License, or (at your option) any later version.
This program is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the GNU General Public License for more details.
You should have received a copy of the GNU General Public License along with this program. If not, see http://www.gnu.org/licenses/.


## Software requirements

RADIATE is run from the Linux command line (Terminal). To compile the software, the *gfortran 5 compiler* (or newer version) needs to be installed.


## Dependencies

RADIATE is an independent software package of VieVS.


## Installation

The source code is hosted at two git repositories:

https://github.com/TUW-VieVS/RADIATE.git (public)

https://git.geo.tuwien.ac.at/vievs/RADIATE/RADIATE.git (inside TU Vienna GEO Domain)

For installation, the following steps are required:
* Create a directory, for example 'RADIATE'
* Clone the VieVS module RADIATE into the new directory using

```
$ mkdir RADIATE
$ cd RADIATE
$ git clone https://github.com/TUW-VieVS/RADIATE.git
```

After installation, unzip DATA/UNDULATIONS/global_undulations_dint_lat0.125_dint_lon0.125.txt.zip, but make sure to exclude the unzipped file from any commit!


## Use the VieVS ray-tracer ##

The Fortran version of the VieVS ray-tracer is to be operated from the Linux command line (Terminal). The procedure is generally divided into two parts: the compilation and the actual ray-tracing. For compilation, the *gfortran 5 compiler* or higher must be installed. On most modern Linux systems this is installed by default. If not, it can be downloaded [here](https://gcc.gnu.org/wiki/GFortran). Apart from the command line, the ray-tracer can also be run from Windows using an IDE like e.g. *Microsoft Visual Studio* together with the *Intel Fortran Compiler*. This makes adapting the script much more comfortable. However, for operational purposes the use of *gfortran* is recommended due to higher computational speed.

In the following, there is a step-by-step description of how to create ray-traced delays.  
1. Before calculation, the following data must be available:
   - the Numerical Weather Models (NWM) in text format of all desired epochs must be stored in *DATA/GRIB/* without subdirectories (examples can be found in *DATA/GRIB/sample/*). These text files have to strictly follow a formatting, which is described in the section "Required format of the NWM text files" below. Please mind that these NWMs must be in 1°x1° horizontal resolution and 25 pressure levels vertical resolution, covering the whole globe! We are aware that the creation of these text format NWMs might be the major challenge for successful usage of the ray-tracer.
   - the specifications for the observations which are to be ray-traced must be available. Depending on the respective input argument (Section "Input arguments to the ray-tracer"), this means that either azel files must be stored in *DATA/AZEL/* (see examples in *DATA/AZEL/sample/*), or the information must be available in *AZEL_spec.txt* / *AZEL_spec_SLR.txt* (for optical ray-tracing) and *AZEL_list.txt* in the *DATA/INPUT/* directory (examples and format see *DATA/INPUT/sample/*).
   - the station coordinate files (Section "Input arguments to the ray-tracer") can be downloaded from http://vmf.geo.tuwien.ac.at/station_coord_files/ and need to be stored in the directory *DATA/STATIONS/*.
2. Open a Terminal window and navigate to the directory *FUN_TEXT/*, where all required functions of the ray-tracer are stored. These functions read the text-file versions of the NWM.
3. As a first step of the ray-tracing, all scripts have to be compiled, that is, an executable file is created. This is handled through a batch file, in which the compilation command is placed twice. To execute the batch file, type `./compile_RADIATE.sh`. This will produce the executable file *radiate*.
4. Now the actual ray-tracing is executed. There are several options to accomplish this, which are listed in the section "Input arguments to the ray-tracer". It is possible to perform the ray-tracing either for single sessions, or for a number of sessions at once. All sessions are referenced through their respective azel filename.
   - Single sessions: Type `./radiate <azel_file> <stations_coord_file>`. The specification of the azel file name and the station coordinates file is the minimum requirement, all other options will get their default values. If further options are to be specified, they have to be appended following a blank.
   - Multiple sessions: In order to evaluate multiple sessions at once, batch files have to be created. One such batch file is `./run_multiple_sessions.sh`, which evaluates all sessions for which azel files are available in *DATA/AZEL/*. Data in subdirectories is not considered. Changes in the input options to the ray-tracer have to be done inside the batch file. 
5. The results of the ray-tracing are stored in *RESULTS/*.


### Input arguments to the ray-tracer ###

The executable file is run by the command `./radiate` from the command line. This command must be followed by a set of input arguments, the first two of which are mandatory. The first mandatory input argument is the azel file (defining the session name), whereas the second mandatory input argument is the station coordinates file. The order of the subsequent optional input arguments is arbitrary. If no optional input arguments are specified, the default settings of each will be applied.

1. Name of azel-file of the session which shall be processed in the format *azel_yyyymmddhh_UNI.txt*, e.g. `azel_2014071200_UNI.txt`
2. Name of the respective station coordinates file:
   - `vlbi.ell`: containing the ellipsoidal coordinates of all VLBI stations (~240)
   - `slr.ell`: containing the ellipsoidal coordinates of SLR stations (~170)
   - `gnss.ell`: containing the ellipsoidal coordinates of most IGS stations (~460)
   - `doris.ell`: containing the ellipsoidal coordinates of all DORIS stations (~200)
   - `gridpoint_coord_5x5`: containing the ellipsoidal coordinates of all grid points of a global 5°x5° grid (2592)
   - `gridpoint_coord_1x1`: containing the ellipsoidal coordinates of all grid points of a global 1°x1° grid (64800)
    
- Method for the vertical interpolation of the meteorological data:
  - `-profilewise` (default)
  - `-gridwise`
- Ray-tracing method (approach for defining intersection points and delays):
  - `-pwl` (default)
  - `-ref_pwl` (only realized in the MATLAB version!)
  - `-Thayer` (only realized in the MATLAB version!)
- Wavelength of observations (choose between ray-tracing for microwave or for optical techniques):
  - `-microwave` (default)
  - `-optical` (implemented wavelength: 532nm)
- Time interpolation method (mode of assigning the observations to epochs):
  - `-one_epoch_per_obs`: each observation is assigned to only that grib-file epoch which it is closest to the observation
  - `-two_epochs_per_obs` (default): each observation is assigned to the two surrounding grib-file epochs, and is linearly interpolated from them. This approach is indeed more precise, however it demands double calculation time. Note: if an observation is exactly at a NWM epoch, the ray-tracing is still carried out for the neigboring NWM as well, although the second epolog entry will not alter the time interpolation result.
- Specify whether a .trp file shall be created and saved to *RESULTS/* in addition to the standardly produced .radiate file:
  - `-trp` (default)
  - `-notrp`
- Specify whether the error log shall be saved to *DATA/ERROR_LOG*:
  - `-errorlog` (default)
  - `-noerrorlog`
- Specify whether the produced session index- and epolog-files shall be cleaned up at the end:
  - `-cleanup`
  - `-nocleanup` (default)
- Specify how sessions are evaluated:
  - `-readAzel` (default): the input settings (stations, azimuths, elevations) for the ray-tracing of the respective session have to be stored in an azel file (from AZimuth and ELevation), containing a line for every single observation. The azel file(s) must be stored in *DATA/AZEL/* without subdirectories. For all the observations contained in it, ray-traced delays will be produced. See Section "Required format of the azel files" for the syntax of azel files. This approach will be relevant in the absolute majority of cases.
  - `-createUniAzel`: in case ray-traced delays for uniformly distributed azimuths and constant elevations are desired (most likely, this will rarely be the case), a further option is possible. The input settings have to be specified in *DATA/INPUT/azel_spec.txt*. Thus, a virtual .azel file is created internally (without being saved as a .txt. file) for all stations from the selected station coordinates file. This means that no .azel file is required in *DATA/AZEL/*. See Section "Required format of the azel files" for the syntax of *azel_spec.txt*.
  
  
### Required format of the NWM text files ###

The VieVS ray-tracer requires the input text files containing the NWM information to follow a strict formatting. In the following, this format is described. In the directory //DATA/GRIB/sample//, there are some example NWMs in the required text format. Please mind that these files may be too large to be opened with standard text editors like Notepad, Wordpad or Notepad++! You may want to use a powerful text viewer such as [lister](https://www.ghisler.com/lister/) for this purpose.

The resolution of the Numerical Weather Models MUST be 1°x1° in the horizontal and 25 pressure levels in the vertical. The 3 quantities which are of interest are:
- geopotential height Z [m]
- specific humidity Q 
- temperature T [K]

##### Header Lines: #####
1. `181 360`: number of latitudes, number of longitudes (hardcoded!)
2. `90.0000000 -90.0000000`: first and last latitudes (hardcoded!)
3. `0.00000000 359.000000`: first and last longitudes (hardcoded!)
4. `1.00000000 1.00000000`: interval of longitudes, interval of latitudes (hardcoded!)
5. `25`: number of pressure levels (hardcoded!)
6. list of pressure levels in [hPa], starting with the top pressure level and ending with the bottom pressure level
7. `3`: number of parameters (hardcoded!)
8. `yyyy mm dd hh mm ss ii`: date of the NWM; *ii* represents the forecast step, but it can contain any number

##### Data lines: #####
In total 75 lines, each value in the format: value (number of pressure level, latitude, longitude), with pressure level #1 being the highest pressure level (in most cases 1 hPa).

![grib_txt file](https://user-images.githubusercontent.com/45286008/49023114-bb073400-f196-11e8-8788-42924bf7085f.PNG)



### How to bring ECMWF NWMs into text format ###

In general, all kinds of NWMs can be converted into the text format described in the section above. At TU Wien, we download NWM data from the European Center for Medium-range Weather Forecasts (ECMWF), which comes in so-called .grib format. However, as this .grib files are not publicly available, we cannot distribute them, which is why all the steps written above are necessary in order to perform the ray-tracing. We are aware that the creation of these text format NWMs might be one of the major challenges of the usage of the ray-tracer.

In case of ECMWF data, we create the .grib files online for the specified height levels, latitudes and longitudes and the meteorological parameters geopotential height, specific humidity and temperature. With the ECMWF-owned Fortran tool *ecCodes* we decipher the binary .grib file and use the internal function `grib_filter` to extract the data columns and write them into the text file using Bash.



### Required format of the azel files ###

Azel files contain all specifications for the observations to be ray-traced. Depending on the optional input arguments, azel files either have to created manually or they are computed automatically (and internally). 

- If the optional input argument `-readAzel` is set (also set by default), then the ray-tracer expects the respective azel file to be stored in 'DATA/AZEL/'. Every line of an azel file specifies one observation, yielding one ray-traced delay in further consequence. Azel files have to consist of the following columns:
  1. scannumber (can also be set to any number)
  2. modified Julian date (including decimal places; must not contradict with the other date specifications!)
  3. year (must not contradict with the modified Julian date!)
  4. day of year (must not contradict with the other data specifications!)
  5. hour (must not contradict with the modified Julian date!)
  6. min (must not contradict with the modified Julian date!)
  7. sec (must not contradict with the modified Julian date!)
  8. station name
  9. azimuth [rad]
  10. elevation [rad]
  11. source (either VLBI quasar or GNSS satellite; can also be set to 'none')
  12. temperature at the site [°C] (can also be set to NaN, if not available)
  13. pressure at the site [hPa] (can also be set to NaN, if not available)
  14. water vapor pressure at the site [hPa] (can also be set to NaN, if not available)

- If the optional input argument `-createUniAzel` is set, then azel files do not need to be created manually. Instead, the azel files are created by RADIATE internally (without writing them to any location), following the specifications in *DATA/INPUT/AZEL_spec.txt*. The syntax for 'AZEL_spec.txt* is described in its header. In order to create *AZEL_list.txt* containing the list of azel file names to be ray-traced, which are based on the date information specified in *AZEL_spec.txt*, the shell script *FUN_ADD/create_AZEL_list.sh* has to be executed. As already mentioned before, this option will most likely not be needed by users.



### Troubleshooting ###

In particular the compilation part is prone to errors. In the following, there is a (incomplete) list of possible error sources, depending on whether the error happened during the compilation, or during the actual ray-tracing.

- Errors during the compilation:
  - Running the compilation batch file *creategf5* does not work, consequently no executable file is produced, but no error message is displayed in the command line: comment out the first line in *creategf5*, saying `scl enable devtoolset-4 bash`. Perhaps you will have to execute this command alone in the command line, though. 
   - Running the compilation batch file *creategf5* does not work, and loads of errors are displayed in the command line, saying that a certain .mod file is missing: apparently there is any defective .mod file in the folder. Delete all .mod files by typing `rm *.mod` into the command line, and then execute *creategf5* at least twice. A set of further error messages will appear, but in the end the compilation should work without any problems.
- Errors during the ray-tracing:
  - In case a certain input argument to the ray-tracer is wrong or missing, the output of the ray-tracer to the command window is generally self-explaining.
  
  
  
### Important hints for compilation with gfortran ###

- A line must not be longer than 132 letters, otherwise it will be truncated. This can be prevented if the flag `-ffree-line-length-none` is used in the compilation following the `gfortran` command.
- Numeric values in types (= structures) which shall get the value 0, must be initialized with this value. In Microsoft VS (with Intel Fortran Compiler) this is not necessary, however in gfortran it is mandatory.
- Divisions by 0 do not yield NaN as in Microsoft VS, but an error.
- To all real or double precision numbers, *d0* MUST be appended.
- Logical comparisons must be *.eqv.* instead of *==*.



### Documented problems of the ray-tracer ###

- in very very rare cases (in 4 of >16000 VLBI sessions in total), but e.g. in session *azel_03JUL16XF.txt* at station CHICHI10, the ray-tracer produces an error. More precisely, in line 541 of module_getRayTrace2D_pwl_global.f90 at some point the location dependent elevation angle *theta* gets *NaN*, perhaps because for some reason the underlying refractivities change too rapidly within the height levels. So far it could not be found out why exactly this is happening, even less how to solve it.


