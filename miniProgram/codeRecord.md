# 一些实现
## 列表数据高亮输入框值

```wxml
<view wx:for="{{result}}" wx:key="BRDBRDNBR" class="resultItem" bindtap="complete" data-item="{{item}}">
    <text wx:for="{{item.bankName}}" wx:for-item="str" wx:for-index="k" wx:key="k"
          class="{{str === inpVal ? 'light' : ''}}">{{str}}</text>
</view>
```