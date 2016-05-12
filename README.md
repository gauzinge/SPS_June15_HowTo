# SPS_NOV15_HowTo
In case there is a problem with the beam call 77500. 
If there are problems with the telescope / DAQ, call G. Auzinger

# IMPORTANT CHECKLIST

It is extremely important that all shifters are aware that severeal parameters / setups have to be regularly monitored during the shift. Your duties do not only include starting and stopping runs but also monitoring the parameters / functionality of the setups and checking the data quality. Therefore you should terminate and restart the telescope DAQ regularly (approx every 5 runs) to get fresh online-monitoring plots from the telescope and the FEI4. In order to avoid further data corruption / loss you should:

-) check / interpret / scrutinize the DQM plots - if something is wrong, try to repeat the run or figure out what is wrong

-) have an eye on the telescope Online Monitoring: this is at least as important

-) carefully fill the ELOG! This is extremely important!

-) carfully fill the list of Good Runs. This is a dedicated spreadsheet on the google doc with the shift schedule / measurement program

-) you should restart the telescope DAQ approx. every 5 runs in order to get fresh plots that should allow you to determine if there is something wrong

-) regularly check the temperature on the hybrid and the relative humidity / dew point. Also make sure to have an eye on the currents and voltages of the bias system - you can find the plots on cmstkph2-beamtest.web.cern.ch

-) play back the recorded data. This allows the experts to run a quasi-online analysis which will greatly help to monitor the parameters we are setting / quality of the data.

-) every parameter change you make, cross-check with your fellow shifter - everybody is tired by now so 4 eyes and 2 brains are better than 2/1.

-) if you notice any irregularity, add an entry in the elog explaining your observations. This does not have to be with a run entry - feel free to add a dedicated entry - be honest about what happened and if you can, give a realistic estimate about what runs you think are affected!

-) check that the MIMOSA current on the NI CRATE PC is not above 3.8A. If it is higher, you need to power cycle the telescope: 
    
    -) kill the eudaq run control
    -) on the NI CRATE PC: press disable followed by enable on the LAB VIEW VI that monitors the current (you should see the current going down to muA and then to ~2.5 A)
    -) re-program the MIMOSAS using the JTAG interface (MI26 icon on the desktop)
    -) follow the instructions at: https://telescopes.desy.de/User_manual#2._Starting_sensor:_MimosaJTAG
    -) be sure to use the Threhsold6 file.
    -) in case of problems / you don't understand what you are doing, call G. Auzinger


## INFRASTRUCTURE

This section briefly describes how to start the DAQ of the EUDET Telescope / TLU / FEI4 plane.
### go to the AIDA Telescope control PC
1) navigate in the eudaq/ directory and execute the STARTRUN.cmstk script

        ./STARTRUN.cmstk

this Kills all previous instance of the  EUDAQ runcontrol start the runcontrol and the EUDAQ log & data collector and the online Monitor + brings up the GUI 
once you start, be sure to use the "cms_May16_ext.conf" configuration file.

Before you can do anything further, be sure that the two LabView VIs on the NICrate PC are running. One is the current monitor for the telescope and the second one is the configuration utility (called ANEMONE). Both have to be open, running and must not show errors. Only if this is the case, the telescope startup procedure will work. Try to never close them.

2) on the AIDA Telescope Control PC, open the Remote Desktop client 

there are 2 PCs: NICratePC & USBPixPC1, you will need RDC connections to both:
    
2a) connect to the NICrate PC, go to C:\opt\eudaq\bin and start the

    startNIandTLU.bat

this will bring up two command line windows, one for the NI Producer (Telescope Data) and one for the TLU (trigger handling).

Furthermore, in the runcontrol window on the AIDA control PC, you will see that the NIProducer & TLU producer have connected to the run control.

2b) connect to the USBPixPC1 and start the STProducer by clicking the icon on the Desktop (Start_STControl_eudaq_something)

then you will see the USBpix producer connect in the main RunControl window.

3) Now om the AIDA Telescope PC go the EUDAQ run control GUI and load the correct config file :

        cms_May16_ext.conf

This file contains all necessary settings for: Mimosa telescope, FEI4 plane & TLU. 

8) click "Config". This configures TLU and Telescope - be aware that there is significant lag in the remote connection so be patient and click only once! If you click twice, there is a chance you will have to start the whole procedure again.

4) During the run monitor the errors of the RunControlLog & OnlineMonitor



## Starting a Telescope Run

1) navigate to the EUDAQ run control & click "Start". This starts a run, issues a run number and gets the TLU ready. Also, be patient and click only once. As soon as the scaler and particle counters are 0 the run starts and you have a run number (you can get it also from the LOG collector)

NOTE: The TLU will not issue any triggers before the GLIB DAQ is started!

2) after a run is finished, click Stop on the EUDAQ run control. Be patient and click only once!

2a) if you want to immediately start the next run, wait a bit (see the log collector) and click "Start" again!


## Configuring the GLIB, CBCs etc

Our DAQ is controlled by 1 XDAQ applications that runs on cmsuptracker008:
    -) GlibSupervisor 

0) As a first step, log into the control Pc and ssh to cmsuptracker008 (in the beam area). 

    ssh -XY xtaldaq@cmsuptracker008

once logged in, you can start the GlibStreamer by calling a shell script:

        /home/xtaldaq/CBCDAQ/GlibStreamer/scripts glibConnect.sh 2

(be sure to put the option 2 after the script, otherwise you will have the wrong config)

then, in a browser (on your local PC), the XDAQ main page can be reached on cmsuptracker008 on port 13000

        cmsuptracker008:13000

this brings you the the hyperdaq page where you will have icons both for the the GLIB Supervisor.

1) first go to the GLIBSupervisor, When you launch the shell script, it will automatically initialize, configure the GLIB, write the I2C settins to the CBCs and read them back. If you need to change global I2C settings for both CBCs, you can do this using the Global_CBC_Register tag in the following file:
/home/xtaldaq/CBCDAQ/GlibStreamer/xml/HWDescription_2CBC.xml

If you need to change the Stub correlation window for the various angular scans, this is done by modifying the 

    <GlobalCBCRegister name="MiscStubLogic"> 0xXE</GlobalCBCRegister>

Parameter in the CBCDAQ/GlibSupervisor/xml/HWDescription_2CBC.xml. x is the width of the correlation window (usually between 4 and 7 [hex = decimal!]) and E is the configuration of the stub logic. Never change the E!!

This will then update the CBC configuration the moment you launch the streamer from the script
Be carful not to alter any GLIB parameters as this could screw up running!

2) change to the "Acquisition" tab on the GlibSupervisor page and the default values should be fine. You should only have to change the filepaths to the .daq (for later playback) and the .raw file (for DQM) - you can get the run number from the elog. 

In the fields labelled "Destination file" and "New DAQ format file" enter the following filepath / name:

        Destination file: /cmsuptrackernas/Raw/2016/runxxx.raw
        New DAQ format file: /cmsuptrackernas/DaqData/2016/runxxx.daq

where xxx is the run number from the ELOG (make sure to fill the TLU run number from the EUDAQ run control in the appropriate field).

In the field "NbofAcquistions" change to desired_number_of_events / 10 000. So 30 for 300k Events!

Eventually, depending on the run type, you might also have to change the number of acquisitions (multiply this number by 10000 and you get the number of events that will be recorded).

Click "OK"

3) click the "Condition Data" link and enter the following values in the following fields:

<!--TODO: fix me!-->

    "Data Type"  |   "Value"    |   "Meaning"

    4            | Rotation Ang | Angle of the rotation stage; 5 on the controller for the rotation stage is 0 for us!

    5            | HV Setting   |   High Voltage settin; nominal value is (-)600

    8            |  StubLatency | Stub latency as configured in the HWDescription_2CBC.xml file under the "stubdata_latency_adjust_fe1" register - default value = 4

## Starting a Run on the GLIB DAQ

Head to the ELOG which is available from the beamtest web page:

    cmstkph2-bt.web.cern.ch

There create a new entry for your run with the correct information and submit it. Be carful to correctly fill all fields!

once the TLU / Telescope are running and waiting for our DAQ (particle counter going up but not the trigger counter), navigate to the GLIBSupervisor application and start it by clicking "Enable", which will start a run with the number of events you specified! You can stop it at any time by clicking "Stop"..

Afterwards, once the triggers stop, make sure to stop the run on the Telescope DAQ by pressing "Stop" - be sure to wait appropriately long. If the status in the Connections field does not update, click in the window!

## DONT FORGET TO UPDATE THE ELOG

## Online monitoring the telescope data
The online monitor for the telescope data is running as part of the run control by default so take your time to look at the plots from time to time.

## Running the playback on cmsuptracker007

Go to http://cmsuptracker007.cern.ch:8080/rcms

login as xtaldaq

press    playbackDAQ2CBC-fix1 in the configuration chooser

press    Create

Should see state: Initial

press    InitialiZe

Should see state: Halted

Go to http://cmsuptracker007.cern.ch:41800 in new window/tab

press    GlibStreamer Stre...

Set:

Short pause duration (ms) = 1

Condition data = check

Fake data = check

Enter data or filename: = /cmsuptrackernas/DataDaq/runXXX.daq (where xx is the run number)

press   ok

Go back to http://cmsuptracker007.cern.ch:8080/rcms

press InitialiSe!!

press   Configure

Should see state: Configured

press   start

Should see: Starting

wait...

Should see: Running

Go back to http://cmsuptracker007.cern.ch:41800/urn:xdaq-application:lid=52/validParam

press   Refresh

Should see: "Complete acquisition"

Acquisition number is the number of events so far.

wait... do not press refresh in your browser, but press the "refresh" button on the application

Once its finished:

Go back to http://cmsuptracker007.cern.ch:8080/rcms

Take the run number, and enter it into the "CMS run nr.:" field of the appropriate elog entry.

Go back to http://cmsuptracker007.cern.ch:8080/rcms

press   Destroy

## Checking the prompt reco data

If the automatic data-reconstruction script is running, basic reconstruction data should be produced within a couple of minutes after a playback run was closed.

Output are produced in /cmsuptrackernas/PromptReco in the hist and plot folders. Furthermore, a .csv file with summary for each run is produced (/cmsuptrackernas/PromptReco/summary.csv). You can just scp it from cmsuptracker007, open it with libreoffice and produce your plots interactively.

If the files are not produced or the csv isn't updated with new runs, connect to cmsuptracker007 and launch the promptReco.py script in the home directory.
