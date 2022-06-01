# How Core-V-Verif farmework works?

Clone and install all the required toochlain if not present:
1. set the $RISCV path to the directory where RISCV toolcahin is installed
2. To run the compliance tests using core-v-verif, "cva6.py" is used which is called using the `source cva6/regress/dv-riscv-compliance.sh` script.
3. It will give error due to non inclusion of `cf_math_pkg.sv` in ` {core-v-verif_home}/core-v-cores/cva6/common/submodules/common_cells/src/lzc.sv`. It can be removed by inlcuding the mentioned package.


First of all "install-cva6.sh" script is called from "dv-rsicv-compliance.sh"
1. **_install-cva6.sh_**:
   
   * sets paths e.g $ROOT_PROJECT, $RISCV if not set already
   * sets $VERILATOR_ROOT path where verilator will be installed
   * Now calls the "install-verilator.sh" to install verilator
   * Installs verilator to the path specified by VERILATOR_ROOT if not already installed
   * Adds verilator and RISCV tool paths to PATH
   * sets variables to clone and install cva6 repo e.g CVA6_REPO(link to cva6 github's repo), CVA6_BR (master)
   * Now clone cva6 git repo and checkout to specific branch and hash
   * Install spike to the path defined by SPIKE_ROOT
  
2.  **_install-riscv-dv.sh script_**:
       
    * It will again check the variable and paths setting
    * make directory cva6/sim if not already present
    * Clone and install google riscv-dv(cva6/sim/dv) and its dependencies defined under requirements.txt
  
3.  **_install-riscv-compliance.sh script_**:
    
    * clone riscv-compliance test suites and environment from git repo to the "cva6/tests/riscv-compliance"
    * sets $DV_TARGET and $DV_SIMULATORS e.g ( DV_TARGET=cv64a6_imafdc_sv39s, DV_SIMULATORS=veri-core,spike)
    * change directory to cva6/sim
    * run the python script "cva6.py" with following flags:
    ```
    --testlist  = ../tests/testlist_riscv-compliance-$DV_TARGET.yaml
    --target    = $DV_TARGET (cv64a6_imafdc_sc39s)
    --iss_yaml  = cva6.yml
    --iss       = $(DV_SIMULATORS) i.e veri-core, spike
    ```
4. This script is actually running compliance tests provided under the test-suite directory on ISS (spike) and Core (cva6) using verilator
  
    * --iss_yaml flag used to configure ISS accroding to the target(core which is being verified).
  
# Whole Verification Flow:
  
  First of all a argument parser is setup to parse command line arguments. ISS is configured according to the arguments passed. We can provide assembly tests using --asm_test flag and c tests using --c_tests flag. But we can also put all of our tests in a yaml file. Then a output directory is created for simulation results and log files. Now tests are run on both core using verilator and spike and results are stored in log files. Finally these results are compared instruction by instruction if there is any mismatch it fails the tests and report the results


More details on test architecture are provide on this [link](https://github.com/riscv-non-isa/riscv-arch-test/blob/master/spec/TestFormatSpec.adoc)

#  How to create and Run tests?

### Test structure in yaml file:
```yaml  
  test: rv64im-REMUW
  iterations: 1
  path_var: TESTS_PATH
  gcc_opts: "-static -mcmodel=medany -fvisibility=hidden -nostdlib -nostartfiles -I<path_var>/riscv-compliance/riscv-test-env/ -I<path_var>/riscv-compliance/riscv-test-env/p/ -I<path_var>/riscv-compliance/riscv-target/spike/"
  asm_tests: <path_var>/riscv-compliance/riscv-test-suite/rv64im/src/REMUW.S 
  
```
```yaml
# ================================================================================
#                  Regression test list format
# --------------------------------------------------------------------------------
# test            : Assembly test name
# description     : Description of this test
# gen_opts        : Instruction generator options
# iterations      : Number of iterations of this test
# no_iss          : Enable/disable ISS simulator (Optional)
# gen_test        : Test name used by the instruction generator
# asm_tests       : Path to directed, hand-coded assembly test file or directory
# rtl_test        : RTL simulation test name
# cmp_opts        : Compile options passed to the instruction generator
# sim_opts        : Simulation options passed to the instruction generator
# no_post_compare : Enable/disable comparison of trace log and ISS log (Optional)
# compare_opts    : Options for the RTL & ISS trace comparison
# gcc_opts        : gcc compile options
# --------------------------------------------------------------------------------
```

## Running all compliance tests present in a yaml file:
  1. Go to core-v-verif home directory
  2. Run the command: `source cva6/regress/dv-riscv-compliance.sh`
  3. It will run all the tests included in the yam file `testlist_riscv-compliance-cv64a6_imafdc_sv39.yaml` present at : `{core-v-verif_home}/cva6/tests`
  4. Here is the full command which is running the tests using yaml file: `python3 cva6.py --testlist=../tests/testlist_riscv-compliance-$DV_TARGET.yaml --target $DV_TARGET --iss_yaml=cva6.yaml --iss=$DV_SIMULATORS $DV_OPTS`

## Running an individual compliance test from test suite:
If you want to run a specific test using the existing flow and test suites then follow below step:
   1. Open the `testlist_riscv-compliance-cv64a6_imafdc_sv39.yaml`
   2. Now comment all the test except the one which you want to run
   3. Another way to run a test indvidually is by appending `--test <name_of_test>` with `python3 cva6.py --testlist=../tests/testlist_riscv-compliance-$DV_TARGET.yaml --target $DV_TARGET --iss_yaml=cva6.yaml --iss=$DV_SIMULATORS $DV_OPTS`

## Running a custom test:
   If you want to run a specific test using the existing flow and test suites then follow below step:
   1. Open the `testlist_riscv-compliance-cv64a6_imafdc_sv39.yaml`
   2. Now by looking at how other tests are added you can add your custom test
   3. create your assembly test and place it under the `{core-v-verif/cva6/tests/<your_custom_test_dir_name>}`
   4. While adding your test to the yaml file provide the correct path to the test.
   5. If you want to run your custom test along with other tests mentioned in yaml file then add its path, name and iterations etc following the same format as for other tests in yaml file  
   6. It will run your custom test along with other tests already included in yaml file
   7. If you want to run your custom test independtly then follow the same process as above as for individual compliance test

There are also other yaml file presents in `{core-v-verif/cva6/tests/` which include different tests based on target. Each yaml file include different set of tests. You can specificy the whichever yaml file you want to run based on your target by using flag `--testlist` along with `python3 cva6.py command` which is defined in the end of `{core-v-verif_home}/cva6/regress/dv-riscv-compliance.sh` script.
   * Increase the verbosity by appending "-v" flag with cva6.py run command
   * There are lot of other useful command line flags that can be used, enter `--help` along with `python3 cva6.py` command
   * Description of each flag is also defined in `cva6.py` script
