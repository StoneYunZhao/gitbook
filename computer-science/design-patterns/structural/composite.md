# Composite

## 介绍

定义：将对象组合成树形结构来表示“整体/部分”的层次结构。组合能让客户以一致的方式处理个别对象及对象组合。GoF：Compose objects into tree structure to represent part-whole hierarchies. Composite lets client treat individual objects and compositions of object uniformly.

优点：清楚地定义分层次的复杂对象，表示对象的全部或者部分层次。

## 类图

* Component：组件，包括叶节点和组合两种。
* Leaf：叶节点。
* Composite：组合，持有一群孩子，孩子可以是组合也可以是叶节点。

![](../../../.gitbook/assets/image%20%28105%29.png)

## 源码

MyBatis 源码，`org.apache.ibatis.scripting.xmltags`：

```java
public interface SqlNode {
  boolean apply(DynamicContext context);
}

public class MixedSqlNode implements SqlNode {
  private List<SqlNode> contents;

  public MixedSqlNode(List<SqlNode> contents) {
    this.contents = contents;
  }

  @Override
  public boolean apply(DynamicContext context) {
    for (SqlNode sqlNode : contents) {
      sqlNode.apply(context);
    }
    return true;
  }
}

public class WhereSqlNode extends TrimSqlNode {
  private static List<String> prefixList = Arrays.asList("AND ","OR ","AND\n", "OR\n", "AND\r", "OR\r", "AND\t", "OR\t");

  public WhereSqlNode(Configuration configuration, SqlNode contents) {
    super(configuration, contents, "WHERE", prefixList, null, null);
  }
}
```

* SqlNode：对应 Component。
* WhereSqlNode：对应 Leaf。
* MixedSqlNode：对应 Composite。

![](../../../.gitbook/assets/image%20%28210%29.png)

