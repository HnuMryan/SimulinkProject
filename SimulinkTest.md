## 1 基本模块

### 1.1 运算部分

#### 1.1.1 selector

模块用于从指定的向量/数组中提取固定位置的元素或数组

![1658229838572](E:\Git\SimulinkTest\SimulinkProject\Note\1658229838572.png)

#### 1.1.2 Gain 矩阵乘法/点乘运算

使用Gain模块进行矩阵乘法/点乘运算等

![1658230186039](E:\Git\SimulinkTest\SimulinkProject\Note\1658230186039.png)



#### 1.1.3 bias/Weighted Sample Time

![1658230326795](E:\Git\SimulinkTest\SimulinkProject\Note\1658230326795.png)

### 1.2 模型验证

注意在这种模式下求解器需配置为离散定步长

#### 1.2.1 Check Discrete Gradient 检测离散梯度

检测每个采样步长内的离散梯度不超过设定梯度，__如下述斜坡信号斜率为5，步长为0.1则此时的梯度为__(再确认)

![1658230698567](E:\Git\SimulinkTest\SimulinkProject\Note\1658230698567.png)

#### 1.2.2 Assertion模块

Assertion模块在输入为1时Pass,在输入为0时failed

![1658230732509](E:\Git\SimulinkTest\SimulinkProject\Note\1658230732509.png)

#### 1.2.3 Check Static Range模块

检测输入信号是否在设定范围内

![1658230743099](E:\Git\SimulinkTest\SimulinkProject\Note\1658230743099.png)

### 1.3 基本滤波

#### 1.3.1 均值滤波

取信号的前四个值做均值处理

#### ![1658231304041](E:\Git\SimulinkTest\SimulinkProject\Note\1658231304041.png)

取信号的中值进行滤波

![1658231327079](E:\Git\SimulinkTest\SimulinkProject\Note\1658231327079.png)

#### 1.3.3 低通滤波

> y(k) = a✖y(k) + (1-a)✖y(k-1)			# 系数a乘以当前值+(1-a)乘以上一时刻值

![1658231338829](E:\Git\SimulinkTest\SimulinkProject\Note\1658231338829.png)

### 1.4 位操作

#### 1.4.1数之间的位与操作

如下图所示：3和6分别解析为2进制进行与操作

![1658231617589](E:\Git\SimulinkTest\SimulinkProject\Note\1658231617589.png)

#### 1.4.2 数组的位或操作

![1658231636899](E:\Git\SimulinkTest\SimulinkProject\Note\1658231636899.png)

#### 1.4.3 数组与掩码的位与

![1658231655821](E:\Git\SimulinkTest\SimulinkProject\Note\1658231655821.png)

## 2 模型覆盖度

### 2.1 决策覆盖度(Decision Coverage)

所有能产生的结果是否都被执行输出如下例子：  
![1658231994063](E:\Git\SimulinkTest\SimulinkProject\Note\1658231994063.png)

如上图的决策覆盖度就是100%

 ![1658234337809](E:\Git\SimulinkTest\SimulinkProject\Note\1658234337809.png)

如上图的决策覆盖度就是50%

### 2.2 条件覆盖度(Condition Coverage)

简单来说所有的条件都需要被执行到  
如下图，这个信号只能做到<=3的验证而>=3无法验证  
![1658234561713](E:\Git\SimulinkTest\SimulinkProject\Note\1658234561713.png)

如下图>=3和<=3两种条件都能够测试得到，此时条件覆盖度达到100%

![1658234650835](E:\Git\SimulinkTest\SimulinkProject\Note\1658234650835.png)



### 2.3 修正条件/决策覆盖度(Modified Condition and Decision Coverage简称MCDC)

* 每个输入至少有一次改变引起输出的改变，覆盖度达到100%   
  

如下图，因为两个Signal都取到过0或者1所以此时条件覆盖度可达到100%，但没有因为输入的改变导致的输出改变，所以此时的MCDC为0%

  ![1658234804749](E:\Git\SimulinkTest\SimulinkProject\Note\1658234804749.png)



如下图信号1有因输入改变而引起输出的改变，信号2也有因此MCDC覆盖度=50%+50%=100%

![1658235195115](E:\Git\SimulinkTest\SimulinkProject\Note\1658235195115.png)



## 3 Simulink Test

针对如下需求——是否具备 成为VIP：

> * 条件1 ： 年龄满18岁
> * 条件2 ： 至少充值100
> * 条件3 ： 游戏年龄满1年
> * 条件4 ： 被举报次数小于3
>
> 1，4必须满足，2、3必须满足其一（1&&4&&(2||3))

针对以上需求建立如下应用模型

![1658236318410](E:\Git\SimulinkTest\SimulinkProject\Note\1658236318410.png)

### 3.1 Test Assessment和Test Sequence语法

#### 3.1.1 Assessment Statements

用于Verify仿真，停止仿真并且返回验证结果，使用Assessment Statements

| 关键字 | 语法                                                         | Description                                         | Example                                                      |
| ------ | ------------------------------------------------------------ | --------------------------------------------------- | ------------------------------------------------------------ |
| verify | verify(expression)  <br />verify(expression,errorMessage)<br />verify(expression,identifier,errorMessage | 评估一个逻辑表达式，将结果显示在diagnostic viewer中 | verify(x > y,'SimulinkTest:greaterThan',' x ad y values are %d, %d',x,y) |
| assert | verify(expression)  <br />verify(expression,errorMessage)    | 评估一个逻辑表达式，如果结果为假将直接停止仿真      | assert(h==0 && k==0,... 'h and k must '... 'initialize to 0') |

#### 3.1.2 Temporal Operators

用于创造一个表达式来评估仿真时间，如果要用到变量一定需要提前定义

| Operator                                                     | Syntax                                          | Description                                                  | Example                                                      |
| ------------------------------------------------------------ | ----------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| [et](https://ww2.mathworks.cn/help/sltest/ref/et.html)       | `et(TimeUnits)`                                 | The elapsed time of the test step in `TimeUnits`. Omitting `TimeUnits` returns the value in seconds.<br />**测试步骤花费的时间单位** | The elapsed time of the test sequence step in milliseconds:`et(msec)` |
| [t](https://ww2.mathworks.cn/help/sltest/ref/t.html)         | `t(TimeUnits)`                                  | The elapsed time of the simulation in `TimeUnits`. Omitting `TimeUnits` returns the value in seconds.<br />**仿真花费的时间单位** | The elapsed time of the simulation in microseconds:`t(usec)` |
| [after](https://ww2.mathworks.cn/help/sltest/ref/after.html) | `after(n, TimeUnits)`                           | Returns `true` if `n` specified units of time in `TimeUnits` elapse since the beginning of the current test step.<br />**从当前测试步骤开始经过n个时间单位后返回真** | After 4 seconds:`after(4,sec)`                               |
| [before](https://ww2.mathworks.cn/help/sltest/ref/before.html) | `before(n, TimeUnits)`                          | Returns `true` until `n` specified units of time in `TimeUnits` elapse, beginning with the current test step.<br />**直到经历了n个时间单位后再开始当前测试步骤** | Before 4 seconds:`before(4,sec)`                             |
| [duration](https://ww2.mathworks.cn/help/sltest/ref/duration.html) | `ElapsedTime = duration (Condition, TimeUnits)` | Returns `ElapsedTime` in `TimeUnits` for which `Condition` has been `true`. `ElapsedTime` is reset when the test step is re-entered or when `Condition` is no longer `true`.<br />**当条件为真的时候返回所花费的时间单位，返回值当测试步骤重新进入或者条件不再为真时重置** | Return `true` if the time in milliseconds since `Phi > 1` is greater than 550:`duration(Phi>1,msec) > 550` |

#### 3.1.3 Transition Operators

使用表达式来评估信号事件，注意u必须是一个输入的data类型

| Operator                                                     | Syntax                 | Description                                                  | Example                                                    |
| ------------------------------------------------------------ | ---------------------- | ------------------------------------------------------------ | ---------------------------------------------------------- |
| [hasChanged](https://ww2.mathworks.cn/help/sltest/ref/haschanged.html) | `hasChanged(u) `       | Returns `true` if `u` changes in value since the beginning of the test step, otherwise returns `false`.`u` must be an input data symbol.<br />**如果信号发生了改变则返回真，否则返回假** | Transition when `h` changes:`hasChanged(h)`                |
| [hasChangedFrom](https://ww2.mathworks.cn/help/sltest/ref/haschangedfrom.html) | `hasChangedFrom(u,A) ` | Returns true if `u` changes from the value `A`, otherwise returns false.`u` must be an input data symbol.<br />**如果信号从A变成了其他值返回真，否则返回假** | Transition when `h` changes from `1`:`hasChangedFrom(h,1)` |
| [hasChangedTo](https://ww2.mathworks.cn/help/sltest/ref/haschangedto.html) | `hasChangedTo(u,B) `   | **Returns true if `u` changes to the value `B`, otherwise returns false.`u` must be an input data symbol.<br />如果信号变成了B则返回真，否则返回假** | Transition when `h` changes to `0`:`hasChangedTo(h,0)`     |

#### 3.1.4 Transition Types

* 标准转移

  标准转移序列从第一个Step开始，根据过度条件行进到下一个Step

  ![1658239953816](E:\Git\SimulinkTest\SimulinkProject\Note\1658239953816.png)

  

* when分解

  分解序列与switch语句类似，你的序列可以按照特定的存在在你的模型中间的条件进行，在一个分解序列，Steps的激活是基于when后面的语句

  ![1658240218505](E:\Git\SimulinkTest\SimulinkProject\Note\1658240218505.png)

### 3.2 Test Assessment

在本例中测试需要确认的是当res(验证结果)为真时，我们的基本逻辑（1&&4&&(2||3))是否满足

![1658241761385](E:\Git\SimulinkTest\SimulinkProject\Note\1658241761385.png)

### 3.3 Test Sequence

这里列举了3个测试用例

![1658241335874](E:\Git\SimulinkTest\SimulinkProject\Note\1658241335874.png)

### 3.4 结果展示

测试框架如下：

![1658241870568](E:\Git\SimulinkTest\SimulinkProject\Note\1658241870568.png)

测试结果如下：

![1658241920237](E:\Git\SimulinkTest\SimulinkProject\Note\1658241920237.png)

