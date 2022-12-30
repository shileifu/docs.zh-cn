# substring

## 功能

从一个字符串中抽取部分字符串返回。

### 语法

```Haskell
VARCHAR SUBSTRING(VARCHAR str, start, length)
```

## 参数说明

- str: 必填, 被截取的字符串。
- start: 必填，起始位置。字符串的第一个位置是 1。
- length: 必填，截取字符的数量，值必须是一个正数。

## 示例

```Plain Text

MySQL > select substring("starrockscluster", 1, 9);
+----------------------------+
| substring("starrockscluster", 1, 9) |
+----------------------------+
| starrocks                      |
+----------------------------+
```

## 关键字

substring,string,sub
