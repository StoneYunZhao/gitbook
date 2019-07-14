# Balking

回顾上节的 Guarded Suspension 模式，本质是有一个状态变量，一个线程会判断这个状态变量是否已经满足要求，若没有满足，则等待直到满足再进行接下来的逻辑。

Balking 模式的本质有点相似，同样有一个状态变量，一个线程会判断这个状态变量是否已经满足要求，若没有满足，则直接返回；若满足，则执行接下来的逻辑。与 Guarded Suspension 模式的差别就在于状态变量没有满足要求时，是直接返回还是等待直到满足要求。

所以 Guarded Suspension 模式通常也被称为“多线程版本的 if”，只是这个 if 是需要等待的，而且很执着。Balking 也是多线程版本的 if，这个 if 不需要等待。

以下是一个自动存盘的逻辑，采用了 Balking 模式：

```java
public final class AutoSave {
    private boolean changed = false;

    private ScheduledExecutorService ses = Executors.newSingleThreadScheduledExecutor();

    private void startAutoSave() {
        ses.scheduleWithFixedDelay(this::autoSave, 5, 5, TimeUnit.SECONDS);
    }

    private void autoSave() {
        synchronized (this) {
            if (!changed) {
                return;
            }
            changed = false;
        }

        execSave();
    }

    private void execSave() {
        System.out.println("save");
    }

    public void edit() {
        System.out.println("edit");

        change();
    }

    private void change() {
        synchronized (this) {
            changed = true;
        }
    }
}
```

[单例模式的双重检测方式](../../../computer-science/design-patterns/singleton.md#xian-cheng-an-quan)本质也是采用了 Balking 模式。

