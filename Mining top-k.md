# Mining top-K frequent itemsets from data streams

## 1.介绍
+ 过去大部分的研究致力于提供提供一种保证，使得出的频繁模式其频率的错误率不超过$\epsilon$
但如果不能首先找到一个核实的阀值s，这种保证就没有什么价值。

+ 这个能给出合适个数频繁模式的阀值s，在不同的数据集甚至同一个数据集的不同子集当中有所不同，而且按一个有序的两级变化。
+ 小项集更容易多次出现，所以频率更高是理所当然的，所以l-itemset 的frequency 阀值应该随l的不同而不同

+ 按照项数由1到L，项集分成不同等级，找出每个等级的前频率在前K个的项集。“第K个项集的频率就是这个基本项集的阀值s”
+ 最后把各个等级的前K个高频项集合并起来就得到top-K 高频项集。


这里我们让用户告诉我们想要的频繁模式的个数使多少？


## 2. Chernoff-based algorithm versus top-K lossy counting algorithm
> Chernoff (英)切尔诺夫 边界是应用马尔可夫（Markov）不等式得出的。
> $Pr\{ |\tilde{x} - x | \geq x \gamma \} \leq 2e^{\frac{-nx\gamma^2}{2}}$
>

### Chernoff-based algorithm
> 对于一个随着时间流失高速无限增长的数据流，如何通过分析一个时间窗口内的数据获得一些规律并保证这些结论在一定的概率程度上可以代表整个数据流的规律。

+ 由错误率阀值$\epsilon$ 和可信度$\delta$ 控制。数据到来的时候自动确定$\epsilon$的值。
$\delta$用来确定可信度，确保得到的是频率前K的项集。
+ 这个算法假设数据是独立性，但可以用3.3节的技术处理数据的依赖性。

+ 把项集分成两组：有希望组和不抱希望组

#### 3.1 分析：
主要考虑如何剪枝不会错误的剪掉top-k项集。
我们从切尔诺夫算法中获得一些保证，在下面的研究中我们发现第K个频繁项集的支持度基本上都大于$\frac{ln(2/\delta)}{2R}$

算法用到两个估计estimation：

+ 用一批数据得出的$s_K$估计整个数据流的s
+ 估计项集的频率
+ 每个估计的不准确都可以导致总体错误
    - Lemma 1：设定一个$s_K$ 可能误差的边界， 然后
    - Theorem 1，2：合并观察支持度的保证与上面的辅助定理一起给出全面的保证。


##### 辅助定理1:
**假设 $\epsilon_{SK} = \sqrt{\frac{2s_K \, ln(2/\delta)}{R}}.
\quad s_K \geq s + \epsilon_{s_K} \, 的概率最大为\delta$ **

**证明：**

+ $s_K$ is the K-th frequent itemset in a batch B of R transection.
+ We are interested in the case where $\mathbf{s_K > s}$
+ Let itemset Y be the K-th frequent itemset of the batch B. Hence $\tilde{s}(Y,R)=s_K$
+ Let $\epsilon_Y = \sqrt{\frac{2s(Y) ~ ln(2/\delta)}{R}}$
+ By the Chernoff bound, with probability $\leq \delta, \tilde{s}(Y,R) \geq s(Y) + \epsilon_Y$
+ Thus, with the probability $p_Y \leq \delta, \\
 s_K \geq s(Y) + \epsilon_Y \quad (3)$
+ **There are three possible cases for itemset Y:**
    1.  Y is the K-th frequent itemset in the entire data stream. Hence, $s(Y) = s$
        - with $p' \le \delta,  s_K \geq s + \epsilon_Y$
        - $s<s_K \Rightarrow \epsilon_Y < \epsilon_{s_K}$
        - So, with probability $p<p'<\delta, s_K \ge s + \epsilon_{s_K}$ _compared to a bigger one_
    2. Y is the one of the top K frequent itemsets, then $s(Y) \gt s$
        - Y is the K-th frequent itemset found in the batch.
        - There are K itemsets Z such that $\tilde s(Z,R) \ge s_K$
        - Let $\epsilon_Z = \sqrt{\frac{2s(Z) \, ln(2/\delta)}{R}}$
        - By the Chernoff bound, with probability $p_Z \leq \delta, \tilde{s}(Z,R) \geq s(Z) + \epsilon_Z ~ (4)$
        - **Subcases:1**
            - If at least one itemset Z in set A is none of the actual top K itemsets, then $s(Z) < s$
            $$\therefore with ~ p_Z \leq \delta, ~ \tilde{s}(Z,R) \geq s + \epsilon_Z$$
            - and $\tilde s(Z,R) \ge s_K$
            $$\therefore ~ 0\le p'\le p_Z\le \delta, s_K \ge s + \epsilon_{Z}$$
            - $\because s(Z)<s$ and $s<s_K$, $s(Z) < s_K$.
            $$\therefore \epsilon_Z < \epsilon_{s_K}$$
            $$\therefore ~ with ~ p, ~ p\le p'\le \delta, s_K \ge s + \epsilon_{s_K}$$
        - **Subcases:2**
            - If all the itemsets in A are the actual top K frequent itemsets in the entire data stream.
            - Then $\exists$ itemset Z in A which is the K-th frequent itemset in the entire data stream
            - $\Rightarrow s = s(Z)$
            - and $\tilde s(Z,R) \ge s_K$
            $$\therefore with ~ p', ~ 0\le p'\le \delta, s_K \ge s + \epsilon_{Z}$$
            - $\because s=s(Z)$ and $s<s_K$, $s(Z) < s_K$.
            $$\therefore \epsilon_Z < \epsilon_{s_K}$$
            $$\therefore ~ with ~ p, ~ p\le p'\le \delta, s_K \ge s + \epsilon_{s_K}$$
    3. Y is not one of the true top K frequent itemsets, then $s(Y) \lt s$  
        - If Y is not one of the true top K frequent itemsets, then $s(Y) < s$
        - From Eq.(3)  

+ 项集Y是batch B（每个批次有R个交易）里面的第K个频繁项集
+ 根据切尔诺夫边界：在概率小于$\delta$ 的情况下： $\tilde{s}(Y,R) \geq s(Y) + \epsilon_Y$


### top-K lossy counting algorithm 损失计数算法
+ 不假定数据的独立性
+ 如果$\epsilon$设置成小于第K个频率的项集的**支持度**的话，结果会很准确，如果大于的话，很多正确的值就会丢失。
+ 但设得太小则损耗计算能力和内存，设得太大会产生很多不正确的结果。
