# Maven

## 可选依赖于依赖排除

Optional 和 Exclusions 都是用来排除jar包依赖使用的。

Project-X 依赖 Project-A，Project-A 依赖 Project-B。

### Optional

```markup
<!-- Project-A 的 pom 文件 -->
<project>
  ...
  <dependencies>
    <dependency>
      <groupId>sample.ProjectB</groupId>
      <artifactId>Project-B</artifactId>
      <version>1.0</version>
      <scope>compile</scope>
      <optional>true</optional>
    </dependency>
  </dependencies>
</project>
```

如上 X 依赖 A，A 依赖 B 用的`<optional>true</optional>`，这时 B 只能在 A 中使用，而不会主动传递到 X 中，X 需要主动引用 B 才有 B 的依赖。

### Exclusions

```markup
<!-- Project-X 的 pom 文件 -->
<dependencies>
    <dependency>
      <groupId>sample.ProjectA</groupId>
      <artifactId>Project-A</artifactId>
      <version>1.0</version>
      <scope>compile</scope>
      <exclusions>
        <exclusion>
          <groupId>sample.ProjectB</groupId>
          <artifactId>Project-B</artifactId>
        </exclusion>
      </exclusions> 
    </dependency>
</dependencies>
```

如果 A 不用`<optional>true</optional>`引用 B，则会传递到 X 中，X 如果不需要 B 则需要主动排除 A 传递过来的 B。

* [http://maven.apache.org/guides/introduction/introduction-to-optional-and-excludes-dependencies.html](http://maven.apache.org/guides/introduction/introduction-to-optional-and-excludes-dependencies.html)
* [https://www.jianshu.com/p/6eee9191f2ab](https://www.jianshu.com/p/6eee9191f2ab)

