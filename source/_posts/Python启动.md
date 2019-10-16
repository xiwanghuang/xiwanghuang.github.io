---
title: Python启动
date: 2019-05-25 14:21:22
tags:
---
# 列表
在python中使用方括号来表示列表。
```python
bicycles = ['treck','cabbondale']
print(bicycles[0])
```
使用[-1]可以访问列表最后一个元素`print(bicycles[-1])`
## 列表的修改
`bicycles[0] = 'giant'`
## 添加元素
append()方法添加元素至列表末尾；  
insert()方法添加元素至指定位置。
`bicycles.insert(1,bike)`
## 删除元素
del()方法删除元素；
`del bicycles[0]`
pop()方法删除元素并返回元素值；
```python
last_ownd=bicycles.pop() //默认删除列表末尾的元素
bicycles.pop(0)          //删除指定位置的元素
```
remove()根据元素值删除元素(仅删除第一个匹配的元素)；
```python
bicycles.remove('treck')
```
## 组织列表
sort() 排序；
```pyhton
bicycles.sort()             //升序
bicycles.sort(reverse=true) //降序
```
reverse()反转列表元素排列
len()返回列表长度
## for循环遍历列表
使用for循环。python根据缩进来判断代码行与前一个代码行的关系。
```python
for bicycle in bicycles:
    print(bicycle)
```
## 使用range()创建数字列表
range(A,B)方法生成从A到B-1的值
函数list()将range()的结果直接转化为列表
```python
numbers = list(range(1,6))  //numbers为数字1-5的列表
```
range()方法还能够指定步长如下，步长为2
`range(1,10,2)`
创建列表
```python
squares = []
for value in range(1,11)
    square = value**2
    squares.append(square)
```
## 列表解析
```python
squares = [value**2 for value in range(1,11)]
```
## 使用列表的一部分——切片
```python
bicycles = ['treck','cabbondale','giant']
print(bicycles[0:1])
print(bicycles[0:])
print(bicycles[:1])
print(bicycles[-3:]) //最后三个
```
## 复制列表
```python
bicycles = ['treck','cabbondale','giant']
my_bikes = bicycles[:]
```
不可以直接写`my_bikes = bicycles`这样无法得到两个列表。
# 选择结构
```python
if (a == b) and (a != 0):
    a=0
elif b!= 0:
    b=0
else:
    a=b
```
判断特定的值是否包含在列表中可使用关键字in，反之则使用not in。
# 字典
一个简单的字典如下
```python
alien_0 = {'color': 'green','points': 5}
```
在Python中，字典是一系列的键值对，键值对之间用逗号分隔。
添加键-值对可以直接输入
```python
alien_0['x_position']=0
alien_0['y_position']=25
```
删除键值对
`del alien_0['points']`
## 字典的遍历
* 使用for循环遍历所有键值对,方法items()返回一个键值对列表
  ```pyhton
  for key,value in alien_0.items():
    print("\nKey:"+key)
  ```
* key()方法返回所有键的列表
  ```python
  for name in alien_0.keys():
    print(name.title())
  ```
* 对遍历的结果进行排序
  ```python
  for name in sorted(alien_0.keys()):
    print(name.title())
  ```
## 字典嵌套
```python
aliens=[alien_0,alien_1,alien_2]
```
# 用户输入和while循环
## 用户输入
* input()进行用户输入,该方法会将用户的输入解读为字符串
`message=input("请输入文本")`
* 使用int()方法可以获取数值输入
## while循环
```python
while message != 'quit':
    message = input()
    print(message)
```
break,continue的用法与c类似
# 函数
## 函数定义
```python
def abc(a)
    print("hello ,world"+str(a))
    return a+1
```