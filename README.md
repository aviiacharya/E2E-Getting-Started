# E2E-Getting-Started
End-to-End Machine Learningrefers to the process of developing, deploying, and maintaining machine learning models from start to finish, encompassing all the stages required to take a model from initial conception to its practical application. The key components of End-to-End Machine Learning are data collection, data preprocessing, feature engineering, model selection, model training, model evaluation, model deployment, model monitoring and maintenance, interpretability and explainability, and scalability and performance optimization.

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
  * git clone https://github.com/rchudasa/MLAnalyzer . 
- Each time you add a new script or compiler to compile CMSSW,  you shoud run the line "scram b" .
- Next we will run the root file. Let's get it from https://cernbox.cern.ch/s/laQStvb8lAJBmc3. Download the file to your local computer. Transfer it to the MLAnalyzer folder.
  *  To transfer files from local system to lxplus or vice-versa, use these commands "scp file username@lxplus.cern.ch:/afs/cern.ch/user/u/username" and "scp abachary@lxplus.cern.ch:/afs/cern.ch/user/u/username/file ." respectively.  
- Now run runRHAnalyzer.py “some textfile.txt” . The ".txt" file will contain your downloaded root file. This will create another output.root file that contains all the necessary plots and graphs. There's a script in RHAnalyzer called runRHAnalyzer.py. Go into here and edit the input file to be your mini sample. Also check that the cfg line is RecHitAnalyzer/python/ConfFile_cfg.py. When you run this, it does cmsRun using ConfFile_cfg.py as your "configuration" file, and with the specified input file. The config file is where you actually tell it to run the ntuplyzer (shouldn't have to change anything here yet). If you see an error message you will most likely have to do 
“if (histos['mass'].GetBinContent(iBinX+1) == 0): continue” before the error line in the RHAnalyzer file .
- Great ! You learned the framework. Next, let's dive into some actual work.

  







### Plotting
- This document explains the process from begining : Starting from plotting the 1D, 2D histograms from the miniaod [root file](https://github.com/aviiacharya/E2E-Getting-Started/blob/main/GetPlotsFromRootFile.ipynb). 

-But you may need to generate your own data sets as AOD or miniAOD files as per your need.
