# Button元素意外触发click事件

`button`, `input` 元素在`focus`以后, 按Space / Enter 键会自动触发元素的`click`事件

避免这种情况, 可以手动的对button元素执行`blur()`