# Prototype

如果对象的创建成本比较大，而同一个类的不同对象之间差别不大（大部分数据相同），我们可以利用已有对象（原型）进行复制来创建新对象，以节约创建时间。

{% hint style="info" %}
使用原型来复制新对象时，注意深拷贝和浅拷贝的区别。
{% endhint %}

对象创建成本比较大的场景：

* 对象中的数据需要经过复杂计算才能得到，比如排序、计算 hash 值
* 需要从 RPC、网络、数据库、文件系统等慢 IO 中读取。

