---
layout: "sfz/opcode"
opcode_name: "egN_dynamic"
---
When 1, causes envelope segment durations and levels to be recalculated when a MIDI CC message modulating those envelopes is received. When 0, envelope segment durations and levels are calculated only at the start of the particular envelope segment.
## Examples

```
<region>
sample=*saw

eg1_ampeg=1     // Create envelope to control amplitude..
eg1_sustain=1
eg1_level1=1
eg1_level2=0
eg1_time2=4     // ..with a release time of 4 seconds

eg1_time2_oncc1=-8  // assign modwheel to modulate release time

eg1_dynamic=1   // 1 = modulation will affect all notes immediately, or 0 (default) = new segments only
```
