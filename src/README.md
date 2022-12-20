## 一点理解

### Verilog

要先在脑中考虑好电路图，再根据电路图连线写 `verilog` 代码

电路本质上其实就是一大堆寄存器之间的连线，连线中间夹杂着一些逻辑门和选择器

#### wire

1. `wire` 就是电线，连接的过程中可以夹杂逻辑门（对数据进行运算）
2. 正因为是电线，所以只能用 `assign` 赋值，并且只能被赋值一次，一条电线肯定只有一种连法
3. 正因为是电线，所以是 `module` 里一开始就被连接好的，无关乎被写在什么位置
4. 可以用三目运算符实现 `if` 的效果

#### reg

1. `reg` 是寄存器，具有储存功能
2. 在 `@always(*)` 中**只**使用 `=` 给 `reg` 赋值，这完全等价于 `wire` 用 `assign` 赋值
3. 在 `@always(posedge clk)` 中**只**使用 `<=` 给 `reg` 赋值，所有赋值操作是并行的，也就是可以用旧版本数据给新版本数据赋值

#### 组合与时序

1. 组合逻辑是顺序执行的；时序逻辑是并行执行的
2. 组合逻辑使用的越多，时延有可能变大（取决于层数）；时序逻辑是并行的，但也因此相当于一个周期只能执行一条语句
    所以对于有先后顺序的语句，应尽量使用层数少的组合逻辑；对于无先后顺序的语句，应尽量使用时序逻辑
    **因此优化的关键就是减少逻辑的层数！！既可以减少组合时延，又可以减少时序周期数**
3. 对于需要直接自己赋值给自己的，必须使用时序逻辑
    对于需要间接自己赋值给自己的，不能使用在一个周期内就完成的组合逻辑，否则会导致综合出 `latch`（锁存器）

#### 总结 / 重点 / 设计原则
1. 尽量减少逻辑层数，合理分配组合与时序，可以流水的能用时序的就用时序，否则用层数少的组合，**一个模块内部各条可能的线路时序必须同步，否则会出现 `structural hazard`**
2. **每个模块在保证所需周期数是最少的前提下用尽量多的时序，如果使用时序会增加周期数，那么才用组合**
3. **每个模块对外输出信息用组合，更新内部信息用时序**
4. `ELSE IF` 会综合出优先级电路，导致逻辑层数变多
5. 尽量不用 `reg = `，能用 `assign wire = ` 的就用
6. 由于是电路，所以一个 `wire` 或 `reg` 都只能被赋值一次，并且只能出现在一个块中，否则会导致`multi driver`（多驱动）
7. 在 `@always(*)` 中只用 `reg = `，在 `@always(posedge clk)` 中只用 `reg <=`
8.  需要自己给自己赋值的，必须使用 `<=`
9.  两种解决方法，防止 `if-else` 综合出 `latch`
       1. 组合逻辑中的 `if-else` 结构必须完整，每个变量赋值都必须同时出现在 `if` 和 `else` 中
       2. 需要出现在 `if-else` 中的变量一定要赋初值，然后用有优先级的 `if-else` 判断，时延较长
10. 只有两个分支的可以用三目运算符，多分支的最好还是用 `if` 或者 `case`
11. 善用 `generate` 生成一层组合逻辑
12. 逻辑门消耗：加法器 > 比较器 > 选择器
    因此先选择，再比较，再作加法