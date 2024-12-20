# JUC

附: [hm_juc(Done)](https://www.yuque.com/mo_ming/gl7b70/gw2xt5)

## 一、Java线程

### 1.1 创建和运行线程

方法一，直接使用 `Thread`

```java
// 创建线程对象
Thread t = new Thread() {
    public void run() {
        // 要执行的任务
    }
};
// 启动线程
t.start();
```

方法二，使用 `Runnable ` 配合 `Thread` 