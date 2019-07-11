## 类型

### Integer的缓存机制

* 通过内部静态类IntegerCache实现缓存
* 缓存本质上一个Integer数组
* 缓存的默认范围是[-128, 127]
* 可通过虚拟机参数设置缓存的上界

* static class IntegerCache

```

//定义缓存的边界
static final int low = -128;
static final int high;

//缓存，Integer数组
static final Integer cache[];

```