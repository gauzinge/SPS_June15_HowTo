# SPS_NOV15_HowTo
In case there is a problem with the beam call 77500. 
If there are problems with the telescope / DAQ, call G. Auzinger
## INFRASTRUCTURE

This section briefly describes how to start the DAQ of the EUDET Telescope / TLU / FEI4 plane.
### go to the AIDA Telescope control PC
1) navigate in the eudaq/ directory and execute the STARTRUN.cmstk script

        ./STARTRUN.cmstk

this Kills all previous instance of the  EUDAQ runcontrol start the runcontrol and the EUDAQ log & data collector and the online Monitor + brings up the GUI 
once you start, be sure to use the "cms_Nov15_internal/external" configuration file

2) on the AIDA Telescope Control PC, open the Remote Desktop client 

there are 2 PCs: NICratePC & USBPixPC1, you will need RDC connections to both:
    
2a) connect to the NICrate PC, go to C:\opt\eudaq\bin and start the

    startNIandTLU.bat

this will bring up two command line windows, one for the NI Producer (Telescope Data) and one for the TLU (trigger handling).

Furthermore, in the runcontrol window on the AIDA control PC, you will see that the NIProducer & TLU producer have connected to the run control.

2b) connect to the USBPixPC1 and start the STProducer by clicking the icon on the Desktop (Start_STControl....)

then you will see the USBpix producer connect in the main RunControl window.

3) Now om the AIDA Telescope PC go the EUDAQ run control GUI and load the correct config file :

        cms_Nov15_internal/external.conf

This file contains all necessary settings for: Mimosa telescope, FEI4 plane & TLU. Be aware that the filename without the ".conf" has to be copied in the "Config" field. cms_Nov15_internal.conf is for running with internal triggers, _ext is for the scintillator trigger.

8) click "Config". This configures TLU and Telescope - be aware that there is significant lag in the remote connection so be patient and click only once! If you click twice, there is a chance you will have to start the whole procedure again.

4) During the run monitor the errors of the RunControlLog & OnlineMonitor



## Starting a Telescope Run

1) navigate to the EUDAQ run control & click "Start". This starts a run, issues a run number and gets the TLU ready. Also, be patient and click only once. As soon as the scaler and particle counters are 0 the run starts and you have a run number (you can get it also from the LOG collector)

NOTE: The TLU will not issue any triggers before the GLIB DAQ is started!

2) after a run is finished, click Stop on the EUDAQ run control. Be patient and click only once!

2a) if you want to immediately start the next run, wait a bit (see the log collector) and click "Start" again!


## Configuring the GLIB, CBCs etc

Our DAQ is controlled by 2 XDAQ applications that run on cmsuptracker008:
    -) GlibSupervisor (for control and configuration)
    -) GlibStreamer (for data readout)

0) As a first step, log into the control Pc and ssh to cmsuptracker008 (in the beam area). 

    ssh -XY xtaldaq@cmsuptracker008

once logged in, you can start GlibStreamer/Supervisor by calling a shell script:

        /home/xtaldaq/CBCDAQ/GlibStreamer/scripts glibConnect.sh 16

(be sure to put the option 16 after the script, otherwise you will have the wrong config)

then, in a browser (on your local PC), the XDAQ main page can be reached on cmsuptracker008 on port 13000

        cmsuptracker008:13000

this brings you the the hyperdaq page where you will have icons both for the GLIBStreamer & the GLIB Supervisor. Open both in new Tabs.

1) first go to the GLIBSupervisor, click initialize, wait for the state-change and then click configure.
2) on the GlibStreamer page, select "Load Parameters from file" and put the following file in the box:
    
    /home/xtaldaq/streamerValues.txt

click "LoadfromFile". This will update the settings and put the correct values in the fields. All you have to do is update the run number to the one you derive from the ELOG (cmstkph2-bt.web.cern.ch). After you have done so, click configure. 

In the fields labelled "Destination file" and "New DAQ format file" enter the following filepath / name:

        Destination file: /cmsuptrackernas/Raw/runxxx.raw
        New DAQ format file: /cmsuptrackernas/DaqData/runxxx.daq

where xxx is the run number from the ELOG (make sure to fill the TLU run number from the EUDAQ run control in the appropriate field).

(old text: Then, in the "Number of Acquisitions" field enter the number of events you would like to record / 1000. (i.e. 400 for 400k Events - the default packet size for our DAQ is 999 which is 1000 in reality as it is zero-indexed).

Click the "Conditions Data" field and fill the fields for HighVoltage, Angle and in the third, enter the current threshold in decimal, then click OK. 

Now the GlibStreamer is configured correctly.)

Eventually, depending on the run type, you might also have to change the number of acquisitions (multiply this number by 1000 and you get the number of events that will be recorded).

**Make sure that the "Break Trigger during block read" is not ticked!**

now go to conditions data and enter the data following the list in the AIDA counting room. 

2) before you can start a run, you need to configure the GLIB readout board and the CBC chips on the module. This could be done with the GLIBSupervisor application running in another tab but it is rather cumbersome and involves a lot of clicking. It is much easier to do the following.

-) open another terminal and 

    ssh -XY xtaldaq@cmsuptracker008

once logged in:

    cd Ph2_ACF
    source setup.sh

If you have to modify settings other than the Threshold (VCth), open the file

        ~/Ph2_ACF/settings/Beamtest_Nov15.xml

and edit accordingly. The Stub latency is a parameter of the GLIB board and is enclosed in a <Register> xml node. Be aware that you have to set it for both front-ends. 

**This, in theory, is expert-only work as it could mess up the data.** 

Now you can configure the setup by typing (in the command line)

    configure -v "threshold in hex"

Since we will mostly be doing threshold scans, check in the elog & the measurement programm what threshold setting should be applied for this run and enter the value in hex, using the format "0xVV" where value stands for the number (8bit!) Be aware that the threshold is increased by lowering this number!

I advise you to re-configure the CBCs & GLIB before every run, also if we are doing angular scans - just to make sure that we use the right parameters and the data is sound. For angular scans at a fixed threshold (0x73), it should suffice to run the configure script without any parameter.

If you need to change the Stub correlation window for the various angular scans, this is done by modifying the 

    <GlobalCBCRegister name="MiscStubLogic"> xE</GlobalCBCRegister>

Parameter in the Beamtest_Nov15.xml. x is the width of the correlation window (usually between 4 and 7 [hex = decimal!]) and E is the configuration of the stub logic. Never change the E!!

this last step will download all settings to the GLIB and the CBCs and read back the threshold. If you get an error message saying that it was not possible to read back a value in 5 iterations, try again.

If you get readback errors all over the place, for all CBCs, you will have to power cycle the front-end GLIB. This is done by going across the bridge to the corridor that runs along the middle of the building. Beside the access door to our zone, there is a green rack and by the foot of that is a power socket. Just unplug the connector there, re-plug it and try to run the configuration again. If it still does not work, repeat until it does. If the problem persists after ~10 tries, call G. Auzinger   

Be carful not to alter any GLIB parameters as this could screw up running!

## Starting a Run on the GLIB DAQ

Head to the ELOG which is available from the beamtest web page:

    cmstkph2-bt.web.cern.ch

There create a new entry for your run with the correct information and submit it. Be carful to correctly fill all fields!

once the TLU / Telescope are running and waiting for our DAQ (particle counter going up but not the trigger counter), navigate to the GLIBStreamer application and start it by clicking start, which will start a run with the number of events you specified! You can stop it at any time by clicking "Stop"..

## Online monitoring the telescope data
The online monitor for the telescope data is running as part of the run control by default so take your time to look at the plots from time to time.

## Running the playback on cmsuptracker007

Go to http://cmsuptracker007.cern.ch:/8080/rcms

login as xtaldaq

press    playbackDAQ

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

Go back to http://cmsuptracker007.cern.ch:/8080/rcms

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

Go back to http://cmsuptracker007.cern.ch:/8080/rcms

Take the run number, and enter it into the "CMS run nr.:" field of the appropriate elog entry.

Go back to http://cmsuptracker007.cern.ch:/8080/rcms

press   Destroy

## Checking the prompt reco data

If the automatic data-reconstruction script is running, basic reconstruction data should be produced within a couple of minutes after a playback run was closed.

Output are produced in /cmsuptrackernas/PromptReco in the hist and plot folders. Furthermore, a .csv file with summary for each run is produced (/cmsuptrackernas/PromptReco/summary.csv). You can just scp it from cmsuptracker007, open it with libreoffice and produce your plots interactively.

If the files are not produced or the csv isn't updated with new runs, connect to cmsuptracker007 and launch the promptReco.py script in the home directory.
