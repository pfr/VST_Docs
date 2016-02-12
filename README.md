# Video Speed Tracker
Documentation for Video Speed Tracker, and related documents.

>The resolution proposed to Charlottesville City Council at the end of the presentation will change.  It needs to reflect that speeding by city vehicles is probably not the fault of the drivers alone.  They likely have supervisors who are pushing them to stay on schedule.


##Overview
The Video Speed Tracker (VST) is an open source, technically sound, vehicle speed measuring system
that can track bidirectional traffic, one lane in each direction (e.g. a typical residential street). Beyond a typical home
computer, the VST requires video from an HD camera, many of which can be purchased for less than
$100 (e.g. the Foscam Fi9103EP, power over Ethernet, outdoor camera). If you plan to modify and
recompile VST then you need to have OpenCV c 2.4.11 installed.
The VST software package comprises two components: 1) the Video Speed Tracker and 2) a final
highlights video processor. The Video Speed Tracker takes user provided setup information and inputs
either a single HD video or a directory of HD videos, containing recording(s) of passing traffic. VST
outputs:

1. A csv (comma separated) file with one entry for each vehicle tracked in the video. Each vehicle
entry includes video file name, start and end frame numbers the vehicle was tracked, its
direction, its profile area and an estimated speed. Profile area can be handy for separating
buses and large trucks from other vehicles. Estimated speed entries support speed data
analysis of any sort imaginable.
2. A trace (debug) file, if you request it. This file contains copious information for determining
how the tracker derived a final speed for a given vehicle.
3. A highlights video, if requested, where the user can select a speed range for identifying vehicles
to be captured in the highlights video. Vehicle area is also used to select among vehicles. For
example, a user can request video for vehicles going between 25 and 40 MPH and/or with the
profile area of buses and trucks to be output to the highlights video. The highlights video is
meant to be used to sort through selected vehicles for the purpose of verifying the speed
tracking quality before making highlights public.

The final highlights video processor takes in a highlights video produced by the Video Speed Tracker
and produces a final video file meant to depict carefully reviewed speeding vehicles. The highlights
video processor supports user viewing of each vehicle as it passes through the speed measuring zone,
replay, slow replay, deletion and keeping for the final highlights video. The final highlights video
should be the sort of thing your lawyer would be comfortable with you posting to the internet.

## Authors
### Original Author and Development Lead
- Paul Reynolds (reynolds@virginia.edu) www.cs.virginia.edu/~pfr

### Contributors
see https://github.com/pfr/VST_Docs/graphs/contributors
