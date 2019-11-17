# Remove Final by Reflection

## 背景

需要通过反射的方式修改某个类的`private static final`字段。

## 重现

```java
// java.lang.reflect.Field
public class FinalFieldModify {
    public static void main(String[] args) throws NoSuchFieldException, IllegalAccessException {
        System.out.println("Before modify: contains c = " + DataHolder.contains("c"));

        Field field = DataHolder.class.getDeclaredField("data");
        field.setAccessible(true);

        List<String> origin = (List<String>) field.get(null);

        final Field modifiersField = Field.class.getDeclaredField("modifiers");
        modifiersField.setAccessible(true);
        modifiersField.setInt(field, field.getModifiers() & ~Modifier.FINAL);

        // 由于 List.of 返回的是不可变 List，
        // 所以 只能新建一个 List，增加元素，再替换原来的 List
        ArrayList<String> cur = new ArrayList<>(origin.size() + 1);

        cur.addAll(origin);
        cur.add("c");

        field.set(null, cur);

        System.out.println("After modify: contains c = " + DataHolder.contains("c"));
    }

    private static class DataHolder {
        private static final List<String> data = List.of("a", "b");

        private static boolean contains(String s) {
            return data.contains(s);
        }
    }
}
```

运行上述代码后会抛出如下异常：**Can not set static final java.util.List field** 。

由于已经通过反射把字段的`private`和`final`两个修饰符都去掉了，所以很诡异。

## 解决方案

只需要把上面的第 8 行挪到第 13 行就行。即在**对字段的修饰符通过反射修改之前，不能通过反射获取该字段的值**。

```java
... 略
field.setAccessible(true);

final Field modifiersField = Field.class.getDeclaredField("modifiers");
modifiersField.setAccessible(true);
modifiersField.setInt(field, field.getModifiers() & ~Modifier.FINAL);

List<String> origin = (List<String>) field.get(null);

// 由于 List.of 返回的是不可变 List，
// 所以 只能新建一个 List，增加元素，再替换原来的 List
ArrayList<String> cur = new ArrayList<>(origin.size() + 1);
```

输出结果：

```text
Before modify: contains c = false
After modify: contains c = true
```

## 原理

通过上面的解决方案可以看出，关键问题在于通过反射获取字段的值这个操作。

{% code title="" %}
```java
public final class Field extends AccessibleObject implements Member 
{

    // Cached field accessor created with override
    private FieldAccessor overrideFieldAccessor;
    
    public Object get(Object obj) {
        return getFieldAccessor(obj).get(obj);
    }
    
    public void set(Object obj, Object value) {
        getFieldAccessor(obj).set(obj, value);
    }
    
    private FieldAccessor getFieldAccessor(Object obj) {
        FieldAccessor a = overrideFieldAccessor;
        return (a != null) ? a : acquireFieldAccessor(true);
    }
    
    private FieldAccessor acquireFieldAccessor(boolean overrideFinalCheck) {
        FieldAccessor tmp = reflectionFactory.newFieldAccessor(this, overrideFinalCheck);
        setFieldAccessor(tmp, overrideFinalCheck);
        return tmp;
    }
    
    private void setFieldAccessor(FieldAccessor accessor, boolean overrideFinalCheck) {
        this.overrideFieldAccessor = accessor;
    }
}
```
{% endcode %}

上面是`java.lang.reflect.Field`精简后的代码，可见 Field 类会在第一次调用 get 方法时会缓存 FieldAccessor，但是修改 Field 的修饰符不会清除缓存，所以还是用的老的 final 类型的 FieldAccessor。

**结论：通过反射修改 Field 的修饰符必须在通过反射访问（get、set） Field 之后。**

