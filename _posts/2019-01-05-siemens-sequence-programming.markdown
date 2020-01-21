---
title: Siemens sequence programming and k-space data structure
layout: post
date:   2019-09-11
---
# Learning material
* The [PDF files](https://app.box.com/s/ga3hg1u7nhnya6mry5nnlrjxdshxuvwe) for operating Siemens scanner
* [Files](https://app.box.com/s/r75m3wor2rfgjuke3udg32gpmuf21izx) and [files](https://app.box.com/s/k041zdunxq5n82zak98xib4nodbtho3s) for learning Siemens pulse sequence programming
* Progress in reading `IDEA_Presentations`
	* 2019-01-06: 1_CppOverview; 2_SequenceInN4; 3_kSpaceInN4; 4_realTimeEvents; 5_Reordering; 6_EventBlock; 7_SeqLoop; 8_libSBB; 9_SequenceTesting; 11_ErrorHandling; 
* type `ideassh` in pc to connect with the linux system

# Conversation with Mahesh 
* 2019-01-14
* In window 7 virtual machine: run IDEA.net program and choose the correct complier environment
* The useful commands in window7 IDEA.net include `menu`, `help`, `seqdir` (going to the correct pulse sequence directory), `regtarg` (which is like prep_psq_dir in GE), `mc` (compiling), `mc clear`, `sys` (choosing the targeted hardware setting: 7-skyra XQ is what we have), `trs2vs` (preparing visual studio solution file appropriately according to the make file), `sim` (simulation: init+prep+run),  `vis` (visualization based on the most recent simulation), `poet` (protocol editing for debugging), `poet2` (protocol editing for release); `mp` is to make protocol.
* In the make file, make sure edit the line ## enter target name ## `LIB(ep2d_bold_nkc)` so that the compiled dll and so files do not overwrite the product sequences.
* The `mc` command can be used to produce binary for debug (e.g., flashd.dll), release (e.g., flash.dll), and linux file (e.g., flash.so). The release and linux files should be copied to the scanner for actually running the sequence.

# Meeting with Chidi and Silu
* 2019 01 17
* on Mars (linux side) of virtual machine: change the CIFS_credential: login: mars ; password: mars
* on windows virtual machine: run IDEA.net; in the terminal, type this `net user mars mars`
* How to use debug? Step 1: Add the following codes (`//debug`) to a_MiniFlash.cpp

``` c++
#include "MrServers/MrImaging/ut/libsequt.h"    
#include <iostream>   	// debug: add this and the next line
using namespace std; 	

NLSStatus MiniFlash::initialize(SeqLim &rSeqLim)
{
    // This string is intended to identify where we are in the code (used for debugging by the MRTRACE macro).
    static const char *ptModule = {"MiniFLASH::initialize"};

    // Default return value
    NLS_STATUS lStatus = SEQU__NORMAL;

    #define DEBUG_MSVS_PAUSE  // debug: add the following 5 lines
    #if defined(DEBUG_MSVS_PAUSE) && defined(DEBUG) && !defined(VXWORKS) 
        cout << "Pausing for debugging purposes.  Attach a debugger and press <enter> to continue" << endl;
        char ptJunk[12];
        cin.getline(ptJunk,10); // wait for keyboard    
    #endif    

```
* How to use debug? Step 2: In Visual Studio: choose `debug`: `attach the process`: choose either `CrPropDEF` or `MriSeqSimDSP`; and then in debug window, we can step into (a function), step through (every line of codes), or step out (of a function).

# Operate the MRI scanners
* In the windows system (scanner console) I need to mainly work on these 3 directories: `MriCustomer`; `MriSiteData`; `log` (all under `C:\MedCom`)
* In `MriCustomer` library we store the sequence binary code and ICE reconstruction codes . For example, under the `seq` directory we can see a lot of .so and .dll files which are the sequence codes.  
* In `MriSiteData` we can see support file such as shaped RF pulse files, tensor direction files (although many tensor direction files are in seq directory under MRI customer) 
* In `log` we can see the output from cout<< in the pulse sequence -- first use command line in windows: cmd  -- then type logviewer  -- then we can see the cout output 
* Before doing all of these I need to become an advanced user in the window system -- type control escape to bring up the window and search for "advanced user" program -- the password is `PASSword$$$$1111` in VE11c (meduser1 in earlier versions)

# Install pulse sequences
* The sequence may contain 2 files: .so and .dll -- in addition, the protocol (with the extension of exar1) may be provided.
* The binary files should be put into the MRI customer / seq directory; the protocol file can be put anywhere -- just drag it to the scanner console through a "special card" or something and then it will call the binary file automatically
* After a sequence has been compiled for the Windows platform (release version) and the appropriate measurement control system platform, these targets need to be copied to the NUMARIS Host. The only allowed directory on the NUMARIS Host is `C:\MedCom\MriCustomer\seq` (It is also allowed to create subdirectories therein).
* Once a sequence is available there, a measurement protocol can be created by using the Exam Explorer. Right-click into a program, choose “Insert Sequence”, and switch the selection box over to “USER”. Then, select the desired sequence. (see page 36/548 of the large PDF file: All sequences to be executed must be inserted into Exam Database. All manipulations of Exam database should be done throughthismenu.)
* Four steps of inserting new pulse sequences:
    1. Within Exam Explorer, select Insert from Menu bar.
    2. Identify folder where sequence file is located - Siemens or User.
    3. Identify specific sequence file to be inserted.
    4. Select `Insert`, and then a default protocol will be generated.
    
# How to get the k-space data from the scanner
* Find the program mrvista (which can be searched with twix from the command line window) -- which is the raw data manager 
* Right click on the files we want and choose "copy total file"

# Compiling pulse sequences
* `Runmode` shows which mode the program is running
* `Runmode RELEASE` switches to the RELEASE mode
* After compiling a pulse sequence, the dll file is located in `.../n4/x86/prod/bin` and the so file is locate din `.../n4/linux/prod/lib`

# Working with FLASHBack_nkc sequence
* 2019-04-27
* the goal here is to implement super-resolution; so I look for places where the slice position is mentioned add where multiple measurement is mentioned, and marked with `// nkc`
* my planned steps:
    1. compile the original sequence and see the diagram
    2. add another loop to simply repeat the sequence
    3. introduce slice position shift for every measurement
    4. built upon the previous step we now introduce slice position shift only for some measurements
* understanding the sequence
    * Initialize
        * `line # 389 to 391` mentioned `setSlices` and `setSliceDistanceFactor`;
        * `line # 399 to 400` mentioned that the slice group is calculated within the UI (__fSUPrepSlicePosArray__) and the actual position of asSLC[0] depends on the the SliceSeriesMode!! 
        * `line # 402` mentioned `rSeqLim.enableSliceShift()` : can I change the slice shift dynamically? across multiple measurements in an addtional loop?
        * `line # 449` mentioned `rSeqLim.setRepititions()`: what I need is probably the measurement loop instead of repitition loop
        * `line # 468, 487, 506`: now I see that RSats means regional saturations; PSat means parallel saturations (which are RSat); TSats means travelling sat that might be used to suppress blood signal (black blood); MSats might mean MTC ?
        * `line #506` mentioned rSeqLim.setMultipleSeriesMode(): I need to understand this
    * Prepare
        * `line # 872` mentioned the number that the kernel would be called
        * `line # 1058`: what is `TokTokSBB`?
        * `line # 1125 to 1133` mentioned RF pulse parameters.
        * `line # 1146 to 1149` mentioned ADC.
        * `line # 1197` mentioned prepPE: prepares for phase-encoding gradient table.
        * `line # 1213 to 1241` talks about the slice gradient.
        * `line # 1544 and 1545` __calculates the positions of slices, most relevant to my interest!__ I think I will need to find a way to further modify __m_asSLC__!!
        * `line # 1650` section: ICE program; I should figure out how to use Mahesh's no_ICE recon program to bypass the ICE recon.
    * Check
        * `line # 1770`
    * Run
        * `line # 1860`: what is ChronologicSlice?
        * `line # 1909` is the __loop__ structure! It looks like that I will need to add another loop myself (measurement) -- pay attention to getMDH()
        * `line # 2014` __run_kernel()__
        * `line # 2089`: this is exactly where i can __modify the code__ -- insteading of grabbing information from `m_asSLC[slice1]` i could grab information from both `m_asSLC[slice1]` and `m_asSLC[slice2]` and average them-- achieving slice position shift. same thing should be applied to ADC, `line # 2100`
        * `line # 2186` shows the event block
        * `line # 2191`: how to modify this line containing `m_asSLC[slicex].getROT_MATRIX()`?
* After understanding the sequence, I think I am ready to add an additional loop (near `line # 1909`) and modify the slice position information (near `line # 2089`).


# Some notes from looking at Ralf Deichmann's single-shot EPI pulse sequence
* 2019-08-23
* We can change the reconsrtruction mode with `pSeqLim->setReconstructionMode   (SEQ::RECONMODE_MAGNITUDE);` Other possible values are: `RECONMODE_PHASE`, `RECONMODE_REAL_PART`, `RECONMODE_MAGN_PHASE`, `RECONMODE_REAL_PHASE`.
* Setting up ICE program (see "ralf_epi_template.cpp")
``` if(pMrProt->wipMemBlock().alFree[0]==0)
  {
   pSeqLim->setICEProgramFilename   ("%SiemensIceProgs%\\IceProgram2D");
  }
```
* Specifying Mdh in Ralf D.'s single-shot EPI pulse sequence
    * `sADC01.Mdh.setKSpaceCentreColumn(int(pMrProt->kSpace().baseResolution()*0.5));`
    * `sADC01.Mdh.setKSpaceCentrePartition (TRUE);` (mandatory, even for 2D sequence)
    * `sADC01.Mdh.setEvalInfoMask (sADC01.Mdh.getEvalInfoMask() | MDH_ONLINE) ;` **what is this for?**
    * `sADC02.Mdh.setCseg            (0);` used for echoes with positive readout
    * `sADC02.Mdh.setCacq            (0);` used for ALL echoes except for 3. nav. echo
    * `sADC03.Mdh.setCseg            (1);` used for echoes with negative readout
    * `sADC03.Mdh.setCacq            (1);` used for 3. nav. echo since it has to be added to the 1. nav. echo
    * `sADC03.Mdh.addToEvalInfoMask (MDH_REFLECT);` used for echoes with negative readout **???**
    * `sADC01.Mdh.setCrep (crep)` in fSEQRun()
    * `sADC01.Mdh.setCslc (cslc);` in fSEQRun()
    * `sADC01.Mdh.setReadOutOffcentre(asSLC[cslc].getSliceOffCenterRO());` in fSEQRun()
    * `sADC01.Mdh.setFirstScanInSlice(1);` in fSEQRunKernel()
    * `sADC01.Mdh.setLastScanInSlice(0);` in fSEQRunKernel()
    * `sADC01.Mdh.setPhaseFT        (0);` in fSEQRunKernel()
    * `sADC01.Mdh.setClin            (centline);` in fSEQRunKernel() 
* To prepare the slice position array
    * `OnErrorReturn(fSUPrepSlicePosArray (pMrProt, pSeqLim, asSLC));`
* In the beginning of fSEQRun() of Ralf D. single-shot EPI pulse sequence:
    * `mSEQTest(pMrProt,pSeqLim,pSeqExpo,RTEB_ORIGIN_fSEQRunStart,0,0,0,0,0);`. Should verify this in other pulse sequences.
* [ ] Why is there `mSEQTest()` in fSEQRun to check TR? Probably related to sequence unit test: see the following
```
//Call sequence unit test
  if (lKernelMode == KERNEL_CHECK) 
      mSEQTest(pMrProt,pSeqLim,pSeqExpo,RTEB_ORIGIN_fSEQCheck,10,0, asSLC[cslc].getSliceIndex(),0,0);
  else
      mSEQTest(pMrProt,pSeqLim,pSeqExpo,RTEB_ORIGIN_fSEQRunKernel,10,0,asSLC[cslc].getSliceIndex(),0,0);
  OnErrorReturn(fRTEBFinish());
  ```

# Modifying the FLASHBack.cpp sequence to change slice location across multiple runs
* 2019-09-06
* In my virtualPC the source codes are in the FLASHBack_nkc2 directory. 
    * I moved the following 4 lines from within `runKernel()` to `run()`, and as a result the slice position is now dependent on `lRepetition`. `trs2vs` in the command line window can prepare the source codes in visual studio.
        * `m_sSRF01zSet.prepSet (m_asSLC[lChronologicSlice*(lRepetition%2)], m_sSRF01) ;` 
        * `m_sSRF01zNeg.prepNeg (m_asSLC[lChronologicSlice*(lRepetition%2)], m_sSRF01) ;` 
        * `m_sADC01zSet.prepSet (m_asSLC[lChronologicSlice*(lRepetition%2)], m_sADC01, m_sGradReadout,lLine - rMrProt.kSpace().echoLine(),lPartition - rMrProt.kSpace().echoPartition()) ;` 
        * `m_sADC01zNeg.prepNeg (m_asSLC[lChronologicSlice*(lRepetition%2)], m_sADC01, m_sGradReadout,lLine - rMrProt.kSpace().echoLine(),lPartition - rMrProt.kSpace().echoPartition()) ;`
    * In command line window I used `mp` to compile the sequences (with options 1, 2, and 4 all turned on: for both simulation and actual scans).
    * The created binary files include: `dll` file located in `.../n4/x86/prod/bin` and `so` file located in `.../n4/linux/prod/lib`
    * In command line window I used `poet` to change parameters in "Doc Cockpit": In many cases I needed to hit `enter`, after changing each parameter, otherwise new parameters were not updated in simulation. I then clicked `Tools`->`SIM` near the top of the "Doc Cockpit" window, and then hit `enter`. When sequence waveforms showed up, I clicked `Window` -> `View Mode` and then increased the `End time (SRT)` to include all the waveforms. With `Diagrams` button I could choose which channels to show. By selecting 3 slices and 4 repeated measurements, I could see that the sequence modification was successful.
    * I transferred the sequence binary files to the scanner: `C:\MedCom\MriCustomer\seq`. 
    * The way to load the pulse sequence in the scanner: 
        1. Click Exam Explorer (near the bottom-right corner), and then click a small arrow to bring up the "Doc Cockpit" window.
        2. Near the right hand side of the window, I could choose "default" to insert FLASHBack_nkc2 pulse sequence (along with other pulse sequences in the `C:\MedCom\MriCustomer\seq` directory).
        3. Save as a "new program".
        4. Now I could load this "new program" in phantom scans.
* The next steps: 
    1. [x] Read the acquired k-space data with my new julia code.
    2. [x] Turn FLASHBack.cpp into an EPI pulse sequence.

# Conversation with Mahesh
* 2019-09-11
* The customers' made RF waveforms are in `\Medcom\MriSiteData\measurement` directory of the scanner PC. The default RF pulse file is `extrf.dat`; I can see that there are a lot of other files with similar names, made by Mahesh. In virtual machine IDEA program I could use `pulsetool` program to load those RF pulse files. With the same `pulsetool` program I could load my own RF pulses and create a new .dat file. In my pulse sequence I just need to call that new .dat file (which should be placed in the `\Medcom\MriSiteData\measurement` directory of the scanner PC). I will use this approach to design my own multi-band RF pulses.
* In the scanner PC I could type the following command line `ideacmdtool`: option 5: I can turn on EmptyIceProgram (for all subsequent scans: so make sure undo this change after phantom scans). Option 1 is for producing simulated physiological signals (e.g., for testing respiration gated scans); Option 2 is to clear the .dll files already loaded into the memory.
* To use EmptyIceProgram in the pulse sequence level: Note that there is an `IceProgramIceTemplate.ipr` file in the `\Medcom\MriCustomer\ice` directory of the scanner PC. This ipr program calls the most basic and default configuration (including saving k-space data) but not performing Fourier transform. In my pulse sequence I just need to call this `IceProgramIceTemplate` and I should be able to just save k-space data without reconstructing images on the scanner. By the way, the `\Medcom\MriProduct\IceConfigurators\` directory of the scanner PC includes Siemens' ICE programs, such as `IceProgramOnline2D.ipr` that's mentioned in the `FLASHBack.cpp` code.
* Note that I can use `\Medcom\MriCustomer\ice\IceProgramIceTemplate.ipr` to retrospectively re-process existing k-space data on the scanner. Just hit `shift` key and then click the FID icon on the scanner: Then click `export edit` and then search `ctrl-F` iceProgram -- on the 0th ky line I should see that IceProgram that's being called -- I could change that to `%CustemerIceProgs%\IceProgramIceTemplate` (similar to the changes I made to the `FLASHBack_nkc2.cpp` code) and then save and then use `shift`+FID_icon to `StartRecon` under the new conditions. In the same `export edit` window I can look into the `PIPE` category and see how the ICE handles the data flow.

# Loading k-space data into julia
* 2019-09-12
* The julia code for reading Siemens' MRI k-space data are [here](https://gist.github.com/nankueichen/1913b36e1ddf3328d87cf396ad06537f)
* The codes (in both jupyter notebook and julia) and the data are located [here](https://drive.google.com/drive/folders/1g5yIu1pfAqKEGxlcl2qtvDYeKfJgNSNr?usp=sharing). The `meas_data.pdf` under the `juliaCodes` directory explains the concept.

# Modifying FLASHBack.cpp sequence to enable EPI
* 2019-09-13
* A few steps might be needed:
    1. Moving the RF pulse, slice-selection gradients, x- and y-prephasing gradients from the most inner loop to an outer loop (done)
    2. Removing the waveforms for the crusher / spoiling gradients in the most inner loop (done)
    3. Reversing the readout gradient polarity for every other $$k_y$$ lines (done)
    4. Adding phase-encoding gradient blips (done)
    5. Maximizing the gradient slew rate for the phase-encoding blips (done)
    6. Making sure the timing is correct (done)
    7. Enabling ramp sampling
* I am working on FLASHbBack_nkc3 directory, for EPI without super-resolution (while FLASHBack_nkc2 could be turned into super-resolution scans)

# Acquring and reconstructing EPI data (based on FLASHBack_nkc3.cpp)
* 2019-09-16
* The data and the reconstruction julia codes (mainly the jupyter notebook) are located [here](https://drive.google.com/drive/folders/12696XFp-vkY60SIlD9jwM4Gcy_2Dafaa?usp=sharing). I successfully produced EPI data with this modified pulse sequence.
* The next steps: 
    1. [ ] Modifying the pulse sequence to enable ramp sampling -- I should look into R Deichmann's sequence and hopefully I can use Siemens built-in function for this.
    2. [ ] Maximizing gradient slew rate of pulse sequence just for blipped gradients but not the phase-encoding prephasing gradient
    3. [ ] Writing a more complete julia reconstruction procedures, that should include Nyquist artifact correction (e.g., with CPR), partial-Fourier reconstruction, SENSE and MUSE data processing.
    4. Trying out different features of the SV-EPI pulse sequence, including 
        1. [x] Swapping x- and y-axes in subsequent scans -- see FLASHBack_nkc4
        2. [ ] Changing the gradient blip amplitudes ($$k_y$$ line dependent) -- this could be done by the `*.prepPE()` function
        3. [ ] Using variable readout acquisition widnow duration ($$k_y$$ line dependent) -- I should look into R Deichmann's sequence.

# EPI with swapping axes
* 2019-09-18
* The EPI pulse sequence, FLASHBack_nkc4, that swaps readout and phase-encoding axes across repeated scans, is located [here](https://drive.google.com/drive/folders/1i3BrpG13lU3XWuOkYuKA3n6NW1W3vAYu?usp=sharing)
* The acquired data (4 repetitions) and the julia reconstruction code are located [here](https://drive.google.com/drive/folders/1i3BrpG13lU3XWuOkYuKA3n6NW1W3vAYu?usp=sharing).









