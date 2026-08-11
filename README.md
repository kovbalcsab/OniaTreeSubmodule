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
git remote add oniaforest git@github.com:MITHIG/OniaTreeSubmodule.git
git fetch oniaforest
git checkout oniaforest/CMSSW_13_2_X_ForestIntegration HiAnalysis
git checkout oniaforest/CMSSW_13_2_X_ForestIntegration HiSkim
git checkout oniaforest/CMSSW_13_2_X_ForestIntegration HeavyIonsAnalysis
scram b -j8
```

4) Enable the Onia Tree in your forest config by adding the following
below somewhere below your `process.forest` initialization:
```python
# Onia J/psi reco + ntuple
from HiAnalysis.HiOnia.oniaTreeAnalyzer_cff import oniaTreeAnalyzer
from HiSkim.HiOnia2MuMu.onia2MuMuPAT_cff import changeToMiniAOD

# No trigger categorization needed if you only want J/psi reco + decay length
oniaTriggerList = {
    'DoubleMuonTrigger': cms.vstring(),
    'SingleMuonTrigger': cms.vstring(
        "HLT_HIUPC_SingleMuOpen_NotMBHF2AND_v",
        "HLT_HIUPC_SingleMuOpen_NotMBHF2AND_MaxPixelCluster1000_v",
        "HLT_HIUPC_SingleMuOpen_BptxAND_MaxPixelCluster1000_v"
    ),
}

oniaTreeAnalyzer(
    process,
    muonTriggerList = oniaTriggerList,
    HLTProName = 'HLT',
    muonSelection = "GlbOrTrk",
    L1Stage = 2,
    isMC = False,
    pdgID = (443), # This is now a vector! add all onia channels enabled in MC
    outputFileName = OUTPUT_FILE_NAME,
    muonlessPV = False, # Set True for non-prompt MC
    doTrimu = False,
    doDimuTrk = False,
    flipJpsiDir = 0,
    OnlySingleMuons = False,
    getObjectsBy = "vector",
)

# J/psi candidate building
process.onia2MuMuPatGlbGlb.dimuonSelection = cms.string("")
process.onia2MuMuPatGlbGlb.lowerPuritySelection = cms.string(
    "abs(eta) < 2.4 && (isTrackerMuon || isGlobalMuon)"
)
process.onia2MuMuPatGlbGlb.onlySoftMuons = cms.bool(False)
process.onia2MuMuPatGlbGlb.addCommonVertex = cms.bool(True)
process.onia2MuMuPatGlbGlb.addMuonlessPrimaryVertex = cms.bool(False)
process.onia2MuMuPatGlbGlb.resolvePileUpAmbiguity = cms.bool(True)

# Keep only the J/psi ntuple content you care about
process.hionia.checkTrigNames = cms.bool(False)
process.hionia.useSVfinder = cms.bool(False)
process.hionia.fillTree = cms.bool(True)
process.hionia.fillHistos = cms.bool(False)
process.hionia.fillSingleMuons = cms.bool(True)
process.hionia.fillRecoTracks = cms.bool(True)
process.hionia.onlySingleMuons = cms.bool(False)
process.hionia.useBeamSpot = cms.bool(False) # writes PV-based ctau: ppdlPV / ppdlPV3D
process.hionia.useEvtPlane = cms.untracked.bool(False)
process.hionia.storeSameSign = cms.bool(True)
process.hionia.applyCuts = cms.bool(False)
process.hionia.AtLeastOneCand = cms.bool(False)
process.hionia.mom4format = cms.string("vector")
process.hionia.isHI = cms.untracked.bool(False)
process.hionia.isPA = cms.untracked.bool(False)
process.hionia.isUPC = cms.untracked.bool(True)
process.hionia.isMC = cms.untracked.bool(True)
process.hionia.genealogyInfo = cms.bool(True)
process.hionia.oniaPDG = cms.vint32(443)
process.hionia.isPromptMC = cms.untracked.bool(True)

# MiniAOD adaptation; keep this minimal for first working setup
changeToMiniAOD(process, addIsolation=False)

# Preserve the forest muon setting after changeToMiniAOD()
process.unpackedMuons.muonSelectors = cms.vstring()

# Replace default PV input for miniAOD compatibility
process.hionia.primaryVertexTag = cms.InputTag("offlineSlimmedPrimaryVertices")
process.onia2MuMuPatGlbGlb.primaryVertexTag = cms.InputTag("offlineSlimmedPrimaryVertices")
process.patMuonsWithoutTrigger.pvSrc = cms.InputTag("offlineSlimmedPrimaryVertices")

process.onia2MuMuPatGlbGlb.genParticles = cms.InputTag("prunedGenParticles")
process.genMuons.src = cms.InputTag("prunedGenParticles")
process.muonMatch.src = cms.InputTag("unpackedMuons")
process.hionia.genParticles = cms.InputTag("prunedGenParticles")

# Separate path; your existing filterSequence-prepend loop will also hit this path
process.oniaPath = cms.Path(process.oniaTreeAna)
```

5) You must also modify the root output lines in your forest config for
the Onia Tree config to use the same file, like this:
```python
# root output
OUTPUT_FILE_NAME = "HiForestWithOnia.root"
process.TFileService = cms.Service(
    "TFileService",
    fileName = cms.string(OUTPUT_FILE_NAME)
)
```
