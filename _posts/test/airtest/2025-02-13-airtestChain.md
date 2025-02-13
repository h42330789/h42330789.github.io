---
title: appium6：airtest查找和点击调用链
author: 独孤流
date: 2025-02-13 13:05:00 +0800
categories: [autotest, airtest]
tags: [autotest]     # TAG names should always be lowercase
---

https://github.com/AirtestProject/Airtest/blob/master/airtest/core/ios/ios.py
https://github.com/AirtestProject/Airtest/blob/master/airtest/core/api.py
https://github.com/openatx/facebook-wda
https://github.com/openatx/facebook-wda/blob/master/wda/__init__.py

```
def touch(v, times=1, **kwargs):
    """
    Perform touch action on the target.
    :param v: Target to touch, can be Template, string or pos.
    :param times: How many times to touch.
    :param kwargs: Other parameters.
    :return: None
    """
    if isinstance(v, Template):
        pos = loop_find(v, timeout=ST.FIND_TIMEOUT)
    elif isinstance(v, (str, unicode)):
        pos = loop_find(v, timeout=ST.FIND_TIMEOUT)
    else:
        pos = v
    G.DEVICE.touch(pos, times=times, **kwargs)
```

源码路径：`airtest/core/ios/ios.py`
```
def touch(self, pos, duration=0.01):
        """
        Args:
            pos: coordinates (x, y), can be float(percent) or int
            duration (optional): tap_hold duration

        Returns: None(iOS may all use vertical screen coordinates, so coordinates will not be returned.)

        Examples:
            >>> touch((100, 100))
            >>> touch((0.5, 0.5), duration=1)

        """
        x, y = pos
        if not (x < 1 and y < 1):
            x, y = int(x * self.touch_factor), int(y * self.touch_factor)
        # 根据用户安装的wda版本判断是否调用快速点击接口
        if self.using_ios_tagent:
            x, y = self._transform_xy(pos)
            self._quick_click(x, y, duration)
        else:
            self.driver.click(x, y, duration)


if parsed not in ["localhost", "127.0.0.1"] and "." in parsed:
            # Connect remote device via url.
            self.is_local_device = False
            self.driver = wda.Client(self.addr)
        else:
            # Connect local device via url.
            self.is_local_device = True
            if parsed in ["localhost", "127.0.0.1"]:
                if not udid:
                    udid = self._get_default_device()
                self.udid = udid
            else:
                self.udid = parsed
            self.driver = wda.USBClient(udid=self.udid, port=8100, wda_bundle_id=self.wda_bundle_id)

// 点击
def click(self, x, y, duration: Optional[float] = None):
        """
        Combine tap and tap_hold

        Args:
            x, y: can be float(percent) or int
            duration (optional): tap_hold duration
        """
        x, y = self._percent2pos(x, y)
        if duration:
            return self.tap_hold(x, y, duration)
        return self.tap(x, y)

// 调用到iPhone上
def tap(self, x, y):
        # Support WDA `BREAKING CHANGES`
        # More see: https://github.com/appium/WebDriverAgent/blob/master/CHANGELOG.md#600-2024-01-31
        try:
            return self._session_http.post('/wda/tap', dict(x=x, y=y))
        except:
            return self._session_http.post('/wda/tap/0', dict(x=x, y=y))
```


发送消息
```
def text(self, text, enter=True):
        """Input text on the device.

        Args:
            text:  text to input
            enter: True if you need to enter a newline at the end

        Examples:
            >>> text("test")
            >>> text("中文")
        """
        if enter:
            text += '\n'
        self.driver.send_keys(text)
// wda里的内容

 def send_keys(self, value):
        """
        send keys, yet I know not, todo function
        """
        if isinstance(value, six.string_types):
            value = list(value)
        return self._session_http.post('/wda/keys', data={'value': value})
```