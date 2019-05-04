# Data Types

Java是一门强类型语言，意味着Java中的每个变量都申明了类型。Java中有8种基本数据类型和引用类型。

## 基本数据类型

| 类型 | 所占空间\(byte\) | 数据范围 | 默认值 | 包装类 |
| :---: | :---: | :---: | :---: | :---: |
| boolean | 1 | true和false | false | Boolean |
| byte | 1 | \[ - 2^7, 2^7 - 1 \] | 0 | Byte |
| char | 2 | Unicode \[0, 65535\] | u0000 | Character |
| short | 2 | \[ - 2^15, 2^15 - 1 \] | 0 | Short |
| int | 4 | \[ - 2^31, 2^31 - 1 \] | 0 | Integer |
| float | 4 | 32位 IEEE754 单精度 | 0.0f | Float |
| long | 8 | \[ - 2^63, 2^63 - 1 \] | 0L | Long |
| double | 8 | 64位 IEEE754 双精度 | 0.0 | Double |

* Java中的数值类型**都是有符号的**，不存在无符号的数值类型。
* Java中还存在一种基本类型void，有对应的包装类Void。

## 类型的自动转换

## 类型的强制转换

