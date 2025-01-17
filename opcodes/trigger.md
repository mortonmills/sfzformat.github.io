---
layout: "sfz/opcode"
opcode_name: "trigger"
---
Values can be:

- **attack** : (Default): Region will play on note-on.
- **release**: Region will play on note-off or sustain pedal off. The velocity
               used to play the note-off sample is the velocity value of the
               corresponding (previous) note-on message.
- **first**: Region will play on note-on, but if there's no other note going on
             (commonly used for or first note in a legato phrase).
- **legato**: Region will play on note-on, but only if there's a note going on
              (notes after first note in a legato phrase).
- **release_key**: Region will play on note-off. Ignores sustain pedal.

## Practical Considerations

This entire section is dedicated to release triggers, which can get quite complex.

Setting trigger to release or release_key will cause the region to play as if
[loop_mode] was set to **one_shot**.

Beyond that, release region behavior varies considerably between SFZ players.

rcg sfz:
* Both release or release_key regions play whether there is a previous attack region or not.
* Release_key regions won't play when releasing the sustain pedal. Release samples only play
when sustain pedal is up (not depressed).
* In sfz v1, there is no polyphony or note_polyphony for limiting the number of simultaneous
release regions playing, which can result in a very large number of release regions triggered
when the sustain pedal is raised.

DropZone and other Cakewalk players:
* Both release or release_key require a previous attack region.
* Release_key regions won't play when releasing the sustain pedal. Release samples only play
when sustain pedal is up (not depressed).
* To make release or release_key plays without any previous attack region, set
[rt_dead] to on.
* [note_polyphony] is now available to control the accumulation of
release voices of repeated notes when Sustain pedal is down.

ARIA and Sforzando:
* release requires a previous attack region.
* release_key doesn't require a previous attack region.
* release responds to Sustain Pedal position.
* release_key ignores Sustain Pedal completely.
* rt_dead is not supported, and defaults to off.
* note_polyphony is available to control the accumulation of release voices.

Release samples can require a corresponding region with trigger set to attack to be active at
the moment when the note-off message is received, or the release region will never play.
In rgc sfz, this is not required. In DropZone and other Cakewalk players, it is required
unless [rt_dead] is set to on. In ARIA, it is required for trigger=release
regions but not for trigger=release_key regions.

For cases where a corresponding attack region is required, here is more detail.
An attack region is considered corresponding if it has the same MIDI note number,
and the same velocity range, as the release region. The velocity which matters here is
the note-on velocity of the initial region - not the velocity of the note-off message
which triggers the release region. Round robins do not need to match, so it is possible
to for example have five round robins for releases and only four round robins for
attacks.

This corresponding attack region is then used to calculate the volume of the release
region based on the attack region's velocity and [rt_decay]. If there is
no corresponding attack region, or the corresponding attack region has finished playing due
to reaching sample end etc, then the release region will not play. This is designed primarily
designed for piano release samples.

Triggering a release sample without a corresponding attack region is is useful for release
samples which are noises not dependent on the volume of any corresponding note, such
as hurdy-gurdy key returns, which will sound the same whether the wheel is turning
or not.

Note that at least in ARIA and Sforzando, a note-on event which triggers multiple regions
(for example a multimic instrument, or one with simulated unison) will have multiple
corresponding regions for the release region, causing the release region to be triggered
multiple times. With seven mics and a separate release for each mic, this would mean a
key release would trigger a total of 49 samples if not controlled with [note_polyphony].
However, setting note_polyphony=1 and giving each mic a different [group] number solves this.

When using releases with round robins and multiple voices, it can be tricky to make
the release sample round robin counter advance correctly. When there are 2 matching
regions playing, ARIA appears to advance the counter for the releases by 2, and if
there are 4 release round robins, only 2 of them will actually be used. One workaround
for that is triggering an extra region of silence to make the round robin counter
advance by 3, but this will only work if the number of regions is consistent and
predictable. With instruments that have release samples with a number of microphone
positions or organ stops, any of which could be on or off, the total number of matching
regions is very difficult to assess, and it's far easier to use lorand/hirand to
select the release samples instead.

[on_loccN / on_hiccN] effectively replace the default trigger=attack,
as it is used for regions which are to be triggered by MIDI CC messages and not MIDI
note messages. For regions which use these opcodes, trigger should be left unspecified.

## Examples

```
trigger=release
trigger=legato
```


[loop_mode]:         loop_mode
[note_polyphony]:    note_polyphony
[on_loccN / on_hiccN]: on_loccN
[rt_dead]:           rt_dead
[rt_decay]:          rt_decay
