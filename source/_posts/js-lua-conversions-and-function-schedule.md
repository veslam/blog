---
title: 从js到lua的写法转变 & 定时调用的实现
date: 2018-05-16 13:54:31
tags: [Cocos2d, JavaScript, lua, schedule]
---
记录最初接触lua时趟的一些坑。

- 语法糖
```
. -> :
```
    > The : is syntax sugar in Lua which effectively implies the object on which the method is called being passed as first parameter.

- 基本语法替换
``` lua
var -> local
this -> self
!= -> ~=
! -> not
|| -> or
// -> --
```

- list转table
``` lua
someList.push(e) ->
table.insert(someTable, e)
```

- 数学
``` lua
parseInt() -> tonumber()

Math.floor(Math.random() * n) -> math.floor(math.random()* n)
Math.PI -> math.pi

cc.pDistance -> cc.pGetDistance
cc.pToAngle -> cc.pToAngleSelf
```

- 当前时刻
``` lua 
local currTime = cc.utils:gettime() -- 单位是秒
```

- update()
``` lua
this.scheduleUpdate()
->
local function _update(dt)
    self:update(dt)
end
self:scheduleUpdateWithPriorityLua(_update, 0)
```

---
- schedule/unschedule
``` lua
schedule(func, interval, cc.REPEAT_FOREVER)
scheduleOnce(func, delayTime)
unschedule(func)
-> 新接口，实现见后文。
function TimerCallFunc:addCallFunc(func, delay, param, group)
function TimerCallFunc:addScheduleFunc(func, interval, param, group)
function TimerCallFunc:clearGroup(group)
function TimerCallFunc:unscheduleFunc(func, group)
```

---
##### lua中的定时调用实现
extern.lua 提供了通过runAction实现的schedule(定期)和performWithDelay(单次)
``` lua
function schedule(node, callback, delay)
    local delay = cc.DelayTime:create(delay)
    local sequence = cc.Sequence:create(delay, cc.CallFunc:create(callback))
    local action = cc.RepeatForever:create(sequence)
    node:runAction(action)
    return action
end

function performWithDelay(node, callback, delay)
    local delay = cc.DelayTime:create(delay)
    local sequence = cc.Sequence:create(delay, cc.CallFunc:create(callback))
    node:runAction(sequence)
    return sequence
end
```

然而，通过Action的实现总让人有些不放心，比如stopAllActions()就会令其失效。有没有更纯粹的方法呢？有的，就是[scheduler:scheduleScriptFunc()](http://doc.zengrong.net/cocos/2.2.5/de/dee/classcocos2d_1_1_c_c_scheduler.html#af236237543cc64e1779b464bf8b4f97e)方法。

``` cpp
unsigned int scheduleScriptFunc (
    unsigned int    nHandler,
    float   fInterval,
    bool    bPaused 
)   
```
>The scheduled script callback will be called every 'interval' seconds.
If paused is YES, then it won't be called until it is resumed. If 'interval' is 0, it will be called every frame. return schedule script entry ID, used for unscheduleScriptFunc(). NA

``` lua
TimerCallFunc = class("TimerCallFunc")

function TimerCallFunc:ctor()
    self.maps = {}
    self._instance = nil
end

function TimerCallFunc:getInstance()
    if not self._instance then
        self._instance = TimerCallFunc.new()
    end
    return self._instance
end

--- 单次调用
function TimerCallFunc:addCallFunc(func, delay, param, group)
    self:_addFunc(func, delay, param, group, false)
end

--- 循环调用
function TimerCallFunc:addScheduleFunc(func, interval, param, group)
    self:_addFunc(func, interval, param, group, true)
end

function TimerCallFunc:_addFunc(func, dt, param, group, loop)
    group = group or cc.Director:getInstance():getRunningScene()
    if not group then return end

    self.maps[group] =  self.maps[group] or {}
    if func and not self.maps[group][func] then
        local function append(loop)
            if self.maps[group] then
                if (not loop) then self:unscheduleFunc(func, group) end
                func(param)
            end
        end
        local scheduler = cc.Director:getInstance():getScheduler()
        local entry = scheduler:scheduleScriptFunc(function()
            append(loop)
        end, dt, false)
        self.maps[group][func] = entry
    end
end

function TimerCallFunc:clearGroup(group)
    if self.maps[group] then
        local scheduler = cc.Director:getInstance():getScheduler()
        for k, v in pairs(self.maps[group]) do
            scheduler:unscheduleScriptEntry(v)
        end
    end

    self.maps[group] = nil
end

function TimerCallFunc:unscheduleFunc(func, group)
    group = group or cc.Director:getInstance():getRunningScene()
    if self.maps[group] and self.maps[group][func] then
        local scheduler = cc.Director:getInstance():getScheduler()
        scheduler:unscheduleScriptEntry(self.maps[group][func])
        self.maps[group][func] = nil
    end
end
```

``` lua
-- 用法
TimerCallFunc:getInstance():addCallFunc(self.someFunc, dt, self, self)
TimerCallFunc:getInstance():addCallFunc(function()
    -- body
end, dt, self, self)
```

