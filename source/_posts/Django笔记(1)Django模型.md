---
abbrlink: ''
categories:
- - Django 笔记 Python
date: '2024-11-10'
tags:
- Django
- 笔记
- Python
title: Django模型详解
toc: true
updated: '2024-11-14T20:22:22.170+08:00'
---
# 模型的字段

### 基础字段

**CharField**

需要使用max_length设置最大长度。

- 如不是必填项，可设置blank = True和default = ‘‘
- 如果想使其唯一，可以设置`unique = True`
- 如果有choice选项，可以设置 choices = XXX_CHOICES

**TextField**

适合大量文本，max_length = xxx选项可选

**DateField() 和DateTimeField() **

可通过default=xx选项设置默认日期和时间。

- 对于DateTimeField: default=timezone.now - 先要`from django.utils import timezone`
- 如果希望自动记录一次修改日期(modified)，可以设置: `auto_now=True`
- 如果希望自动记录创建日期(created),可以设置`auto_now_add=True`

**EmailField() **

如不是必填项，可设置blank = True和default = ‘。一般Email用于用户名应该是唯一的，建议设置unique = True

**IntegerField(), SlugField(), URLField()，BooleanField()**

可以设置blank = True or null = True。对于BooleanField一般建议设置`defaut = True or False`

**FileField(upload_to=None, max_length=100) - 文件字段 **

- upload_to = “/some folder/”：上传文件夹路径
- max_length = xxxx：文件最大长度

**ImageField (upload_to=None, max_length=100,)- 图片字段 **

- upload_to = “/some folder/”: 指定上传图片路径

### 关系字段

**OneToOneField(to, on_delete=xxx, options) - 单对单关系**

- to必需指向其他模型，比如 Book or ‘self’ .
- 必需指定`on_delete`选项(删除选项): i.e, “`on_delete = models.CASCADE`” or “`on_delete = models.SET_NULL`” .
- 可以设置 “`related_name = xxx`” 便于反向查询。

**ForeignKey(to, on_delete=xxx, options) - 单对多关系**

- to必需指向其他模型，比如 Book or ‘self’ .
- 必需指定`on_delete`选项(删除选项): i.e, “`on_delete = models.CASCADE`” or “`on_delete = models.SET_NULL`” .
- 可以设置”default = xxx” or “null = True” ;
- 如果有必要，可以设置 “`limit_choices_to =` “,
- 可以设置 “`related_name = xxx`” 便于反向查询。

**ManyToManyField(to, options) - 多对多关系**

- to 必需指向其他模型，比如 User or ‘self’ .
- 设置 “`symmetrical = False` “ 表示多对多关系不是对称的，比如A关注B不代表B关注A
- 设置 “`through = 'intermediary model'` “ 如果需要建立中间模型来搜集更多信息。
- 可以设置 “`related_name = xxx`” 便于反向查询。

示例：一个人加入多个组，一个组包含多个人，我们需要额外的中间模型记录加入日期和理由。

```python
from django.db import models

class Person(models.Model):
    name = models.CharField(max_length=128)

    def __str__(self):
        return self.name

class Group(models.Model):
    name = models.CharField(max_length=128)
    members = models.ManyToManyField(Person, through='Membership')

    def __str__(self):
        return self.name

class Membership(models.Model):
    person = models.ForeignKey(Person, on_delete=models.CASCADE)
    group = models.ForeignKey(Group, on_delete=models.CASCADE)
    date_joined = models.DateField()
    invite_reason = models.CharField(max_length=64)
```

对于`OneToOneField`和`ForeignKey`, `on_delete`选项和`related_name`是两个非常重要的设置，前者决定了了关联外键删除方式，后者决定了模型反向查询的名字。

### on_delete删除选项

Django提供了如下几种关联外键删除选项, 可以根据实际需求使用。

- `CASCADE`：级联删除。当你删除publisher记录时，与之关联的所有 book 都会被删除。
- `PROTECT`: 保护模式。如果有外键关联，就不允许删除，删除的时候会抛出ProtectedError错误，除非先把关联了外键的记录删除掉。例如想要删除publisher，那你要把所有关联了该publisher的book全部删除才可能删publisher。
- `SET_NULL`: 置空模式。删除的时候，外键字段会被设置为空。删除publisher后，book 记录里面的publisher_id 就置为null了。
- `SET_DEFAULT`: 置默认值，删除的时候，外键字段设置为默认值。
- `SET()`: 自定义一个值。
- `DO_NOTHING`：什么也不做。删除不报任何错，外键值依然保留，但是无法用这个外键去做查询。

### related_name选项

`related_name`用于设置模型反向查询的名字，非常有用。在文初的`Publisher`和`Book`模型里，我们可以通过`book.publisher`获取每本书的出版商信息，这是因为`Book`模型里有`publisher`这个字段。但是`Publisher`模型里并没有`book`这个字段，那么我们如何通过出版商反查其出版的所有书籍信息呢？

Django对于关联字段默认使用`模型名_set`进行反查，即通过`publisher.book_set.all`查询。但是`book_set`并不是一个很友好的名字，我们更希望通过`publisher.books`获取一个出版社已出版的所有书籍信息，这时我们就要修改我们的模型了，将`related_name`设为`books`, 如下所示：

```python
# models.py
from django.db import models
 
class Publisher(models.Model):
    name = models.CharField(max_length=30)
    address = models.CharField(max_length=60)
 
    def __str__(self):
        return self.name

# 将related_name设置为books
class Book(models.Model):
    name = models.CharField(max_length=30)
    description = models.TextField(blank=True, default='')
    publisher = ForeignKey(Publisher,on_delete=models.CASCADE, related_name='books')
    add_date = models.DateField(auto_now_add=True)
 
    def __str__(self):
        return self.name
```

1. 设置`related_name`前：`publisher.book_set.all`
2. 设置`related_name`后：`publisher.books.all`

## 模型的META选项

- `abstract=True`: 指定该模型为抽象模型
- `proxy=True`: 指定该模型为代理模型
- `verbose_name=xxx`和`verbose_name_plural=xxx`: 为模型设置便于人类阅读的别名
- `db_table= xxx`: 自定义数据表名
- `odering=['-pub-date']`: 自定义按哪个字段排序，`-`代表逆序
- `permissions=[]`: 为模型自定义权限
- `managed=False`: 默认为True，如果为False，Django不会为这个模型生成数据表
- `indexes=[]`: 为数据表设置索引，对于频繁查询的字段，建议设置索引
- `constraints=`: 给数据库中的数据表增加约束。
- 除此以外，我们经常自定义方法或Manager方法

### 示例一：自定义方法

~~~python
```
# 为每篇文章生成独一无二的url
def get_absolute_url(self):
    return reverse('blog:article_detail', args=[str(self.id)])

# 计数器
def viewed(self):
    self.views += 1
    self.save(update_fields=['views'])
```
~~~

### 示例二：自定义Manager方法

~~~python
```
# First, define the Manager subclass.
class DahlBookManager(models.Manager):
    def get_queryset(self):
        return super().get_queryset().filter(author='Roald Dahl')

# Then hook it into the Book model explicitly.
class Book(models.Model):
    title = models.CharField(max_length=100)
    author = models.CharField(max_length=50)

    objects = models.Manager() # The default manager.
    dahl_objects = DahlBookManager() # The Dahl-specific manager.
```
~~~
