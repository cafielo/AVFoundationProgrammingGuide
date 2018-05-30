# AVFoundation ProgrammingGuide
<br>

## Introduction
<br>
#### About AVFoundation 
- play, create time based audiovisual media
<br>
#### Framework architecture
![frameworksblockdiagram_2x](https://user-images.githubusercontent.com/5119286/39769839-42848828-5328-11e8-8470-1c203fccde16.png)

For Simple Task
- Play: use AVKit 
- Record Video: use UIImagePickerController


<br>
#### At a Glance
Two facets of API 
- Video
- Audio


<br>
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
- Reading, Writing, and Reencording Assets
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

<br>
#### Concurrent Programming with AVFoundation 
- Callbacks from AVFoundation are invoked on which its internal tasks
- 2 guideline for notification and threading
    - UI related noti occur on main thread
    - class or method that require create a queue will return notification on the queue
- Must See code
    - https://developer.apple.com/library/content/samplecode/AVCam/Introduction/Intro.html#//apple_ref/doc/uid/DTS40010112-Intro-DontLinkElementID_2

<br>
#### Prerequisites
- basic playback: https://developer.apple.com/documentation/avkit




<br><br>
## Using Assets


Assets can come from various way(File, iPod Library, Server…etc)
- all the information of asset might not be  immediately available


<br>
#### Creating an Asset Object

Using URL to create asset with AVURLAsset

코드 필요 

Options for Initializing an Asset
- AVURLAsset take an url and options(optional and dictionary type)
- use AVURLAssetPreferPreciseDurationAndTimingKey for getting exact duration

코드 필요 

Accessing the User’s Assets
- To access the asset in Photo Application(or iPod Library), you need to get a URL
    - iPod Library
        - Use MPMediaQuery, and get url by using MPMediaItemPropertyAssetURL
    - Photo Application 
        - Use ALAssetLibrary

코드 필요

<br>
#### Preparing an Asset for Use

Initializing an asset does not mean that information of item is immediately available
Load information asynchronously
- Use AVAsynchronousKeyValueLoading protocol 
    - call loadValuesAsynchronouslyForKeys:completionHandler
    - get property at completionHandler block
    - handle the property depend on status
- AVAsset, AVAssetTrack conform to AVAsynchronousKeyValueLoading protocol 

코드 필요


<br>
#### Getting Still Images from a Video


To get thumbnail 
- Use AVAssetImageGenerator
    - create a AVAssetImageGenerator with asset
    - before create a imagegenerator, check tracksWithMediaType to identify videoType track in asset
    - *Note: keep strong reference to the image generator until it get all the images

코드 필요 

Generating a single image
- Use copyCGImageAtTime:actualTime:error: to generate single image

Generating a Sequence of Images
- Use generateCGImagesAsynchronouslyForTimes:completionHandler:
    - completionHandler provide 3 result 
        - The Image
        - The Time requested, and time for the image generated
        - error object with reason


 Ensure that you keep a strong reference to the image generator until it has finished creating the images.

코드 필요 

<br>
#### Trimming and Transcoding a Movie

Trim and Transcoding 
- use AVAssetExportSession

Work flow 
￼![export_2x](https://user-images.githubusercontent.com/5119286/40695420-2694e240-63fc-11e8-84e4-c468b3d6fa47.png)


How to use ExportSession
- initialize with asset and preset
- specify output url and file type
- (optional) specify timeRange( if you want to trim) 
- exportAsynchronouslyWithCompletionHandler to export file
- (optional) you can cancel 

How to check compatible preset
- use exportPresetsCompatible(asset:) method in AVExportSession


The case export will be canceled
- overrite an existing file
- write file outside of the app sandbox
- incoming call
- enter background and start playback from another app



<br><br>
## Playback






<br><br>
## Editing 










<br><br>
## Still and Video Media Capture

Use AVCaptureSession to configure data flow(from input to output)

AVCaptureDevice
- represent input Device(camera, mic)
AVCaptureInput
- configure the ports from input device
AVCaptureOutput
- manage the output to movie file or still image
AVCaptureSession
- coordinate the data flow from input to output
AVCaptureVideoPreviewLayer



single session can configure multiple inputs and outputs
![captureoverview_2x](https://user-images.githubusercontent.com/5119286/40695431-2f7a9c92-63fc-11e8-83e5-e24725390bf9.png)




AVCaptureConnection 
- **Connection between a capture input and output is represented by AVCaptureConnection** 
- AVCaptureInput have one or more input ports(AVCaptureInputPort)
- AVCaptureOutput can accept one or more sources(ex. AVCaptureMovieFileOutput accept both video and audio)
- **When you add inputs and outputs to a session, session makes connections between all the compatible capture inputs’ ports and capture outputs’**
- Use AVCaptureConnection to enable or disable data flow
- Use AVCaptureConnection to monitor audio power level


![capturedetail_2x](https://user-images.githubusercontent.com/5119286/40695437-34bdb216-63fc-11e8-8594-ddf920462cb5.png)


<br>
#### Use a Capture Session to Coordinate Data Flow


AVCaptureSession is the central coordinating object to manage data capture.

```swift
let session = AVCaptureSession()
// Add inputs and outputs
session.startRunning()
```

Configuring a Session 
- Use a Preset to specify quality and resolution 
- Presets 
    - AVCaptureSessionPresetHigh
    - AVCaptureSessionPresetMedium
    - AVCaptureSessionPresetLow
    - AVCaptureSessionPreset640x480
    - AVCaptureSessionPreset1280x720
    - AVCaptureSessionPresetPhoto


You can check whether it is supported

```swift
if session.canSetSessionPreset(AVCaptureSession.Preset.hd1280x720) {
    session.sessionPreset = .hd1280x720
} else {
    // handle failure
}
```

You can adjust session parameter, make changes to a running session 
- surround changes with beginConfiguration and commitConfiguration 

```swift
session.beginConfiguration()
// remove an existing capture device
// add a new capture deive
// reset the preset
session.commitConfiguration()
```

Monitoring Capture Session State
- session posts notifications
    - start, stop, interrupted, error
- observe running and interrupted property to post notification on the main thread


<br>
#### An AVCaptureDevice Object Represents and Inputs Device

AVCaptureDevice(abstract of physical device) provides input data to AVCaptureSession

Find out available devices
- Use AVCaptureDevice.devices() - it is deprecated though,


Device Characteristics
- Use hasMediaType() ,supportsAVCaptureSessionPreset() to check supporting certain type of media or preset
- You can find out position of Camera

**Note**
- Media capture does not support simultaneous capture of both front and back camera
 


Device Capture Settings
- Different devices has different capabilities
- In all cases, lock the device before changing mode of particular feature

```swift

let devices = AVCaptureDevice.devices()
var torchDevices = [AVCaptureDevice]()

devices.forEach { device in
    print("name: \(device)")
    
    if device.hasMediaType(.video) {
        // do some with video capture device
    }
    
    if device.hasTorch && device.supportsSessionPreset(.high) {
        torchDevices.append(device)
    }
}
```

Focus Modes
- 3 modes:
    - AVCaptureFocusModeLocked
    - AVCaptureFocusModeAutoFocus (useful to camera app)
    - AVCaptureFocusModeContinuousAutoFocus


Exposure Modes
- 2 modes:
    - AVCaptureExposureModeContinuousAutoExposure
    - AVCaptureExposureModeLocked


Flash Modes
- 3 modes
    - AVCaptureFlashModeOff
    - AVCaptureFlashModeOn
    - AVCaptureFlashModeAuto

Torch Mode
- 3 modes
    - AVCaptureTorchModeOff
    - AVCaptureTorchModeOn
    - AVCaptureTorchModeAuto

Video Stabilization 
- Cinematic video stabilization is available for connections that operate on video, depending on the specific device hardware(not all source formats and video resolution are supported)
- Use enablesVideoStabilizationWhenAvailable property to stabilization automatically

White Balance
- 2 modes
    - AVCaptureWhiteBalanceModeLocked
    - AVCaptureWhiteBalanceModeContinuousAutoWhiteBalance

Setting Device Orientation 
- use AVCaptureConnection to set desired orientation(orientation will affect to AVCaptureOutput)


Configuring a Device
- To set capture properties, 
    - first, call lockForConfiguration() on device



Switching Between Devices 
- To switch device
    - surround configuration changes between beginConfiguration() and commitConfiguration()
    - this ensure smooth transition


<br>
#### Use Capture Inputs to Add a Capture Device to a Session 

Use AVCaptureDeviceInput 

```swift
do {
        let input = try AVCaptureDeviceInput(device: device)
        
        if session.canAddInput(input) {
            session.addInput(input)
        } else {
            // handle failure
        }
    } catch let error {
        // handle error
    }
```

<br>
#### Use Capture Output to Get Output from a Session 

To get output from capture session,  you add outputs. 
- **AVCaptureMovieFileOutput** to output to a movie file
- **AVCaptureVideoDataOutput** if you want to process frames from the video being captured, for example, to create your own custom view layer
- **AVCaptureAudioDataOutput** if you want to process the audio data being captured
- **AVCaptureStillImageOutput** if you want to capture still images with accompanying metadata

Saving to a Movie File
- Use AVCaptureMovieFileOutput
    - you can set maxDuration, maximumFileSize
    - resolution depends on preset(but, default is H.264 for video, AAC for audio)
    - 
- Start a record
    - need to supply url 
- Ensuring that the file was written successfully
    - implement `captureOutput:didFinishRecordingToOutputFileAtURL:fromConnections:error:` method
- Adding meta data to file

Processing Frames of Video(**Important to Image process !!!**)
- `AVCaptureVideoDataOutput` uses delegation to vend video frames
- Create serial queue 
- Set delegate using `setSampleBufferDelegate:queue:`  
- Frames are presented in delegate method `captureOutput:didOutputSampleBuffer:fromConnection:`
    - use BGRA type which is works well both opengl and Core Graphics
- Performance considerations for processing video
    - set the session output the lowest practical resolution 

Capturing Still Images
- Use `AVCaptureStillImageOutput`
- Pixel and Encoding Formats
    - Different image format supported by different devices
    - set the outputSettings dictionary to specify the image format

```swift
let stillImageOutput = AVCaptureStillImageOutput()
let dic = [AVVideoCodecKey: AVVideoCodecJPEG]
stillImageOutput.outputSettings = dic
```

- Capturing an Image 
    - use `captureStillImageAsynchronouslyFromConnection:completionHandler:` to capture

<br>
#### Showing the User What’s Being  Recorded

Video Preview
- use `AVCaptureVideoPreviewLayer`  to show what is being recorded
    - you can also set mirror property
    - Video Gravity Modes
      - AVLayerVideoGravityResizeAspect
      - AVLayerVideoGravityResizeAspectFill (확대시 이거 사용하면 될듯)
      - AVLayerVideoGravityResize

```swift
let preview = UIView()
let previewLayer = AVCaptureVideoPreviewLayer(session: session)
preview.layer.addSublayer(previewLayer)
```

Showing Audio Levels
- use `AVCaptureAudioChannel` to monitor power level of audio

<br>
#### Putting It All Together: Capture Video Frames as UIImage Objects

- Create an AVCaptureSession object to coordinate the flow of data from an AV input device to an output
- Find the AVCaptureDevice object for the input type you want
- Create an AVCaptureDeviceInput object for the device
- Create an AVCaptureVideoDataOutput object to produce video frames
- Implement a delegate for the AVCaptureVideoDataOutput object to process video frames
- Implement a function to convert the CMSampleBuffer received by the delegate into a UIImage object


```swift
// Create and Configure Capture Session
let session = AVCaptureSession()
session.sessionPreset = AVCaptureSession.Preset.medium


// Create and configure Device and the Device input
let device = AVCaptureDevice.default(for: .video)

do {
    let input = try AVCaptureDeviceInput(device: device!)
    session.addInput(input)
} catch let error {
    // handle error
}

// Create and Configure the Video Data Output
let output = AVCaptureVideoDataOutput()
session.addOutput(output)
let outputSetting: [String: Any] = [String(kCVPixelBufferPixelFormatTypeKey): kCVPixelFormatType_32BGRA]
output.videoSettings = outputSetting

let vc = UIViewController()
let queue = DispatchQueue(label: "my")
output.setSampleBufferDelegate(vc as! AVCaptureVideoDataOutputSampleBufferDelegate, queue: queue)

// Implement the Sample Buffer Delegate Method
func captureOutput(_ output: AVCaptureOutput, didDrop sampleBuffer: CMSampleBuffer, from connection: AVCaptureConnection) {
    let image: UIImage = imageFromSampleBuffer(sampleBuffer: sampleBuffer)
    // add your code here that uses the image
}

// Starting and Stopping Record
// ensure that permission(code is omitted here. check out permission grant code)
session.startRunning()
```

<br>
#### High Frame Rate Video Capture

Use AVCaptureDeviceFormat to determine the capture capbilities
- slow and fast motion
- check out sloPoke sample code

Playback
- AVPlayer manages playback speed automatically by setting rate property
- AVPlayerItem support audioTimePitchAlgorithm to specify how audio is played when the movie is played at various frame rates using the Time Pitch algorithm setting

Editing
- Create AVMutableComposition 
- Insert your video asset using the insertTimeRange:ofAsset:atTime:error: method.
- Set the time scale of a portion of the composition using scaleTimeRange:toDuration:

Export
- Exporting 60 fps video uses the AVAssetExportSession class to export an asset. The content can be exported using two techniques:

Recording 
- You capture high frame rate video using the AVCaptureMovieFileOutput class, which automatically supports high frame rate recording. It will automatically select the correct H264 pitch level and bit rate.

- To do custom recording, you must use the AVAssetWriter class, which requires some additional setup
- 





<br><br>
## Export







<br><br>
## Time and Media Representations






