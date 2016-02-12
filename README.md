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

## Using the VST
When you’ve got your camera with a street view set up, frame counts for a calibration vehicle passing
each way through the speed measuring zone, and video files your camera has captured, the next step
for performing traffic speed analysis is to start up the Video Speed Tracker. Currently the routine steps
on startup are carried out in a command line window (ripe opportunity for a GUI developer with time
to do better). In sequence, you will be asked to select a directory from the <path prefix> \\ IPCam
directory, where you have done a one-time setup of the <path prefix> directory, e.g. “g:\locustData”.
Figure 1 depicts the steps I went through to select the directory “2016 02 05”, located in
“g:\\locustData\IPCam” on my system, and containing all of my video files from February 5th, 2016.
Next, within that directory I chose to process the specific video file named
“manual_20160205_115810.avi”. Following that, as you can see, I was asked about a trace file, the
speed limit, the egregious speed lower bound, the starting frame number to process, whether I wanted
a highlights video file, and whether the yellow region, as depicted in figure 2, was acceptable. This
yellow region in figure 2 is the “analysis box.” Currently, if you answer that the analysis box, etc. are
not OK, the program aborts (because problems would be fixed by changing constants in the code.

>Insert Figure 1. and Figure 2.

The yellow analysis box depicted in Figure 2 serves a number of purposes: It crops out across the street
neighbors’ houses, it removes useless information in the foreground, and it greatly enhances the
quality of the frame differencing method I use to detect motion: when the wind blows the leaves
move, which creates bothersome noise in the difference image.

The white vertical lines are the end points of the speed measuring zone. A vehicle’s speed is measured
by counting how many 1/30th of a second video frames it was in as it passed though the speed
measuring zone. More specifically, the frame counter starts when the vehicle’s front bumper first
crosses the first white line in its path, and the frame counter stops when the vehicle’s front bumper
first crosses the second white line in its path. Its frame count is then compared to the frame count for
a calibration vehicle that was driven through the same speed measuring zone at the speed limit
(multiple vehicles, many times in my case, to be certain). A first order speed estimate for a vehicle
analyzed by the Video Speed Tracker is derived by determining the value of:

```
Observed vehicle speed =
  int (0.5 + ( (observed vehicle frame count) / (calibration vehicle frame count)) * speed limit).
```
Further analysis is performed in the Video Speed Tracker to home in on the likely actual speed but that
analysis needn’t be discussed here.

The Video Speed Tracker is currently set up to support the description of one vertical obstruction in the
foreground. In Figure 2 you can see the maple tree in my front yard, with two vertical yellow lines, one
on either side of it. Because the VST uses a frame differencing method to determine motion, it has to
account for objects that will keep that method from working as needed. When the front bumper of a
vehicle passes behind the tree, the frame differencing method will not detect the motion of the
bumper, behind the tree. Because VST uses a “predictive tracker” this is not a problem: the occluded
bumper is “dead reckoned” (and the rear bumper) as it passes behind the tree. If you have more than
one foreground obstruction in your scenario, you’ll have to modify the VST code a bit. It won’t be hard
if you’re a programmer of any sort. I tell you what to do in the appendix.

Finally there are the hubcap lines: one horizontal orange line and one horizontal purple line as
depicted in Figure 2. As you’ll appreciate as you read on, shadows cast on the pavement by the
vehicles move! For example, in the bottom window (the enhanced difference image) in Figure 3, you
can see the “beaver tail” trailing behind the vehicle that is heading left to right. That’s its shadow
moving. (In the same difference image window you can also see the tree passing up through the back
part of the car’s hood –no motion detected behind the tree). There’s a lot of literature about detecting
and removing shadows from video images. I read some of it and decided to hold off on any further
consideration of shadow removal. There was an easier way for the straight, relatively flat stretch of
road I’m analyzing: don’t look at the pavement in front of a car (It’s the shadows in front of a car that
could cause problems for the VST because they can interfere with quality front bumper tracking, which
is what VST does.) The orange (for right to left traffic) and purple (for left to right traffic) lines are at
hubcap level. VST doesn’t consider motion information below the orange line for right-to-left vehicles,
or the purple line for left to right vehicles. The bottom yellow line of the analysis box could be the
purple line, but then the resulting video wouldn’t look as good.

If your street has a hill or dip, the straight hubcap lines idea won’t work well. You’d need to be able to
describe a curved hubcap line and to have the VST sample the y value of that curve given the x
coordinate of the front bumper of the vehicle under analysis. That shouldn’t be hard. I just haven’t
done it (another Github postable issue for VST).

Once you say yes to the prompt about the acceptability of the analysis box, the VST starts its analysis.
Figure 3 depicts the two new windows you’ll see open up. The top window is the region of the current
video frame being analyzed by the VST.

As can be seen in the upper window in figure 3, each vehicle being tracked has two rectangles around
it. A green rectangle depicts what the predictive tracker’s information is about the vehicle. The blue
line at the leading edge of the green rectangle is where the predictive tracker has predicted the front
bumper should be in the displayed image. In order to do an acceptable job of determining a vehicle’s
speed, the VST has to have locked the blue line onto the front bumper before the vehicle hits its first

>Insert figure 3.

speed measuring zone line (the white vertical line). And it needs to be locked on when the vehicle
crosses the second white line in its path. In actuality it only needs to be locked onto the same point on
the vehicle as it crosses each white line, but VST attempts to make that be the front bumper. Also, the
blue line doesn’t have to be locked onto the front bumper between the white lines, although not being
locked on at those times is rare if it’s locked on at each of the white lines.

Vehicles traversing left to right also have purple rectangles around them, and right to left vehicles have
orange rectangles. These rectangles represent what the current frame differencing data indicated
about the current extent and position of the vehicle, as loosely constrained by the green box. You’d
think the predictive tracker is failing if the blue line isn’t coincident with the vertical purple (orange)
line representing the leading edge of the vehicle. That’s not always the case. The frame differencing
operation can produce noisy data, for example when a front bumper passes behind a tree, when
there’s glare off a front windshield, when the differencing operation fails some because there are
pixels similar to the color of the vehicle, behind it, and others. This is why tracking radars like the ones
used to track aircraft into and out of your local airport generally use predictive tracking also. The “real
data” can be noisy, but you can generally pull enough useful data (“signal”) out of the noise to get a
very good representation of what’s really going on. That’s what VST’s predictive tracker does. Usually,
the blue line at the leading edge of the green prediction rectangle is telling the truth better than the
current snapshot of “real data” gotten from the frame differencing.

VST really only needs to track the front bumper of a vehicle to get speed results. Tracking height and
rear bumper are done only to support computation of profile area (to separate buses from cars from
motorcycles) and to handle issues that arise when two opposing vehicles pass each other.

> Insert figure 4.

There are two decent technical accomplishments in VST: the quality of its predictive tracker (look
where the blue lines are in Figure 4) and its ability to continue to track opposite direction passing
vehicles with high probability of success. The difference images for the two vehicles in Figure 4 are
about to collide. Without predictive tracking, all would most likely be lost. Figure 5 shows what
happens next.

> Insert figure 5.

As soon as a pair of front bumpers of opposing vehicles passes each other, VST turns the leading blue
lines red to indicate that each vehicle is being dead-reckoned, meaning that predicted data is being
used as input to the predictive tracker rather than actual data derived from the difference image.
Without performing some serious image analysis on the blob depicted in the bottom window of Figure
5, it’s essentially impossible to pick out the front bumpers of the two vehicles. The quality of the
prediction is better than the quality of the real data at this point.

As soon as the front bumper of a vehicle proceeds past the rear bumper of the opposing vehicle (Rear
bumper may be predicted, or based on actual data), VST starts using the difference image data again to
feed the tracking filter for the front bumper of the first vehicle. You can now see why locations of rear
bumpers need to be known: so that VST can determine when to turn off dead reckoning a front
bumper.

VST also, separately, dead reckons rear bumpers as needed. Figure 6 shows the dead reckoned rear
bumpers of two passing vehicles at the last moments of their rear bumpers being dead reckoned. VST
stops dead reckoning rear bumpers when it determines the predicted positions of the rear bumpers
are past each other. Notice how tightly the tracker lock has stayed on the front bumpers.

> Insert figure 6.

Finally, as seen in Figure 7, the vehicles have crossed their respective second measuring zone white
lines, VST has computed their respective speeds, automated quality checks have been passed, and
speeds are posted. And so it goes.

> Insert figure 7.

While the VST is executing, the user has various options. When the analysis window (e.g. the top
window in Fig. 7) is highlighted, the user can enter a small set of single letter commands to change
processing behavior:

1. “p” will toggle pausing the analysis and resuming it
2. “v” toggles display of the lower window in Fig. 7, and stops refresh of the upper window,
allowing processing to proceed faster (VST tends to be I/O bound).
3. “f” makes the display update of the two windows in Fig. 7 occur five times faster to a delay
lower limit of 10. Consequently the execution speed of VST goes up accordingly.
4. “s” makes the display update of the two windows in Fig. 7 occur five times slower to a delay
upper limit of 1250. Consequently the execution speed of VST goes down accordingly.

##Producing a Highlights Video File in VST
The user is given an option to have a highlights video file produced as a byproduct of the execution of
VST. In Fig. 1 the user said “no” to such a highlights video file. Figure 8 depicts the interactive session
when the user says “yes”. Consequently the user is given the chance to enter qualifying information
for vehicle speeds leading to the vehicle’s inclusion in the highlights video file.

> Insert figure 8. and Figure 9. 

After indicating the Analysis Box is acceptable, a video compression selection box will pop up, as
depicted on the left in Figure 9. You need to have installed a codec for the h264 codec or whatever
codec you choose to use. It is strongly advised that you use a codec that does high compression or you
will produce a very large highlights file (orders of magnitude larger than what h264 produces). Scroll
down in the list and select the codec you choose to use and select OK. The VST will start up, and you
will see the two new windows depicted in Figure 3.

##Further Notes on the VST

Every once in a while a horizontal red line will pop up through the middle of the Analysis Box. This
means that VST has determined there’s too much going on to produce valid results with high
likelihood. Currently the threshold for abandoning current analysis is that there are two opposite
direction vehicles being detected, as well as a third vehicle, and a fourth has just appeared. Four
vehicles, with at least two opposing, means that too much dead reckoning will have to be done. This
can be visually confirmed by inspecting the scene in any of figures 3 through 7. VST can handle an
arbitrary number of vehicles all traveling in the same direction.

Sometimes VST will fail to track a vehicle well. There are many circumstances that can cause this
outcome: a vehicle too close in color to the background behind it, low quality differencing images for
any of many other reasons, too much dead reckoning past a very large vehicle too soon after the
vehicle was first put in track, etc. VST attempts to identify bad tracks but doesn’t always catch them.
That’s why there’s a post processor for the highlights video.

VST abandons tracks when it detects a vehicle is moving backwards relative to what VST was expecting.
VST abandons a track when, twice in a row, it doesn’t see any frame differencing image where a
vehicle is expected to be.

VST is aggressive about keeping moving objects in track, even if it lost track on them previously. This is
done primarily to keep the quality of opposite direction tracking high. Bad things happen if a tracked
vehicle passes another when the tracker doesn’t know the second one is there. You’ll see evidence of
Figure 9. CODEC Selection.

VST’s aggressiveness on occasion, when track is lost on a vehicle, and then picked right back up the
next frame. If the vehicle is more than half way across the analysis box VST will try to place it in track
in the opposite direction it’s traveling. Soon after, track will be dropped, for backwards motion, and
then picked right back up again. No need to worry. All of those futile attempts are thrown away.
If a track is lost for a vehicle when its front bumper is in the speed measuring zone, no speed for that
vehicle will be entered into final statistics. No vehicle that is first put into track after its front bumper
has passed the first white speed measuring marker will have its speed entered into final statistics. In
general, if a speed is posted for a vehicle in the upper corner of its destination end of the analysis box,
then that speed is recorded into the spreadsheet (csv) data file. Posted speeds can be seen in the top
window of Fig. 7.

##Post-Processing with the Final Highlights Video Processor
Input to the second program, the final highlights video processor (FHVD), is a video highlights file
produced by the VST. The purpose of the FHVD is to provide the user with an opportunity to hand
select examples of speeding vehicles for public analysis and review, and an opportunity for you to
ensure no mistracked vehicle is ever presented as having been tracked correctly.
When the FHVD is started it lists the avi files found in the HiLites subdirectory, one-by-one, as can be
seen in Figure 10. You select the file to be edited, and then confirm the cropping box is acceptable,
see figure 11. Note the cropping box is 640 x 360 pixels. This smaller size is used to make posts of the
output file to the web more easily viewed. 1280 wide files do no show well on Facebook, for example.

>Figure 10. FHVD Input File Selection.

Next, the codec selection popup will occur, as seen in figure 9. The codec you choose will be used to
encode the edited highlights video that FHVD outputs. Once you’ve chosen the codec (h264 highly
recommended), the editing process will begin.
FHDV presents the motion of vehicles, one at a time, with the last frame displayed being the one
showing the vehicle across the end of the speed measuring zone, and with the speed of that vehicle
shown above it. At this point you have the following options:

1. [R]eplay – Show the motion of the vehicle through the speed measuring zone again.
2. [S]low replay – do a slow motion replay.
3. [D]elete – do not write the current vehicle motion to the output file. Proceed to next vehicle.
4. [K]eep – do write the current vehicle motion to the output file. Proceed to the next vehicle.
5. [Q]uit – stop processing vehicles.

FHVD’s output file has the prefix “forPost” and a suffix taken from the original video file processed by
VST. The forPost file is written to the subdirectory <path prefix>\HiLites\forPosting.
Figure 11. Cropping Box for FHVD Output
Set-up Checklist

1. You really need about 100 to 120 feet of open view (a maple tree trunk in the foreground is
OK) of the street. Your speed measuring zone should be at least one second wide for a vehicle
going the speed limit. For a speed limit of 25 MPH, that’s about 37 feet. My speed measuring
zone is 1-1/3 seconds wide, which translates to about 50 feet for a vehicle going 25 MPH. I’ve
been able to track vehicles up to 71 MPH (Nailed that sucker!) in the zone you see in Figure 2.
2. You need an HD (1280 x 720) video input stream for the stretch of street you want to analyze.
That requires a camera. I’m using the Foscam FI9803EP outdoor, HD, power over ethernet
camera. It’s about $90 online. I have no particular loyalty to Foscam, but I can say I’ve had
about a half dozen of their cameras and only one (an FI9821) has developed an issue (when it
pans, the video signal cuts out). That’s a decent record for an inexpensive cam that does HD
and 30FPS. My 9803EP has been working about three weeks, including through the infamous
Middle Atlantic January 2016 Snowzilla.
3. Aim the camera well. Look at Figure 2. Aim the camera so that the street passes left to right
about half way up the overall image. This should minimize the effects of lens distortion. A
vehicle moving a constant velocity cross the lens moves at different pixel rates, frame to frame,
as it proceeds from an edge towards the center and back towards the far edge. My tracker is
what’s known as a “piecewise, linear least squares” tracker. The piecewise part (throwing out
the oldest data) is what allows it to adapt to changing pixel rates as a vehicle moves through
the scene. Setting up as you see in Figure 2 helps the tracker produce high quality results.
4. Set critical one time setup values, as documented in the appendix and then compile VST. You’ll
need OpenCV 2.4.11 installed to have a successful compile. Some of the one-time values you’ll
have to set in the code are annoying, in that they are not set from a configuration file or
through a user interface. There aren’t too many, and one of the first Github repo issues for VST
will be to remove setting of these data objects from the code and place their values into a
binary independent config file and/or user interface. Keep a watch on the VST Github site.
5. Create a location (the “prefixPath”) for five directories used by VST (and the final highlights
video processor):
  1. IPCam -- You put directories for collections of .avi video files here.
  2. HiLites -- VST outputs highlights videos into this directory
    4. forPosting -- Subdirectory for output of final highlights video processor
  3. Stats – VST puts csv files with tracked vehicle data here
  4. Trace – VST puts debug files here.

All of the files VST creates are given names derived from the input video file name (or the video file
directory if you answer “*” when asked for which file in a directory full of video files to process.) For
the IPCam directory, I create subdirectories with syntax yyyymmdd (e.g. 20160205) in which I place
that day’s video files.

>TODO Appendix

## Authors
### Original Author and Development Lead
- Paul Reynolds (reynolds@virginia.edu) www.cs.virginia.edu/~pfr
