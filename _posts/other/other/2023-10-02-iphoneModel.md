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

- https://www.theiphonewiki.com/wiki/Models
- https://theapplewiki.com/wiki/Models

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
// https://www.theiphonewiki.com/wiki/Models
// https://theapplewiki.com/wiki/Models
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
        // https://theapplewiki.com/wiki/Models
        case "iPod1,1": return "iPod Touch 1"
        case "iPod2,1": return "iPod Touch 2"
        case "iPod3,1": return "iPod Touch 3"
        case "iPod4,1": return "iPod Touch 4"
        case "iPod5,1": return "iPod Touch 5"
        case "iPod7,1": return "iPod Touch 6"
        case "iPod9,1": return "iPod Touch 7"
            
        // iPhone
        // https://everymac.com/systems/apple/iphone/index-iphone-specs.html
        // https://theapplewiki.com/wiki/Models
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

----
增加从`https://theapplewiki.com/wiki/Models`自动抓取生成model的脚本

`apple_device_parser.py`
```
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

"""
从 https://theapplewiki.com/wiki/Models 网站抓取 Apple 设备型号信息并生成 Swift 代码。
生成的代码包含设备标识符到设备代数的映射关系。

依赖安装:
pip3 install requests beautifulsoup4

执行示例:
python3 apple_device_parser.py Device+Models.swift
"""

import requests
from bs4 import BeautifulSoup
from typing import List, Tuple, Dict, Set, Optional
import time
import random
import os
import re
import argparse

class AppleDeviceParser:
    """用于解析Apple设备型号信息的类"""
    
    def __init__(self):
        self.url = "https://theapplewiki.com/wiki/Models"
        self.headers = {
            'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36',
            'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8',
            'Accept-Language': 'zh-CN,zh;q=0.9,en;q=0.8',
            'Referer': 'https://theapplewiki.com/',
        }
        # 使用数组保存解析结果，每个元素是一个字典 {identifier: generation}
        self.device_list = []

    def download_webpage(self) -> str:
        """步骤1：下载网页内容"""
        try:
            time.sleep(random.uniform(1, 3))  # 添加随机延迟
            response = requests.get(self.url, headers=self.headers)
            response.raise_for_status()
            print("✅ 成功下载网页内容")
            return response.text
        except requests.RequestException as e:
            print(f"❌ 下载网页失败: {e}")
            return ""

    def find_target_tables(self, html_content: str) -> List[Tuple[BeautifulSoup, dict]]:
        """查找包含Generation和Identifier的表格，并记录表格的关键信息
        返回: [(table, table_info)]
        table_info包含：
        - total_columns: 总列数
        - generation_index: Generation列的索引
        - identifier_index: Identifier列的索引
        """
        soup = BeautifulSoup(html_content, 'html.parser')
        all_tables = soup.find_all('table', {'class': 'wikitable'})
        target_tables = []
        
        for table in all_tables:
            # 获取第一行（标题行）
            header_row = table.find('tr')
            if not header_row:
                continue
                
            # 获取标题行的所有单元格文本
            header_cells = header_row.find_all(['td', 'th'])
            header_texts = [self.get_cell_text(cell).lower() for cell in header_cells]
            
            # 检查是否包含必要的列
            if 'generation' in header_texts and 'identifier' in header_texts:
                table_info = {
                    'total_columns': len(header_cells),
                    'generation_index': header_texts.index('generation'),
                    'identifier_index': header_texts.index('identifier')
                }
                target_tables.append((table, table_info))
                print(f"找到目标表格:")
                print(f"  总列数: {table_info['total_columns']}")
                print(f"  Generation列索引: {table_info['generation_index']}")
                print(f"  Identifier列索引: {table_info['identifier_index']}")
                
        print(f"共找到 {len(target_tables)} 个目标表格")
        return target_tables

    def get_cell_text(self, cell: BeautifulSoup) -> str:
        """获取单元格的文本内容，去除引用标记和其他无关内容"""
        # 移除所有引用标记
        for ref in cell.find_all('sup', {'class': 'reference'}):
            ref.decompose()
        # 移除所有链接的样式，只保留文本
        for link in cell.find_all('a'):
            link.replace_with(link.get_text())
        return cell.get_text(strip=True)

    def process_table(self, table_info: Tuple[BeautifulSoup, dict]) -> None:
        """处理单个表格"""
        table, info = table_info
        rows = table.find_all('tr')
        if not rows:
            return
            
        total_columns = info['total_columns']
        generation_index = info['generation_index']
        identifier_index = info['identifier_index']
        
        print(f"\n处理表格:")
        print(f"总列数: {total_columns}")
        print(f"Generation列索引: {generation_index}")
        print(f"Identifier列索引: {identifier_index}")
        
        # 记录当前的Generation
        current_generation = None
        
        # 处理数据行
        for row_idx, row in enumerate(rows[1:], 1):  # 从第1行开始
            cells = row.find_all(['td', 'th'])
            row_texts = [self.get_cell_text(cell) for cell in cells]
            
            print(f"\n处理第 {row_idx} 行:")
            print(f"列数: {len(cells)}")
            print(f"内容: {row_texts}")
            
            # 检查是否是完整行，记录Generation
            if len(cells) == total_columns:
                current_generation = row_texts[generation_index]
                print(f"更新Generation: {current_generation}")
            
            # 检查是否包含Identifier（包括完整行）
            if len(cells) >= (total_columns - identifier_index):
                # 提取Identifier
                identifier_cell_index = len(cells) - (total_columns - identifier_index)
                if identifier_cell_index >= 0 and current_generation:
                    identifier_text = row_texts[identifier_cell_index]
                    print(f"检查Identifier文本: {identifier_text}")
                    
                    # 提取设备标识符
                    device_pattern = re.compile(r'(iPhone|iPad|iPod|Watch|Mac|iMac|MacBook|MacBookPro|MacBookAir|Macmini)\d+,\d+')
                    matches = device_pattern.finditer(identifier_text)
                    
                    for match in matches:
                        device_id = match.group()
                        # 将每个identifier-generation对作为独立的字典添加到列表中
                        self.device_list.append({device_id: current_generation})
                        print(f"✅ 添加设备: {device_id} -> {current_generation}")

    def extract_device_data(self, tables: List[Tuple[BeautifulSoup, dict]]) -> None:
        """步骤3：从表格中提取设备标识符和代数信息"""
        for table_info in tables:
            self.process_table(table_info)
        
        print(f"\n✅ 提取了 {len(self.device_list)} 条设备数据")
        
        # 打印所有提取的数据
        print("\n提取的所有数据:")
        for device_dict in self.device_list:
            for device_id, generation in device_dict.items():
                print(f"标识符: {device_id:<15} -> 代数: {generation}")

    def generate_swift_code(self) -> str:
        """生成Swift代码，使用device_list保持顺序"""
        if not self.device_list:
            return "// 没有找到设备数据"
            
        # Swift文件头部
        code_lines = [
            "//",
            "// DeviceModels.swift",
            "// 自动生成的设备型号映射代码",
            "// 数据来源: https://theapplewiki.com/wiki/Models",
            "//",
            "",
            "import UIKit",
            "",
            "public extension UIDevice {",
            "    ",
            "    static var modelName: String {",
            "        var systemInfo = utsname()",
            "        uname(&systemInfo)",
            "        let machineMirror = Mirror(reflecting: systemInfo.machine)",
            "        let identifier = machineMirror.children.reduce(\"\") { identifier, element in",
            "            guard let value = element.value as? Int8, value != 0 else { return identifier }",
            "            return identifier + String(UnicodeScalar(UInt8(value)))",
            "        }",
            "        ",
            "        switch identifier {",
        ]
        
        # 按设备类型分组但保持原始顺序
        current_type = None
        for device_dict in self.device_list:
            for device_id, generation in device_dict.items():
                # 获取设备类型
                device_type = device_id.split(',')[0].rstrip('0123456789')
                
                # 如果是新的设备类型，添加注释
                if device_type != current_type:
                    current_type = device_type
                    code_lines.append(f"\n            // {device_type} 设备")
                
                code_lines.append(f'            case "{device_id}":')
                code_lines.append(f'                return "{generation}"')
        
        code_lines.extend([
            "",
            "            // 模拟器的名称默认就是型号的名称",
            '            case "i386", "x86_64":',
            '                return "\\(UIDevice.current.name) Simulator"',
            "            default:",
            "                return identifier",
            "        }",
            "    }",
            "}",
        ])
        
        return '\n'.join(code_lines)
    
    def save_swift_file(self, swift_code: str, output_path: str) -> None:
        """步骤5：保存Swift代码到文件"""
        try:
            # 获取当前脚本所在目录
            script_dir = os.path.dirname(os.path.abspath(__file__))
            # 在脚本目录下创建输出文件
            output_path = os.path.join(script_dir, output_path)
            
            # 写入文件
            with open(output_path, 'w', encoding='utf-8') as f:
                f.write(swift_code)
            print(f"✅ Swift代码已保存到: {output_path}")
        except Exception as e:
            print(f"❌ 保存文件失败: {e}")

def main():
    # 设置命令行参数
    parser = argparse.ArgumentParser(description='从theapplewiki.com抓取Apple设备型号信息并生成Swift代码')
    parser.add_argument('output_path', help='生成的Swift文件路径')
    args = parser.parse_args()
    
    parser = AppleDeviceParser()
    
    # 步骤1：下载网页
    html_content = parser.download_webpage()
    if not html_content:
        print("程序终止：无法获取网页内容")
        return
        
    # 步骤2：查找目标表格
    target_tables = parser.find_target_tables(html_content)
    if not target_tables:
        print("程序终止：未找到包含所需信息的表格")
        return
        
    # 步骤3：提取数据
    parser.extract_device_data(target_tables)
    if not parser.device_list:
        print("程序终止：未能提取到设备数据")
        return
        
    # 步骤4：生成Swift代码
    swift_code = parser.generate_swift_code()
    
    # 步骤5：保存到文件
    parser.save_swift_file(swift_code, args.output_path)

if __name__ == "__main__":
    main() 

# 初次执行，需要安装依赖
# pip3 install requests
# pip3 install beautifulsoup4
# 每次直接命令行执行
# python3 apple_device_parser.py Device+Models.swift
```