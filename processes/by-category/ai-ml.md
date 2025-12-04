# AI/ML Processes

Processes involved in machine learning, Apple Intelligence, and on-device inference.

## Apple Intelligence Platform

### intelligenceplatformd
**Path**: `/System/Library/PrivateFrameworks/IntelligencePlatformCore.framework/intelligenceplatformd`

Primary orchestration daemon for Apple Intelligence:
- Feature availability checks
- Request routing
- Model coordination
- Privacy enforcement

**Log predicate**:
```bash
--predicate 'process == "intelligenceplatformd"'
```

### IntelligencePlatformComputeService
**Path**: XPC service within `IntelligencePlatformCompute.framework`

ML compute backend:
- Model inference execution
- Neural Engine coordination
- GPU/CPU fallback

---

## Model Management

### modelcatalogd
**Path**: `/System/Library/PrivateFrameworks/ModelCatalogRuntime.framework/modelcatalogd`

Model catalog management:
- Available models inventory
- Model version tracking
- Download coordination

**Key logs**:
- Model availability queries
- Download requests
- Version checks

### modelmanagerd
**Path**: `/usr/libexec/modelmanagerd`

Model lifecycle:
- Model loading/unloading
- Memory management
- Cache management

### mlhostd
**Path**: `/usr/libexec/mlhostd`

ML model hosting:
- Process isolation for models
- Resource allocation
- Inference hosting

---

## Siri Intelligence

### siriinferenced
**Path**: `/System/Library/PrivateFrameworks/SiriInference.framework/Support/siriinferenced`

On-device Siri ML:
- Intent classification
- Named entity recognition
- Query understanding

**High forensic value** - Shows local Siri processing activity.

### assistantd
**Path**: `/System/Library/PrivateFrameworks/AssistantServices.framework/assistantd`

Main Siri daemon:
- Voice recognition coordination
- Intent handling
- Response generation

### siriknowledged
**Path**: `/usr/libexec/siriknowledged`

Siri knowledge graph:
- Entity relationships
- Personal context
- Query augmentation

### siriactionsd
**Path**: `/System/Library/PrivateFrameworks/VoiceShortcuts.framework/Support/siriactionsd`

Shortcuts and actions:
- Shortcut execution
- App intent handling
- Automation triggers

### sirittsd
**Path**: `/System/Library/PrivateFrameworks/SiriTTSService.framework/sirittsd`

Text-to-speech:
- Voice synthesis
- Audio output

---

## Media Analysis

### mediaanalysisd
**Path**: `/System/Library/PrivateFrameworks/MediaAnalysis.framework/mediaanalysisd`

Media ML processing:
- Video analysis
- Audio analysis
- Content classification

**Forensic interest**: Shows ML processing of user media.

### photoanalysisd
**Path**: `/System/Library/PrivateFrameworks/PhotoAnalysis.framework/photoanalysisd`

Photo ML:
- Face detection/recognition
- Scene classification
- Object detection
- Memory curation

**Critical for forensics** - Processes all photos with ML.

---

## Search & Indexing

### corespotlightd
**Path**: System service (multiple locations)

Spotlight indexing:
- Content indexing
- Semantic search
- Cross-app search

**Key metric**: High activity indicates content processing.

### spotlightknowledged
**Path**: `/System/Library/Frameworks/CoreSpotlight.framework/spotlightknowledged`

Search knowledge:
- Search ranking
- Relevance scoring
- Personalization

---

## Behavioral Intelligence

### duetexpertd
**Path**: `/usr/libexec/duetexpertd`

Prediction engine:
- App usage prediction
- Timing optimization
- Resource pre-allocation

### coreduetd
**Path**: `/usr/libexec/coreduetd`

Duet scheduler:
- Background task scheduling
- Power-aware execution
- Activity prediction

### contextstored
**Path**: `/System/Library/PrivateFrameworks/CoreDuetContext.framework/Resources/contextstored`

Context storage:
- User context tracking
- Location context
- Activity context

---

## Personalization

### parsecd
**Path**: `/System/Library/PrivateFrameworks/CoreParsec.framework/parsecd`

Siri/Search personalization:
- Query personalization
- Contact ranking
- App ranking

**High forensic value** - Personal behavior modeling.

### parsec-fbf
**Path**: `/System/Library/PrivateFrameworks/CoreParsec.framework/parsec-fbf`

Parsec feedback:
- Model feedback
- Personalization updates

---

## Fitness Intelligence

### fitnessintelligenced
**Path**: `/System/Library/PrivateFrameworks/FitnessIntelligenceDaemonCore.framework/fitnessintelligenced`

Fitness ML:
- Workout detection
- Activity classification
- Health predictions

### FitnessIntelligenceInferenceService
**Path**: XPC service

Fitness inference:
- Real-time activity recognition
- Motion analysis

---

## Call Intelligence

### callintelligenced
**Path**: `/System/Library/PrivateFrameworks/CallIntelligence.framework/callintelligenced`

Call ML:
- Voicemail transcription
- Call classification
- Spam detection

---

## Services Intelligence

### servicesintelligenced
**Path**: `/System/Library/PrivateFrameworks/ServicesIntelligence.framework/servicesintelligenced`

Services ML:
- App suggestions
- Service recommendations

---

## Process Counting

```bash
# Count AI/ML processes
awk 'NR>1 {print $NF}' ps.txt | \
    grep -iE "intelligence|siri|ml|model|inference|neural|media|photo|context|duet|parsec" | \
    wc -l
```

## Log Analysis

```bash
# Combined AI process logs
log show --archive system_logs.logarchive \
    --predicate '
        process == "intelligenceplatformd" OR
        process == "modelcatalogd" OR
        process == "siriinferenced" OR
        process == "photoanalysisd" OR
        process == "mediaanalysisd"
    ' \
    --style json
```

## Delta Comparison

```bash
for proc in intelligenceplatformd modelcatalogd siriinferenced; do
    echo "=== $proc ==="
    for archive in baseline enabled disabled; do
        count=$(log show --archive "$archive/system_logs.logarchive" \
            --predicate "process == \"$proc\"" \
            --style json 2>/dev/null | grep -c '"timestamp"')
        echo "$archive: $count events"
    done
done
```

---

## See Also

- [privacy.md](privacy.md) - Privacy processes
- [../index.md](../index.md) - All processes
- [../../subsystems/intelligence.md](../../subsystems/intelligence.md) - AI subsystems
