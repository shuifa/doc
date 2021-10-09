# 字符串

PHP字符串以 `ZEND_STRING` 为载体，实现了字符串的构建、扩充、截断、判等、复制、取长度、释放等操作。

## 字符串的结构

```c
    struct _zend_string {
        zend_refcounted_h   gc;     /* 8 字节，内嵌的gc，引用计数及字符串类别存储 */
        zend_ulong          h;      /* 哈希值，8字节，字符串的哈希值 */
        size_t              len;    /* 8字节，字符串的长度 */
        char                val[1]; /* 柔性数组，占1字节，字符串的值存储位置 */
    }
```
```c
    
```

