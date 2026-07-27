# Setup for Heavy Ion Foresting
Implements modifications by [@kovbalcsab](https://github.com/kovbalcsab) to
enable integration of the CMS Dilepton group's 
[OniaTreeSubmodule](https://github.com/CMS-HIN-dilepton/OniaTreeSubmodule/tree/CMSSW_13_2_X).

> [!NOTE]
> The original instructions can be found here:
> https://twiki.cern.ch/twiki/bin/view/CMS/HiDileptonWorkingAreaSetting

1) From lxplus, get a recent CMSSW_13_2_X version:
```bash
cmsRel CMSSW_13_2_15
cd CMSSW_13_2_15/src/
cmsenv
```

2) Merge the Heavy Ion foresting tools, and compile:
```bash
git cms-merge-topic CmsHI:forest_CMSSW_13_2_X
git remote add cmshi git@github.com:CmsHI/cmssw.git
scram b -j8
```

3) Add this repo and recompile:
```bash
git remote add oniaforest git@github.com:jdlang/OniaTreeSubmodule.git
git fetch oniaforest
git checkout oniaforest/CMSSW_13_2_X_ForestIntegration HiAnalysis
git checkout oniaforest/CMSSW_13_2_X_ForestIntegration HiSkim
git checkout oniaforest/CMSSW_13_2_X_ForestIntegration HeavyIonsAnalysis
scram b -j8
```

4) Up
