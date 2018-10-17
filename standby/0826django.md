django 笔记
1. django框架简介
2. django的MVT设计模式
3. django的工作流程
4. django项目实战



##  Python3基础

>Python3 的六个标准数据类型中：  
不可变数据（3 个）：Number（数字）、String（字符串）、Tuple（元组）；  
可变数据   （ 3 个）：Dictionary（字典）、List（列表） 、Set（集合）。  

## 1、Number(数字) 常用函数&方法：

`type(a)`  		检测a的数据类型  
`int(x) `   	 	将x转换为一个整数。  
`float(x) ` 		将x转换到一个浮点数。  
`complex(x)` 将x转换到一个复数，实数部分为 x，虚数部分为 0  

## 2、String(字符串) 常用函数&方法：

`max(str)、min(str)`   求最大最小值  
`str.capitalize() ` 首字母大写，非字母不作处理  
`str.isalnum() ` 检测字符串是否由字母和数字组成  
`str.isalpha() ` 检测字符串是否只由字母组成  
`str.rstrip() `	删除string 字符串末尾指定包含的字符串（无序、默认空格）  
`str.upper()` 	字母小写转换为大写  
`str.swapcase()` 	大小写互换  

## 3、Tuple(元组) 常用函数：

`len(tuple)` 	 计算元组中元素的个数  
`max(tuple)`	 返回元组中元素的最大值  
`min(tuple) `	最小值  
`tuple(list)` 	将列表转换为元组  

## 4、Dictionary(字典) 常用函数&方法：

`len(dict) `计算字典元素个数  
`str(dict) ` 以字符串形式输出字典  
`type(varlable）`返回输入的变量类型  
`dict.clear() `删除字典中所有的元素  
`dict.copy()` 返回一个字典的浅复制， 深拷贝键；不拷贝值，还是引用  
`dict.fromkeys(seq, value)` 创建一个新字典，以序列seq中元素做字典的键，value为字典所有键对应的初始值  
`key in dict `如果键在字典dict中返回True  
`dict1.update(dicts)` 把字典参数 dict2 的 key/value(键/值) 对更新到字典 dict1 里  
`dict.keys()` 以列表返回一个字典所有的键  
`dict.get(key, default=None)` 返回指定键的值，如果值不在字典中返回默认值  
`dict.setdefault(key, default=None)` 如果键不存在于字典中，将会添加键并将值设为默认值  
`dict.items() `以列表返回可遍历的(键, 值) 元组数组  
`dict.popitem()` 随机返回并删除字典中的一对键和值(默认删除末尾对）  

## 5、List(列表) 常用函数&方法：
`len(list)、max(list)、min(list)、list(seq)`	将元组转换为列表  
`list_a.append(b) `    # 拼接字符串到尾部，列表拼接出现情况：【1，2，【1，2】】  
`list_a.extend(list_b)  `    # 可以拼接列表  
`list.sort(reverse=True)`  # 排序、反转  
`list.pop()`      # 根据索引移除列表中的指定一个元素  
`list.clear() `# 清空列表  
`del list[i] `# 删除列表元素  
`list.reverse()` # 列表反向排序  
`list.insert(1,2) `# 在索引1的位置插入2  

## 6、Set(集合) 常用函数&方法：

`s.add( x )` 			将元素 x 添加到集合 s 中，如果元素已存在，则不进行任何操作  
`s.update( x )`		 添加元素，且参数可以是列表，元组，字典等  
`s.remove( x ) `		将元素 x 添加到集合 s 中移除，如果元素不存在，则会发生错误  
`s.discard( x )`	 	移除集合中的元素，且如果元素不存在，不会发生错误  
`s.pop() `			设置随机删除集合中的一个元素  
`len(s) `				计算集合 s 元素个数  
`s.clear() `			清空集合  
`x in s`	 			判断元素是否在集合中存在，存在返回 True，不存在返回 False  

## 7、条件判断
7.1-Python中if语句的一般形式如下所示：  
```py
if condition_1:
    statement_block_1
elif condition_2:
    statement_block_2
else:
    statement_block_3
```
## 8、循环语句

8.1-Python中while语句的一般形式：
```py
while 判断条件：
     语句
else:
     语句
```
8.2-Python for循环可以遍历任何序列的项目，如一个列表或者一个字符串。
for循环的一般格式如下：
```py
for <variable> in <sequence>:
    <statements>
else:
    <statements>
```
8.3-range()函数
如果你需要遍历数字序列，可以使用内置range()函数。它会生成数列
```py
>>>for i in range(5):
...     print(i)
...
```
8.4-Python pass是空语句，是为了保持程序结构的完整性。
pass 不做任何事情，一般用做占位语句，如下实例
实例
```py
>>>while True:
...     pass  # 等待键盘中断 (Ctrl+C)
```
