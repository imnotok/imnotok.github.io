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

Apple development document

{% highlight objc %}

- (instancetype)initWithURL:(NSURL *)url  settings:(NSDictionary<NSString *, id> *)settings  error:(NSError * _Nullable *)outError {
}

{% endhighlight %}

the url is the file system location to record to. And also need settings for the recording session,On success, an initialized AVAudioRecorder object. If nil, the outError parameter contains an NSError describing the failure.

So we  need some steps,an examble for acc settting

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
  
So we can start a recording like this  
                      
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

