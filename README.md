# AVFoundation ProgrammingGuide
<br>

## Introduction 
<br>

#### About AVFoundation 
- play, create time based audiovisual media

#### Framework architecture
![frameworksblockdiagram_2x](https://user-images.githubusercontent.com/5119286/39769839-42848828-5328-11e8-8470-1c203fccde16.png)

For Simple Task
- Play: use AVKit 
- Record Video: use UIImagePickerController



#### At a Glance
Two facets of API 
- Video
- Audio



#### Representing and  Using Media with AVFoundation 
- AVAsset
    - primary class for representing media
    - not tied to particular data format(like mp4, mp3, mov…etc)
    - individual media data is a uniform type(called Track)
    - It is just representing media(does not mean ready for use)
- Playback
    - Separate presentation state(play, displaying..) from the asset itself
    - presentation state is managed by player item
    - play player item(eg. avplayeritem) using player(eg. avplayer )
- Reading, Writing, and Recording Assets
    - Reencode: use Export Session, Asset Reader, Asset Writer
      - Visual representation of audio: use Asset Reader
- Thumbnails
    - generate thumbnail: use AVAssetImageGenerator
- Editing 
    - create new assets: 
      - use Compositions
      - you can also use sample buffer or asset writer 
    - add, remove, reorder: use Mutable Compositions
    - A Composition reside in memory, you can export them as a file using export session
- Still and Video Media Capture
    - Recording av data from device(cam, mic) is managed by Capture Session
    - Capture Session manages flow data from input(cam) to output(display)
    - Manage data flow !!


#### Concurrent Programming with AVFoundation 
- Callbacks from AVFoundation are invoked on which its internal tasks
- 2 guideline for notification and threading
    - UI related noti occur on main thread
    - class or method that require create a queue will return notification on the queue
- Must See code
    - https://developer.apple.com/library/content/samplecode/AVCam/Introduction/Intro.html#//apple_ref/doc/uid/DTS40010112-Intro-DontLinkElementID_2

#### Prerequisites
- basic playback: https://developer.apple.com/documentation/avkit



