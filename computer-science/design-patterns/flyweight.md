# FlyWeight

[String](../../java/grammar/data-types.md#string) 和 [包装类](../../java/grammar/data-types.md#bao-zhuang-lei)使用了**享元模式**（蝇量模式）。

本质是一个**对象池**：创建之前看看对象池中是否已经存在对象，若存在，则直接返回对象池中的对象；若不存在，则创建一个新对象，并把这个对象放入对象池中。

