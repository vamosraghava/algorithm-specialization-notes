# Introduction / 简介

> update: 2017/12/05

求解：两个n位整数相乘

输入：两个n位的整数x与y

输出：两者的乘积

解决方案：

1. Naive方法：确定基本运算，即y的每一位都要与x中的n位相乘，就是n次基本运算了，乘积十位上的数字还要进上去，这就涉及一些额外的加法运算，但是总之在任何情况下，基本运算的总量最多是2n。同理，得到每一部分乘积最多都需要2n步运算，而部分乘积的总次数为n，即要做$2n^2$次基本运算。然而还没有结束，还要把这些部分乘积累加起来，才能得到最终结果，这一步的求和运算量最多也是$2n^2$次，因此计算两个n位整数的乘积的基本运算操作量为$4n^2$，是输入长度n的二次函数。

2. Karatsuba方法：利用分治的思想，把x表示为ab，y表示成cd，即a=56，b=78，c=12，d=34。通过以下前三个递归步骤，利用$x\cdot y = 10000\cdot a\cdot c + 100\cdot a\cdot d + 100\cdot b\cdot c + b\cdot d$这一思想来求得最终的结果。可以发现，第三步化简之后就是ad+bc，但是在具体操作时不是分别计算ad和bc的乘积再求和，而是使用化简之前的表示，先各自求得a与b之和、c与d之和，然后求积，最后减去已知的ac与bd。这样是为了减少一次递归次数，毕竟加减法才是最基本的运算。

   形式化的方式来归纳以下，就是：

   - $x = 10^{\frac{1}{2}}\cdot a + b$，$y= 10^{\frac{1}{2}}\cdot c + d$
   - $x\cdot y=10^{n}a\cdot c + 10^{\frac{1}{2}}(a\cdot d+ b\cdot c) + b\cdot d$，$n=4$ ($\star$)
   - 递归计算ac，ad，bc，bd，然后代入上面的星式求得最终乘积
   - 为了减少一次递归运算，不需要单独计算ad与bc的乘积

   ![](http://7xwggp.com1.z0.glb.clouddn.com/karatsuba.jpg)

> 对于一个优秀的算法设计者而言，最重要的原则就是拒绝满足。
>
> Perhaps the most important principle for the good algorithm designer is to refuse to be content.
>
> The algorithm design space is surprisingly rich!
>
> 算法的设计空间，比我们想象中的要广阔得多！

讲解模式：

1. 确定输入与输出
2. 给出解决方案，即算法，使得输入转化为输出

# About the course /关于这门课程

> update: 2017/12/06

课程涵盖了5个话题：

1. 设计与**分析算法性能**所需要的基本知识，比如大O表示法
2. **分治算法**的设计与分析，使用场景很多，比如上一节课提到的Karatsuba算法，还有其他的比如排序、矩阵乘法等，需要分析类似递归算法的运行之间
3. **随机化算法**的设计与分析，涉及到快速排序、图分解、哈希等
4. **图论分析**的基本知识，涉及计算连通信息量、最短路径、社交网络的结构等
5. 基本**数据结构**的实现与运用，涉及堆、平衡二叉搜索树、哈希表及其变种，比如布隆过滤器等

后续课程可能会涵盖的话题：

1.  **贪心**算法，涉及最小生成树、调度问题和信息编码理论
2.  **动态规划**算法，涉及基因序列和社交网络中的最短路径
3.  **NP完全**问题，涉及是什么以及解法
   - 能够解决特殊问题的快速算法
   - 高效的有可证效率的回溯算法
   - 具有指数时间复杂度的算法，本质上会比暴力搜索优化

从这门课学到什么：视频里说了不少，我挑我最感兴趣且觉得最重要的几点说说

1. Become a better programmer.虽说读博对代码技巧要求未必很高，但还是希望自己除了学术分析、写作之上，有扎实的代码功底。毕竟还是希望自己毕业之后能去工业界待一段时间的。
2. Shapen mathematica analytical skills.这是实实在在我需要提神的一项技能！不仅老板指出来了，我自己也深有感触，写paper时构造不出定理与证明，实在是一大败笔！
3. Explain why things are the way the are, why we analyze the algorithms in the way that we do. 除了设计一个高效的算法，更重要的是你需要能够给别人讲懂，为什么是这样设计，为什么用这种方法来分析。这就回到上一点，良好的算法思路需要扎实功底的数学分析，这样才能充分理解！

# Divide and Conquer / 分治

## Merge Sort ：Example/ 归并排序：示例

> update: 2017/12/07

为什么在这里要讲解Merge Sort？有以下五点原因。

1. Merge sort是一个著名的、古老但是很有用的排序算法，现在已经被列入许多标准库中了
2. Merge sort完美体现了分治的思想：把一个大问题分解为多个小问题，然后递归地解决小问题，最后合并小问题的求解结果，比Selection / Insetion / Bubble sort算法更直观更有优势
3. Merge sort可以为学生的未来课程做更好的定位，即calibrate your preparation，后续的算法讲解会越来越复杂，所以这是一个很好的热身，来帮助我们判断是否适合这个课程
4. Merge sort帮助看清，分析算法与分析其他事物有所不同，需要在分析之前做假设性前提，分析worst-case，然后采用asymptotic analytics渐近分析法来观察算法效率的增长
5. Merge sort是利用Recusive-tree递归树来分析的，这是一个Master method

然后简要讲解了Merge sort是解决什么问题的？当然是解决乱序数组的排序问题啦。

输入：n个无序数字，假设没有重复数

输出：n个有序排列的数字

处理：把输入的数分成两半，先递归地解决左半部分，再解决右半部分，最后整合出结果，如下图所示，第一步可以想象成在递归调用之前先把左右两部分各自拷贝到新的数组中。

![](http://7xwggp.com1.z0.glb.clouddn.com/merge_sort.png)

## Merge sort：pseudocode / 归并排序：伪代码

> update：2017/12/08

伪代码：不管归并排序的子程序具体是如何实现的，假设子程序已经存在了，那就直接合并。所以Merge sort的伪代码就比较明了，如下三步。

1. 递归地对输入数组第一半子数组进行排序
2. 递归地对输入数组第二半子数组进行排序
3. 将两个排好序的子数组进行合并

递归算法需要一个基准，就是当输入为什么的时候，算法就得停止了，返回一个结果。那么在排序中，这个基准就是子数组中只剩下0或者1个数字的时候，不需要再进行计算，直接返回这个数字就可以了。算法实现的细节视频直接忽略了，比如如果数组长度为奇数钙怎么没办？也不会给出递归排序的具体细节，比如在递归调用中，如何把子数组的值返回给函数？这个视频要讨论的是抽象出来的有关算法的概念！

比较hard的部分：归并部分。下面给出归并部分的伪代码。

![](http://7xwggp.com1.z0.glb.clouddn.com/merge_preudocode.png)

分析归并排序的运行时间：从直观上看，应该从一个调试者的角度来考虑算法的运行，也就是说算法运行的时间就是所执行操作的数量，可以理解为实际执行代码的行数。虽然这是一个复杂的问题，但是这次视频忽略了递归相关的操作数量，只考虑归并操作。初始化有2步，进入for循环，每一次迭代有3步，然后还有循环本身的递增，即每一次迭代有4步。把这些加到一起，就得到归并操作的运行时间。给定一个有M数字的数组，最多执行$4\cdot M+2$步操作。这个上界可以放宽到$6\cdot M$，因为可能你要考虑循环递增数与总长度的比较，这类小细节操作。但这不是重点。分析Merge sort的主程序会更复杂，因为它在不断调用自身，所以需要分析递归调用的次数，这个次数是呈**指数级**增长的。*现在还有一个矛盾*，那就是进行递归调用时，输入数组会不断的减小，每次都是之前的一半大小。一方面是子问题的分裂造成的膨胀， 而另一面又是子问题会越来越小，二者之间形成了牵制 ，要解决这二者之间的矛盾就要取决于是什么在驱动Merge sort。后续视频会给出证明，这里先给出一个结论，就是算法的总步骤不超过$6\cdot N\cdot log N + 6\cdot N$。其他类似冒泡排序算法的总步骤是$N^2$，这两个的比较可以看下图，N越大，优势就越明显。

![](http://7xwggp.com1.z0.glb.clouddn.com/runtime.png)

## Merge sort：analysis / 归并排序：运行时间分析
> update：2017/12/09
>

这节课的视频是分析Merge sort的运行时间，用数学方式证明：递归Merge sort算法对给定的包含n个数字的数组进行排序，输出有序数组总共需要$6\cdot n\cdot log_{2}n + 6\cdot n$次操作。

证明方法是使用recursion tree递归树，在树结构中写下Merge sort所做的全部工作，一个节点每次递归调用就创建两个孩子节点，如下图所示。根节点root是首次对Merge sort的调用，这一层称作level-0。level-1对应root接下来的两次递归调用，输入是原数组的一半。以此类推，直到最后子数组里面数字的个数为0或者1。显然，叶子节点所在的层是level-$log_{2}n+1$。

![](http://7xwggp.com1.z0.glb.clouddn.com/recursion_tree.png)

确定好树的深度，需要计算每层的操作数量。首先回答两个问题：

1. 对于给定的第j层，有多个sub problem？
2. 对于第j层的每个sub problem，输入子数组的size是多少？

答案也显然，第j层有$2^j$个sub problem，每个sub problem的输入size是$\frac{n}{2^j}$。整个递归树的总操作数量是每层的操作数量之和。由于递归调用本身可以忽略不计，因此只考虑merge操作里的操作数量。分析如下图，第j层的merge次数是$2^{j}\cdot 6\cdot \frac{n}{2^j}$，其中$6$在上一节课的视频中分析过。最后，总共有$log_{2}n+1$层，所以最终结果是$6\cdot (log_2n+1)$，证明结束。

![](http://7xwggp.com1.z0.glb.clouddn.com/operation_time.png)

## Guiding principles for analysis of algorithms
> update：2017/12/11
>

这一节课的视频是回过头来，介绍算法分析的三项指导原则，也可以说是三项假设，从而帮助我们分析推到算法，并给出"fast"算法的定义。

1. 只考虑worst-case。和avarage-case和benchmark analytics相反，最坏情况的分析对输入没有要求，不需要domain knowledge。
2. 忽略小的常数因子。原因比较简单，第一，简化分析；第二，就这个课程而言，纠结于常数因子对算法的影响没有意义，毕竟常数因子对算法的影响很大程度上还取决于硬件结构、编译器、使用的编程语言、程序员编码习惯等。
3. 关注大规模输入asymptotic analytics。当输入规模逐渐增大至无限，分析算法的性能。关注小规模输入没有意思，以排序为例，也许只对100个数字排序，Merge sort的对数级运行时间会高于其他平方级算法。但是某个临界点过后，对数级的运行时间优势就越来越明显。

<!-- more -->



