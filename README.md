# E2E-Getting-Started
End-to-End Machine Learningrefers to the process of developing, deploying, and maintaining machine learning models from start to finish, encompassing all the stages required to take a model from initial conception to its practical application. The key components of End-to-End Machine Learning are data collection, data preprocessing, feature engineering, model selection, model training, model evaluation, model deployment, model monitoring and maintenance, interpretability and explainability, and scalability and performance optimization.

Doing E2E ML with the CMS experiment involves processing raw data from the CMS detector(or you simulate your own data as per CMS requirements, engineering relevant features, training models for particle identification and event classification, deploying these models for real-time or offline analysis, continuously monitoring their performance, and interpreting results to gain insights into particle physics.

Hence to start doing analysis at CMS, you should have a CERN account first. Once you have it, you can connect to lxplus CERN server or lpc server from FermiLab.

### Setting up LXPLUS account
- Start by connecting to the system with ssh username@lxplus.cern.ch (SSH or Secure Shell is a protocol used to securely connect to a remote server or computer over a network).
- Go to the home directory(user). Follow these steps
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
(You can change the CMSSW version as per requirement)






### Plotting
- This document explains the process from begining : Starting from plotting the 1D, 2D histograms from the miniaod [root file](https://github.com/aviiacharya/E2E-Getting-Started/blob/main/GetPlotsFromRootFile.ipynb). 

-But you may need to generate your own data sets as AOD or miniAOD files as per your need.
