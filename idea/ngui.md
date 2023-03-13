# ngui

# uicarmera

负责ui的消息处理

3dui 对应 boxcollider

2dui 对应 boxcollider2d

# particleSystem !!!!!

Unity 渲染与 NGUI 渲染 兼容性很差

在改变Partcle System 的 order in layout 时，只能实现overlap 在ngui 上，无法实现穿插效果
也就是说，这是不完全的sort

真正想要控制其代码顺序需要在脚本中重新改变renderedorder的值，或许本质上是unity的rendered只能通过代码控制的问题？

# UICamera 

UICamra在LateUpdate函数中检测屏幕尺寸变化，当屏幕尺寸发生变化时会广播UpdateAnchors消息给其下所有组件。在每一个UIWeidget都继承自UIRect，而UIRect中包含了UpdateAnchors函数用于接受UIRoot的消息

# UIAnchor

UIAnchor在Update中进行位置的调整，在取消RunOnlyOnce属性时只会运行一次，这里可以通过接受来自UIRoot的广播进行屏幕自适应

# NGUI 与 Shader

NGUI对Shader的支持并不好，这是由于NGUI在进行drawcall合并时会对需要渲染的物体的Shader进行一次拷贝，并在后面按照统一的渲染批次进行渲染。这种方式会忽略掉由其他代码注入的Shader参数，并且由于渲染延迟了，Game视图的时效性，Shader上属性的时效性也会受到影响。

# NGUI 控件大小单位

NGUI中的控件大小单位都是整型，但是存储类型使用的是float型，使用整型存储会使NGUI内部计算出来的UI排布更加规整，但是代价是牺牲了精度。。。

# SendMessage
SendMessage 最多支持传递一个参数，并且当存在无参的重载时会优先调用无参的重载忽略带参的重载
当具有多个一个参数重载时，只会选择对应参数那个，并且不会有隐式转换，因此不匹配即报错

无法发送消息给 false Active 的物体...

由于受到其性能影响在聚合度较高的地方慎用

# 单向接口

由于C#不支持多继承以及接口无法序列化的原因，在Unity中使用接口回调的形式无法序列化到inspector窗口上显示，只能通过sendmessage的形式调用指定接口，因此接口回调指支持单向通信。

# 垃圾ngui动画警告

当使用ngui的tweener动画时，由动画控制的变量将无法正常读取，这将会导致许多不可预知的bug，强烈建议不要使用