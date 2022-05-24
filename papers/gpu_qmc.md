# A Design of GPU-Based Quantitative Model Checking[^0]

> YoungMin Kwon & Eunhee Kim 

## Abstract

In this paper, we implement a GPU-based quantitative model checker and compare its performance with a CPU-based one. Linear Temporal Logic for Control (LTLC) is a quantitative variation of LTL to describe properties of a linear system and LTLC-Checker is an implementation of its model checking algorithm. In practice, its long and unpredictable execution time has been a concern in applying the technique to real-time applications such as automatic control systems. In this paper, we design an LTLC model checker using a GPGPU programming technique. The resulting model checker is not only faster than the CPU-based one especially when the problem is not simple, but it has less variation in the execution time as well. Furthermore, multiple counterexamples can be found together when the CPU-based checker can find only one.

## Glossary

* MPC: Model Predictive Control

    有一个视界（Horizon），决策时最小化 $[t,t+H]$ 内 Output 和 Reference 的 Difference，但只保留第一步

* LTI: Linear Time Invariant (System Model)

    $$M=<U,Y,X,A,B,C,D>$$
    
    $U$: input, $X$: state, $Y$: output.

    $${\mathrm x}(t+1)=A\cdot {\mathrm x}(t) + B\cdot{\mathrm u}(t)$$

    $${\mathrm y}(t+1)=C\cdot {\mathrm x}(t) + D\cdot{\mathrm u}(t)$$


* LTLC: Linear Temporal Logic for Control 时序逻辑
    
    在一般 Logic 上加入了时间谓词

## Key Idea

LTLC + Horizon Constraits 可以~~去掉时序约束~~，将时间有限化，然后展开约束中就不存在时间谓词了。因为是线性系统，最后写成 DNF，里面每个 conjunction 是个线性规划。

线性规划就可以在 GPGPU 上跑单纯形了。

搜索策略：BFS+DFS，在 Büchi automaton 上搜索。

整体 BFS，但是从节点扩展路径选择 DFS 深度 $l$ 的所有路径，交 GPU 验证保留 satisfiable 的。

## Tricks

* 为了避免内存瓶颈，Memcpy 用 stream style。`#stream++, stream_size--` 来优化线性规划大小不一致的 overhead。
* 限制 simplex 迭代次数，没有算完扔进下一次迭代

## Questions

实际上 LTLC 转线性规划时利用的是 Büchi automaton (for Restricted Second Order Arithmetic，详见[^1])，具体算法可以看 [^2]

1. 真实 program 能否转 Büchi automaton 是个问题
2. 具体搜索深度仍有限，例子是 $d=40, t\approx 1 \min$，对于字符串处理的循环，约束过长，是否会裂开？
3. ~~**Linear**，异或是不是裂开了~~，[好像可以弄](https://yetanothermathprogrammingconsultant.blogspot.com/2016/02/xor-as-linear-inequalities.html)，0-1 整数规划，（单纯形仍适用？）
    
    确实裂开了，0-1 LP NP-hard

4. 主要问题在于，符号执行的约束 (SMT) 能不能转 LP？如果能，复杂度能不能接受

5. 全局变量的处理？扩大约束规模？

## Reference

[^0]: <https://link.springer.com/chapter/10.1007/978-3-030-67067-2_20>

[^1]: On a Decision Method in Restricted Second Order Arithmetic

[^2]: <http://jst.tsinghuajournals.com/article/2014/1000-0054/1000-0054-54-2-281.html>

