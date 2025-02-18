---
title: appium6：appium查找和点击调用链
author: 独孤流
date: 2025-02-13 12:05:00 +0800
categories: [autotest, appium]
tags: [autotest]     # TAG names should always be lowercase
---

![image](/assets/img/test/appium/appium.drawio.png)

1、python文件里从查找元素到点击元素
```
welcomeStartBtn = WebDriverWait(driver, 2.5).until(
            EC.element_to_be_clickable((By.XPATH, '//XCUIElementTypeButton[@label="马上开始"]'))
        )
```

2、appium收到的请求
```
POST /session/c5cf401e-596f-44dc-87fe-49d1969d532a/element {"using":"xpath","value":"//XCUIElementTypeButton[@label=\"马上开始\"]"}
```

3、XCUITestDriver处理请求
```
[c5cf401e][XCUITestDriver@e9eb] Calling AppiumDriver.findElement() with args: ["xpath","//XCUIElementTypeButton[@label=\"马上开始\"]","c5cf401e-596f-44dc-87fe-49d1969d532a"]
[c5cf401e][XCUITestDriver@e9eb] Executing command 'findElement'
[c5cf401e][XCUITestDriver@e9eb] Valid locator strategies for this request: xpath, id, name, class name, -ios predicate string, -ios class chain, accessibility id, css selector
[c5cf401e][XCUITestDriver@e9eb] Waiting up to 0 ms for condition
[c5cf401e][XCUITestDriver@e9eb] Matched '/element' to command name 'findElement'
[c5cf401e][XCUITestDriver@e9eb] Proxying [POST /element] to [POST http://127.0.0.1:8100/session/4F15956B-A028-4B39-8BCE-C5D573EA023B/element] with body: {"using":"xpath","value":"//XCUIElementTypeButton[@label=\"马上开始\"]"}
```

4、`wda`的`FBFindElementCommands`收到请求处理
```
@implementation FBFindElementCommands
+ (NSArray *)routes
{
  return
  @[
    [[FBRoute POST:@"/element"] respondWithTarget:self action:@selector(handleFindElement:)],
    [[FBRoute POST:@"/elements"] respondWithTarget:self action:@selector(handleFindElements:)],
    [[FBRoute POST:@"/element/:uuid/element"] respondWithTarget:self action:@selector(handleFindSubElement:)],
    [[FBRoute POST:@"/element/:uuid/elements"] respondWithTarget:self action:@selector(handleFindSubElements:)],
    [[FBRoute GET:@"/wda/element/:uuid/getVisibleCells"] respondWithTarget:self action:@selector(handleFindVisibleCells:)],
#if TARGET_OS_TV
    [[FBRoute GET:@"/element/active"] respondWithTarget:self action:@selector(handleGetFocusedElement:)],
#else
    [[FBRoute GET:@"/element/active"] respondWithTarget:self action:@selector(handleGetActiveElement:)],
#endif
  ];
}

+ (id<FBResponsePayload>)handleFindElement:(FBRouteRequest *)request
{
// activeApplication是根节点
  FBSession *session = request.session;
  XCUIElement *element = [self.class elementUsing:request.arguments[@"using"]
                                        withValue:request.arguments[@"value"]
                                            under:session.activeApplication];
  if (!element) {
    return FBNoSuchElementErrorResponseForRequest(request);
  }
  return FBResponseWithCachedElement(element, request.session.elementCache, FBConfiguration.shouldUseCompactResponses);
}

- (XCUIApplication *)makeApplicationWithBundleId:(NSString *)bundleIdentifier
{
  return nil != self.testedApplication && [bundleIdentifier isEqualToString:(NSString *)self.testedApplication.bundleID]
    ? self.testedApplication
    : [[XCUIApplication alloc] initWithBundleIdentifier:bundleIdentifier];
}

```

[HTTP] --> GET /session/c5cf401e-596f-44dc-87fe-49d1969d532a/element/0E000000-0000-0000-C7FD-000000000000/displayed {}

Proxying [GET /session/c5cf401e-596f-44dc-87fe-49d1969d532a/element/0E000000-0000-0000-C7FD-000000000000/displayed] to [GET http://127.0.0.1:8100/session/4F15956B-A028-4B39-8BCE-C5D573EA023B/element/0E000000-0000-0000-C7FD-000000000000/displayed]

----
滑动操作
```
[HTTP] --> POST /session/c5cf401e-596f-44dc-87fe-49d1969d532a/actions {"actions":[{"type":"pointer","parameters":{"pointerType":"touch"},"id":"touch","actions":[{"type":"pointerMove","duration":250,"x":375,"y":406,"origin":"viewport"},{"type":"pointerDown","duration":0,"button":0},{"type":"pointerMove","duration":250,"x":0,"y":406,"origin":"viewport"},{"type":"pointerUp","duration":0,"button":0}]}]}
```

代理执行转发
```
[XCUITestDriver@e9eb] Proxying [POST /actions] to [POST http://127.0.0.1:8100/session/4F15956B-A028-4B39-8BCE-C5D573EA023B/actions] with body: {"actions":[{"type":"pointer","parameters":{"pointerType":"touch"},"id":"touch","actions":[{"type":"pointerMove","duration":250,"x":375,"y":406,"origin":"viewport"},{"type":"pointerDown","duration":0,"button":0},{"type":"pointerMove","duration":250,"x":0,"y":406,"origin":"viewport"},{"type":"pointerUp","duration":0,"button":0}]}]}
```

`WDA` -> `FBTouchActionCommands.m`
```
+ (NSArray *)routes
{
  return
  @[
    [[FBRoute POST:@"/actions"] respondWithTarget:self action:@selector(handlePerformW3CTouchActions:)],
  ];
}
+ (id<FBResponsePayload>)handlePerformW3CTouchActions:(FBRouteRequest *)request
{
  XCUIApplication *application = request.session.activeApplication;
  NSArray *actions = (NSArray *)request.arguments[@"actions"];
  NSError *error;
  if (![application fb_performW3CActions:actions elementCache:request.session.elementCache error:&error]) {
    return FBResponseWithUnknownError(error);
  }
  return FBResponseWithOK();
}

[XCUIDevice.sharedDevice eventSynthesizer] synthesizeEvent:record
```
----
click
点击转发：
```
Proxying [POST /session/c5cf401e-596f-44dc-87fe-49d1969d532a/element/0E000000-0000-0000-C7FD-000000000000/click] to [POST http://127.0.0.1:8100/session/4F15956B-A028-4B39-8BCE-C5D573EA023B/element/0E000000-0000-0000-C7FD-000000000000/click]
```
WDA->`FBElementCommands.m`
```
[[FBRoute POST:@"/element/:uuid/click"] respondWithTarget:self action:@selector(handleClick:)],

+ (id<FBResponsePayload>)handleClick:(FBRouteRequest *)request
{
  FBElementCache *elementCache = request.session.elementCache;
  XCUIElement *element = [elementCache elementForUUID:(NSString *)request.parameters[@"uuid"] checkStaleness:YES];
#if TARGET_OS_IOS
  [element tap];
#elif TARGET_OS_TV
  NSError *error = nil;
  if (![element fb_selectWithError:&error]) {
    return FBResponseWithStatus([FBCommandStatus invalidElementStateErrorWithMessage:error.description traceback:nil]);
  }
#endif
  return FBResponseWithOK();
}
```

----
设置内容

```
emailInput.send_keys("a1@test.com")
```

appium 收到请求

```
[HTTP] --> POST /session/652863ad-9fd4-4b98-b658-e67061595f9f/element/4F000000-0000-0000-4903-010000000000/value {"text":"a1@test.com","value":["a","1","@","t","e","s","t",".","c","o","m"]}
```

XUITestDriver处理请求并转发
```
[652863ad][XCUITestDriver@7e47] Calling AppiumDriver.setValue() with args: [["a","1","@","t","e","s","t",".","c","o","m"],"4F000000-0000-0000-4903-010000000000","652863ad-9fd4-4b98-b658-e67061595f9f"]
[652863ad][XCUITestDriver@7e47] Executing command 'setValue'
[652863ad][XCUITestDriver@7e47] Matched '/element/4F000000-0000-0000-4903-010000000000/value' to command name 'setValue'
[652863ad][XCUITestDriver@7e47] Added 'text' property "a1@test.com" to 'setValue' request body
[652863ad][XCUITestDriver@7e47] Proxying [POST /element/4F000000-0000-0000-4903-010000000000/value] to [POST http://127.0.0.1:8100/session/E932FD60-FD2E-4BEF-B8BC-61A3DB079C2F/element/4F000000-0000-0000-4903-010000000000/value] with body: {"value":["a","1","@","t","e","s","t",".","c","o","m"],"text":"a1@test.com"}
```

WDA -> FBElementCommands.m
```
[[FBRoute POST:@"/element/:uuid/value"] respondWithTarget:self action:@selector(handleSetValue:)],

+ (id<FBResponsePayload>)handleSetValue:(FBRouteRequest *)request
{
  FBElementCache *elementCache = request.session.elementCache;
  XCUIElement *element = [elementCache elementForUUID:(NSString *)request.parameters[@"uuid"]
                                       checkStaleness:YES];
  id value = request.arguments[@"value"] ?: request.arguments[@"text"];
  if (!value) {
    return FBResponseWithStatus([FBCommandStatus invalidArgumentErrorWithMessage:@"Neither 'value' nor 'text' parameter is provided" traceback:nil]);
  }
  NSString *textToType = [value isKindOfClass:NSArray.class]
    ? [value componentsJoinedByString:@""]
    : value;
  XCUIElementType elementType = [element elementType];
#if !TARGET_OS_TV
  if (elementType == XCUIElementTypePickerWheel) {
    [element adjustToPickerWheelValue:textToType];
    return FBResponseWithOK();
  }
#endif
  if (elementType == XCUIElementTypeSlider) {
    CGFloat sliderValue = textToType.floatValue;
    if (sliderValue < 0.0 || sliderValue > 1.0 ) {
      return FBResponseWithStatus([FBCommandStatus invalidArgumentErrorWithMessage:@"Value of slider should be in 0..1 range" traceback:nil]);
    }
    [element adjustToNormalizedSliderPosition:sliderValue];
    return FBResponseWithOK();
  }
  NSUInteger frequency = (NSUInteger)[request.arguments[@"frequency"] longLongValue] ?: [FBConfiguration maxTypingFrequency];
  NSError *error = nil;
  if (![element fb_typeText:textToType
                shouldClear:NO
                  frequency:frequency
                      error:&error]) {
    return FBResponseWithStatus([FBCommandStatus invalidElementStateErrorWithMessage:error.description traceback:nil]);
  }
  return FBResponseWithOK();
}

// 调用系统方法
BOOL FBTypeText(NSString *text, NSUInteger typingSpeed, NSError **error)
{
  NSString *name = text.length <= MAX_TEXT_ABBR_LEN
    ? [NSString stringWithFormat:@"Type '%@'", text]
    : [NSString stringWithFormat:@"Type '%@…'", [text substringToIndex:MAX_TEXT_ABBR_LEN]];
  XCSynthesizedEventRecord *eventRecord = [[XCSynthesizedEventRecord alloc] initWithName:name];
  XCPointerEventPath *ep = [[XCPointerEventPath alloc] initForTextInput];
  [ep typeText:text
      atOffset:0.0
   typingSpeed:typingSpeed
  shouldRedact:NO];
  [eventRecord addPointerEventPath:ep];
  return [FBXCTestDaemonsProxy synthesizeEventWithRecord:eventRecord error:error];
}
```

----
调用卸载、安装app的流程

模拟器
https://github.com/appium/appium/tree/master
https://github.com/appium/appium-xcuitest-driver
https://github.com/appium/appium-ios-simulator
https://github.com/appium/node-simctl

真机
https://github.com/appium/appium/tree/master
https://github.com/appium/appium-xcuitest-driver
https://github.com/appium/appium-ios-device
调用各个服务
real-device.js 
services.startInstallationProxyService
AfcService
com.apple.crashreportcopymobile

调用真机或模拟器的device初始化
determineDevice()
const device = await getSimulator(this.opts.udid） {getSimulator} from 'appium-ios-simulator';
const device = new RealDevice(this.opts.udid, this.log); appium-ios-device

https://github.com/appium/appium-xcuitest-driver/blob/master/lib/simulator-management.js
-> https://github.com/appium/appium-ios-device
-> https://github.com/appium/appium-ios-simulator/blob/master/lib/extensions/applications.js
-> https://github.com/appium/node-simctl/blob/master/lib/simctl.js
-> https://github.com/appium/node-simctl/blob/master/lib/subcommands/uninstall.js
-> this.exec('uninstall',...)
-> https://github.com/appium/node-simctl/blob/master/lib/simctl.js -> 'simctl xxxx' 
-> `xcrun simctl xxxxx`
`xcrun simctl uninstall booted com.example.myapp`
`xcrun simctl uninstall ABCD-1234-5678-90EF com.example.myapp`


安装同理
-> https://github.com/appium/node-simctl/blob/master/lib/subcommands/install.js
-> this.exec('install',...)
-> https://github.com/appium/node-simctl/blob/master/lib/simctl.js -> 'simctl xxxx' 
`xcrun simctl install ABCD-1234-5678-90EF /xx/xx.ipa|app`


