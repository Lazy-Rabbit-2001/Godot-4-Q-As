# 小技巧（仅限Godot 4）
本篇会分享一些会在Godot中经常用到的一些小技巧，供大家参考

## 你不知道的帧循环
### 神奇的`SceneTree`信号——`process_frame`信号和`physics_frame`信号
一般情况下，我们的帧循环都是通过`_process()`和`_physics_process()`来实现的。然而，这些帧循环都有个特点——它们都只能在节点上运行。可如果我希望让一个不是节点的对象也可以使用帧循环该怎么办？  
好消息是，Godot已经在`SceneTree`类中提供了两个信号：`process_frame`和`physics_frame`，它们其实就分别对应了`_process()`和`_physics_process()`。如果我们能够获取到一个节点树的引用，那么我们就可以在非`Node`对象里也能实现帧循环：
```GDScript
extends RefCounted

var tree: SceneTree


func _process() -> void:
    pass # 自行实现

func _physics_process() -> void:
    pass # 自行实现


func tree_set(to: SceneTree) -> void:
    if !is_instance_valid(to): # 防止传入了对无效的SceneTree实例的引用
        return
    tree = to # 传引用给tree成员
    tree.process_frame.connect(_process) # 连接给_process()方法
    tree.physics_frame.connect(_physics_process) # 连接给_physics_process()方法
```
这样，只需要传入一个SceneTree实例的引用，就可以在非`Node`对象里使用帧循环了，但唯一的缺点就是无法获得delta参数，解决办法也比较简单——把传入的`SceneTree`实例换成`Node`即可，此时`tree`应改名为`node`，并将`tree.process_frame`改成`node.get_tree().process_frame`，对于`physics_frame`也如法炮制。对于这些方法里的delta参数，则可以按如下方式设置：
```GDScript
func _process() -> void:
    var delta := node.get_process_delta_time() # 如果是_physics_process，就把函数名中的process换成physics_process
```
### 具有条件限制的帧循环
不过，在Godot中，我们也可以对一个循环体通过协程等待一个帧循环信号的发出来将循环体转构造成一个帧循环体：
```GDScript
# 此处省略若干行代码
while <条件>:
    <循环体代码>
    await get_tree().process_frame # 这里相当于执行了_process()，如果想要执行_physics_process()，请换成physics_frame
```
有些情况下，我们通过这种循环结构可以省略一条变量声明，并且还能对该帧循环进行控制，从而在最大程度上节约内存消耗。
