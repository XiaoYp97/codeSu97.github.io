---
title: "引用类型"
description:
date: "2024-03-24T21:51:52+08:00"
slug: "reference-type"
image: ""
license: false
hidden: false
comments: false
draft: true
tags: ["Java", "引用类型"]
categories: ["Java", "引用类型"]
# weight: 1 # You can add weight to some posts to override the default sorting (date descending)
---

在 Java 中，引用类型决定了对象如何被垃圾回收器（GC）处理。除了常见的强引用，Java 还提供了软引用、弱引用和幻象引用，它们分别适用于不同的内存敏感场景。

|引用类型| GC 回收条件 |是否阻止对象被回收 |典型用途|
| ---   | --- | --- | ---|
|强引用| 永远不会被 GC 自动回收 |是 |普通对象引用（默认行为）|
|软引用| 内存不足时会被回收 |否（除非内存不足） |缓存、内存敏感资源|
|弱引用| 下一次 GC 发生时强制回收 |否 |临时缓存、监听器列表|
|幻象引用| 对象被回收后触发通知 |否（且无法获取对象） |资源清理、对象回收跟踪|

## 引用类型

### 强引用（Strong Reference）

普通对象赋值（默认引用方式）。

```java
Object obj = new Object(); // 强引用
```

- 只要强引用存在，对象永远不会被 GC 回收。
- 若 `obj = null`，则对象变为**不可达**，GC 会回收。

**使用场景**：

- 绝大多数普通对象的创建和引用。

### 软引用（Soft Reference）

通过 `SoftReference` 类实现，**内存不足**时被回收。

```java
SoftReference<byte[]> softRef = new SoftReference<>(new byte[1024]);
byte[] data = softRef.get(); // 可能返回 null（如果已被回收）
```

- 内存充足时，对象不会被回收。
- 内存不足时（接近 `OutOfMemoryError`），GC 会回收软引用对象。
- 适合缓存需要保留但允许丢失的数据。

**使用场景**：

- 图片缓存、临时计算结果缓存。
- 避免因缓存导致内存溢出（如 Android 中的 `LruCache` 默认用强引用，可改用软引用）。

### 弱引用（Weak Reference）

通过 `WeakReference` 类实现，GC 运行时立即回收。

```java
WeakReference<Object> weakRef = new WeakReference<>(new Object());
Object obj = weakRef.get(); // GC 后可能为 null
```

- 无论内存是否充足，对象在 GC 时被回收。
- 适合存储短暂存在的辅助数据。

**使用场景**：

- `WeakHashMap` 的键（键对象无强引用时自动清理条目）。
- 监听器列表（防止因未注销监听器导致内存泄漏）。

### 幻象引用（Phantom Reference）

通过 `PhantomReference` 类实现，需配合 `ReferenceQueue` 使用。

```java
ReferenceQueue<Object> queue = new ReferenceQueue<>();
PhantomReference<Object> phantomRef = new PhantomReference<>(new Object(), queue);
```

- 无法通过 `get()` 获取对象（始终返回 `null`）。
- 对象被回收时，幻象引用会加入关联的引用队列。
- 用于在对象回收后执行**资源清理**操作（如关闭文件句柄）。

**使用场景**：

- 跟踪对象回收事件，确保释放堆外内存（如 `DirectByteBuffer` 的 `Cleaner` 机制）。
- 避免忘记清理资源（如数据库连接）。

## 引用队列（ReferenceQueue）

与软/弱/幻象引用配合，监听对象被回收的事件。

```java
ReferenceQueue<Object> queue = new ReferenceQueue<>();
WeakReference<Object> weakRef = new WeakReference<>(new Object(), queue);

// 轮询队列，当对象被回收时，队列会收到引用
Reference<?> ref = queue.remove(); // 阻塞直到有引用入队
```

## 典型场景对比

| **场景**              | **推荐引用类型**         | **原因**                          |
|----------------------|------------------------|-----------------------------------|
| 缓存图片              | 软引用                  | 内存不足时自动释放，避免 OOM。         |
| 临时监听器列表         | 弱引用                  | 监听器无用时自动清理，防止内存泄漏。     |
| 跟踪对象回收后的清理    | 幻象引用 + 引用队列       | 确保在对象销毁后执行资源释放。         |
| 常规对象持有           | 强引用                  | 默认行为，对象生命周期由代码显式控制。   |

## 注意事项

1. **强引用滥用**：
   大量未释放的强引用会导致内存泄漏（如静态集合缓存对象）。
2. **软引用与缓存策略**：
   软引用缓存可能被提前回收，需结合 LRU 算法优化。
3. **幻象引用的资源清理**：
   需通过 `ReferenceQueue` 轮询或线程处理，避免阻塞主逻辑。

## 代码示例

### 软引用缓存

```java
// 缓存大型数据
public class ImageCache {
    private final Map<String, SoftReference<Bitmap>> cache = new HashMap<>();

    public Bitmap getImage(String key) {
        SoftReference<Bitmap> ref = cache.get(key);
        return (ref != null) ? ref.get() : null;
    }

    public void putImage(String key, Bitmap bitmap) {
        cache.put(key, new SoftReference<>(bitmap));
    }
}
```

### 弱引用监听器

```java
// 自动清理无效监听器
public class EventManager {
    private final List<WeakReference<EventListener>> listeners = new ArrayList<>();

    public void addListener(EventListener listener) {
        listeners.add(new WeakReference<>(listener));
    }

    public void notifyListeners() {
        listeners.removeIf(ref -> ref.get() == null); // 清理已被回收的监听器
        listeners.forEach(ref -> ref.get().onEvent());
    }
}
```

### 幻象引用资源清理

```java
// 跟踪对象回收并释放资源
public class ResourceCleaner {
    public static void trackResource(Object resource, Runnable cleanupTask) {
        ReferenceQueue<Object> queue = new ReferenceQueue<>();
        PhantomReference<Object> ref = new PhantomReference<>(resource, queue);

        new Thread(() -> {
            try {
                queue.remove(); // 阻塞直到资源被回收
                cleanupTask.run(); // 执行清理操作
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }).start();
    }
}

// 使用示例：释放 DirectByteBuffer 的本地内存
ByteBuffer buffer = ByteBuffer.allocateDirect(1024);
ResourceCleaner.trackResource(buffer, () -> {
    ((DirectBuffer) buffer).cleaner().clean();
});
```

## 总结

- **强引用**：默认行为，对象生命周期由代码控制。
- **软引用**：内存敏感缓存，自动释放避免 OOM。
- **弱引用**：自动清理临时数据，防止内存泄漏。
- **幻象引用**：对象回收后的资源清理，确保安全释放。

合理使用这些引用类型，可以显著优化内存管理，避免常见的内存泄漏和溢出问题。
