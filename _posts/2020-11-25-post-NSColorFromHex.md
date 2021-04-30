---
title: "NSColor from HEX.."
categories:
  - objctive-c
tags:
  - Obj-C
  - UI
---

## HEX 값으로 NSColor 생성 후 setTextColor 로 적용하는 방법.

### 순서.
1. NSString 값을 HexInt 로 변환.
2. HexInt 값을 RGBA 값으로 변환하여 NSColor 생성.
3. setTextColor에 NSColor 적용.

### 1. NSString 값을 HexInt 로 변환.
~~~objectivec
unsigned rgbValue = 0;
NSScanner *scanner = [NSScanner scannerWithString:[#eb5757]];
[scanner setScanLocation:1]; // bypass '#' character
[scanner scanHexInt:&rgbValue];
~~~
### 2. HexInt 값을 RGBA 값으로 변환하여 NSColor 생성.
~~~objectivec
//매크로 등록..
 #define NSColorFromRGB(rgbValue) [NSColor colorWithCalibratedRed:((float)((rgbValue & 0xFF0000) >> 16))/255.0 green:((float)((rgbValue & 0xFF00) >> 8))/255.0 blue:((float)(rgbValue & 0xFF))/255.0 alpha:1.0]

NSColor *hexColor = NSColorFromRGB(rgbValue);
~~~
### 3. setTextColor에 NSColor 적용.
~~~objectivec
[self.textField setTextColor:hexColor];
~~~

### etc : RGB 값으로 NSColor 설정하기.
~~~objectivec
NSColor *myColor = [NSColor colorWithCalibratedRed:37.0/255.0
                                            green:193.0/255.0
                                            blue:111.0/255.0
                                            alpha:1.0f];
~~~
