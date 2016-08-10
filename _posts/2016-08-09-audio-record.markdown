---
---

iOS provides various framework to let you work with sound in your app. One of the frameworks that you can use to play and record audio file is the AV Foundation Framework. In this tutorial, I’ll walk you through the  the framework and show you how to manage audio recording

### AVAudioRecorder ?
An instance of the AVAudioRecorder class, called an audio recorder, provides audio recording capability in your application. Using an audio recorder you can:

- Record until the user stops the recording

- Record for a specified duration

- Pause and resume a recording

- Obtain input audio-level data that you can use to provide level metering
 
### How to initialize an AVAudioRecorder

To prepare for recording using an audio recorder:

- Specify a sound file URL.
- Set up the audio session.
- Configure the audio recorder’s initial state.


{% highlight objc %}

- (instancetype)initWithURL:(NSURL *)url 
                   settings:(NSDictionary<NSString *, id> *)settings  
                      error:(NSError * _Nullable *)outError {
}

{% endhighlight %}

the url is the file system location to record to. And also need settings for the recording session,On success, an initialized AVAudioRecorder object. If nil, the outError parameter contains an NSError describing the failure.

an acc type record setting .for example

{% highlight objc %}

- (NSDictionary *)configRecorderSettingWithSampleRate:(float)rate
                                              bitRate:(int)bitRate {
    NSMutableDictionary *recordSettings = [[NSMutableDictionary alloc] initWithCapacity:10];
    [recordSettings setObject:[NSNumber numberWithInt: kAudioFormatMPEG4AAC] forKey: AVFormatIDKey];
    [recordSettings setObject:[NSNumber numberWithFloat:rate] forKey: AVSampleRateKey];
    [recordSettings setObject:[NSNumber numberWithInt:1] forKey:AVNumberOfChannelsKey];
    [recordSettings setObject:[NSNumber numberWithFloat:bitRate] forKey:AVEncoderBitRateKey];
    [recordSettings setObject:[NSNumber numberWithInt:32] forKey:AVLinearPCMBitDepthKey];
    [recordSettings setObject:[NSNumber numberWithInt: AVAudioQualityMedium] forKey: AVEncoderAudioQualityKey];
    return  [recordSettings copy];
}

{% endhighlight %}   

to ensure optimum performance and quality, you need to pick the right audio format and audio codec type. see  [Multimedia Programming Guide](https://developer.apple.com/library/prerelease/content/documentation/AudioVideo/Conceptual/MultimediaPG/UsingAudio/UsingAudio.html)             
so we can start a recording like this (In production code, you would include appropriate error handling)
                      
{% highlight objc %}

- (void)startRecorderWithPath:(NSString *)savePath
                     settings:(NSDictionary  *)settings
                        error:(NSError *__autoreleasing *)error {
    
    __autoreleasing NSError *errorInfo = nil;
    
    AVAudioSession *audioSession = [AVAudioSession sharedInstance];
    [audioSession setCategory:AVAudioSessionCategoryPlayAndRecord error:&errorInfo];
    if (errorInfo && error) {
        *error = errorInfo;
        return;
    }
    [audioSession setActive:YES error:&errorInfo];
    
    if (errorInfo && error) {
        *error = errorInfo;
        return;
    }
    
    
    
    NSURL *url = [NSURL fileURLWithPath:savePath];
    AVAudioRecorder * recorder;
    recorder = [[ AVAudioRecorder alloc]
                initWithURL:url
                settings:settings
                error:&errorInfo];
    
    if (errorInfo && error) {
        *error = errorInfo;
        NSLog(@"%@", errorInfo.localizedDescription);
        return;
    }
    
    if ([recorder prepareToRecord] == YES){
        [recorder record];
    }
}

{% endhighlight %} 


some times we also need a callback to tell the recording progress, some exception handling When the recording is interrupted by the call.
To handle interruptions and the completion of recording, we need to add the AVAudioRecorderDelegate protocol names to the interface declaration for your implementation.

{% highlight objc %}
// add notification
[[NSNotificationCenter defaultCenter]
     addObserver:self
     selector:@selector(handleInterruption:)
     name:AVAudioSessionInterruptionNotification
     object:[AVAudioSession sharedInstance]];
     
// Handle AVAudioSessionInterruptionNotification
- (void)handleInterruption:(NSNotification *)notification {
    NSDictionary *info = notification.userInfo;
    AVAudioSessionInterruptionType type =
    [info[AVAudioSessionInterruptionTypeKey] unsignedIntegerValue];
    
    if (type == AVAudioSessionInterruptionTypeBegan && notification.object == [AVAudioSession sharedInstance]) {
        // Handle AVAudioSessionInterruptionTypeBegan
        // Save state and context
        // Update user interface
    } else {
        // Handle AVAudioSessionInterruptionTypeEnded
        // Restore state and context
        // Reactivate audio session if app appropriate
        // Update user interface
    }
}
{% endhighlight %} 

### AudioHelper

 A record class which is userfriendly and esay to use by developers.  

### How to use AudioHelper

- **config AudioHelper**

{% highlight objc %}
- (void)configRecord {
    // initialize an AudioHelper
    AudioHelper *audioHelper = [[AudioHelper alloc] init];
    // config the recording settings, type, sample rate, bit rate
    [audioHelper configRecorderSettingWithEncodingType:kRecordType sampleRate:kSampleRate bitRate:kBitRate];
    
    //config the power peak, recording progress, completion
    [audioHelper
     configChannelPowerBlockWithPowerBlock:^(float peakPowerForChannel) {
         NSLog(@"power:[%f]", peakPowerForChannel);
     }
     progress:^(NSTimeInterval times) {
         NSLog(@"recording times:[%lf]", times);
     }
     recordStopBlock:^(AudioStopType stopType, NSString *path, NSTimeInterval times) {
         NSLog(@"path:[%@], times:[%.2f]", path, times);
     }];
}

{% endhighlight %} 

- **start recording**

{% highlight objc %}
- (void)startRecording {
    __autoreleasing NSError *error;
    NSString *path = [NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES) firstObject];
    [path stringByAppendingString:@"test.caf"];
    [self.audioHelper startRecorderWithPath:path maxTime:60.0 error:&error];
}
{% endhighlight %} 


- **stop, pause, resume**

{% highlight objc %}
// stop
[self.audioHelper stopRecord];
//stop and delete the temp file
[self.audioHelper stopAndDeleteFile];
// pause
[self.audioHelper pauseRecord];
// resume
[self.audioHelper resumeRecord];
{% endhighlight %} 


for more infomation .see [github]()



