[TOC]

# NCL intro

NCL中许多内置函数和绘图程序需要载入特定的库才能使用，在本教程中，规定如下
- 如果你的版本号在 6.2.0 及以上，无需载入
- 版本号在6.2.0以下，加入以下
  ```
  load "$NCARG_ROOT/lib/ncarg/nclscripts/csm/gsn_code.ncl"
  load "$NCARG_ROOT/lib/ncarg/nclscripts/csm/gsn_csm.ncl"
  load "$NCARG_ROOT/lib/ncarg/nclscripts/csm/contributed.ncl"
  ```

## 1 读取数据
NCL支持多种数据格式的直接读取，常见如 **NetCDF** 、**GRIB** 、**HDF** 等等。打开这些格式支持的文件非常简单，只需要使用 **addfile** 函数：

```
f = addfile("hgt_2009_2012_monthly.nc", "r")
```
- 函数将文件指针返回给文件对象f，以便进行下一步的操作

文件被打开后，可通过对文件对象的操作查看文件的基本信息和文件变量信息：
```
print(f)
list_filevars(f)
```
- 打印子程序**print**用于文件对象时，将文件的全部信息，包括文件的基本信息、文件的全局属性、维和所有的**文件变量概略**到命令行窗口（标准输出）
- 子程序**list_filevars**仅将文件中的所有**变量概略**输出到命令行窗口

通常打开文件就是为了读取文件变量，使用 -> 来读取文件中的变量：
```
hgt = short2flt(f->hgt)
printVarSummary(hgt)
```
- 文件变量hgt使用了short打包，因此使用函数**short2flt**自动解包
- 函数 **printVarSummary** 打印NCL变量的概略信息

## 2 选取数据 切片
实际分析中，我们需要选择多维数组变量中某特定的 天/高度层/位置 的数据，不可避免地要进行数据选取工作。NCL支持一般语言中的**数组下标**（整数）和其针对气象格点数据专门开发的**坐标下标**来选取数据

### 2.1 数组下标
可以使用整数来选取数组中特定位置的元素-**下标从0开始**
```
arr1 = ispan(1, 10, 1)
print(arr1(4))
print(hgt(0, 4, 1, 10))
```
- **ispan** 函数生成等间隔的整数数组，这里arr1的值为 (/1,2,3,4,5,6,7,8,9,10/)
- **arr1(4)** 获取arr1数组中的第5个元素
- **hgt(0, 4, 1, 10)** 获取hgt数组第1个时次，第5层，第2个纬度、第11个经度上的值

对某一个连续片段的选取
```
print(arr1(2:4))
print(arr1(1::2))
print(hgt(:15:2, 0, 15, 29))
```
- **arr1(2:4)** 选取arr1数组中第3个-第5个元素
- **arr1(1::2)** 选取arr1数组中第2个-最后，**每隔一个元素选取**
- **hgt(:15:2, 0, 15, 29)** 选取hgt数组中第1个时次-第16个时次每隔一个时次 | 第1层 | 第16个纬度 | 第30个经度的片段

### 2.2 坐标下标
选取位势高度变量 hgt 第一个时次 | 500 hPa 气压层 | 所有经纬度 的 数据部分：
```
hgt_500 = hgt(0, {500}, :, :)
```
- 整数索引值**从 0 开始**
- 使用**大括号**直接索引指定层 500 hPa, 同理可用于指定经纬度
- **冒号 :** 可以索引整个维

借助于维名称，可以实现**维顺序的重新排列**，二维数组的维序交换就是转置，高维数组的维序变化较难用语言解释,下句选取数组hgt的第1个时次 | 500-700hPa | 90-120°E | 所有纬度 的部分数据

```
hgt_500_0 = hgt(time| 0, {level| 500:700}, {lon| 90:120}, lat| :)
```
- 使用维序重排时需要**将所有的维名称全部列出**，即使索引的是整个维
- 使用坐标来选取片段同样使用**冒号 :**

### 2.3 不连续片段选取
使用切片语法只能选取连续的数组片段，不连续的数组片段需要使用**整数数组**进行选取

下例选取add1数组中第2个、第4个、第5个、第6个、第8个、第9个元素组成的新数组
```
idx = (/1,3,4,5,7,8/)
print(arr1(idx))
```
- 数组直接创建的方法是使用**括号和左斜杠 (/ /)** 将每一个用**逗号分隔**的元素包裹起来

### 2.4 条件选取
如果要获取变量中满足一定条件的值的位置怎么办？这时候需要使用条件索引技巧
使用 **ind** 函数**返回一维数组中满足条件的元素的位置**
```
a = (/1, 2, 3, 4, 5, 5, 4, 3, 2, 1, 1, 2, 3, 4, 5/) ; 定义一维数组
a@_FillValue = 5  ; 将5设置为缺测
valid = ind(ismissing(a))  ; 找到数组a中所有缺测值的位置
a(valid) = 0  ; 将缺测值所在位置元素赋值为0
a(ind(a.ne.0)) = -a(ind(a.ne.0)) * 2 + 1 ; 对所有不等于0的元素操作
```
- 设定变量的缺测值属性，给变量 **_FillValue** 属性赋值
- **ismissing** 函数返回数组中每一个元素缺测测试的结果 (True | False)
- 关系运算符包括 **.gt.** (大于)  |  **.ge.** (大于或等于)  |  **.lt.** (小于)  |  **.le.** (小于或等于)  | **.eq.** (等于)  | **.ne.** (不等于)

多维数组条件选取没有特定的函数，需要联合 **ndtooned** 和**ind_resolve**

## 3 操作
实际应用中，常需对数组进行一定的操作，从而实现特定的目的

### 3.1 转置
```
arr2 = (/(/1, 2, 3, 4/), (/5, 6, 7, 8/)/)
arr3 = transpose(arr2)
```
- 与前一节的维序重排不同，未定义维名称的二维数组变量需使用函数 **transpose** 来进行转置

### 3.2 扩展
NCL强类型语言的性质决定了变量的扩展实际上是一个新变量的建立
在数组 arr1 后新添两个元素 11 和 12

```
arr1 := array_append_record(arr1, (/11, 12/), 0)
```
- 限于强类型语言的特点，赋值过的变量大小确定后不可改变，因此要么重新另一个变量等于右侧
- 另一种方法是使用**重赋值算符冒号和等号 :=** ， 其自动销毁当前的已被赋值的变量，重新赋值
- 重赋值算符在不需要重赋值的地方也可以使用，此时其退化为赋值运算符

将第二个数组与第一个数组按行连结
```
x1 = (/(/-3.71, -3.70, -3.03/),\
       (/-1.72, -3.70, -3.64/),\
       (/-1.91, -4.92, -4.85/),\
       (/-4.17, -4.68, -4.41/)/)
x2 = (/(/14.31, 15.95, 19.75/),\
       (/14.00, 10.11, 12.65/)/)
a = table_attach_rows (x1, x2, 0)
write_matrix(a, "3f6.2", False)
```
- **反斜杠 \  用于续行**，当一行无法写下时，使用其将多行连接
- **write_matrix**按格式打印二维数组

输出如下：
```
-3.71  -3.70  -3.03
-1.72  -3.70  -3.64
-1.91  -4.92  -4.85
-4.17  -4.68  -4.41
14.31  15.95  19.75
14.00  10.11  12.65
```

将第二个数组与第一个数组按列连结
```
y1 = (/(/-3.04, -2.05, -4.29/),\
       (/-2.07, -3.66, -1.46/),\
       (/-2.49, -3.11, -3.83/),\
       (/-4.44, -3.24, -3.08/),\
       (/-2.14, -4.68, -2.01/)/)
y2 = (/(/153.50, 167.58, 115.26, 143.38, 154.57/),\
       (/190.60, 138.17, 113.66, 172.26, 150.34/),\
       (/150.35, 189.73, 146.19, 159.03, 188.25/),\
       (/148.74, 176.38, 100.36, 155.45, 196.51/),\
       (/188.72, 122.79, 176.24, 117.69, 174.37/)/)
y1!0 = "row"
y1!1 = "col"
y2!0 = "row"
y2!1 = "col"
b = table_attach_columns(y1, y2, 0)
write_matrix(b, "8f8.2", False)
```
- **table_attach_columns函数要求连结的变量必须有维名称**
- **维名称定义使用 ! 接维序号**，例中给y1和y2变量第一维赋值名称为"row", 第二维赋值名称为"col"

输出如下
```
-3.04   -2.05   -4.29  153.50  167.58  115.26  143.38  154.57
-2.07   -3.66   -1.46  190.60  138.17  113.66  172.26  150.34
-2.49   -3.11   -3.83  150.35  189.73  146.19  159.03  188.25
-4.44   -3.24   -3.08  148.74  176.38  100.36  155.45  196.51
-2.14   -4.68   -2.01  188.72  122.79  176.24  117.69  174.37
```

将包含12个元素的一维数组arr1扩展到多维（4行12列）
```
arr4 = onedtond(arr1, (/4,12/))
write_matrix (arr4, "12I3", False)
```
- **onedtond**将一维变量扩展到多维，按最右维填入数值

输出如下
```
1  2  3  4  5  6  7  8  9 10 11 12
1  2  3  4  5  6  7  8  9 10 11 12
1  2  3  4  5  6  7  8  9 10 11 12
1  2  3  4  5  6  7  8  9 10 11 12
```

按指定匹配维扩展到与指定变量同等大小
假定数组q的维数为 nt x ny x nx x nz  ，而数组dz 为一维数组，长度为nz
按 q 的第3维扩展 dz 到 q 的大小
```
dzConform = conform(q, dz, 3)
```

### 3.3 变形
```
arr5 = reshape(arr1, (/2, 6/))
arr6 = reshape(arr5, (/3, 4/))
```
- **reshape**函数改变数组的形状，改变的维数大小应等于改变前的维数大小
- 上例中，arr1为12个元素的一维数组，因此其可以改变为 2x6 | 6x2 | 3x4 | 4x3 四种形状

### 3.4 蒙版
顾名思义就是将不需要的东西给盖住，在程序中就是置为缺测

```
x = ispan(-10, 10, 1)
x = mask(x, x.lt.0, True)
```
- **mask**函数，第二个表达式参数等于第三个参数，对应于第一个参数的位置上的值保护起来
- 上例中，x小于0为真的元素被保护起来，其余的值全部被置为缺测

下例中读取温度变量TS和下垫面标识变量ORO，并用下垫面标识来做海洋或陆地的蒙版
```
begin
    f = addfile("ts_oro.nc", "r")  ; 打开文件
    TS = short2flt(f->TS) ; 读取地面温度
    ORO = short2flt(f->ORO)  ; 读取下垫面标识，海洋(0) | 陆地(1)

    land_only  = TS ; 拷贝TS副本到land_only变量
    ocean_only = TS ; 拷贝TS副本到ocean_only变量

    land_only  = mask(TS, ORO, 1)   ; 当ORO=1时，返回TS的值，其余置缺测
    ocean_only = mask(TS, ORO, 0)   ; 当ORO=0时，返回TS的值，其余置缺测
end
```
- 上述方法将海洋或陆地的值做蒙版，以便只绘制海洋或陆地的白化图形

### 3.5 条件操作
条件操作指的是对数组中满足条件的值和不满足条件的分别进行的操作，功能非常强大
```
arr6 = ispan(-6, 6, 1)
arr6 = where(arr6.lt.0, arr6+256, arr6*2)  ; 将小于0的值加上256, 大于等于0的值做平方

arr6@_FillValue = default_fillvalue(typeof(arr6))
arr6_inv = 1. / where(arr6.ne.0, arr6, arr6@_FillValue)  ; 禁用0除

v1 = (/(/12, 43, 18, 23/), (/84, 14, 32, 54/)/)
v2 = (/(/99, 47, 32, 53/), (/75, 45, 54, 16/)/)
vmin = where(v1.lt.v2, v1, v2) ; 取两数组对应元素最小值
vmax = where(v1.gt.v2, v1, v2) ; 取两数组对应元素最大值
```
- **where**函数，相当于三元运算，第1个参数条件表达式满足时，结果等于第2个参数，不满足等于第3个参数
- 必要时可以嵌套使用where函数处理更复杂的条件分支结构
- **typeof**函数，给定变量，返回变量的类型
- **default_fillvalue**返回给定类型的默认缺测值
- 逻辑运算符包括  **.and.** （与）| **.or. （或）** |  **.not.** （非） | **.xor. （异或）**

## 4 基础代数 

### 4.1 算数运算
NCL中标量运算和数组运算是一致的

+(加)  |  - (减)  |  * (乘)  |  / (除)   |  % (取模)   |  ^ (幂)

```
a = 2.2  ; 浮点型变量a, 赋值为 2.2
b = 5  ; 整型变量b, 赋值 5
c = "mcs"  ; 字符串型变量c, 赋值 "mcs"
arr7 = ispan(2, 12, 2)  ; (/2, 4, 6, 8, 10, 12/)
arr8 = fspan(1, 6, 6)  ; (/1., 2., 3., 4., 5., 6./)
;; 标量的算数运算
print("a+b = "+(a+b)+"   a-b = "+(a-b))  ; 浮点和整数的加和减
print("a*b = "+(a*b)+"    a/b = "+(a/b))  ; 浮点和整数的乘和除
print("b%2 = "+(a%b)+"  a^b="+(a^b))  ; 整型取模和幂运算(a的b次方)
print("a+c = "+(a+c)+"   b+c = "+(b+c))  ; 整型+字符串
;; 同样大小的数组支持跟标量一致的算数运算
print(arr7 + arr8)  ; 整型数组和浮点数组相加
print(arr7 - arr8)  ; 整型数组和浮点数组相减
print(arr7 * arr8)  ; 整型数组和浮点数组相乘
print(arr7 / arr8)  ; 整型数组和浮点数组相除
print(arr7^3+arr8^2)  ; arr7的立方加arr8的平方
;; 多维数据算数运算也是一致的
hgt_diff = hgt(:, {500}, :, :) - hgt(:, {1000}, :, :)
copy_VarCoords(hgt(:, {500}, :, :), hgt_diff)  ; 复制变量坐标
```
- **fspan** 通过给定起始值、终值和元素个数来生成等间隔的浮点数组
- NCL支持自动类型转换，当整型变量和浮点型变量运算时，整型将自动转换为浮点型参与计算
- 同理，任何类型变量与字符串相加时，都将首先转为字符串型，然后执行字符串连接运算
- **算数运算将导致变量元数据（坐标，维，属性）的丢失**
- **copy_VarCoords**可以将一个变量的坐标复制给另一个变量

### 4.2 平均
对变量求整体的平均
```
print(avg(arr7))
```

指定维的平均 **dim_avg** | **dim_avg_n** |**dim_avg_n_Wrap**
```
print(dim_avg(hgt))  ; 最右边维的平均
print(dim_avg_n(hgt, 0))  ; 指定维的平均
print(dim_avg_n_Wrap(hgt, 1)) ; 指定维的平均，保留变量元数据
```

### 4.3 最大值/最小值
子程序 **printMinMax** 可以输出一个变量的最大值和最小值

```
printMinMax(hgt)
```
函数 **min** |**dim_min**|**dim_min_n**|**dim_min_n_Wrap**可以获取整体最小值、最右维最小值、指定维最小值、保留元数据
```
print(min(hgt))  ; hgt的最小值
print(dim_min(hgt))  ; 最右边维的最小值
print(dim_min_n(hgt, 0))  ; 指定维的最小值
print(dim_min_n_Wrap(hgt, 1)) ; 指定维的最小值，保留变量元数据
```
- **Wrap**函数，当一个函数以Wrap结尾时，代表其运算结果能自动保留运算变量的元数据(维、属性、坐标)

函数 **max** |**dim_max**|**dim_max_n**|**dim_max_n_Wrap**可以获取整体最大值、最右维最大值、指定维最大值、保留元数据

```
arr9 = (/9, 3, 2, 5, 2, 1, 4, 8, 1, 9/)
print(minind(arr9))  ; 5
print(maxind(arr9))  ; 0
```
- 有多个最小值或最大值时，只返回第一个最小值或最大值的索引

## 5 时间序列
时间序列处理有非常广泛的应用，比如转换日期格式，格式化输出日期时间，气候序列分析等等

### 5.1 解析时间字符串
NCL6.4.0中新增了一个非常有用的函数 **cd_inv_string** 可以将给定的字串按格式转为日期时间

读取一些文本文件（如下）时，难免要碰到按一定格式存储的日期，只有将其转换为NCL的日期时间，才能真正利用起来
```
2010-10-13 00 1 118 1414 1004      13
2010-10-13 06 1 119 1411 1002      15
2010-10-13 12 2 120 1409  998      18
2010-10-13 18 2 121 1406  998      18
2010-10-14 00 2 122 1400  995      20
```

```
load "cd_inv_string.ncl"  ; 将下载的文件置于当前目录后，载入到命名空间
begin
    f = asciiread("megi.txt", -1, "string") ; 以字符串格式读取文件所有行
    time_str = str_get_cols(f, 0, 13)  ; 获取字符串变量f中的日期子串数组
    time = cd_inv_string(time_str, "%Y-%N-%D %H")  ; 转换日期子串到NCL时间

    lat = tofloat(str_get_field(f, 4, " "))*0.1  ; 读取纬度，转换为浮点数后，转换单位
    lon = tofloat(str_get_field(f, 5, " "))*0.1  ; 读取经度，转换为浮点数后，转换单位

    print(time+"    "+lon+" "+lat) ; 输出变量
    printVarSummary(time)  ; 输出时间变量的概略信息
end
```

- **asciiread** 函数可以读取文本文件
- **str_get_cols** 指定起始和结束的列序号来获取字符串/串组的子串
- **cd_inv_string**按指定格式将字符串/串组解析为NCL日期时间
- **str_get_field** 按指定分隔符划分字符串/串组的域，通过域编号来获取子串
- 加号 + 在NCL中可以用于连接字符串或同样大小的串组
- **to**函数，**以to开头的函数用于强制转换变量类型**，注意不能直接返回给原变量（新变量 | :=）
- **tofloat**将输入的变量类型转换为浮点型

### 5.2 转换NCL时间到格式化字符串
将NCL日期时间转换为格式化字符串是与字符串解析执行相反的操作，使用函数 **cd_string** 来进行，通常的应用场景有时间刻度标签、气候时间序列选取

```
load "$NCARG_ROOT/lib/ncarg/nclscripts/contrib/cd_string.ncl"
print(cd_string(hgt&time(:10), "%Y-%N-%D %H:%M:%S"))  ; (/"2009-01-01 00:00:00", .../)
ndims = dimsizes(hgt)  ; (/48, 12, 37, 144/)
print(cd_string(hgt&time(ndims(0)-10:), "%c %Y")) ; (/"Mar 2012", "Apr 2012", .../)
mon = toint(cd_string(hgt&time, "%n"))  ; idx
idx_MAM =   ind(mon.ge. 3 .and. mon.le.5)  ; spring
hgt_MAM_avg = dim_avg_n_Wrap(hgt(idx_MAM, :, :, :), 0)  ; spring average
```
- **cd_string**需要载入到命名空间才能使用
- 变量的每个维，使用 **&** 符号，连接维名称， 即可取出维变量，维变量也是个标准的NCL变量
- **dimsizes**可以获取变量的每一维的大小，返回到一个数组，要获取变量有几维，嵌套使用**dimsizes**
- **toint** 转换输入的变量类型到整型，注意不是原地转换

### 5.3 转换NCL时间到世界时
将NCL中的时间变量转换到世界时，以便于下一步的计算或查看，使用**cd_calendar**

```
time = (/17522904, 17522928, 17522952, 17522976, 17523000/)  ; 时间
time@units = "hours since 1-1-1 00:00:0.0" ; 设定时间单位
utc_date = cd_calendar(time, 0)  ; 转换到世界时
year   = tointeger(utc_date(:,0))
month  = tointeger(utc_date(:,1))
day    = tointeger(utc_date(:,2))
hour   = tointeger(utc_date(:,3))
minute = tointeger(utc_date(:,4))
second = utc_date(:,5)
date_str = sprinti("%0.2iZ ", hour) + sprinti("%0.2i ", day) + \
           month_abbr(month) + " "  + sprinti("%0.4i", year)
print(date_str)
```
- **cd_calendar**的第二个参数可以指定返回值的格式，从而应付不同的情景

## 6 流程控制
流程控制用来实现程序设计的两种重要结构，**重复** 和 **选择**，大多数现实的问题都会碰到这两种结构

### 6.1 循环
循环用来实现代码的重复，从而节省代码
NCL中提供两种循环实现语句，一种是 **do** 语句，另一种是 **do while** 语句
**基于重复次数的已知与否来选用 do 还是do while**

循环初始化变量，生成九九乘法表
```
arr10 = new((/9, 9/), integer)
do i = 1, 9
    do j = 1, 9
        arr10(i-1, j-1) = i*j
    end do
end do
write_matrix(arr10, "9I3", False)
```
- **new**语句创建一个未初始化的指定大小的数组，你可以在第三个参数中指定缺测值，否则使用类型默认缺测值
- do 循环语句以，**do 循环变换 = 初值, 终值, 步长 为开始，以 end do结束，循环变量步长必须为整数，默认为1**，可省略

**do while** 语句**使用条件判断来控制循环的结束**，因此需要注意循环条件的设置，避免死循环的出现

```
i = 0 ;创建循环变量
do while (i < 20)
    print("i = "+i) ; 打印循环变量
    i = i + 1 ; 控制循环变量在循环体内的变化，如果循环过程 i 一直不变可就死循环了哦
end do
```

循环语句中可以使用 **break** 和 **continue** 来控制循环过程，其中:
- **break** 语句用于中断整个循环，转而执行循环体后面的语句
- **continue** 语句用于中断本次循环，继续下一次循环过程

### 6.2 条件判断
条件判断用于控制流的分支，实现选择结构，使用 **if** 语句来进行条件判断，有点可惜的是NCL中并没有 switch 语句

```
wind = (/20, 25, 25, 30, 35, 40, 45, 50, 50, 45/) ; 热带气旋最大风速
do i = 0, dimsizes(wind)-1  ; 循环
    if wind(i) .gt. 50.9 then ; 条件一
        print("Intensity of TCs: "+wind(i) +" scale: Super Typhoon")
    else if wind(i) .gt. 41.4 then ; 条件二（条件一的分支）
        print("Intensity of TCs: "+wind(i) +" scale: Intense Typhoon")
    else if wind(i) .gt. 32.6 then  ; 条件三（条件二的分支）
        print("Intensity of TCs: "+wind(i) +" scale: Typhoon")
    else if wind(i) .gt. 24.4 then  ; 条件四（条件三的分支）
        print("Intensity of TCs: "+wind(i) +" scale: Intense Tropic Strom")
    else if wind(i) .gt. 17.1 then  ; 条件五（条件四的分支）
        print("Intensity of TCs: "+wind(i) +" scale: Tropicl Strom")
    else if wind(i) .ge. 10.8 then  ; 条件六（条件五的分支）
        print("Intensity of TCs: "+wind(i) +" scale: Tropic Depression")
    else  ; 条件六 分支
         print("Not classified")  
    end if  ; 结束条件六
    end if  ; 结束条件五
    end if  ; 结束条件四
    end if  ; 结束条件三
    end if  ; 结束条件二
    end if  ; 结束条件一
end do
```
- 一个if语句怎么那么多 end if,不要怀疑，那是因为NCL中并没有 else if 语句，而上述代码中的else if不过是将 else 和 if 两个语句放在同一行，造成else if 的假象罢了，这点有点坑哦
- if 语句以 if 条件判断 then 开始，以end if 结束，可使用 if ... then - else - end if 来展开分支结构

## 7 函数
NCL中既包括大量的气象诊断和绘图函数，同时也可以自定义函数和子程序，采用函数式的编程思路，尽管支持的函数式编程功能有限，但依然能够满足基本的需求，也是需要认真掌握的一部分内容

## 8 绘图
绘图是NCL最强大的功能，它能帮助你生成高质量的、可用于发表的图形。
通常默认图形能够用于快速分析，要生成论文级别的图形需要设定很多的图形属性，掌握起来略有难度，需要不断积累

### 8.1 折线图
```
begin  ;开始代码块
    wks = gsn_open_wks ("png","xy") ; 将图形发送到xy.png文件中
    res = True  ; 建立源变量
    res@tiMainString = "Basic XY plot" ; 通过给源变量设置属性 tiMainString 来设定图形标题
    plot = gsn_csm_xy (wks, hgt&lat, hgt(0, {500}, :, {120}), res) ; 调用折线图绘图函数绘图
end  ;结束代码块
```

### 8.2 等值线图
```
begin
  wks = gsn_open_wks("ps","ce")  ; 将图形写入ce.ps文件
  hgt_avg = dim_avg_n_Wrap(hgt, 0)  ; 计算所有时次的平均位势高度场
  res = True ; 建立源变量
  res@mpMinLonF = 100. ; 设定源变量属性 mpMinLonF 指定地图最小经度
  res@mpMaxLonF = 150. ; 设定源变量属性 mpMaxLonF 指定地图最大经度
  res@mpMinLatF = 10. ; 设定源变量属性 mpMinLatF 指定地图最小纬度
  res@mpMaxLatF = 45. ; 设定源变量属性 mpMaxLatF 指定地图最大纬度
  plot = gsn_csm_contour_map(wks, hgt_avg({500}, :, :), res)  ; 调用地图等值线图函数绘图
end
```

## 9 输出变量到文件 
### 9.1 输出到NetCDF文件
由于NCL数据结构基于NetCDF定制，因此在NCL中将变量输出到NetCDF文件无疑是最明智的选择
最简单的输出方式
```
fout = addfile("hgt_annually.nc", "c")
fout->hgt = hgt_avg
```
- 注意，此时函数 addfile 的第2个参数是 **"c" , 代表创建**
- 为节约存储空间，我们可以用**pack_values函数将变量压缩为short型**，再存入文件

### 9.2 输出到文本文件
```
npts = 100
i    = ispan(1,npts,1)
j    = generate_unique_indices(npts)
k    = generate_unique_indices(npts)
x    = random_uniform(-10,10,npts)
y    = random_uniform(0,1000.,npts)
write_table("file2.txt","w",[/j,x,i,y,k/], "string_%03i %8.2f %4.0i %8.1f     string_%03i")
```
- 函数**generate_unique_indices** 生成随机的索引值
- 函数**random_uniform** 生成均匀分布的随机数
- 函数 **write_table** 将列表变量按指定格式字符串写入文件
- 这里使用了列表类型，创建列表使用**中括号和左斜杠 [/ /]** 将变量包裹起来





