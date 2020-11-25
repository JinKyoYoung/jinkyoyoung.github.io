---
title: "GetSystemVolumName"
categories:
  - objctive-c
tags:
  - Obj-C
  - System
---

## MacOS System Volume Name 구하는 방법.

### why?
- "Macintosh HD" 이름이 아닌, System Volume Name을 구하고 싶다.
- 간단한 리서치를 통해서는 방법을 찾을 수 없었다.
  (혹시 간단한 방법으로 알 수 있는 방법이 있으면 댓글 부탁드려요~^^;)

### diskuitl 명령어를 통해 disk의 여러 정보를 구할 수 있다.
~~~objectivec
luckyo$ diskuitl apfs list;
APFS Container (1 found)
|
+-- Container disk1 00000000-0000-0000-0000-000000000000
    ====================================================
    APFS Container Reference:     disk1
    Size (Capacity Ceiling):      250685575168 B (250.7 GB)
    Capacity In Use By Volumes:   237907656704 B (237.9 GB) (94.9% used)
    Capacity Not Allocated:       12777918464 B (12.8 GB) (5.1% free)
    |
    +-< Physical Store disk0s2 00000000-0000-0000-0000-000000000000
    |   -----------------------------------------------------------
    |   APFS Physical Store Disk:   disk0s2
    |   Size:                       250685575168 B (250.7 GB)
    |
    +-> Volume disk1s1 00000000-0000-0000-0000-000000000000
    |   ---------------------------------------------------
    |   APFS Volume Disk (Role):   disk1s1 (Data)
    |   Name:                      Macintosh HD - 데이터 (Case-insensitive)
    |   Mount Point:               /System/Volumes/Data
    |   Capacity Consumed:         223089950720 B (223.1 GB)
    |   FileVault:                 No
    |
    +-> Volume disk1s2 00000000-0000-0000-0000-000000000000
    |   ---------------------------------------------------
    |   APFS Volume Disk (Role):   disk1s2 (Preboot)
    |   Name:                      Preboot (Case-insensitive)
    |   Mount Point:               Not Mounted
    |   Capacity Consumed:         105754624 B (105.8 MB)
    |   FileVault:                 No
    |
    +-> Volume disk1s3 00000000-0000-0000-0000-000000000000
    |   ---------------------------------------------------
    |   APFS Volume Disk (Role):   disk1s3 (Recovery)
    |   Name:                      Recovery (Case-insensitive)
    |   Mount Point:               Not Mounted
    |   Capacity Consumed:         1046724608 B (1.0 GB)
    |   FileVault:                 No
    |
    +-> Volume disk1s4 00000000-0000-0000-0000-000000000000
    |   ---------------------------------------------------
    |   APFS Volume Disk (Role):   disk1s4 (VM)
    |   Name:                      VM (Case-insensitive)
    |   Mount Point:               /private/var/vm
    |   Capacity Consumed:         2148728832 B (2.1 GB)
    |   FileVault:                 No
    |
    +-> Volume disk1s5 00000000-0000-0000-0000-000000000000
        ---------------------------------------------------
        APFS Volume Disk (Role):   disk1s5 (System)
        Name:                      Macintosh HD (Case-insensitive)
        Mount Point:               /
        Capacity Consumed:         11378089984 B (11.4 GB)
        FileVault:                 No
~~~
~~~objectivec
luckyo$ diskutil apfs listVolumeGroups;
APFS Container (1 found)
|
+-- Container disk1 00000000-0000-0000-0000-000000000000
    |
    +-> Volume Group 00000000-0000-0000-0000-000000000000
        =================================================
        APFS Volume Disk (Role):   disk1s1 (Data)
        Name:                      Macintosh HD - 데이터
        Volume UUID:               00000000-0000-0000-0000-000000000000
        Capacity Consumed:         223088590848 B (223.1 GB)
        -------------------------------------------------
        APFS Volume Disk (Role):   disk1s5 (System)
        Name:                      Macintosh HD
        Volume UUID:               00000000-0000-0000-0000-000000000000
        Capacity Consumed:         11378089984 B (11.4 GB)
~~~
### 위와같은 정보를 code 로 구하고 싶었다.
~~~objectivec
- (NSString*)getDefaultVolumeName
{
    NSString *defaultVolumeName = @"";
    NSArray<NSURL*> *vols = [NSFileManager.defaultManager mountedVolumeURLsIncludingResourceValuesForKeys:nil options: 0 ];
    vols = [vols arrayByAddingObject:[NSURL fileURLWithPath:@"/System/Volumes/Data"]]; // the root's Data vol isn't added by default
    NSMutableArray<NSString*> *lines = [NSMutableArray new];
    for (NSURL *vol in vols) {
        // vol 값이 "file:///System/Volumes/Data/" 일 경우만 값을 구하면 될 듯..
        NSDictionary *d = [vol resourceValuesForKeys:@[
            NSURLVolumeIsBrowsableKey,
            NSURLVolumeIsRootFileSystemKey,
            NSURLVolumeIdentifierKey,
            NSURLVolumeNameKey
        ] error:nil];

        //output..
        id isRootPath = volumeInfo[NSURLVolumeIsRootFileSystemKey];
        NSInteger isRootFileSystem = [(NSNumber *)isRootPath integerValue];

        if (isRootFileSystem == 1)
        {
            defaultVolumeName = volumeInfo[NSURLVolumeNameKey];
            break;
        }

//        struct statfs fsinfo;
//        statfs(vol.path.UTF8String, &fsinfo);
//        NSString *bsdName = [NSString stringWithUTF8String:fsinfo.f_mntfromname];
//        bsdName = [bsdName lastPathComponent];
//
//        [lines addObject:[NSString stringWithFormat:@"%@, %@, %@, %@", bsdName, vol.path, d[NSURLVolumeIsBrowsableKey], d[NSURLVolumeNameKey]]];
    }
    NSLog(@"\n%@", [lines componentsJoinedByString:@"\n"]);
    return defaultVolumeName;
}


~~~
### NSDictionary *d에 대한 output
~~~
Printing description of d:
{
    NSURLVolumeIdentifierKey = {length = 8, bytes = 0x6745640000000000};
    NSURLVolumeIsBrowsableKey = 1;
    NSURLVolumeIsRootFileSystemKey = 1;
    NSURLVolumeNameKey = "Macintosh HD";
}
Printing description of d:
{
    NSURLVolumeIdentifierKey = {length = 8, bytes = 0xc623650000000000};
    NSURLVolumeIsBrowsableKey = 0;
    NSURLVolumeIsRootFileSystemKey = 0;
    NSURLVolumeNameKey = VM;
}
Printing description of d:
{
    NSURLVolumeIdentifierKey = {length = 8, bytes = 0x6998660000000000};
    NSURLVolumeIsBrowsableKey = 0;
    NSURLVolumeIsRootFileSystemKey = 0;
    NSURLVolumeNameKey = home;
}
Printing description of d:
{
    NSURLVolumeIdentifierKey = {length = 8, bytes = 0x51dc680000000000};
    NSURLVolumeIsBrowsableKey = 1;
    NSURLVolumeIsRootFileSystemKey = 0;
    NSURLVolumeNameKey = "WD Drive";
}
Printing description of d:
{
    NSURLVolumeIdentifierKey = {length = 8, bytes = 0xcd7c6d0000000000};
    NSURLVolumeIsBrowsableKey = 1;
    NSURLVolumeIsRootFileSystemKey = 0;
    NSURLVolumeNameKey = FasooStorage;
}
Printing description of d:
{
    NSURLVolumeIdentifierKey = {length = 8, bytes = 0xf241700000000000};
    NSURLVolumeIsBrowsableKey = 1;
    NSURLVolumeIsRootFileSystemKey = 0;
    NSURLVolumeNameKey = Lclient;
}
Printing description of d:
{
    NSURLVolumeIdentifierKey = {length = 8, bytes = 0x6745640000000000};
    NSURLVolumeIsBrowsableKey = 1;
    NSURLVolumeIsRootFileSystemKey = 1;
    NSURLVolumeNameKey = "Macintosh HD";
}

~~~
