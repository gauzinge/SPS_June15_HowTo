# SPS_NOV15_HowTo
In case there is a problem with the beam call 77500. 
## INFRASTRUCTURE

This section briefly describes how to start the DAQ of the EUDET Telescope / TLU / FEI4 plane.
### on the cmsuptracker006 machine in the barrack
1)  ssh into the telescope PC:

        ssh -XY telescope@pcaidarc

2) on the remote machine (pcaidarc) navigate in the eudaq/ directory and execute the STARTRUN.cmstk script

        ./STARTRUN.cmstk

this Kills all previous instance of the  EUDAQ runcontrol start the runcontrol and the EUDAQ log collector and brings up the GUI 
once you start, be sure to use the "cms_Nov15_internal/external" configuration file

3) On the CMSuptracker006 open a new terminal, start a remote connection to the tbdaqpc01.cern.ch (STControl) via 

        rdesktop-vrdb -g 1600x900 tbdaqpc01.cern.ch
        User: TestBeamAdmin
You will find the PW attached to the monitor

4) Open the Developer Command Line on the Desktop and launch: 

        E:\icwiki_svn\USBPix\host\tags\release-5.3\bin\STControl_eudaq.exe -r 192.168.5.251

this is the producer for the FEI4 plane
4) Start a remote connection to the flhaida.cern.ch (STControl) via

        rdesktop-vrdb -g 1600x900 flhaida.cern.ch
        User: telescope
You will find the PW attached to the monitor

5) on the NI Remote Desktopt navigate to C:\Desktop\old in the windows explorer and launch the 

        StartNIproducer_and_TLU.bat

this connnects the TLU and the telescope to the EUDAQ run control


6) Now om cmsuptracker go the EUDAQ run control GUI and load the correct config file (remember, you are working on a remote PC!):

        cms_Nov15_internal/external.conf

This file contains all necessary settings for: Mimosa telescope, FEI4 plane & TLU. Be aware that the filename without the ".conf" has to be copied in the "Config" field.

7) click "Config". This configures TLU and Telescope - be aware that there is significant lag in the remote connection so be patient and click only once!

8) During the run monitor the errors of the RunControlLog



## Starting a Telescope Run

1) navigate to the EUDAQ run control & click "Start". This starts a run, issues a run number and gets the TLU ready. Also, be patient and click only once. As soon as the scaler and particle counters are 0 the run starts and you have a run number (you can get it also from the LOG collector)

NOTE: The TLU will not issue any triggers before the GLIB DAQ is started!

2) after a run is finished, click Stop on the EUDAQ run control. Be patient and click only once!

2a) if you want to immediately start the next run, wait a bit (see the log collector) and click "Start" again!


## Configuring the GLIB, CBCs etc

Our DAQ is controlled by 2 XDAQ applications that run on cmsuptracker008:
    -) GlibSupervisor (for control and configuration)
    -) GlibStreamer (for data readout)

they have to be launched together by calling a shell script:

        /home/xtaldaq/glibConnect.sh

then, in a browser, the XDAQ main page can be reached on the localhost on port 13000

        127.0.0.1:13000

you can either configure GLIB & CBCs by using the GLIBSupervisor which is rather cumbersome because you have to click many buttons in the correct order and you can not screw up. The second possibility is to use the Ph2_ACF framework to write the parameters to the CBCs (the GLIB is in it's default config anyway). 

To do so, in a terminal do the following:

        cd Ph2_ACF
        source setup.sh

open an editor of your choice and open the file:

        ~/Ph2_ACF/settings/Beamtest_Nov15.xml

this holds the GLIB and CBC parameters. In order to change the threshold for example, just change the HEX number in the line that says <GlobalCBCRegister> . The three lines below point to the CBC config files that are currently being used. Editing the Global_CBC_Register node avoids having to change all CBC files manually. Things defined in this node take precedence over the settings from the CBC files!

Then, on the terminal, use the systemtest application to upload the config to GLIB & CBCs:

        ~/Ph2_ACF/bin/systemtest -f settings/Beamtest_Nov15.xml -c

or from the ACF folder:

        systemtest -f settings/Beamtest_Nov15.xml -c

Be carful not to alter any GLIB parameters as this could screw up running!

## Starting a Run on the GLIB DAQ

once the TLU / Telescope are running and waiting for our DAQ, navigate to the GLIBStreamer application and configure it by clicking "configure".

In the fields labelled "Destination file" and "New DAQ format file" enter the following filepath / name:

        Destination file: /cmsuptrackernas/raw/runxxx.raw
        New DAQ format file: /cmsuptrackernas/DaqData/runxxx.daq

where xxx is the TLU run number read from the EUDAQ run control.

Then, in the "Number of Acquisitions" field enter the number of events you would like to record / 1000. (i.e. 400 for 400k Events - the default packet size for our DAQ is 999 which is 1000 in reality as it is zero-indexed).

Click the "Conditions Data" field and fill the fields for HighVoltage, Angle and in the third, enter the current threshold in decimal, then click OK. 

Now the GlibStreamer is configured correctly.

Head to the ELOG which runs on cmsuptracker003 (if a tunnel is currently running). You will find it in the browser history on 

        localhost:8080

There create a new entry for your run with the correct information and submit it. Be carful to correctly fill all fields!

Once this is done, click "Start" which will start a run with the number of events you specified! You can stop it at any time by clicking "Stop".



## Online monitoring the telescope data
Once you have started the run you may have a look at the online data of the telescope: 
Connect to the telescope master control pc:

Use the Programm:

        /home/telescope/BELLETEST/eudaq-neu/build/monitors/onlinemon/OnlineMon.exe -sc 1000 -f data/run000369.raw
        
you can play around with the options if you do not want to wait for all events to be processed. 


