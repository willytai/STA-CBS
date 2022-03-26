# STA Common Benchmark Suite
Here lists the detailed instructions for running this benchmark suite.  
There are 27 benchmarks in total. Each benchmark will be tested with pre-route STA, post-route STA. Results are dumped as json files.


## Benchmarks

- Combinational

| c1355 | c17 | c1908 | c2670 | c3540 | c3\_slack | c432 | c499 | c5315 | c6288 | c7552 | c880 |
|-------|-----|-------|-------|-------|-----------|------|------|-------|-------|-------|------|

- Sequential

| s1196 | s1494 | s27 | s344 | s349 | s386 | s400 | s510 | s526 | ac97\_ctrl | aes\_core | des\_perf | vga\_lcd |
|-------|-------|-----|------|------|------|------|------|------|------------|-----------|-----------|----------|

- Small testcases from OpenTimer/OpenSTA

| simple | example |
|--------|---------|


## File Structures
```
common_benchmark
├── c1355
│    └── *.v
│    └── *.db
│    └── *.lib
│    └── *.sdc
│    └── *.spef
│    └── *_preroute_pt.tcl
│    └── *_preroute_ot.tcl
│    └── *_postroute_pt.tcl
│    └── *_postroute_ot.tcl
│    └── pt_preroute.json
│    └── ot_preroute.json
│    └── pt_postroute.json
│    └── ot_postroute.json
│    └── ...
├── c17
│    └── ...
├── c1908
│    └── ...
.
.
.
├── testScripts
│    └── ...
├── translator.tcl
├── CMakelists.txt
```
Each of the benchmark contains
| File                  | Description                                                   |
|-----------------------|---------------------------------------------------------------|
| \*.v                  | verilog netlist                                               |
| \*.db                 | converted timing library for PrimeTime                        |
| \*.lib                | Liberty timing library                                        |
| \*.sdc                | timing constraints                                            |
| \*.spef               | parasitics, for post-route STA                                |
| \*\_preroute\_pt.tcl  | PrimeTime script for pre-route STA                            |
| \*\_preroute\_ot.tcl  | OpenTimer script for pre-route STA                            |
| \*\_postroute\_pt.tcl | PrimeTime script for post-route STA                           |
| \*\_postroute\_ot.tcl | OpenTimer script for post-route STA                           |
| pt\_preroute.json     | PrimeTime pre-route output of the data points in json format  |
| ot\_preroute.json     | OpenTimer pre-route output of the data points in json format  |
| pt\_postroute.json    | PrimeTime post-route output of the data points in json format |
| ot\_postroute.json    | OpenTimer post-route output of the data points in json format |

## JSON Output Format

Refer to `output_format.pptx` for the output json format.

## Running the benchmarks

### Dependencies  
- [OpenTimer](https://github.com/OpenTimer/OpenTimer.git)  
- CMake >= 2.8  
- Python3.0+  
- PrimeTime

### Run
Create a testing directory
```
mkdir test && cd test
```
Build CTest target with CMake
```
cmake .. -DOT_EXEC=<path/to/the/OpenTimer/executable>
```
Run all tests
```
make test
```
Logs of the tests will be located at `Testing/Temporary/LastTest.log`.  

**_If error occurs, make sure that_**
- PrimeTime is available
- lc\_shell is available
- OpenTimer is correctly linked


## Dumping SDF
The command for dumping sdf in PrimeTime is simply `write_sdf <output filename>`.  
To generate the SDF for each benchmark, remove all `get_attribute` commands in \<benchmark\>/\*pt.tcl and add `write_sdf <output filename>` in the end.  
For example, this is the script for dumping pre-route SDF for the `simple` benchmark:
```tcl
set_app_var search_path "."
set_app_var link_path "simple_Late.db"
read_verilog simple.v
link_design simple
set_min_library simple_Late.db -min_version simple_Early.db
read_sdc simple.sdc
set timing_remove_clock_reconvergence_pessimism true
set timing_save_pin_arrival_and_slack true
write_sdf simple_preroute.sdf
quit
```
and this is for post-route SDF:
```tcl
set_app_var search_path "."
set_app_var link_path "simple_Late.db"
read_verilog simple.v
link_design simple
set_min_library simple_Late.db -min_version simple_Early.db
read_parasitics simple.spef
read_sdc simple.sdc
set timing_remove_clock_reconvergence_pessimism true
set timing_save_pin_arrival_and_slack true
write_sdf simple_postroute.sdf
quit
```
