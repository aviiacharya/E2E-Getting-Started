# E2E-Getting-Started
End-to-End Machine Learning refers to the process of developing, deploying, and maintaining machine learning models from start to finish, encompassing all the stages required to take a model from initial conception to its practical application. The key components of End-to-End Machine Learning are data collection, data preprocessing, feature engineering, model selection, model training, model evaluation, model deployment, model monitoring and maintenance, interpretability and explainability, and scalability and performance optimization.

Doing E2E ML with the CMS experiment involves processing raw data from the CMS detector(or you simulate your own data as per CMS requirements, engineering relevant features, training models for particle identification and event classification, deploying these models for real-time or offline analysis, continuously monitoring their performance, and interpreting results to gain insights into particle physics.

Hence to start doing analysis at CMS, you should have a CERN account first. Once you have it, you can connect to lxplus CERN server or lpc server from FermiLab.

### Setting up LXPLUS account
- Start by connecting to the system with "ssh username@lxplus.cern.ch" (SSH or Secure Shell is a protocol used to securely connect to a remote server or computer over a network). You can do the same thing for your lpc userspace. I will explain the log-in details below after this explanation.
- Alright! Now go to the home directory(user). Follow these steps
- Get your grid certificate from https://ca.cern.ch/ca/user/Request.aspx?template=ee2user and import it to your user space ("public" advised).
- Make Sure your account is registered in CERN VOMS(use firefox to do this and have the myCertificate.p12 file inserted in the browser). https://twiki.cern.ch/twiki/bin/view/CMSPublic/SWGuideLcgAccess#How_to_register_in_the_CMS_VO
- In your user space do
  * mkdir .globus
  * cd .globus/
  * mv /afs/cern.ch/user/u/username/myCertificate.p12 
  * rm -f usercert.pem
  * rm -f userkey.pem
  * openssl pkcs12 -in myCertificate.p12 -clcerts -nokeys -out usercert.pem
  * openssl pkcs12 -in myCertificate.p12 -nocerts -out userkey.pem
  * chmod 400 userkey.pem
  * chmod 400 usercert.pem
  * voms-proxy-init --rfc --voms cms -valid 192:00

- Now you have access to make a CMS workfield and install all the required libraries. To make a CMS environment do the following,
  * Go to eos cd /eos/user/u/username
  * export SCRAM_ARCH=slc7_amd64_gcc1
  * cmsrel CMSSW_10_6_20
  * cd CMSSW_10_6_20/
  * cd src/
  * cmsenv
  
- You can change the CMSSW version as per requirement. The “src” directory is where we’ll be doing most work in. “cmsenv” activates an environment that gives you some commands in the command prompt which we’ll need. Cmsenv sets the environment so each time you have to do cmsenv before you run a script. (This is a side note, If you want to check a script or a root file you gonna be working on, it’s better if you download root software to your local computer. I did it using xcode and homebrew installer.)

- Now you have to get all the MLAnalyzer files and scripts. To do this inside your user space, do export SCRAM_ARCH=slc7_amd64_gcc10 , and then do "cmsrel CMSSW_10_6_20". Here you are setting up the CMSSW_10_6_20 installation . Now you should have a CMSSW_10_6_20 directory. Now go inside CMSSW_10_6_20/src . Here you will keep all your MLAnalyzer files and scripts. It is a good idea to keep all the data file and plots etc in a separate folder other than src.

- In this location do cmsenv. Activate Git integration with CMSSW by running the line git cms-init. As you should be already aware Git is our version control software which we use to track changes to our code. Next you will add all the packages . So do,
  * git cms-addpkg Configuration/Generator
- Next to final line adds MLAnalyzer, the repo we’ll actually work on is,
  * git clone https://github.com/rchudasa/MLAnalyzer . (Ruchi's code is the most updated so far.)
- Each time you add a new script or compiler to compile CMSSW,  you shoud run the line "scram b" .
- Next we will run the root file. Let's get it from https://cernbox.cern.ch/s/laQStvb8lAJBmc3. Download the file to your local computer. Transfer it to the MLAnalyzer folder.
  *  To transfer files from local system to lxplus or vice-versa, use these commands "scp file username@lxplus.cern.ch:/afs/cern.ch/user/u/username" and "scp username@lxplus.cern.ch:/afs/cern.ch/user/u/username/file ." respectively.  
- Now run runRHAnalyzer.py “some textfile.txt” . The ".txt" file will contain your downloaded root file. This will create another output.root file that contains all the necessary plots and graphs. There's a script in RHAnalyzer called runRHAnalyzer.py. Go into here and edit the input file to be your mini sample. Also check that the cfg line is RecHitAnalyzer/python/ConfFile_cfg.py. When you run this, it does cmsRun using ConfFile_cfg.py as your "configuration" file, and with the specified input file. The config file is where you actually tell it to run the ntuplyzer (shouldn't have to change anything here yet). If you see an error message you will most likely have to do 
“if (histos['mass'].GetBinContent(iBinX+1) == 0): continue” before the error line in the RHAnalyzer file .
- Great ! You learned the framework. Next, let's dive into some actual work.

#### Our main goal for now is to develop a regressor.
- Remember the steps are Generation - Simulation - NTuplization(RecHitAnalysis)- Training. An N-tuple is essentially a structured data format that organizes information in rows and columns, much like a table or a spreadsheet, but is optimized for handling large datasets typical of particle physics such as our CMS detector physiscs.
 
- To start, check whether you have the GeneratorInterface directory inside the src directory. If not, click on [GeneratorInterface](https://github.com/aviiacharya/E2E-Getting-Started/blob/main/GeneratorInterface.zip) . You will have the folder downloaded. Now upload the file in the src folder inside lxplus. Unpack it and do scram b. The filters will be compiled. This is a collection of plugins which are needed for doing the Monte Carlo generation. It includes the main thing, "Pythia8PtGunV3" which we give certain parameters and it simulates these particle decays. So this is a sample generation config file. It calls upon Py8PtGunV3 and gives it a kinematic range (mass and pt), and tells it the particle ID to give the mother particle (25, for Higgs). But it also calls on a "genHToEleEleFilter" which is an EDFilter that applies a cut on pt, eta, and dR. You can write your own EDFilter for this task, or see if Ana Maria's code already includes one. "genHToEleEleFilter" is just an example. The place to put your new EDFilter will be in "CMSSW_10_6_20/src/GeneratorInterface/GenFilters/plugins/". The EDFilter is a process which looks at an event and determines which particles pass the criteria established in the code. What you have to do is modify the logic in the code, like, for example, on line 56 of GenHToEleEleFilter.cc: "if ( abs(iGen->pdgId()) != 25 || iGen->numberOfDaughters() != 2 ) continue;" This is saying, if the particle isn't a Higgs (id = 25) or the number of daughter particles (particles emerging from the decayed particle) isn't 2, skip this particle. You could copy this plugin but change the pdgId to 6 for the top quark, instead of 25 for the Higgs. Now there is a gen_..._cfg.py script that is the config file which runs with CMSSW to do the generation. It sets everything up, imports all the packages we need, and at line 83, you can see that we call on the Pythia8PtGunV3 to generate certain decays.

- Okay? All Good ? Let’s generate the samples. Run the gen_..._cfg.py script using cmsRun. "cmsRun gen_..._cfg.py". The output will be a .root file containing the Generated sample. You will also have do the necessary changes in this file before running. The output will be a .root file containing the Generated sample. These are just the particle based data without the detector physics. But we can do it in our workspace for only a small number of events, say 1k. To run a large number of events you will have to use [condor workspace](https://abpcomputing.web.cern.ch/computing_resources/cernbatch/)  from lxplus or [crab files](https://twiki.cern.ch/twiki/bin/view/CMSPublic/SWGuideCrab) for lpc workspace.
- Now there is a sim_..._cfg.py file or we call it sim script what takes the generated data (or "gen-level" data) and simulates according to the CMS detector. Now it actually is formatted like the data you would get from the detector, with raw detector data from pixels, ECAL, HCAL, etc., as well as reconstructed information like jets and tracks. You can use the output of this in an EDAnalyzer just like real CMS data.
- So, that was a lot of information. I just want to be thorough.
- Next is ntupules. And Parquet conversion. Use RHAnalyzer the same way to get ntuple files. There's a script in MLAnalyzer called "convert_root2pq_jet_tracker.py". It takes one ntuple root file as input and turns it into the parquet file. Set up a python2 environment, because the parquet conversion (at the moment) is only compatible with python 2. Everything you are doing, are within the src directory. I will have a separate block to explain crab file setup and the running process.
- Alright, now that you have the parquet files, you can start the actual training.
- But first let's generate the files.

### Gen script and sim script 
- Instructions [here](https://github.com/aviiacharya/CMSFiles/)
  
### Training

### Condor and Crab setup

 

  







### Plotting
- This document explains the process from begining : Starting from plotting the 1D, 2D histograms from the miniaod [root file](https://github.com/aviiacharya/E2E-Getting-Started/blob/main/GetPlotsFromRootFile.ipynb). 

-But you may need to generate your own data sets as AOD or miniAOD files as per your need.
