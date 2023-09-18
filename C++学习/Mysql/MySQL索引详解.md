# 1、什么是 MySQL 索引？

- 官方上面说索引是帮助MySQL`高效获取数据`的`数据结构`，通俗点的说，数据库索引好比是一本书的目录，可以直接根据页码找到对应的内容，目的就是为了`加快数据库的查询速度`。
- 索引是对数据库表中一列或多列的值进行排序的一种结构，使用索引可快速访问数据库表中的特定信息。
- 一种能帮助mysql提高了查询效率的数据结构：**索引数据结构**。

## 1.1、索引原理

索引的存储原理大致可以概括为一句话：**以空间换时间**。

一般来说索引本身也很大，不可能全部存储在内存中，因此`索引往往是存储在磁盘上的文件中的`（可能存储在单独的索引文件中，也可能和数据一起存储在数据文件中）。

数据库在未添加索引进行查询的时候默认是进行全文搜索，也就是说有多少数据就进行多少次查询，然后找到相应的数据就把它们放到结果集中，直到全文扫描完毕。

## 1.2、索引的分类

**主键索引：`primary key`**

- 设定为主键后，数据库自动建立索引，InnoDB为聚簇索引，主键索引列值不能为空（Null）。

**唯一索引：**

- 索引列的值必须唯一，但允许有空值（Null），但只允许有一个空值（Null）。

**复合索引：**

- 一个索引可以包含多个列，多个列共同构成一个复合索引。

**全文索引：**

- Full Text（MySQL5.7之前，只有MYISAM存储引擎引擎支持全文索引）。
- 全文索引类型为FULLTEXT，在定义索引的列上支持值的全文查找允许在这些索引列中插入重复值和空值。全文索引可以在**Char、VarChar** 上创建。

**空间索引：**

- MySQL在5.7之后的版本支持了空间索引，而且支持OpenGIS几何数据模型，MySQL在空间索引这方年遵循OpenGIS几何数据模型规则。

**前缀索引：**

- 在文本类型为char、varchar、text类列上创建索引时，可以指定索引列的长度，但是数值类型不能指定。

## 1.3、索引的优缺点

**优点：**

- 大大提高数据查询速度。
- 可以提高数据检索的效率，降低数据库的IO成本，类似于书的目录。
- 通过索引列对数据进行排序，降低数据的排序成本降低了CPU的消耗。
- 被索引的列会自动进行排序，包括【单例索引】和【组合索引】，只是组合索引的排序需要复杂一些。
- 如果按照索引列的顺序进行排序，对order 不用语句来说，效率就会提高很多。

**缺点：**

- 索引会占据磁盘空间。
- 索引虽然会提高查询效率，但是会降低更新表的效率。比如每次对表进行增删改查操作，MySQL不仅要保存数据，还有保存或者更新对应的索引文件。
- 维护索引需要消耗数据库资源。

**综合索引的优缺点：**

- `数据库表中不是索引越多越好，而是仅为那些常用的搜索字段建立索引效果最佳!`

## 1.4、创建索引的基本操作

**创建主键索引：**

```
#建表时，主键默认为索引
create table user(
    id varchar(11) primary key,
    name varchar(20),
    age int
)

#查看user表中的索引
show index from user;
```

![在这里插入图片描述](https://ucc.alicdn.com/images/user-upload-01/8d3b5da08f7e4ec1af87ee3eb06e9763.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiA5a6_5ZCb,size_20,color_FFFFFF,t_70,g_se,x_16 "在这里插入图片描述")

**创建单列索引：**

```
#创建单列索引，只能包含一个字段
create index name_index on user(name);
```

![**在这里插入图片描述**](https://ucc.alicdn.com/images/user-upload-01/f622c18a4f214155852be61ee0397b74.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiA5a6_5ZCb,size_20,color_FFFFFF,t_70,g_se,x_16 "**在这里插入图片描述**")

**创建唯一索引：**

```
#创建唯一索引，只能有一个列
create unique index age_index on user(age);
```

![在这里插入图片描述](https://ucc.alicdn.com/images/user-upload-01/892db10dd3ae48c4a26aecb446b446ac.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiA5a6_5ZCb,size_20,color_FFFFFF,t_70,g_se,x_16 "在这里插入图片描述")

**创建符合索引：**

```
#复合索引
create index name_age_index on user(name,age);
```

![在这里插入图片描述](https://ucc.alicdn.com/images/user-upload-01/cc6ac014998b4c63967c2bb78433435e.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiA5a6_5ZCb,size_20,color_FFFFFF,t_70,g_se,x_16 "在这里插入图片描述")

**满足复合索引的查询的两大原则：**  
**假如创建的复合索引为三个字段，按顺序分别是(name,age,sex)  
在查询时能利用复合索引的查询条件如下：**

- 1、最左前缀原则(如下四种都满足条件)
    
    ```
    select * from user where name = ? 
    select * from user where name = ? and age = ?
    select * from user where name = ? and sex = ?
    select * from user where name = ? and age = ? and sex = ?
    ```
    
- 1.1、如下是不满足最前左缀的条件（但是不是全部都不生效，如下第2原则解释）
    
    ```
    select * from user where name = ? and sex = ? and age = ?
    select * from user where age = ? and sex = ? and name = ?
    select * from user where sex = ? and age = ? and name = ?
    select * from user where age = ? and sex = ?
    …………等等
    ```
    
- 2.MySQL 引擎在执行查询时，为了更好地利用索引，在查询过程中会动态调整查询字段的顺序！**(也就是说，当条件中的字段全部达到复合索引中的字段时，可以动态调整字段顺序，使其满足最前左缀)**
    
    ```
    #可以使用复合索引：索引中包含的字段数都有，只是顺序不正确，在执行的时候可以动态调整为最前左缀
    select * from user where sex = ? and age = ? and name = ?
    select * from user where age = ? and sex = ? and name = ?
    
    #不可以使用复合索引：因为缺少字段，并且顺序不正确
    select * from user where sex = ? and age = ? 
    select * from user where age = ? and name = ?
    select * from user where age = ?
    select * from user where sex = ? 
    ```
# 2、索引的数据结构

> 本文借鉴丙哥文章：[一文搞懂MySQL索引所有知识点](https://blog.csdn.net/qq_35190492/article/details/109257302?spm=a2c6h.12873639.article-detail.7.3c4f7ae0SxjGPT&ops_request_misc=%257B%2522request%255Fid%2522%253A%2522163667559716780264065137%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=163667559716780264065137&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_positive~default-1-109257302.first_rank_v2_pc_rank_v29&utm_term=%E7%B4%A2%E5%BC%95)

MySQL索引使用的数据结构主要有`BTree索引`和`hash索引`。

对于hash索引来说，底层的数据结构就是哈希表，因此在绝大多数需求为单条记录查询的时候，可以选择哈希索引，查询性能最快；其余大部分场景建议选择BTree索引。

## 2.1、根据存储引擎的不同，实现方式也不同

MySQL的索引数据结构`最常使用的是B树中的B+Tree`，但对于主要的两种存储引擎的实现方式是不同的。

> ==**InnoDB中data阈存储的是行数据，而MyISAM中存储的是磁盘地址。**==

- **MyISAM**：
    
    > B+Tree叶节点的data域存放的是数据记录的地址。在索引检索的时候，首先按照B+Tree搜索算法搜索索引，如果指定的Key存在，则根据data域中磁盘地址到磁盘中寻址定位到对应的磁盘块，然后读取相应的数据记录，这被称为“非聚簇索引”。
    
- **InnoDB**：
    
    > ==其数据文件本身就是索引文件==。相比MyISAM，索引文件和数据文件是分离的，其表数据文件本身就是按照B+Tree组织的一个索引结构，树的叶节点data域保存了完整的数据记录。这个索引的Key是数据表的主键，因此InnoDB表数据文件本身就是主索引。这被称为“**聚簇索引（聚集索引）**”。而其余的索引都作为辅助索引，辅助索引的data域存储相应记录主键的值而不是地址，这也是和MyISAM不同的地方。
    
    - 在根据主索引搜索时，直接找到Key所在的节点即可取出数据；
    - 在根据辅助索引查找时，则需要先取出主键的值，再走一遍主索引。
    - 因此在设计表的时候，不建议使用过长的字段作为主键，也不建议使用非单调的字段作为主键，这样会造成主索引频繁分裂。

## 2.2、Hash表

Hash表，在Java中的HashMap，TreeMap就是Hash表结构，以键值对的形式存储数据。我们使用hash表存储表数据结构，**Key可以存储索引列，Value可以存储行记录或者行磁盘地址**。Hash表在等值查询时效率很高，时间复杂度为O(1)；但是不支持范围快速查找，范围查找时只能通过扫描全表的方式，筛选出符合条件的数据。

**显然这种方式，不适合我们经常需要查找和范围查找的数据库索引使用。**

## 2.3、二叉树

![在这里插入图片描述](https://ucc.alicdn.com/images/user-upload-01/9cfe610e7fbd4e66a0afaa2607a61971.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiA5a6_5ZCb,size_20,color_FFFFFF,t_70,g_se,x_16 "在这里插入图片描述")  
上面这个图就是我们常说的二叉树：**每个节点最多有两个分叉节点，左子树和右子树数据按顺序左小右大。**

`二叉树的特点就是为了保证每次查找都可以进行折半查找，从而减少IO次数。`

`但是二叉树不是一直保持二叉平衡，因为二叉树很考验根节点的取值，因为很容易在某个节点下不分叉了，这样的话二叉树就不平衡了，也就没有了所谓的能进行折半查找了，如下图：`

![在这里插入图片描述](https://ucc.alicdn.com/images/user-upload-01/efacc9b39f3f411fb6b968305faeabef.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiA5a6_5ZCb,size_20,color_FFFFFF,t_70,g_se,x_16 "在这里插入图片描述")  
**显然这种不稳定的情况，我们在选择存储数据结构的时候就会尽量避免这种的情况发生。**

## 2.4、平衡二叉树

平衡二叉树采用的是`二分法思维`，平衡二叉查找树除了具备二叉树的特点，最主要的特征是树的左右两个子树的层级最多差1。在插入删除数据时通过`左旋/右旋`操作保持二叉树的平衡，不会出现左子树很高、右子树很矮的情况。

使用平衡二叉查找树查询的性能接近与二分查找，时间复杂度为O(log2n)，查询id=6，只需要两次IO。  
![在这里插入图片描述](https://ucc.alicdn.com/images/user-upload-01/9cfe610e7fbd4e66a0afaa2607a61971.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiA5a6_5ZCb,size_20,color_FFFFFF,t_70,g_se,x_16 "在这里插入图片描述")  
就上述平衡二叉树的特点来看，其实是我们理想的状态下，然而其实内部还是存在一些问题：

- 时间复杂度和树的高度有关。树有多高就需要检索多少次，每个节点的读取，都对应一次磁盘的IO操作。树的高度就等于每次查询数据时磁盘IO操作的次数。磁盘每次寻道的时间为10ms，在数据量大时，查询性能会很差。（`1百万的数据量，log2n约等于20次磁盘IO读写，时间消耗约等于：20*10=0.2S`）。
- 平衡二叉树不支持范围查询快速查找，范围查询需要从根节点多次遍历，查询效率不高。

## 2.5、B树：改造二叉树

MySQL的数据是存储在磁盘文件中的，查询处理数据时，需要先把磁盘中的数据加载到内存中，磁盘IO操作非常耗时，`所以我们优化的重点就是尽量减少磁盘的IO操作`。访问二叉树的每个节点都会发生一次IO，`如果想要减少磁盘IO操作，就需要尽量降低树的高度`。

**那如何降低树的高度呢？**

假如key为bigint=8字节，每个节点有两个指针，每个指针为4个字节，一个节点占用的空间为（8+4*2=16）。

因为在MySQL的InnoDB引擎的一次IO操作会读取一页的数据量（默认一页大小为16K），而二叉树一次IO操作的有效数据量只有16字节，空间利用率极低。为了最大化的利用一次IO操作空间，一个解决方法就是在一个节点处存储多个元素，在每个节点尽可能多的存储数据。每个节点可以存储1000个索引（16k/16=1000），这样就将二叉树改造成了多叉树，`通过增加树的分叉树，将树的体型从高瘦变成了矮胖`。构建1百万条数据，树的高度需要2层就可以（1000*1000=1百万），也就是说只需要两次磁盘IO操作就可以查询到数据，磁盘IO操作次数变少了，查询数据的效率整体也就提高了。

这种数据结构我们称之为B树，==B树是一种多叉平衡查找树==，如下图主要特点：

1. B树的节点中存储这多个元素，每个内节点有多个分叉。
2. 节点中的元素包含键值和数据，节点中的键值从大到小排列。也就是说，在所有的节点中都存储数据。
3. 父节点当中的元素不会出现在子节点中。
4. 所有的叶子节点都位于同一层，叶子节点具有相同的深度，叶子节点之间没有指针连接。

**B树数据结构大致如下：**  
![在这里插入图片描述](https://ucc.alicdn.com/images/user-upload-01/f1dbeadce2c14cef9210a119d6f195f5.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiA5a6_5ZCb,size_20,color_FFFFFF,t_70,g_se,x_16 "在这里插入图片描述")  
举个简单的例子，在B树中查询数据的情况：

> 假如我们要查询key等于10对应的数据data，根据上图我们可知在磁盘中的查询路径是：**磁盘块1->磁盘块2->磁盘块6**

- 第一次磁盘IO：将磁盘块1加载到内存中，在内存中从头遍历比较，10<15，走左子树，到磁盘中寻址到磁盘块2。
- 第二次磁盘IO：将磁盘块2加载到内存中，在内存中从头遍历比较，10>7，走右子树，到磁盘中寻址到磁盘块6。
- 第三次磁盘IO：讲磁盘块6加载到内存中，在内存中从头遍历比较，10=10，找到key=10的位置，取出对应的数据data，如果data存储的是行记录，直接取出数据，查询结束；如果data存储的是行磁盘地址，还需要根据磁盘地址到对应的磁盘中取出数据，查询结束。

> 相比较二叉平衡查找树，在整个查找过程中，虽然数据的比较次数并没有明显减少，但是对于磁盘IO的次数会大大减少，同时，由于我们是在内存中进行的数据比较，所以比较数据所消耗的时间可以忽略不计。B树的高度一般2至3层就能满足大部分的应用场景，所以使用B树构建索引可以很好的提升查询的效率。

过程如下图所示：  
![在这里插入图片描述](https://ucc.alicdn.com/images/user-upload-01/df8512d3bfde44b99c17b9c0d90b1252.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiA5a6_5ZCb,size_20,color_FFFFFF,t_70,g_se,x_16 "在这里插入图片描述")  
**看到上面的情况，觉得B树已经很理想了，但是其中还是存在可以优化的地方：**

- B树不支持范围查询的快速查找，例如：仍然根据上图，我们想要查询10到35之间的数据，查找到10之后，需要回到根节点重新遍历查找，需要从根节点进行多次遍历，查询效率有待提高。
- 如果data存储的是行记录，行的大小随着列数的增加，所占空间会变大，这时一页中可存储的数据量就会减少，树相应就会变高，磁盘IO次数就会随之增加，有待优化。

## 2.6、B+树：改造B树

B+树，作为B树的升级版，MySQL在B树的基础上继续进行改造，使用B+树构建索引。B+树和B树最主要的区别在于==非叶子节点是否存储数据==的问题。

- B树：叶子节点和非叶子节点都会存储数据。
- B+树：只有叶子节点才会存储数据，非叶子节点只存储键值key；叶子节点之间使用双向指针连接，最底层的叶子节点形成了一个双向有序链表。

**B+树的大致数据结构：**  
![在这里插入图片描述](https://ucc.alicdn.com/images/user-upload-01/e85b73936ca14d75b3652cf256cbfa17.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiA5a6_5ZCb,size_20,color_FFFFFF,t_70,g_se,x_16 "在这里插入图片描述")

> B+树的最底层叶子节点包含了所有的索引项。从上图可以看出，B+树在查找数据的时候，由于数据都存放在最底层的叶子节点上，所以每次查找都需要检索到叶子节点才能查询到数据。所以在查询数据的情况下每次的磁盘IO次数跟树的高度有直接的关系；但是从另一方面来说，由于数据都被存放到了叶子节点，所以存放索引的磁盘块，所存放的的索引数量会随之增加，所以相对于B树来说，B+树的树高理论情况下是比B树树高要矮的。  
> **但是也存在索引覆盖查询的情况，在索引中数据满足了查询语句所需要的全部数据，此时只需要找到索引即可立刻返回，不需要检索到最底层的叶子节点。**

**等值查询实例：**

假如我们要查询key为9对应的数据data，查询路径为：**磁盘块1->磁盘块2->磁盘块6**。

- 第一次磁盘IO：将磁盘块1加载到内存中，在内存中从头遍历比较，9<15，走左子树，到磁盘寻址定位到磁盘块2。
- 第二次磁盘IO：将磁盘块2加载到内存中，在内存中从头遍历比较，7<9<12，到磁盘中寻址定位到磁盘块6。
- 第三次磁盘IO：将磁盘块6加载到内存中，在内存中从头遍历比较，在第三个索引中找到9，取出对应的数据data，如果data存储的是行记录，直接取出data，查询结束；如果存储的是磁盘地址，还需要根据磁盘地址再次寻址定位到指定磁盘取出数据，查询终止。

![在这里插入图片描述](https://ucc.alicdn.com/images/user-upload-01/0cf6f390478e4f83a72c82325c329926.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiA5a6_5ZCb,size_20,color_FFFFFF,t_70,g_se,x_16 "在这里插入图片描述")  
**范围查询实例：**

假如我们想要查找9和26之间的数据，查找路径为：**磁盘块1->磁盘块2->磁盘块6->磁盘块7**

- 前三次磁盘IO：首先查找到键值为9对应的数据（定位到磁盘块6），然后缓存大结果集中。这一步和前面等值查询流程一样，发生了三次磁盘IO。
- 继续查询，查找到节点15之后，底层的所有叶子节点是一个有序列表，我们从磁盘块6中的键值9开始向后遍历筛选出所有符合条件的数据。
- 第四次磁盘IO：根据磁盘块6的后继指针到磁盘中寻址定位到磁盘块7，将磁盘块7加载到内存中，在内存中从头遍历比较，9<25<26，9<26<=26，将数据data缓存到结果集中。
- 逐渐具备唯一性（后面不会再有<=26的数据），不需要再向后查找，查询结束，将结果集返回给用户。

![在这里插入图片描述](https://ucc.alicdn.com/images/user-upload-01/d7e3168feae043cca47600b95b901c41.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiA5a6_5ZCb,size_20,color_FFFFFF,t_70,g_se,x_16 "在这里插入图片描述")

> ==**由上述实例可知：B+树可以保证等值和范围查询的快速查找，MySQL 的索引采用的就是 B+树的结构。**==

# 3、MySQL 的索引实现

暂时分析MySQL的两种存储引擎的索引实现：==**MyISAM引擎和InnoDB引擎**==。

## 3.1、MyISAM索存储引擎索引

以一个简单的user表为例，user表存在两个索引，id列为主键索引，age列为普通索引。

```
CREATE TABLE `user`
(
  `id`       int(11) NOT NULL AUTO_INCREMENT,
  `username` varchar(20) DEFAULT NULL,
  `age`      int(11)     DEFAULT NULL,
  PRIMARY KEY (`id`) USING BTREE,
  KEY `idx_age` (`age`) USING BTREE
) ENGINE = MyISAM
  AUTO_INCREMENT = 1
  DEFAULT CHARSET = utf8;
```

MyISAM的数据文件和索引文件是分开存储的，MyISAM使用B+树构建索引树时，叶子节点中键值key存储的是索引列的值，数据data存储的是索引所在行的磁盘地址。

**主键ID列索引：**  
![在这里插入图片描述](https://ucc.alicdn.com/images/user-upload-01/e91e5091172346568d93c9bc3814e5f9.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiA5a6_5ZCb,size_20,color_FFFFFF,t_70,g_se,x_16 "在这里插入图片描述")  
表user的索引存储在索引文件`user.MYI`中，数据存储在数据文件`user.MYD`中。

### 3.1.1、根据主键等值查询数据

```
select * from user where id = 28
```

1. 第一次磁盘IO：先在主键索引树中从根节点开始检索，将根节点加载到内存中，比较28<75，所以走左子树。
2. 第二次磁盘IO：将左子树节点加载到内存中，比较16<28<47，向下检索。
3. 第三次磁盘IO：检索到叶子节点，将节点加载到内存中遍历，从16<28，18<28，28=28，查找到键值等于28的索引项。
4. 第四次磁盘IO：从索引项中获取磁盘地址，然后到数据文件`user.MYD`中获取对应整行记录。
5. 将记录返回给客户端。

**磁盘IO次数：3次索引检索+1次记录数据检索：**  
![在这里插入图片描述](https://ucc.alicdn.com/images/user-upload-01/7c1104d6f12845478fbb7e8cdf9de622.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiA5a6_5ZCb,size_20,color_FFFFFF,t_70,g_se,x_16 "在这里插入图片描述")

### 3.1.1、根据主键范围查询数据

```
select * from user where id between 28 and 47;
```

1. 第一次磁盘IO：先在主键索引树中从根节点开始检索，将根节点加载到内存中，比较28<75，所以走左子树。
2. 第二次磁盘IO：将左子树节点加载到内存中，比较16<28<47，向下检索。
3. 第三次磁盘IO：检索到叶子节点，将节点加载到内存中遍历，从16<28，18<28，28=28，查找到键值等于28的索引项，根据次磁盘地址从数据文件中获取行记录缓存到结果集中。
4. 第四次磁盘IO：我们在查询时时根据范围查找，将下一个节点加载到内存中，遍历比较，28<47=47，根据磁盘地址从数据文件中获取地址缓存到结果集中。
5. 根据上图可知，最后得到两条符合筛选条件的结果集，将结果返回给客户端。

**磁盘IO次数：4次索引检索+1次记录数据检索。**  
![在这里插入图片描述](https://ucc.alicdn.com/images/user-upload-01/6e6567bdd8e34a77b5d4d246b9a6c054.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiA5a6_5ZCb,size_20,color_FFFFFF,t_70,g_se,x_16 "在这里插入图片描述")

- 提示：
    
    > 以上分析仅供参考，如果使用MyISAM存储引擎在查询时，会将索引节点缓存在MySQL缓存中，**而数据缓存依赖于操作系统自身的缓存，** 这里只是为了演示分析索引的使用过程。
    

### 3.1.3、辅助索引

- 在MyISAM存储引擎中，辅助索引和主键索引的结构是一样的，没有任何区别，叶子节点中data阈存储的都是行记录的磁盘地址。
- 主键列索引的键值是唯一的，而辅助索引的键值是可以重复的。
- 查询数据时，由于辅助索引的键值不唯一，可能存在多个拥有相同的记录，所以即使是等值查询，也需要按照范围查询的方式在辅助索引树种检索数据。

## 3.2、InnoDB存储引擎索引

### 3.2.1、主键索引（聚簇索引）

每个InnoDB表都有一个聚簇索引，聚簇索引使用B+树构建，叶子节点的data阈存储的是整行记录。一般情况下，聚簇索引等同于主键索引，当一个表没有创建主键索引时，InnoDB会自动创建一个ROWID字段来构建聚簇索引。InnoDB创建索引的具体规则如下：

- 在创建表时，定义主键PRIMARY KEY，InnoDB会自动将主键索引用作聚簇索引。
- 如果表没有定义主键，InnoDB会选择第一个不为NULL的唯一索引列用作聚簇索引。
- 如果以上两个都没有，InnoDB会自动使用一个长度为6字节的ROWID字段来构建聚簇索引，该ROWID字段会在插入新的行记录时自动递增。

除聚簇索引之外的所有索引都被称为辅助索引。在InnoDB中，辅助索引中的叶子节点键值存储的是该行的主键值。在检索时，InnoDB使用此主键在聚餐索引中搜索行记录。

这里以user_innodb表为例，user_innodb的id列为主键，age列为普通索引。

```
CREATE TABLE `user_innodb`
(
  `id`       int(11) NOT NULL AUTO_INCREMENT,
  `username` varchar(20) DEFAULT NULL,
  `age`      int(11)     DEFAULT NULL,
  PRIMARY KEY (`id`) USING BTREE,
  KEY `idx_age` (`age`) USING BTREE
) ENGINE = InnoDB;
```

- InnoDB的数据和索引存储在`t_user_innodb.ibd`文件中，InnoDB的数据组织方式，是聚簇索引。
- 主键索引的叶子节点会存储数据行，辅助索引的叶子节点只会存储主键值。

![在这里插入图片描述](https://ucc.alicdn.com/images/user-upload-01/29a27130b5a843b59c5c560cd5c731b2.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiA5a6_5ZCb,size_20,color_FFFFFF,t_70,g_se,x_16 "在这里插入图片描述")  
**等值查询数据：**

```
select * from user_innodb where id = 28;
```

**发生三次磁盘IO：**  
![在这里插入图片描述](https://ucc.alicdn.com/images/user-upload-01/43285bda9e754299bc0a614282bdd4a6.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiA5a6_5ZCb,size_20,color_FFFFFF,t_70,g_se,x_16 "在这里插入图片描述")

### 3.2.2、辅助索引

- 除聚簇索引之外的所有索引都被称为辅助索引，InnoDB的辅助索引只会存储值而不存储磁盘地址。
- 以user_innodb的age列为例，age列的辅助索引结构如下图：

![在这里插入图片描述](https://ucc.alicdn.com/images/user-upload-01/23bdd860117243ff909bcff49b6b1e09.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiA5a6_5ZCb,size_20,color_FFFFFF,t_70,g_se,x_16 "在这里插入图片描述")

- 辅助索引的底层叶子节点是按照（age，id）的顺序排序，先按照age列从小到大排序，age相同时按照id列从小到大排序。
- 使用辅助索引需要检索两遍索引：首先检索辅助索引获得主键，然后根据主键到主键索引中检索获得数据记录。

**辅助索引等值查询的情况：**

```
select * from t_user_innodb where age=19;
```

![在这里插入图片描述](https://ucc.alicdn.com/images/user-upload-01/175b62fdfe7c48d6a017e3506272f953.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiA5a6_5ZCb,size_20,color_FFFFFF,t_70,g_se,x_16 "在这里插入图片描述")  
**在辅助索引树中获取到主键id，再根据主键id到主键索引数中检索数据的的过程称为`回表查询`。**

**磁盘IO数（从根节点开始）：辅助索引3次 + 回表过程3次。**

### 3.2.3、组合索引

- 以表abc_innodb为例，id列为主键索引，创建一个联合索引`idx_abc(a，b，c)`。

```
CREATE TABLE `abc_innodb`
(
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `a`  int(11)     DEFAULT NULL,
  `b`  int(11)     DEFAULT NULL,
  `c`  varchar(10) DEFAULT NULL,
  `d`  varchar(10) DEFAULT NULL,
  PRIMARY KEY (`id`) USING BTREE,
  KEY `idx_abc` (`a`, `b`, `c`)
) ENGINE = InnoDB;
```

**组合索引的数据结构：**  
![在这里插入图片描述](https://ucc.alicdn.com/images/user-upload-01/1c72e1da60724e9d9f9a73e6ab625079.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiA5a6_5ZCb,size_20,color_FFFFFF,t_70,g_se,x_16 "在这里插入图片描述")  
**组合索引的查询过程：**

```
select * from abc_innodb where a = 13 and b = 16 and c = 4;
```

![在这里插入图片描述](https://ucc.alicdn.com/images/user-upload-01/d7c8ff7d92b347738c3adceba17d8dee.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiA5a6_5ZCb,size_20,color_FFFFFF,t_70,g_se,x_16 "在这里插入图片描述")

### 3.2.4、最左匹配原则

最左前缀匹配原则和联合索引的`索引存储结构和检索方式`是有关系的。

在组合索引树中，最底层的叶子节点按照第一列a列从左到右递增排序，但是b列和c列是无序的，b列只有在a列值相等的情况下小范围内有序递增；而clie只能在a和b两列值相等的情况下小范围内有序递增。

就像上面的查询，B+ 树会先比较a列来确定下一步应该检索的方向，往左还是往右。如果a列相同再比较b列，但是如果查询条件中没有a列，B+树就不知道第一步应该从那个节点开始查起。

可以说创建的idx_(a，b，c)索引，相当于创建了(a)、(a，b)、(a，b，c)三个索引。

**组合索引的最左前缀匹配原则：**

> 使用组合索引查询时，mysql会一直向右匹配直至遇到范围查询(>、<、between、like)等就会停止匹配。

### 3.2.5、覆盖索引

> **==覆盖索引并不是一种索引结构，覆盖索引是一种很常用的优化手段==。因为在使用辅助索引的时候，我们只可以拿到相应的主键值，想要获取最终的数据记录，还需要根据主键通过主键索引再去检索，最终获取到符合条件的数据记录。**
> 
> **在上面的 abc_innodb 表中的组合索引查询时，如果我们查询的结果只需要 a、b、c 这三个字段，那我们使用这个 idx_index (a，b，c)组合索引查询到叶子节点时就可以直接返回了，而不需要再次回表查询，这种情况就是==覆盖索引==。**


# 4、回表和联合索引的应用

## 4.1、回表查询

在InnoDB的存储引擎中，使用辅助索引查询的时候，因为辅助索引叶子节点保存的数据不是当前数据记录，而是当前数据记录的主键索引。如果需要获取当前记录完整的数据，就必须要再次根据主键从主键索引中继续检索查询，这个过程我们称之为**回表查询**。

由此可见，在数据量比较大的时候，回表必然会消耗很多的时间影响性能，所以我们要尽量避免回表的发生。

## 4.2、如何避免回表

使用覆盖索引，举个例子：现有User表

```
CREATE TABLE `user`
(
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name`  int(11)     DEFAULT NULL,
  `sex`  char(3)     DEFAULT NULL,
  `address`  varchar(10) DEFAULT NULL,
  `hobby`  varchar(10) DEFAULT NULL,
  PRIMARY KEY (`id`) USING BTREE,
  KEY `i_name` (`name`)
) ENGINE = InnoDB;
```

如果有一个场景:

```
select id,name,sex from user where name = 'zhangsan';
```

这个语句在业务上频繁使用到，而user表中的其他字段使用频率远低于这几个字段，在这个情况下，如果我们在建立name字段的索引时，不是使用单一索引，而是使用`联合索引（name，sex）`，这样的话再执行这个查询语句，根据这个辅助索引`（name，sex）`查询到的结果就包括了我们所需要的查询结果的所有字段的完整数据，这样就不需要再次回表查询去检索sex字段的数据了。

> **以上就是一个典型的使用覆盖索引的优化策略减少了回表查询的情况。**

## 4.3、联合索引的使用

**联合索引：**

> 在建立索引的时候，尽量在多个单列索引上判断是否可以使用联合索引。联合索引的使用不仅可以节省空间，还可以更容易的使用到覆盖索引。

**节省空间：**

> 试想一下，索引的字段越多，是不是更容易满足查询需要返回的数据呢？比如联合索引`（a，b，c）`是不是等于有了：`a`、`（a，b）`、`（a，b，c）`三个索引，这样是不是节省了空间，当然节省的空间并不是三倍于`a`、`（a，b）`、`（a，b，c）`三个索引所占用的空间，但是联合索引中data字段数据所占用的空间确实节省了不少。

**联合索引的创建原则：**

> 在创建联合索引的时候应该把频繁使用的列、区分度高的列放在前面，频繁使用代表索引利用率高，区分度高代表筛选力度大，这些都是在创建索引的时候需要考虑到的优化场景，也可以将常需要作为查询返回的字段增加到联合索引中。
> 
> **如果在联合索引上增加了一个字段，而恰好满足了使用覆盖索引的情况，这种情况建议使用联合索引。**

**联合索引的使用：**

- 考虑到当前是否已经存在多个可以合并的单列索引，如果有，那么推荐将当前的多个单列索引创建为一个联合索引。
- 当前索引存在频繁使用，在结果中有一个列也被频繁查询，而这个列不在当前频繁使用的索引中，那么这个时候就可以考虑这个列是否可以加入到当前频繁使用的索引中，使其查询语句可以使用到覆盖索引。