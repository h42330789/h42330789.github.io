---
title: iPhone 型号整理更新到iPhone 16系列
author: 独孤流
date: 2023-10-02 23:12:00 +0800
categories: [other, 其他]
tags: [mac]     # TAG names should always be lowercase
---
参考：
- [iPod系列：https://everymac.com/systems/apple/ipod/index-ipod-specs.html](https://everymac.com/systems/apple/ipod/index-ipod-specs.html)
- [iPad系列：https://everymac.com/systems/apple/ipad/index-ipad-specs.html](https://everymac.com/systems/apple/ipad/index-ipad-specs.html)
- [iPhone系列：https://everymac.com/systems/apple/iphone/index-iphone-specs.html](https://everymac.com/systems/apple/iphone/index-iphone-specs.html)

- [swift 获取设备型号](https://blog.csdn.net/GSSGoodLuck/article/details/122085870)
- [获取手机系统型号 https://www.theiphonewiki.com/wiki/Models](https://www.theiphonewiki.com/wiki/Models)
- [获取最新型号查询地址 https://everymac.com](https://everymac.com)

> ### 前言
> 每次项目里需要查找iPhone型号时到处找代码，但是适配的型号都不是最新的，于是把找到更新最勤的几个网站数据同步下来，同时将网站加入到文档中，下次有更新时第一时间去把网站数据更新下来

由于模拟器返回的x86_64,但是模拟器的名称不能修改且是型号，所以模拟器使UIDevice.current.name

Swift参考代码如下
```
//
//  UIDeviceExt.swift
//  SysModel
//
//  Created by MacBook Pro on 10/2/23.
//

import UIKit

// https://blog.csdn.net/GSSGoodLuck/article/details/122085870
//获取手机系统型号
//https://www.theiphonewiki.com/wiki/Models
// 获取最新型号查询地址
// https://everymac.com/ultimate-mac-lookup/?search_keywords=iPhone16%2C2

public extension UIDevice {
    
    static var modelName: String {
        var systemInfo = utsname()
        uname(&systemInfo)
        let machineMirror = Mirror(reflecting: systemInfo.machine)
        let identifier = machineMirror.children.reduce("") { identifier, element in
            guard let value = element.value as? Int8, value != 0 else { return identifier }
            return identifier + String(UnicodeScalar(UInt8(value)))
        }
        
        switch identifier {
        // iPod
        // https://everymac.com/systems/apple/ipod/index-ipod-specs.html
        case "iPod1,1": return "iPod Touch 1"
        case "iPod2,1": return "iPod Touch 2"
        case "iPod3,1": return "iPod Touch 3"
        case "iPod4,1": return "iPod Touch 4"
        case "iPod5,1": return "iPod Touch 5"
        case "iPod7,1": return "iPod Touch 6"
        case "iPod9,1": return "iPod Touch 7"
            
        // iPhone
        // https://everymac.com/systems/apple/iphone/index-iphone-specs.html
        case "iPhone1,1": return "iPhone"
        case "iPhone1,2": return "iPhone 3G"
        case "iPhone2,1": return "iPhone 3GS"
        case "iPhone3,1", "iPhone3,2", "iPhone3,3": return "iPhone 4"
        case "iPhone4,1": return "iPhone 4s"
        case "iPhone5,1", "iPhone5,2": return "iPhone 5"
        case "iPhone5,3", "iPhone5,4": return "iPhone 5c"
        case "iPhone6,1", "iPhone6,2": return "iPhone 5s"
        case "iPhone7,2": return "iPhone 6"
        case "iPhone7,1": return "iPhone 6 Plus"
        case "iPhone8,1": return "iPhone 6s"
        case "iPhone8,2": return "iPhone 6s Plus"
        case "iPhone8,4": return "iPhone SE"
        case "iPhone9,1", "iPhone9,3": return "iPhone 7"
        case "iPhone9,2", "iPhone9,4": return "iPhone 7 Plus"
        case "iPhone10,1", "iPhone10,4": return "iPhone 8"
        case "iPhone10,2", "iPhone10,5": return "iPhone 8 Plus"
        case "iPhone10,3", "iPhone10,6": return "iPhone X"
        case "iPhone11,2": return "iPhone XS"
        case "iPhone11,4", "iPhone11,6": return "iPhone XS MAX"
        case "iPhone11,8": return "iPhone XR"
        case "iPhone12,1": return "iPhone 11"
        case "iPhone12,3": return "iPhone 11 Pro"
        case "iPhone12,5": return "iPhone 11 Pro Max"
        case "iPhone12,8": return "iPhone SE 2"
        case "iPhone13,1": return "iPhone 12 mini"
        case "iPhone13,2": return "iPhone 12"
        case "iPhone13,3": return "iPhone 12 Pro"
        case "iPhone13,4": return "iPhone 12 Pro Max"
        case "iPhone14,4": return "iPhone 13 mini"
        case "iPhone14,5": return "iPhone 13"
        case "iPhone14,2": return "iPhone 13 Pro"
        case "iPhone14,3": return "iPhone 13 Pro Max"
        case "iPhone14,6": return "iPhone SE 3"
        case "iPhone14,7": return "iPhone 14"
        case "iPhone14,8": return "iPhone 14 Plus"
        case "iPhone15,2": return "iPhone 14 Pro"
        case "iPhone15,3": return "iPhone 14 Pro Max"
        case "iPhone15,4": return "iPhone 15"
        case "iPhone15,5": return "iPhone 15 Plus"
        case "iPhone16,1": return "iPhone 15 Pro"
        case "iPhone16,2": return "iPhone 15 Pro Max"
        case "iPhone17,3": return "iPhone 16"
        case "iPhone17,4": return "iPhone 16 Plus"
        case "iPhone17,1": return "iPhone 16 Pro"
        case "iPhone17,2": return "iPhone 16 Pro Max"
            
        // iPad
        // https://everymac.com/systems/apple/ipad/index-ipad-specs.html
        case "iPad1,1": return "iPad"
        case "iPad1,2": return "iPad 3G"
        case "iPad2,1", "iPad2,2", "iPad2,3", "iPad2,4": return "iPad 2"
        case "iPad3,1", "iPad3,2", "iPad3,3": return "iPad 3"
        case "iPad3,4", "iPad3,5", "iPad3,6": return "iPad 4"
        case "iPad6,11", "iPad6,12": return "iPad 5"
        case "iPad7,5", "iPad7,6": return "iPad 6"
        case "iPad7,11", "iPad7,12": return "iPad 7"
        case "iPad11,6", "iPad11,7": return "iPad 8"
        case "iPad12,1", "iPad12,2": return "iPad 9"
        case "iPad13,18", "iPad13,19": return "iPad 10"
            
        // iPad Mini
        case "iPad2,5", "iPad2,6", "iPad2,7": return "iPad Mini"
        case "iPad4,4", "iPad4,5", "iPad4,6": return "iPad Mini 2"
        case "iPad4,7", "iPad4,8", "iPad4,9": return "iPad Mini 3"
        case "iPad5,1", "iPad5,2": return "iPad mini 4"
        case "iPad11,1", "iPad11,2": return "iPad mini 5"
        case "iPad14,1", "iPad14,2": return "iPad mini 6"
            
        // iPad Air
        case "iPad4,1", "iPad4,2", "iPad4,3": return "iPad Air"
        case "iPad5,3", "iPad5,4": return "iPad Air 2"
        case "iPad11,3", "iPad11,4": return "iPad Air 3"
        case "iPad13,1", "iPad13,2": return "iPad Air 4"
        case "iPad13,16", "iPad13,17": return "iPad Air 5"
        case "iPad14,8", "iPad14,9": return "iPad Air M2 11"
        case "iPad14,10", "iPad14,11": return "iPad Air M2 13"
            
        // iPad  pro
        case "iPad6,3", "iPad6,4": return "iPad Pro (9.7-inch)"
        case "iPad6,7", "iPad6,8": return "iPad Pro (12.9-inch)"
        case "iPad7,1", "iPad7,2": return "iPad Pro 2(12.9-inch)"
        case "iPad7,3", "iPad7,4": return "iPad Pro (10.5-inch)"
        case "iPad8,1", "iPad8,2", "iPad8,3", "iPad8,4": return "iPad Pro (11-inch)"
        case "iPad8,5", "iPad8,6", "iPad8,7", "iPad8,8": return "iPad Pro 3 (12.9-inch)"
        case "iPad8,9", "iPad8,10": return "iPad Pro 2 (11-inch)"
        case "iPad8,11", "iPad8,12": return "iPad Pro 4 (12.9-inch)"
        case "iPad13,4", "iPad13,5", "iPad13,6", "iPad13,7": return "iPad Pro 4 (11-inch)"
        case "iPad13,8", "iPad13,9", "iPad13,10", "iPad13,11": return "iPad Pro 5 (12.9-inch)"
        case "iPad14,5", "iPad14,6": return "iPad Pro 6 (12.9-inch)"
        case "iPad16,3", "iPad16,4": return "iPad Pro M4 11"
        case "iPad16,5", "iPad16,6": return "iPad Pro M4 13"
          
        case "AppleTV2,1": return "Apple TV 2"
        case "AppleTV3,1", "AppleTV3,2": return "Apple TV 3"
        case "AppleTV5,3": return "Apple TV 4"
            
        // 模拟器的名称默认就是型号的名称
        case "i386", "x86_64": return "\(UIDevice.current.name) Simulator"
            
        // iMac
        case "iMac21,1", "iMac21,2": return "iMac (24-inch, M1, 2021)"
            
        // Mac mini
        case "Macmini9,1": return "Mac mini (M1, 2020)"
            
        // MacBook Air
        case "MacBookAir10,1": return "MacBook Air (Late 2020)"
            
        // MacBook Pro
        case "MacBookPro17,1": return "MacBook Pro (13-inch, M1, 2020)"
        case "MacBookPro18,3", "MacBookPro18,4": return "MacBook Pro (14-inch, 2021)"
        case "MacBookPro18,1", "MacBookPro18,2": return "MacBook Pro (16-inch, 2021)"
            
        default: return identifier
        }
    }
}

```

Objective-C参考代码
```
+ (NSString *)sysModelName {
    struct utsname systemInfo;
    uname(&systemInfo);
    NSString *deviceModel = [NSString stringWithCString:systemInfo.machine encoding:NSUTF8StringEncoding];

    ///iPhone
    ///https://everymac.com/systems/apple/iphone/index-iphone-specs.html
    if ([deviceModel isEqualToString:@"iPhone3,1"])    return @"iPhone 4";
    if ([deviceModel isEqualToString:@"iPhone3,2"])    return @"iPhone 4";
    if ([deviceModel isEqualToString:@"iPhone3,3"])    return @"iPhone 4";
    if ([deviceModel isEqualToString:@"iPhone4,1"])    return @"iPhone 4S";
    if ([deviceModel isEqualToString:@"iPhone5,1"])    return @"iPhone 5";
    if ([deviceModel isEqualToString:@"iPhone5,2"])    return @"iPhone 5";
    if ([deviceModel isEqualToString:@"iPhone5,3"])    return @"iPhone 5c";
    if ([deviceModel isEqualToString:@"iPhone5,4"])    return @"iPhone 5c";
    if ([deviceModel isEqualToString:@"iPhone6,1"])    return @"iPhone 5s";
    if ([deviceModel isEqualToString:@"iPhone6,2"])    return @"iPhone 5s";
    if ([deviceModel isEqualToString:@"iPhone7,1"])    return @"iPhone 6 Plus";
    if ([deviceModel isEqualToString:@"iPhone7,2"])    return @"iPhone 6";
    if ([deviceModel isEqualToString:@"iPhone8,1"])    return @"iPhone 6s";
    if ([deviceModel isEqualToString:@"iPhone8,2"])    return @"iPhone 6s Plus";
    if ([deviceModel isEqualToString:@"iPhone8,4"])    return @"iPhone SE";
    if ([deviceModel isEqualToString:@"iPhone9,1"])    return @"iPhone 7";
    if ([deviceModel isEqualToString:@"iPhone9,2"])    return @"iPhone 7 Plus";
    if ([deviceModel isEqualToString:@"iPhone9,3"])    return @"iPhone 7";
    if ([deviceModel isEqualToString:@"iPhone9,4"])    return @"iPhone 7 Plus";
    if ([deviceModel isEqualToString:@"iPhone10,1"])   return @"iPhone_8";
    if ([deviceModel isEqualToString:@"iPhone10,4"])   return @"iPhone_8";
    if ([deviceModel isEqualToString:@"iPhone10,2"])   return @"iPhone_8_Plus";
    if ([deviceModel isEqualToString:@"iPhone10,5"])   return @"iPhone_8_Plus";
    if ([deviceModel isEqualToString:@"iPhone10,3"])   return @"iPhone X";
    if ([deviceModel isEqualToString:@"iPhone10,6"])   return @"iPhone X";
    if ([deviceModel isEqualToString:@"iPhone11,2"])   return @"iPhone XS";
    if ([deviceModel isEqualToString:@"iPhone11,4"])   return @"iPhone XS MAX";
    if ([deviceModel isEqualToString:@"iPhone11,6"])   return @"iPhone XS MAX";
    if ([deviceModel isEqualToString:@"iPhone11,8"])   return @"iPhone XR";
    if ([deviceModel isEqualToString:@"iPhone12,1"])   return @"iPhone 11";
    if ([deviceModel isEqualToString:@"iPhone12,3"])   return @"iPhone 11 Pro";
    if ([deviceModel isEqualToString:@"iPhone12,5"])   return @"iPhone 11 Pro Max";
    if ([deviceModel isEqualToString:@"iPhone12,8"])   return @"iPhone SE 2";
    if ([deviceModel isEqualToString:@"iPhone13,1"])   return @"iPhone 12 mini";
    if ([deviceModel isEqualToString:@"iPhone13,2"])   return @"iPhone 12";
    if ([deviceModel isEqualToString:@"iPhone13,3"])   return @"iPhone 12 Pro";
    if ([deviceModel isEqualToString:@"iPhone13,4"])   return @"iPhone 12 Pro Max";
    if ([deviceModel isEqualToString:@"iPhone14,4"])   return @"iPhone 13 mini";
    if ([deviceModel isEqualToString:@"iPhone14,5"])   return @"iPhone 13";
    if ([deviceModel isEqualToString:@"iPhone14,2"])   return @"iPhone 13 Pro";
    if ([deviceModel isEqualToString:@"iPhone14,3"])   return @"iPhone 13 Pro Max";
    if ([deviceModel isEqualToString:@"iPhone14,6"])   return @"iPhone SE 3";
    if ([deviceModel isEqualToString:@"iPhone14,7"])   return @"iPhone 14";
    if ([deviceModel isEqualToString:@"iPhone14,8"])   return @"iPhone 14 Plus";
    if ([deviceModel isEqualToString:@"iPhone15,2"])   return @"iPhone 14 Pro";
    if ([deviceModel isEqualToString:@"iPhone15,3"])   return @"iPhone 14 Pro Max";
    if ([deviceModel isEqualToString:@"iPhone15,4"])   return @"iPhone 15";
    if ([deviceModel isEqualToString:@"iPhone15,5"])   return @"iPhone 15 Plus";
    if ([deviceModel isEqualToString:@"iPhone16,1"])   return @"iPhone 15 Pro";
    if ([deviceModel isEqualToString:@"iPhone16,2"])   return @"iPhone 15 Pro Max";
    if ([deviceModel isEqualToString:@"iPhone17,3"])   return @"iPhone 16";
    if ([deviceModel isEqualToString:@"iPhone17,4"])   return @"iPhone 16 Plus";
    if ([deviceModel isEqualToString:@"iPhone17,1"])   return @"iPhone 16 Pro";
    if ([deviceModel isEqualToString:@"iPhone17,2"])   return @"iPhone 16 Pro Max";
    
    ///iPod
    ///https://everymac.com/systems/apple/ipod/index-ipod-specs.html
    if ([deviceModel isEqualToString:@"iPod1,1"])      return @"iPod Touch 1G";
    if ([deviceModel isEqualToString:@"iPod2,1"])      return @"iPod Touch 2G";
    if ([deviceModel isEqualToString:@"iPod3,1"])      return @"iPod Touch 3G";
    if ([deviceModel isEqualToString:@"iPod4,1"])      return @"iPod Touch 4G";
    if ([deviceModel isEqualToString:@"iPod5,1"])      return @"iPod Touch (5 Gen)";
    
    ///iPad
   ///https://everymac.com/systems/apple/ipad/index-ipad-specs.html
    if ([deviceModel isEqualToString:@"iPad1,1"])      return @"iPad";
    if ([deviceModel isEqualToString:@"iPad1,2"])      return @"iPad 3G";
    if ([deviceModel isEqualToString:@"iPad2,1"])      return @"iPad 2 (WiFi)";
    if ([deviceModel isEqualToString:@"iPad2,2"])      return @"iPad 2";
    if ([deviceModel isEqualToString:@"iPad2,3"])      return @"iPad 2 (CDMA)";
    if ([deviceModel isEqualToString:@"iPad2,4"])      return @"iPad 2";
    if ([deviceModel isEqualToString:@"iPad2,5"])      return @"iPad Mini (WiFi)";
    if ([deviceModel isEqualToString:@"iPad2,6"])      return @"iPad Mini";
    if ([deviceModel isEqualToString:@"iPad2,7"])      return @"iPad Mini (GSM+CDMA)";
    if ([deviceModel isEqualToString:@"iPad3,1"])      return @"iPad 3 (WiFi)";
    if ([deviceModel isEqualToString:@"iPad3,2"])      return @"iPad 3 (GSM+CDMA)";
    if ([deviceModel isEqualToString:@"iPad3,3"])      return @"iPad 3";
    if ([deviceModel isEqualToString:@"iPad3,4"])      return @"iPad 4 (WiFi)";
    if ([deviceModel isEqualToString:@"iPad3,5"])      return @"iPad 4";
    if ([deviceModel isEqualToString:@"iPad3,6"])      return @"iPad 4 (GSM+CDMA)";
    if ([deviceModel isEqualToString:@"iPad4,1"])      return @"iPad Air (WiFi)";
    if ([deviceModel isEqualToString:@"iPad4,2"])      return @"iPad Air (Cellular)";
    if ([deviceModel isEqualToString:@"iPad4,4"])      return @"iPad Mini 2 (WiFi)";
    if ([deviceModel isEqualToString:@"iPad4,5"])      return @"iPad Mini 2 (Cellular)";
    if ([deviceModel isEqualToString:@"iPad4,6"])      return @"iPad Mini 2";
    if ([deviceModel isEqualToString:@"iPad4,7"])      return @"iPad Mini 3";
    if ([deviceModel isEqualToString:@"iPad4,8"])      return @"iPad Mini 3";
    if ([deviceModel isEqualToString:@"iPad4,9"])      return @"iPad Mini 3";
    if ([deviceModel isEqualToString:@"iPad5,1"])      return @"iPad Mini 4 (WiFi)";
    if ([deviceModel isEqualToString:@"iPad5,2"])      return @"iPad Mini 4 (LTE)";
    if ([deviceModel isEqualToString:@"iPad5,3"])      return @"iPad Air 2";
    if ([deviceModel isEqualToString:@"iPad5,4"])      return @"iPad Air 2";
    if ([deviceModel isEqualToString:@"iPad6,3"])      return @"iPad Pro 9.7";
    if ([deviceModel isEqualToString:@"iPad6,4"])      return @"iPad Pro 9.7";
    if ([deviceModel isEqualToString:@"iPad6,7"])      return @"iPad Pro 12.9";
    if ([deviceModel isEqualToString:@"iPad6,8"])      return @"iPad Pro 12.9";
    if ([deviceModel isEqualToString:@"iPad14,8"])      return @"iPad Air M2 11";
    if ([deviceModel isEqualToString:@"iPad14,9"])      return @"iPad Air M2 11";
    if ([deviceModel isEqualToString:@"iPad14,10"])      return @"iPad Air M2 13";
    if ([deviceModel isEqualToString:@"iPad14,11"])      return @"iPad Air M2 13";
    if ([deviceModel isEqualToString:@"iPad14,3"])      return @"iPad Pro M4 11";
    if ([deviceModel isEqualToString:@"iPad14,4"])      return @"iPad Pro M4 11";
    if ([deviceModel isEqualToString:@"iPad14,5"])      return @"iPad Pro M4 13";
    if ([deviceModel isEqualToString:@"iPad14,6"])      return @"iPad Pro M4 13";

    if ([deviceModel isEqualToString:@"AppleTV2,1"])      return @"Apple TV 2";
    if ([deviceModel isEqualToString:@"AppleTV3,1"])      return @"Apple TV 3";
    if ([deviceModel isEqualToString:@"AppleTV3,2"])      return @"Apple TV 3";
    if ([deviceModel isEqualToString:@"AppleTV5,3"])      return @"Apple TV 4";

    if ([deviceModel isEqualToString:@"i386"] || [deviceModel isEqualToString:@"x86_64"]) {
        // 模拟器的名称默认就是型号的名称
        return [NSString stringWithFormat:@"%@ Simulator",UIDevice.currentDevice.name];
    }

    return deviceModel;
}
```