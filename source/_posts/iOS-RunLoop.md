---
title: iOS RunLoop
date: 2016-10-14 12:46:57
tags:
- iOS
- Code
---

# RunLoop
## `CFRunLoop` vs `NSRunLoop`
`CFRunLoop` 是 `NSRunLoop` 在 `CoreFoundation` 中的底层形式，即 `NSRunLoop` 是 `CFRunLoop` 的封装。<br>

`CFRunLoop` 有一个通用启动方法，`NSRunLoop` 通过指定该方法的 `timeout` 参数和 `stopAfterHandle` ，来实现
```obj-c
[[NSRunLoop currentRunLoop] run];
```
和
```obj-c
[[NSRunLoop currentRunLoop] runUntilDate:[NSDate date]];
```

## `source0` vs `source1`
我们平时在 `NSThread` 层手动创建线程的时候，会手动启动一个 `NSRunLoop`。代码形如
```obj-c
[[NSRunLoop currentRunLoop] addPort:[NSMachPort port] forMode:NSDefaultRunLoopMode];
while (running) {
	[[NSRunLoop currentRunLoop] run];
}
```
在代码中向 `NSRunLoop` 添加的 `[NSMachPort port]` 即是一个端口，通常用于通过 `Mach` 实现进程间通信。在此添加是为了不让 `runloop` 一直处于循环之中，添加 `port` 可以使 `runloop` 在没有事件需要处理的时候进入等待状态。<br>
`NSMachPort` 就属于一种 `source1` ，添加到 `runloop` 当中。其他进程可以通过其暴露的 `port` 向其通信，并且唤醒 `runloop` 开始处理事件。<br>
比如：

- 硬件方面
  - 各种传感器传来的事件
  - 用户点击屏幕的事件等

- 底层事件
  - 如网络请求等

`source1` 被处理时会触发添加 `source` 时声明的回调函数。<br>
比如用户触摸屏幕会由系统将 `source1` 标记为已准备，而 `app` 层面的 `UIEvent` 则是在该 `source1` 的回调函数中手动触发某 `source0`，然后唤醒 `runloop` 处理该 `source0`。因此，接收到 `UIEvent` 是从 `source0` 处被调用。

## `CFRunLoop` 流程

{% raw %}
<textarea id="flowchart_code" style="display: none;">
start=>start: CFRunLoop
end=>end: Exit
op1=>operation: Get CurrentMode
op2=>operation: doObserver: kCFRunLoopEntry
op3=>operation: Entry: currentMode
op4=>operation: doObserver: kCFRunLoopBeforeTimers
op5=>operation: doObserver: kCFRunLoopBeforeSources
op6=>operation: doBlocks: from dispatch (?)
op7=>operation: doSource0: 调用被CFRunLoopSourceSignal(source)标记的source0
op8=>operation: doBlocks
cond1=>condition: source1 ready?
op9=>operation: doObserver: kCFRunLoopBeforeWaiting
op10=>operation: mach_msg(msg, MACH_RCV_MSG, port)
cond2=>condition: WakeUp?
op11=>operation: doObserver: kCFRunLoopAfterWaiting
op12=>operation: handleMsg
op13=>operation: doTimers/do dispatch到main_queue的block/doSource1
op14=>operation: doBlocks
cond3=>condition: ModeIsEmpty?
op15=>operation: doObserver: kCFRunLoopExit

start->op1->op2->op3->op4->op5->op6->op7->op8
op8->cond1
cond1(yes)->op12
cond1(no, left)->op9->op10
op10->cond2
cond2(yes)->op11
op11(right)->op12->op13->op14->cond3
cond3(no, left)->op4
cond3(yes, right)->op15->end
</textarea>
<div id="flowchart_canvas"></div>
{% endraw %}

