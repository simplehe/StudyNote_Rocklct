## ThreadLocal实践

有关ThreadLocal相关用法可见笔记，这里尝试自己动手实现一个ThreadLocal

### API总结
总结一下，ThreadLocal需要实现的API：

 - public void set(T value);
 - public T get();
 - public void remove();
 - protected T initialValue();

initalValue方法是protect，是为了提醒程序员这个方法需要被实现，需要给线程局部变量设置初值。

### 代码

``` java
public class MyThreadLocal<T> {
    private Map<Thread, T> container = Collections.synchronizedMap(new HashMap<Thread, T>());

    public void set(T value){
        container.put(Thread.currentThread(),value);
    }

    public T get() {
        Thread thread = Thread.currentThread();
        T value = container.get(thread);
        if(value == null && !container.containsKey(thread)){
            value = initiaValue();
            container.put(thread,value);
        }
        return value;
    }

    public void remove() {
        container.remove(Thread.currentThread());
    }


    protected T initiaValue() {
        return null;
    }
}
```
