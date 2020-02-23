---
title: iOS视频 裁剪／拼接／压缩（时间维度）
date: 2016-07-26 23:11:03
tags: [coding, iOS, AVFoundation, video]
---
之前已经实现了个需求是录屏分享，然而如今，野生的新需求出现了，那就是根据用户触发事件的时间点，裁剪/拼接视频。
查了很久资料，得知AVFoundation可用，推荐[Apple官方说明文档](https://developer.apple.com/library/mac/documentation/AudioVideo/Conceptual/AVFoundationPG/Articles/03_Editing.html#//apple_ref/doc/uid/TP40010188-CH8-SW1)，十分详细。

首先要说说一个用到的结构，影片时间点结构 **CMTime**
``` c
typedef struct
{
    CMTimeValue value;          /* value/timescale = seconds. */
    CMTimeScale timescale;
    CMTimeFlags flags; 
    CMTimeEpoch epoch;
} CMTime;

```
**value** 表示此时刻在整个视频排第几帧，
**timescale** 表示一秒有几帧，
两者相除，当然得到 seconds，即此时刻在视频的第几秒。

只需记住 **value** / **timescale** = seconds 的关系就行。需要注意的是，**timescale** 不是你可以随意调的，而是在录制时决定的，而且，它的值不等于fps。比如，我的录制参数fps = 15，然而得到视频的 **timescale** = 44100

易知：CMTimeMake(value, timescale) <=> CMTimeMakeWithSeconds(value/timescale, timescale)

---
下面就是实现啦，
我参考的是一个[StackOverflow提问](http://stackoverflow.com/questions/6575128/how-to-combine-video-clips-with-different-orientation-using-avfoundation)

单视频裁剪（视频路径videoPath，头部裁掉frontOffset秒，尾部裁掉endOffset秒）---
``` c
void clipVideo(NSString* videoPath, float frontOffset, float endOffset)
{
    AVMutableComposition *composition = [AVMutableComposition composition];
    // Create the video composition track.
    AVMutableCompositionTrack *compositionVideoTrack = [composition addMutableTrackWithMediaType:AVMediaTypeVideo preferredTrackID:kCMPersistentTrackID_Invalid];
    // Create the audio composition track.
    AVMutableCompositionTrack *compositionAudioTrack = [composition addMutableTrackWithMediaType:AVMediaTypeAudio preferredTrackID:kCMPersistentTrackID_Invalid];

    NSURL *pathUrl = [[NSURL alloc] initFileURLWithPath: videoPath];
    AVURLAsset *asset = [AVURLAsset URLAssetWithURL:pathUrl options:nil];
    [pathUrl release];
    
    AVAssetTrack *videoTrack = [[asset tracksWithMediaType:AVMediaTypeVideo] firstObject]; // or use 'objectAtIndex:0'
    AVAssetTrack *audioTrack = [[asset tracksWithMediaType:AVMediaTypeAudio] firstObject];
    [compositionVideoTrack setPreferredTransform:videoTrack.preferredTransform];
    
    // CMTime 
    CMTime trackDuration = videoTrack.timeRange.duration;
    CMTimeScale trackTimescale = trackDuration.timescale;
    // 用timescale构造前后截取位置的CMTime
    CMTime startTime = CMTimeMakeWithSeconds(frontOffset, trackTimescale);
    CMTime endTime = CMTimeMakeWithSeconds(endOffset, trackTimescale);
    CMTime intendedDuration =  CMTimeSubtract([asset duration], CMTimeAdd(startTime, endTime));
    
    // 下面函数的参数很关键，insertTimeRange是来源视频的区间，atTime是目标视频流中的位置
    bool ok = [compositionVideoTrack insertTimeRange:CMTimeRangeMake(startTime, intendedDuration) ofTrack:videoTrack atTime:kCMTimeZero error:nil];
    ok = [compositionAudioTrack insertTimeRange:CMTimeRangeMake(startTime, intendedDuration) ofTrack:audioTrack atTime:kCMTimeZero error:nil];
    
    //export the combined video
    NSString *combinedPath = /*产出视频目标存储路径*/
    
    NSURL *url = [[NSURL alloc] initFileURLWithPath: combinedPath];
    AVAssetExportSession *exporter = [[[AVAssetExportSession alloc] initWithAsset:composition presetName:AVAssetExportPresetHighestQuality] autorelease];
    exporter.outputURL = url;
    [url release];
    
    exporter.outputFileType = AVFileTypeMPEG4;
    //exporter.outputFileType = [[exporter supportedFileTypes] firstObject];
    [exporter exportAsynchronouslyWithCompletionHandler:^(void)
     {
     dispatch_async(dispatch_get_main_queue(), ^{
        NSLog(@"Export Complete %ld %@", (long)exporter.status, exporter.error);
        // 成功回调
    });
     }];
}

```

多视频顺序连接（视频路径std::vector<NSString*> videoPaths）---
多视频可以采用单视频采用不同的处理函数，同时传入一组timeRange和一组track，个人感觉比自己手动拼接效果好，后面视频的起始时间点不用自己算了，拼接处也不会有闪黑：
``` C
(BOOL)insertTimeRanges:(NSArray<NSValue *> *)timeRanges ofTracks:(NSArray<AVAssetTrack *> *)tracks atTime:(CMTime)startTime error:(NSError * _Nullable * _Nullable)outError NS_AVAILABLE(10_8, 5_0);
```

``` c
void combineVideo(std::vector<NSString*> videoPaths)
{
    AVMutableComposition *composition = [AVMutableComposition composition];
    // Create the video composition track.
    AVMutableCompositionTrack *compositionVideoTrack = [composition addMutableTrackWithMediaType:AVMediaTypeVideo preferredTrackID:kCMPersistentTrackID_Invalid];
    // Create the audio composition track.
    AVMutableCompositionTrack *compositionAudioTrack = [composition addMutableTrackWithMediaType:AVMediaTypeAudio preferredTrackID:kCMPersistentTrackID_Invalid];

    // 准备 timeRanges 和 tracks
    NSMutableArray<NSValue *> * videoTimeRanges = [NSMutableArray array];
    NSMutableArray<NSValue *> * audioTimeRanges = [NSMutableArray array];
    NSMutableArray* videoTracks = [NSMutableArray array];
    NSMutableArray* audioTracks = [NSMutableArray array];
    
    for (int i = 0; i < videoPaths.size(); i++)
    {
        NSURL *pathUrl = [[NSURL alloc] initFileURLWithPath: (videoPaths[i])];
        AVURLAsset *asset = [AVURLAsset URLAssetWithURL:pathUrl options:nil];
        [pathUrl release];
        
        AVAssetTrack *videoTrack = [[asset tracksWithMediaType:AVMediaTypeVideo firstObject];
        AVAssetTrack *audioTrack = [[asset tracksWithMediaType:AVMediaTypeAudio firstObject];
        
        //set the orientation
        if (i == 0)
        {
            [compositionVideoTrack setPreferredTransform:videoTrack.preferredTransform];
        }
        
        [videoTimeRanges addObject: [NSValue valueWithCMTimeRange: CMTimeRangeMake(kCMTimeZero, [asset duration])]];
        [videoTracks addObject: videoTrack];
        [audioTracks addObject: audioTrack];
    }

    bool ok = [compositionVideoTrack insertTimeRanges:videoTimeRanges ofTracks:videoTracks atTime:kCMTimeInvalid error:nil];
    assert(ok);
    ok = [compositionAudioTrack insertTimeRanges:audioTimeRanges ofTracks:audioTracks atTime:kCMTimeInvalid error:nil];
    assert(ok);

    //export the combined video
    NSString *combinedPath =  /*产出视频目标存储路径*/
    NSURL *url = [[NSURL alloc] initFileURLWithPath: combinedPath];
    AVAssetExportSession *exporter = [[[AVAssetExportSession alloc] initWithAsset:composition presetName:AVAssetExportPresetHighestQuality] autorelease];
    exporter.outputURL = url;
    [url release];
    
    exporter.outputFileType = AVFileTypeMPEG4;
    //exporter.outputFileType = [[exporter supportedFileTypes] firstObject];
    [exporter exportAsynchronouslyWithCompletionHandler:^(void)
     {
     dispatch_async(dispatch_get_main_queue(), ^{
        NSLog(@"Export Complete %ld %@", (long)exporter.status, exporter.error);
        // 成功回调
    });
     }];
}
```

压缩视频（核心代码）其他部分与前面类似。

``` C
void compressVideo(NSString* videoPath, float intendedDurationInSecs)
{
    ...

    // 计算压缩率
    CMTime trackDuration = asset.duration;
    CMTimeScale trackTimescale = trackDuration.timescale;
    float totalLengthOfSecs = (float)trackDuration.value / trackTimescale;
    float keepPercentage = intendedDurationInSecs / totalLengthOfSecs;
    
    //assert(keepPercentage < 1.0f);
    if (keepPercentage >= 1)
    { return; }

    CMTimeRange currentTimeRange = CMTimeRangeMake(kCMTimeZero, [asset duration]);
    CMTime finalDuration = CMTimeMake(int(intendedDurationInSecs * 25), 25);
    
    [compositionVideoTrack scaleTimeRange: currentTimeRange toDuration: finalDuration];
    [compositionAudioTrack scaleTimeRange: currentTimeRange toDuration: finalDuration];
    
    ...
}
```
**音轨相关**
情景1. 无音轨
上面的代码其实没有考虑视频无音轨的情况，然而这种情况可能存在，需要判断和处理。
``` C
bool hasAudio = true;
...
// 检测 是否有音轨
if ([[asset tracksWithMediaType:AVMediaTypeAudio] count] == 0)
{
    hasAudio = false;
    [composition removeTrack: compositionAudioTrack];
}
//根据flag决定是否处理音频
```

情景2. 需要去掉声音（音轨静音）
目前成功的方法是最笨的：准备一个静音音频资源，从中读音轨，用来覆盖掉需要静音的区间。
``` C
AVAsset* emptyAudioAsset = [AVURLAsset URLAssetWithURL:[NSURL fileURLWithPath:[[NSBundle mainBundle] pathForResource:@"mute_stereo" ofType:@"mp3"]] options:nil];
emptyAudioTrack = [[emptyAudioAsset tracksWithMediaType:AVMediaTypeAudio] firstObject];
```

---
欢迎讨论，但要在看过[Apple官方说明文档](https://developer.apple.com/library/mac/documentation/AudioVideo/Conceptual/AVFoundationPG/Articles/03_Editing.html#//apple_ref/doc/uid/TP40010188-CH8-SW1)之后哦！我只涉及了一个小子集。