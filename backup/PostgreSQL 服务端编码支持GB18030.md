# 编码介绍
目前熟知的编码方式如下：

- 基础ASCII 单字节 7位128个
- 扩展ASCII 单字节 8位256个
- GB2312-80  可变长编码（1-2字节） 6763个汉字 国标
- GBK 可变长编码（1-2字节）21003个汉字 商业非国标
- UTF Unicode标准
- UTF-16 可变长编码（2或4字节） 不兼容ASCII
- UTF-8 可变长编码（1-4字节 ASCII、希腊字母、汉字、其他平面符号）
- GB18030 
- GB18030-2000/2005
- [GB18030-2022](https://openstd.samr.gov.cn/bzgk/gb/newGbInfo?hcno=A1931A578FE14957104988029B0833D3) 可变长编码（1ASCII，2GBK，4字其他文字扩展节） CJK 统一汉字87875个

问题：GB18030完全兼容GBK吗？

尊重UTF-8是目前应用最广泛的编码，给出一个示例：

<!--StartFragment-->
  | 第1字节 | 第2字节 | 第3字节 | 第4字节
-- | -- | -- | -- | --
单字节示例 | 0xxxxxxx |   |   |  
双字节示例 | 110xxxxx | 10xxxxxx |   |  
三字节示例 | 1110xxxx | 10xxxxxx | 10xxxxxx |  
四字节示例 | 11110xxx | 10xxxxxx | 10xxxxxx | 10xxxxxx
<!--EndFragment-->

# Posgres中的[支持编码集](https://www.postgresql.org/docs/current/multibyte.html)
截至pg18，不支持服务端的GB18030编码
![Image](https://github.com/user-attachments/assets/9f177f06-df11-4c29-8244-0fcb64137c69)

# 支持GB18030思路

- [ ] 创建 GB18030 编码的数据库，可考虑实现多版本2005/2022
- [ ] GB18030作为服务器编码时，要能够在pg_conversion中和客户端其他的字符集编码进行转换
- [ ] 适配识别GB18030的字符串处理函数
- [ ] 适配GB18030字符集的排序、索引