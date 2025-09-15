
<!-- image -->

<!-- image -->

<!-- image -->

NIVH

## Disclaimer and proprietary information notice:

## Copyright

© 2024 Hailo Technologies Ltd ('Hailo'). All Rights Reserved.

No part of this document may be reproduced or transmitted in any form without the expressed, written permission of Hailo. Nothing contained in this document should be construed as granting any license or right to use proprietary information for that matter, without the written permission of Hailo.

This version of the document supersedes all previous versions.

## General Notice

Hailo, to the fullest extent permitted by law, provides this document 'as -is' and disclaims all warranties, either express or implied, statutory, or otherwise, including but not limited to the implied warranties of merchantability, noninfringement of third parties' rights, and fitness for a particular purpose.

Although Hailo used reasonable efforts to ensure the accuracy of the content of this document, it is possible that this document may contain technical inaccuracies or other errors. Hailo assumes no liability for any error in this document, and for damages, whether direct, indirect, incidental, consequential, or otherwise, that may result from such error, including, but not limited to loss of data or profits.

The content in this document is subject to change without prior notice and Hailo reserves the right to make changes to the content of this document without providing a notification to its users.

## Trademark Acknowledgement

Micron® is a registered trademark of Micron Technology, Inc. in the US and/or elsewhere.

<!-- image -->

## NIVH 口

## Table of Contents

| 1.                                                                                                                                                                                                                                                                                                                    | Overview...................................................................................................................................................................5                                               |
|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 2. Change Log ..............................................................................................................................................................6                                                                                                                                         |                                                                                                                                                                                                                            |
| 3.                                                                                                                                                                                                                                                                                                                    | Pre-Requisites........................................................................................................................................................7                                                    |
| 3.1. Supported                                                                                                                                                                                                                                                                                                        | DRAM...........................................................................................................................................7                                                                           |
| 4. Installation and Setup                                                                                                                                                                                                                                                                                             | .........................................................................................................................................8                                                                                 |
| 4.1. Board                                                                                                                                                                                                                                                                                                            | Setup................................................................................................................................................... 8                                                                 |
| 4.2.                                                                                                                                                                                                                                                                                                                  | DDRTool Installation ................................................................................................................................. 8                                                                   |
| 4.3.                                                                                                                                                                                                                                                                                                                  | DDRIntegration Tool Configurations ....................................................................................................9                                                                                   |
| 5. DDRTest..................................................................................................................................................................11                                                                                                                                        |                                                                                                                                                                                                                            |
| 5.1. Overview.........................................................................................................................................................11                                                                                                                                              |                                                                                                                                                                                                                            |
| 5.2. F0                                                                                                                                                                                                                                                                                                               | Tests...........................................................................................................................................................11                                                         |
| 5.3. F1\F2 Tests:                                                                                                                                                                                                                                                                                                     | ..................................................................................................................................................12                                                                       |
| 5.4.                                                                                                                                                                                                                                                                                                                  | Usage &Parameters .................................................................................................................................14                                                                      |
| 5.5.                                                                                                                                                                                                                                                                                                                  | Output.............................................................................................................................................................15                                                      |
| 5.5.1.                                                                                                                                                                                                                                                                                                                | Output Reports Summary.................................................................................................................... 15                                                                              |
| 5.5.2. ddr_configurations Output                                                                                                                                                                                                                                                                                      | Example...............................................................................................16                                                                                                                   |
| 5.5.3.                                                                                                                                                                                                                                                                                                                | ddr_test_f0 Output Example.............................................................................................................. 17                                                                                |
| 5.5.4. ddr_test_f1\f2                                                                                                                                                                                                                                                                                                 | Output Example.........................................................................................................18                                                                                                  |
| 5.5.5. rd_margin_test Output                                                                                                                                                                                                                                                                                          | Example.......................................................................................................18                                                                                                           |
| 5.5.6.                                                                                                                                                                                                                                                                                                                | ca_margin_test Output Example.......................................................................................................19                                                                                     |
| 5.6. Execution Examples                                                                                                                                                                                                                                                                                               | ..................................................................................................................................19                                                                                       |
| 6.                                                                                                                                                                                                                                                                                                                    | DDRDumpRegisters.........................................................................................................................................20                                                                |
| 6.1.                                                                                                                                                                                                                                                                                                                  | Overview.......................................................................................................................................................20                                                          |
| 6.2. Usage and Parameters                                                                                                                                                                                                                                                                                             | ............................................................................................................................20                                                                                             |
| 6.3.                                                                                                                                                                                                                                                                                                                  | Source Files Structure...............................................................................................................................21                                                                    |
| 6.4.                                                                                                                                                                                                                                                                                                                  | Output............................................................................................................................................................23                                                       |
| 6.5.                                                                                                                                                                                                                                                                                                                  | Execution Examples .................................................................................................................................24                                                                     |
| Table Figure steps 1 to 3 Figure 2. steps 4 and 5                                                                                                                                                                                                                                                                     |                                                                                                                                                                                                                            |
| of Figures 1. Installation Installation                                                                                                                                                                                                                                                                               | (example).....................................................................................................9 (example)................................................................................................9 |
| Figure 3.YAML file before editing......................................................................................................................10 Figure 4.YAML file after editing.........................................................................................................................10 |                                                                                                                                                                                                                            |
| Figure 5. hailo15_ddr_test usage and parameters .......................................................................................15                                                                                                                                                                             |                                                                                                                                                                                                                            |
| Figure 6. ddr_configuration                                                                                                                                                                                                                                                                                           | output file example..........................................................................................16                                                                                                            |

## NIVH 口

| Figure 7. ddr_test_f0 output example ..............................................................................................................17   |
|---------------------------------------------------------------------------------------------------------------------------------------------------------|
| Figure 8. ddr_test_f1\f2 output example ........................................................................................................18      |
| Figure 9. Read margin test output example...................................................................................................18          |
| Figure 10. ca margin test output example.......................................................................................................19       |
| Figure 11. haio15_ddr_tests console output example..................................................................................19                  |
| Figure 12. hailo15_dump_regt usage and parameters.................................................................................21                    |
| Figure 13 hailo15_dump_reg output example................................................................................................23             |
| Figure 14. hailo15_dump_reg console example............................................................................................24               |

eliminary

Preli

<!-- image -->

NIVH

## 1. Overview

The purpose of this user guide is to provide a step-by-step direction, from installation to execution of the DDR tool.

The Hailo-15 AI vision processor uses external memory for large software systems and data storage, and an unstable external memory interface could result in system failure or interfere with software development.

As part of the bring-up phase of a board designed with H15 SOC, the DDR interface 's successful initialization and stability shall be confirmed. To achieve this confirmation the DDR integration tests must be executed and passed.

<!-- image -->

NIVH

## 2. Change Log

## Version 0.1 (March 2022)

- Initial release.

## Version 0.2 (April 2022)

- Minor log fixes.

## Version 0.3 (April 2022)

- Changing FW versions.

## Version 1.3 (May 2024)

- New concept of a closed testing environment instead of a debug environment without a real test notion.
- F0 tests implementation
- o hailo15\_ddr\_test
- F1/F2 tests implementation
- o hailo15\_dump\_reg
- CA, RD-Rise, RD-Fall, WR 1D margin tests implementation
- parsing the txt file and extracting the following fields:
- OFFSET
- KEYWORD
- FIELD\_NAME
- LENGTH
- START\_BIT
- Performing read to the registers according to the fields above.
- Enable the proper rank.
- Parsing RANK, [rank], START line.
- o Creating CSV results files for:
- F1/F2 tests
- F0 test
- CA, RD-Rise, RD-Fall, WR '1D ' Margin test.
- MT53E1G32D2FW
- o Supporting three Micron® DDR Devices
- MT53E2G32D4DE
- MT53E512M32D1

<!-- image -->

NIVH

## 3. Pre-Requisites

The following prerequisite components are required:

- X86-64-based PC.
- Supported OS: Ubuntu 20 / Ubuntu 22.
- Supported Python revisions 3.8.10 or 3.10.12.
- USB to UART cable that includes USB to UART bridge.
- The UUT board after successfully passing the SPI Flash Programming. There is an example described in the Hailo-15 ™ SBC Quick Start Guide That can be used as a reference.

## 3.1. Supported DRAM

The following table describes the DRAMs supported by Hailo SW:

Table 1. Supported DDR part numbers

| Vendor Part    | Part Number              | Capacity   |
|----------------|--------------------------|------------|
| Micron® LPDDR4 | MT53E1G32D2FW-046 IT:B   | 4GB        |
| Micron® LPDDR4 | MT53E512M32D1ZW-046 IT:B | 2GB        |
| Micron® LPDDR4 | MT53E2G32D4DE-046 WT:C   | 8GB        |

<!-- image -->

## 4. Installation and Setup

## 4.1. Board Setup

To set up the board, perform the following actions:

1. Ensure that the bootstrap is configured to: Boot from UART
2. Connect the board to the host via UART1. Please note that the host must be connected to the UART via USB to UART converter e.g., FTDI.
3. Power up the board.
4. The Hailo DDR tool can now be activated.

## 4.2. DDR Tool Installation

To install the DDR tool SW, perform the following steps:

<!-- image -->

1. Create a virtual python environment: python -m venv ddr\_tool\_venv (this step is optional)
2. Activate the Virtual Environment: source ddr\_tool\_venv/bin/activate (this step is applicable only if Step 1 was performed)
3. Install the following wheels:
- a. pip install hailo15\_board\_tools-&lt;version&gt;.whl
- b. pip install hailo15\_ddr\_integration\_tool-&lt;version&gt;.whl
4. DDR tool install: hailo15\_ddr\_integration\_tool\_install --work\_dir &lt;ddr\_integ\_dir\_path&gt;
5. The DDR integration tool is being configured by editing a YAML file which located at ddr\_integ\_dir\_path given in Step 4 : vi &lt;ddr\_integ\_dir\_path&gt;/ hailo15\_ddr\_integration\_tool.yaml
6. The tool is now ready to run.

Figure 1 and Figure 2 display screen examples of DDR tool installation.

<!-- image -->

NIVH

Figure 1. Installation steps 1 to 3 (example)

<!-- image -->

## hailol5\_ddr\_integration\_tool\_install--work\_dir ddr\_tool\_workdir vi ddr\_tool workdir/hailo15\_ddr\_integration\_tool.yaml

Figure 2. Installation steps 4 and 5 (example)

## 4.3. DDR Integration Tool Configurations

Before executing the DDR integration tool 's tests and as part of the tool's setup , the user must configure the following attributes in the hailo\_ddr\_tool.yaml

Figure 3 and Figure 4 displays examples of YAML file screen before and after editing.

Table 2. Mandatory attributes for DDR integration tools

| Attribute        | Description                                                                                                           | Example                                                         |
|------------------|-----------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------|
| output_dir       | Path to the directory that will contain execution results and logs.                                                   | $HOME/hailo15_ddr_integartion_t ool_results                     |
| uboot_dtb_file   | Path to the original UBOOT DTB file (u-boot.dtb.signed). This file is part of the image of the Hailo deployment.      | $HOME/hailo15_board_deploy_i mage/u-boot.dtb.signed             |
| fw_recovery_file | Path to the recovery firmware (hailo15_uart_recovery_fw.bin). This file is part of the image of the Hailo deployment. | $HOME/hailo15_board_deploy_i mage/hailo15_uart_recovery_fw. bin |

<!-- image -->

NIVH

| customer_certification _key_file   | Path to the customer certification key (customer key). This file can be taken from the image of the Hailo deployment or supplied by the user.                            | $HOME/hailo15_board_deploy_i mage/customer.key   |
|------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------------------------|
| ddr_reconfig_dtsi_file             | Path to the to one of the supported DDR reconfigurations DTSI files. The files are located and can be found https://github.com/hailo- ai/hailo-u-boot under arch/arm/dts | mt53e1g32d2fw- 046_regconfig_ca_odtb_pu.dtsi     |
| uart_path                          | Path to the serial port on the host that is connected to the H15 UART                                                                                                    | /dev/ttyUSB3                                     |
| secured_chip                       | True in case the H15 IC is a secured device or False in case the device is an Engineering sample                                                                         | True                                             |

paths\_configurations:

- #output\_dir:'$HoME/hailo15\_ddr\_integartion\_tool\_results' uboot\_dtb\_file:#Path to the original UBooT DTB file（u-boot.dtb.signed).This file ispart of theimage of the Hailo deployment
- #Change allof thevariablevaluesaccordingto your environment output\_dir:#Path tothe directory thatcontainesall theresultsandlogsforexmple.
- #uboot\_dtb\_file:'$HoME/hailo15\_board\_deploy\_image/u-boot.dtb.signed
- #fw\_recovery\_file:·$HoME/hailo15\_board\_deploy\_image/hailo15\_uart\_recovery\_fw.bin #customer\_certification\_key\_file:*$HoME/hailo15\_board\_deploy\_image/customer.key
- #ddr\_reconfig\_dtsi\_file:'mt53e1g32d2fw-046\_regconfig\_ca\_odtb\_pu.dtsi
- #uart\_path:'/dev/ttyuSB0 secured\_chip:True#Trueincase theH15ICisasecured deviceorotherwiseFalse-DefaultvalueisFalse

Figure 3. YAML file before editing

## paths\_configurations:

- #changeallof thevariablevaluesaccording toyourenvironment
- output\_dir:"&lt;MY-PATH&gt;/output dir"
- uboot\_dtb\_file:"&lt;hailo\_vision\_processor\_sw\_package&gt;/prebuilt/sbc/hailo15-sbc/u-boot.dtb.signed"
- customer\_certification\_key\_file:"&lt;hailo\_vision\_processor\_sw\_package&gt;/prebuilt/sbc/customer.key"
- fw\_recovery\_file:"&lt;hailo\_vision\_processor\_sw\_package&gt;/prebuilt/sbc/hailoi5\_uart\_recovery\_fw.bin"
- ddr\_reconfig\_dtsi\_file:"&lt;hailou-boot-repo&gt;/arch/arm/dts/hailo15\_ddrMT53E1G32D2Fw-046\_regconfig\_caodtb pu\_4GB.dtsi"
- uart\_path:"/dev/ttyusBo"
- secured\_chip:True

Figure 4. YAML file after editing

<!-- image -->

## 5. DDR Test

## 5.1. Overview

The Hailo DDR sub-system supports three frequency-set-points (FSP):

- F0 -DDR initialization frequency
- F1 -Operating frequency, as defined in the hailo15\_ddr\_configuration.dtsi (for example 1600 MHz, 3200 MTs)
- F2 - Operating frequency, as defined in the hailo15\_ddr\_configuration.dtsi (for example 2000 MHz, 4000 MTs)

The DDR test command line will execute different tests for F0 (initialization frequency) and F1\F2 (operating frequency).

The DDR test generic flow:

<!-- image -->

## 5.2. F0 Tests

The Hailo DDR integration tool will execute the following tests to verify basic functionality and connectivity in the initialization frequency:

<!-- image -->

NIVH

Table 3. Basic functionality and connectivity verification tests

| Test Name   | Goal                                                           | Test flow & Pass Criteria                                                                                                  |
|-------------|----------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------|
| Init        | Verify the DDR subsystem initialized successfully in F0        | The DDR integration tool reads the DDR sub-system status and confirms IO calibration completed with no errors detected.    |
| MRR         | Verify the DRAM mode registers are read successfully           | The DDR integration tool reads the DRAM mode registers and compares expected vs actual read data.                          |
| MRW         | Verify the DRAM mode registers are written successfully        | The DDR integration tool performs a read after write to the DRAM mode registers and compares expected vs actual read data. |
| DRAM access | Verify the DRAM memory is successfully accessed (write \ read) | The DDR integration tool performs a read after write to the DRAM and compares expected vs actual read data.                |

## 5.3. F1\F2 Tests:

For the operational frequencies, the Hailo DDR Integration Tool will execute four tests to confirm both reliability and quality.

The first three tests are 'stress' tests that can be execute d several times to verify reliability and increase the 'stress' level. They verify the initialization (calibration + training) and reliability of the interfaces using DDR subsystem BISTs. The BIST is performed by using an internal IP state-machine that executes different pattern reads\writes in F1\F2 frequencies with high DDR interface utilization.

The fourth 'q uality ' test provides measurements for the different data\command signals margin.

## Stress Tests:

| Test name   | Goal                                                          | Test flow & Pass Criteria                                                                                                                                                                         |
|-------------|---------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Init        | Verify the DDR subsystem was initialized successfully in F1\2 | The DDR integration tool reads the DDR sub-system status and confirms that the DDR subsystem initialized successfully, IO calibration and training has been completed and memory BIST was passed. |

<!-- image -->

NIVH

| Controller BIST   | Verify the DDR interface stability using DDR controller's internal BIST flow.   | The DDR integration tool initiates the DDR controller's internal BIST flow and verifies it passed without any error.   |
|-------------------|---------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------|
| PI BIST           | Verify the DDR interface stability using DDR PI' internal BIST flow .           | The DDR integration tool initiates DDR PI's internal BIST flow and verif ies it passed without any error.              |

## Quality Test:

| Test name   | Goal                                             | Test flow & Pass Criteria                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
|-------------|--------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Margin test | Measure the electrical margin of the CA\DQ lanes | No pass criteria for this test. Testing the 1D margin of the Vref: • Robustness of the signal level • Maintaining trained delays values while changing the Vref parameter • The margin is determined by the upper\lower bound values of Vref for which BISTs are still passing. Testing the 1D margin of the delay ' s parameter • Robustness of the signal width • Maintaining trained Vref values while changing the delays parameter The margin is determined by the upper\lower bound values of delays for which BISTs are still passing. Log the margin of the CA\DQ lanes. |

<!-- image -->

NIVH

## 5.4. Usage &amp; Parameters

The following section describes the main functionality of the DDR Integration Tool which is the DDR sub-system testing.

Usage: hailo15\_ddr\_test [-h] [--fsp {all,f0,f1,f2}] [--op\_fsp\_iterations OP\_FSP\_ITERATIONS] [--margin\_test\_granularity {bit,slice}] [--run\_dump\_scripts [RUN\_DUMP\_SCRIPTS [RUN\_DUMP\_SCRIPTS ...]]] [--stop\_on\_fail] [--skip\_margin\_test]

- Optional parameters: (default is used if not set)

| Parameter Name          | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      | Default Value   |
|-------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------|
| fsp                     | Define which FSP is tested: [all] - execute the test of f0\f1\f2 [f0] - execute the f0 tests [f1] - execute the f1 tests [f2] - execute the f2 tests                                                                                                                                                                                                                                                                                                                                                                             | all             |
| op_fsp_iterations       | Number of iterations to run the operation frequencies stress tests. Note: For F0 the value is always 1                                                                                                                                                                                                                                                                                                                                                                                                                           | 50              |
| run_dump_scripts        | For more information see section 6.1. A list of files containing DDR registers to log during the stress tests of f1\f2. The log files will be stored in the output_dir source_file_1 source_file_2… source_file_N. Notes : • The sources files are located in '<hailo15_ddr_integration_tool>/ ddr_dump_reg_scripts/' directory. • In case of empty list, all files located in the ddr_dump_reg_scripts directory will be used. • If the run_dump_scripts flag is not added to command line, no dump registers will be executed. | NA              |
| margin_test_granularity | Specify the margin test granularity for DQs margin test. The test can combine and test                                                                                                                                                                                                                                                                                                                                                                                                                                           | slice           |

<!-- image -->

NIVH

Figure 5. hailo15\_ddr\_test usage and parameters

|              | the margin of a slice (8 bits) or test eachDQ individually. Running in slice mode will be x8 times faster, but the margin will be given for the full slice. [slice] - run in slice granularity. [bit] - run in DQ granularity   |       |
|--------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------|
| stop_on_fail | The flag indicates that the DDR test execution will stop upon first failure. If flag is not added to command line, the tests ' execution will continue even if a test had failed.                                               | False |
| skip_margin  | The flag indicated to run stress tests only and skip the margin test execution                                                                                                                                                  | False |

<!-- image -->

## 5.5. Output

## 5.5.1. Output Reports Summary

The DDR Integration tool will generate the following reports to the output\_dir:

| File name                                      | Description                                                       |
|------------------------------------------------|-------------------------------------------------------------------|
| ddr_tests_f0_yyyy_mm_dd _hh.mm.ss.csv          | Test result per f0 executed test                                  |
| ddr_tests_f1_yyyy_mm_dd _hh.mm.ss.csv          | Test results per f1 executed 'stress' test                        |
| ddr_tests_f2_yyyy_mm_dd _hh.mm.ss.csv          | T est results per f2 executed 'stress' test                       |
| ddr_configuration_f1_ yyyy_mm_dd _hh.mm.ss.csv | Main DDR IO configurations in f1, such as ODT\ DRV\ Trained VREF. |
| ddr_configuration_f2_ yyyy_mm_dd _hh.mm.ss.csv | Main DDR IO configurations in f2, such as ODT\ DRV\ Trained VREF. |

<!-- image -->

NIVH

| rd_margin_test_f1_1d_fall_yyyy_mm_dd _hh.mm.ss.csv   | DQ read direction, falling edge, margin test for f1   |
|------------------------------------------------------|-------------------------------------------------------|
| rd_margin_test_f1_1d_rise_yyyy_mm_dd _hh.mm.ss.csv   | DQ read direction, rising edge, margin test for f1    |
| wr_margin_test_f1_1d_yyyy_mm_dd _hh.mm.ss.csv        | DQ write direction margin test for f1                 |
| rd_margin_test_f2_1d_fall_yyyy_mm_dd _hh.mm.ss.csv   | DQ read direction, falling edge, margin test for f2   |
| rd_margin_test_f2_1d_rise_yyyy_mm_dd _hh.mm.ss.csv   | DQ read direction, rising edge, margin test for f2    |
| wr_margin_test_f2_1d_yyyy_mm_dd _hh.mm.ss.csv        | DQ write direction margin test for f2                 |
| ca_margin_test_f1_1d_ yyyy_mm_dd _hh.mm.ss.csv       | CA margin test for f1                                 |
| ca_margin_test_f2_1d_ yyyy_mm_dd _hh.mm.ss.csv       | CA margin test for f2                                 |

In addition, the DDR Integration tool will dump all source scripts values as provided in the command line parameter ' run\_dump\_scripts ' to the following file (per fsp):

dump\_ddr\_script\_name\_&lt;fsp&gt;\_yyyy\_mm\_dd\_hh.mm.ss  (e.g., dump\_rdlvl\_delays\_rank\_0\_f1\_2024-04-11\_11-45-36.csv)

## 5.5.2. ddr\_configurations Output Example

Figure 6. ddr\_configuration output file example

| GENERAL         | START TIME          | 9/4/20249:47   |
|-----------------|---------------------|----------------|
| GENERAL         | TEST DURATION       | 0:00:47        |
| GENERAL         | DDR FREQ(MT/s)      | 4000 MT/s      |
| GENERAL         | DDRFSP              | f2             |
| GENERAL         | CA ODT CHB          | PU             |
| GENERAL         | edge                |                |
| CADRV           | CK DRV PU           | R_40           |
| CADRV           | CK DRV PD           | R_40           |
| CA DRV          | CS DRV PU           | R_40           |
| CA DRV          | CS DRV PD           | R_40           |
| CADRV           | CA DRV PU           | R_34p3         |
| CADRV           | CA DRVPD            | R_80           |
| CADRV           | DRAMCABUSODT        | R_80           |
| CA VREF         | CA STARTVREF        | 20.00%         |
| CA VREF         | CAINITVREF          | 20.00%         |
| CA VREF         | CATRAINED VREF      | 28.00%         |
| CAVREF          | CA STOP VREF        | 30.00%         |
| WRITEDQDRV/ODT  | PHYDRVPU            | R_40           |
| WRITEDQDRV/ODT  | PHYDRV PD           | R_40           |
| WRITEDQ DRV/ODT | DRAM DQ ODT         | R_48           |
| WRITE DQ VREF   | DRAM STARTVREF      | 20.00%         |
| WRITE DQ VREF   | DRAM INITVREF       | 20.00%         |
| WRITE DQ VREF   | DRAM TRAINEDVREF    | 23.20%         |
| WRITE DQVREF    | DRAMSTOPVREF        | 30.00%         |
| READDQDRV/ODT   | DRAMDQDRV           | R_40           |
| READDQDRV/ODT   | PHYDQ ODT           | R_40           |
| READDQVREF      | PHYDQ STARTVREF     | 10.70%         |
| READ DQ VREF    | PHYDQINIT VREF      | 13.40%         |
| READDQVREF      | PHY DQ TRAINED VREF | 13.90%         |
| READ DQ VREF    | PHYDQSTOPVREF       | 17.30%         |

<!-- image -->

NIVH

## 5.5.3. ddr\_test\_f0 Output Example

The following example shows f0 run in which all tests have passed.

|        |        | MRR TEST MRWTEST ACCESSTEST   |
|--------|--------|-------------------------------|
| Passed | Passed | Passed                        |

Figure 7. ddr\_test\_f0 output example

<!-- image -->

NIVH

## 5.5.4. ddr\_test\_f1\f2 Output Example

The following example shows f2, 10 iterations, run in which all tests have passed.

Figure 8. ddr\_test\_f1\f2 output example

|           | ITERATION INITTEST CTRL BIST   | PIBIST   |
|-----------|--------------------------------|----------|
| 1 Passed  | Passed                         | Passed   |
| 2Passed   | Passed                         | Passed   |
| 3 Passed  | Passed                         | Passed   |
| 4 Passed  | Passed                         | Passed   |
| 5 Passed  | Passed                         | Passed   |
| 6 Passed  | Passed                         | Passed   |
| 7 Passed  | Passed                         | Passed   |
| 8 Passed  | Passed                         | Passed   |
| 9 Passed  | Passed                         | Passed   |
| 10 Passed | Passed                         | Passed   |

## 5.5.5. rd\_margin\_test Output Example

Showing below example of read margin test, falling edge, while the read margin rising edge and the write margin have the same structure.

Figure 9. Read margin test output example

|        |         | Vref reg_val   | Vref reg_val   | Vref reg_val   | Vref reg_val   | delay reg_val   | delay reg_val   | delay reg_val   | delay reg_val   | delay reg_val   |
|--------|---------|----------------|----------------|----------------|----------------|-----------------|-----------------|-----------------|-----------------|-----------------|
|        |         | trained        | margins        | margins        | margins        | trained         | margins         | margins         | margins         | margins         |
| rank # | Slice # | trained        | min            | max            | size           | trained         | min             | center          | max             | size            |
| 0      | 0       | 38             | 2              | 74             | 73             | 126             | 39              | 137             | 235             | 197             |
| 0      | 1       | 34             | 6              | 66             | 61             | 132             | 46              | 136             | 226             | 181             |
| 0      | 2       | 34             | 2              | 74             | 73             | 126             | 36              | 140             | 244             | 209             |
| 0      | 3       | 34             | 2              | 70             | 69             | 138             | 35              | 132             | 229             | 195             |

<!-- image -->

NIVH

## 5.5.6. ca\_margin\_test Output Example

The following example shows the output of the CA margin test:

Figure 10. ca margin test output example

|        |      | Vref reg_val   | Vref reg_val   | Vref reg_val   | Vref reg_val   | delay reg_val   | delay reg_val   | delay reg_val   | delay reg_val   | delay reg_val   |
|--------|------|----------------|----------------|----------------|----------------|-----------------|-----------------|-----------------|-----------------|-----------------|
|        |      | trained        | margins        | margins        | margins        | trained         | margins         | margins         | margins         | margins         |
| rank # | CA # | trained        | min            | max            | size           | trained         | min             | center          | max             | size            |
| 0      | 0    | 41             | 1              | 49             | 49             | 801             | 593             | 809             | 1025            | 433             |
| 0      | 1    | 41             | 1              | 49             | 49             | 815             | 583             | 815             | 1047            | 465             |
| 0      | 2    | 41             | 1              | 49             | 49             | 828             | 588             | 820             | 1052            | 465             |
| 0      | 3    | 41             | 1              | 49             | 49             | 819             | 563             | 803             | 1043            | 481             |
| 0      | 4    | 41             | 1              | 49             | 49             | 827             | 595             | 815             | 1035            | 441             |
| 0      | 5    | 41             | 1              | 49             | 49             | 822             | 582             | 814             | 1046            | 465             |

## 5.6. Execution Examples

Figure 11 shows the console output during execution:

Figure 11. haio15\_ddr\_tests console output example.

<!-- image -->

<!-- image -->

NIVH

## 6. DDR Dump Registers

## 6.1. Overview

The Hailo DDR Integration Tool supports reading and dumping DDR subsystem's registers. This information helps obtain the status, training results, and more.

## 6.2. Usage and Parameters

Usage: hailo15\_dump\_reg [-h] [--run\_dump\_scripts [RUN\_DUMP\_SCRIPTS [RUN\_DUMP\_SCRIPTS ...]]] [--no\_reset]

<!-- image -->

| Parameter name   | Description                                                                                                                                                                                                                                                                                                                                                                                | Default value     |
|------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------------|
| run_dump_scripts | A list of files containing DDR registers to log during the stress tests of f1\f2. The log files will be stored in the output_dir. source_file_1 source_file_2… source_file_N. Notes : • The sources files are located in '<hailo15_ddr_integration_tool>/ ddr_dump_reg_scripts/' directory. • In case of empty list, all files located in the ddr_dump_reg_scripts directory will be used. | NA                |
| no_reset         | The flag indicates not to perform board reset before and after the test. The flag shall be used in the following use-case only : the user wants to run the dump_reg after a failure in the ddr_test, and to get the current values without board reset, which will set all registers to default values.                                                                                    | Reset is executed |

<!-- image -->

NIVH

Figure 12. hailo15\_dump\_regt usage and parameters

<!-- image -->

## 6.3. Source Files Structure

The script file will be in a text format with one of the following line structures:

1. Register dump:  ' &lt;KEYWORD&gt;, &lt;OFFSET&gt;, &lt;FIELD\_NAME&gt;, &lt;START\_BIT&gt;, &lt;LENGHT&gt;'
- KEYWORD : The DDR sub-system from which the fields is read from. One of the following strings: [CTL, PI, PHY]
- OFFSET : A number between 0 -to max offset of each keyword type.  The offset can be in dec or hex.
4. o CTL max offset: 0x19E
5. o PI max offset: 0x12B
6. o PHY max offset: 0x58E
- FIELD\_NAME : a string describing the field name (will be reflected in the output file)
- START\_BIT : The field's LSB, a number between 0 31
- LENGTH : field number of bits.
- Several 'triplets' of ( FIELD\_NAME, START\_BIT, LENGTH) can be included in each line of the file.
- Examples:
12. o CTL, 20, NO\_AUTO\_MRR\_INIT, 24, 1
13. o CTRL, 20, NO\_AUTO\_MRR\_INIT, 24, 1, TINIT5\_F2, 0, 24

In the above example, the keyword is CTL, with an offset of 20, and two different fields are read from the same offset: NO\_AUTO\_MRR\_INIT &amp; TINIT5\_F2

2. Set rank: 'RANK, [rank], START'
- rank -
3. o [0]: rank 0
4. o [1]: rank 1

Some registers hold different values per DRAM rank. To read the value related to any of the ranks, the line can be inserted in the text.

For example:

<!-- image -->

## NIVH 口

- •
- Reading the PHY\_RDDQS\_RISE\_SLAVE\_DELAY\_0 of rank 0: RANK, 0, START
- PHY,121, PHY\_RDDQS\_DQ0\_RISE\_SLAVE\_DELAY\_0, 8, 10
- PHY,122, PHY\_RDDQS\_DQ1\_RISE\_SLAVE\_DELAY\_0, 16, 10
- PHY,123, PHY\_RDDQS\_DQ2\_RISE\_SLAVE\_DELAY\_0, 16, 10
- PHY,124, PHY\_RDDQS\_DQ3\_RISE\_SLAVE\_DELAY\_0, 16, 10

PHY,125, PHY\_RDDQS\_DQ4\_RISE\_SLAVE\_DELAY\_0, 16, 10

- PHY,126, PHY\_RDDQS\_DQ5\_RISE\_SLAVE\_DELAY\_0, 16, 10
- Reading the PHY\_RDDQS\_RISE\_SLAVE\_DELAY\_0 of rank 1: RANK, 1, START

PHY,121, PHY\_RDDQS\_DQ0\_RISE\_SLAVE\_DELAY\_0, 8, 10 PHY,122, PHY\_RDDQS\_DQ1\_RISE\_SLAVE\_DELAY\_0, 16, 10 PHY,123, PHY\_RDDQS\_DQ2\_RISE\_SLAVE\_DELAY\_0, 16, 10 PHY,124, PHY\_RDDQS\_DQ3\_RISE\_SLAVE\_DELAY\_0, 16, 10 PHY,125, PHY\_RDDQS\_DQ4\_RISE\_SLAVE\_DELAY\_0, 16, 10 PHY,126, PHY\_RDDQS\_DQ5\_RISE\_SLAVE\_DELAY\_0, 16, 10

<!-- image -->

NIVH

## 6.4. Output

The results of the hailo15\_dump\_reg will be written in files located in the output\_dir. The file name will be in the following format:

Dump\_ddr\_script\_name\_&lt;fsp&gt;dd\_mm\_yyyy\_hh.mm.ss (e.g., dump\_rdlvl\_delays\_rank\_0\_f1\_2024-04-11\_11-45-36)

## Output Examples:

| KEYWORD   |   OFFSET | FIELD_NAME                       |   START_BIT |   LEN | FIELD_VALUE                           |
|-----------|----------|----------------------------------|-------------|-------|---------------------------------------|
| PHY       |      121 | PHY_RDDQS_DQO_RISE_SLAVE_DELAY_O |           8 |    10 | PHY_RDDQS_DQO_RISE_SLAVE_DELAY_O=Ox8a |
| PHY       |      122 | PHY_RDDQS_DQ1_RISE_SLAVE_DELAY_0 |          16 |    10 | PHY_RDDQS_DQ1_RISE_SLAVE_DELAY_0=0x90 |
| PHY       |      123 | PHY_RDDQS_DQ2_RISE_SLAVE_DELAY_0 |          16 |    10 | PHY_RDDQS_DQ2_RISE_SLAVE_DELAY_0=0x96 |
| PHY       |      124 | PHY_RDDQS_DQ3_RISE_SLAVE_DELAY_0 |          16 |    10 | PHY_RDDQS_DQ3_RISE_SLAVE_DELAY_0=0x9c |
| PHY       |      125 | PHY_RDDQS_DQ4_RISE_SLAVE_DELAY_0 |          16 |    10 | PHY_RDDQS_DQ4_RISE_SLAVE_DELAY_O=0x8a |
| PHY       |      126 | PHY_RDDQS_DQ5_RISE_SLAVE_DELAY_0 |          16 |    10 | PHY_RDDQS_DQ5_RISE_SLAVE_DELAY_0=0x90 |
| PHY       |      127 | PHY_RDDQS_DQ6_RISE_SLAVE_DELAY_0 |          16 |    10 | PHY_RDDQS_DQ6_RISE_SLAVE_DELAY_0=0x8a |
| PHY       |      128 | PHY_RDDQS_DQ7_RISE_SLAVE_DELAY_0 |          16 |    10 | PHY_RDDQS_DQ7_RISE_SLAVE_DELAY_O=0x8a |
| PHY       |      129 | PHY_RDDQS_DM_RISE_SLAVE_DELAY_0  |          16 |    10 | PHY_RDDQS_DM_RISE_SLAVE_DELAY_O=0x8a  |
| PHY       |      377 | PHY_RDDQS_DQO_RISE_SLAVE_DELAY_1 |           8 |    10 | PHY_RDDQS_DQ0_RISE_SLAVE_DELAY_1=0xa2 |
| PHY       |      378 | PHY_RDDQS_DQ1_RISE_SLAVE_DELAY_1 |          16 |    10 | PHY_RDDQS_DQ1_RISE_SLAVE_DELAY_1=0x96 |
| PHY       |      379 | PHY_RDDQS_DQ2_RISE_SLAVE_DELAY_1 |          16 |    10 | PHY_RDDQS_DQ2_RISE_SLAVE_DELAY_1=0x96 |
| PHY       |      380 | PHY_RDDQS_DQ3_RISE_SLAVE_DELAY_1 |          16 |    10 | PHY_RDDQS_DQ3_RISE_SLAVE_DELAY_1=0x90 |
| PHY       |      381 | PHY_RDDQS_DQ4_RISE_SLAVE_DELAY_1 |          16 |    10 | PHY_RDDQS_DQ4_RISE_SLAVE_DELAY1=0x9c  |
| PHY       |      382 | PHY_RDDQS_DQ5_RISE_SLAVE_DELAY_1 |          16 |    10 | PHY_RDDQS_DQ5_RISE_SLAVE_DELAY_1=0x9c |
| PHY       |      383 | PHY_RDDQS_DQ6_RISE_SLAVE_DELAY_1 |          16 |    10 | PHY_RDDQS_DQ6_RISE_SLAVE_DELAY_1=0x8a |
| PHY       |      384 | PHY_RDDQS_DQ7_RISE_SLAVE_DELAY_1 |          16 |    10 | PHY_RDDQS_DQ7_RISE_SLAVE_DELAY_1=0x90 |
| PHY       |      385 | PHY_RDDQS_DM_RISE_SLAVE_DELAY_1  |          16 |    10 | PHY_RDDQS_DM_RISE_SLAVE_DELAY_1=0xa2  |
| PHY       |      633 | PHY_RDDQS_DQO_RISE_SLAVE_DELAY_2 |           8 |    10 | PHY_RDDQS_DQ0_RISE_SLAVE_DELAY_2=0x84 |
| PHY       |      634 | PHY_RDDQS_DQ1_RISE_SLAVE_DELAY_2 |          16 |    10 | PHY_RDDQS_DQ1_RISE_SLAVE_DELAY_2=0x90 |
| PHY       |      635 | PHY_RDDQS_DQ2_RISE_SLAVE_DELAY_2 |          16 |    10 | PHY_RDDQS_DQ2_RISE_SLAVE_DELAY_2=0x90 |
| PHY       |      636 | PHY_RDDQS_DQ3_RISE_SLAVE_DELAY_2 |          16 |    10 | PHY_RDDQS_DQ3_RISE_SLAVE_DELAY_2=0x90 |
| PHY       |      637 | PHY_RDDQS_DQ4_RISE_SLAVE_DELAY_2 |          16 |    10 | PHY_RDDQS_DQ4_RISE_SLAVE_DELAY_2=0x90 |
| PHY       |      638 | PHY_RDDQS_DQ5_RISE_SLAVE_DELAY_2 |          16 |    10 | PHY_RDDQS_DQ5_RISE_SLAVE_DELAY_2=0x90 |
| PHY       |      639 | PHY_RDDQS_DQ6_RISE_SLAVE_DELAY_2 |          16 |    10 | PHY_RDDQS_DQ6_RISE_SLAVE_DELAY_2=0x84 |
| PHY       |      640 | PHY_RDDQS_DQ7_RISE_SLAVE_DELAY_2 |          16 |    10 | PHY_RDDQS_DQ7_RISE_SLAVE_DELAY2=0x8a  |
| PHY       |      641 | PHY_RDDQS_DM_RISE_SLAVE_DELAY_2  |          16 |    10 | PHY_RDDQS_DM_RISE_SLAVE_DELAY_2=0x84  |
| PHY       |      889 | PHY_RDDQS_DQ0_RISE_SLAVE_DELAY_3 |           8 |    10 | PHY_RDDQS_DQ0_RISE_SLAVE_DELAY_3=0x90 |
| PHY       |      890 | PHY_RDDQS_DQ1_RISE_SLAVE_DELAY_3 |          16 |    10 | PHY_RDDQS_DQ1_RISE_SLAVE_DELAY_3=0x8a |
| PHY       |      891 | PHY_RDDQS_DQ2_RISE_SLAVE_DELAY_3 |          16 |    10 | PHY_RDDQS_DQ2_RISE_SLAVE_DELAY_3=0x9c |
| PHY       |      892 | PHY_RDDQS_DQ3_RISE_SLAVE_DELAY_3 |          16 |    10 | PHY_RDDQS_DQ3_RISE_SLAVE_DELAY_3=0xa2 |
| PHY       |      893 | PHY_RDDQS_DQ4_RISE_SLAVE_DELAY_3 |          16 |    10 | PHY_RDDQS_DQ4_RISE_SLAVE_DELAY_3=0x9c |
| PHY       |      894 | PHY_RDDQS_DQ5_RISE_SLAVE_DELAY_3 |          16 |    10 | PHY_RDDQS_DQ5_RISE_SLAVE_DELAY_3=0x9c |
| PHY       |      895 | PHY_RDDQS_DQ6_RISE_SLAVE_DELAY_3 |          16 |    10 | PHY_RDDQS_DQ6_RISE_SLAVE_DELAY_3=0x96 |
| PHY       |      896 | PHY_RDDQS_DQ7_RISE_SLAVE_DELAY_3 |          16 |    10 | PHY_RDDQS_DQ7_RISE_SLAVE_DELAY_3=0xa2 |

`

Figure 13 hailo15\_dump\_reg output example.

<!-- image -->

NIVH

## 6.5. Execution Examples

<!-- image -->

Figure 14. hailo15\_dump\_reg console example.