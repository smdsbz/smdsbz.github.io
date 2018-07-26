---
layout: article
title: WeChat Mini-App Dev Notes
tags: Web WeChatMiniapp
---

Random notes on WeChapp mini-app.

### HowTo

#### Line Input Field

`weui` actually changes the context of style-sheets, to make the distinguish
between different fields.  

Take note of the uses of `weui-cell_after-title` et. al.

```xml
<view class="page-section">
  <view class="page-section-title">Your name</view>
  <view class="weui-cells weui-cells_after-title">
    <view class="weui-cell weui-cell_input">
      <view class="weui-cell_bd">
        <input class="weui-input" name="name" placeholder="" />
      </view>
    </view>
  </view>
</view>
```

