---
title: 参与HackerRank竞赛WeekOfCode26
date: 2016-12-05 20:19:17
tags: [HackerRank, OJ, coding, dynamic programming, python, nim game]
---
[Week of Code 26](https://www.hackerrank.com/contests/w26/challenges) 平生第一次参加OJ竞赛，一周的比赛时间到北京时间今天下午16点截止，每天放出一道题，我也度过了充满激情的充实七天，工作日的晚上以及周六日的全天，基本都面对着笔记本，码代码或是思考如何码。HackerRank界面简洁，易用性很高，以及排名机制什么的成功激起了我做题的欲望。

![最终名次](https://github.com/veslam/ImagesForBlog/raw/master/res/20161205_01_HackerRankWeekOfCode26.png)
最终成绩是 [__521__ out of 6960](https://www.hackerrank.com/results/w26/veslam) 
虽然看起来还行，名列前10%，但考虑到分母里有很多新手或是抱着参与着玩玩的心态的人，其实成绩差强人意，距离榜首的各位满分更是差距一百多分。
然而比赛的意义并不只是排名，就像高三刷题一样，每个题涉及一或多个关键的知识点。下面我会具体总结一下知识方面，确实学到蛮多。最后记录下首次参赛的体会心得。
我的代码也会上传到GitHub，[链接在这里](https://github.com/veslam/HackerRankPython/tree/master/Contests/WeekOfCode26)。由于还有多道问题没有高效的解决，甚至没有解法，所以我会在找到尽量标准的答案后更新。

---
##### 题目解析 #####
1. [Army Game](https://www.hackerrank.com/contests/w26/challenges/game-with-cells)
很简单的送分题。先考虑X方向覆盖，需要数目为(列数/2)上取整，Y方向同理。两者独立，答案为两者乘积。
2. [Best Divisor](https://www.hackerrank.com/contests/w26/challenges/best-divisor)
题意为，找_n_的约数中，各位数值相加最大的条件下，本身值最小的数。
从小到大遍历所有约数，计算各位和并更新记录就行。有优化的可能？已知各位和，枚举数值？
3. [Twins](https://www.hackerrank.com/contests/w26/challenges/twins) （从这题开始就有难度了，此后我就没得到过满分……）
两个数被称为_Twins_，当满足2个条件：1) 均为质数 2) 值差为2。求[_n_, _m_]区间的_Twins_组数。
大体思路较明确，第一步肯定要判断[_n_, _m_]区间每个数是否为质数，第二步遍历区间。
感觉不可能直接从_n_判断起，所以我还是采用从_1_筛查到_m_的方法，创建bool数组_isPrime[m+1]_，如果检查到元素_i_，_isPrime[i]==True_，则_i_是质数，同时_isPrime[所有i的倍数]=False_。
问题在于，所有i的倍数都要写一遍吗？答案是不用的。
i x (1, 2, ... k)其实分为两部分，i x (1, 2, ... i-1) 和 i x (i, i+1, ... k)，第一部分一定已经在之前的筛查中被标记过，所以只需对于第二部分标记就行啦。
这就是[Sieve of Eratosthenes](https://en.wikipedia.org/wiki/Sieve_of_Eratosthenes)算法。
![图示](https://github.com/veslam/ImagesForBlog/raw/master/res/20161205_02_Sieve_of_Eratosthenes_animation.gif)
收获：标准答案里用i*i<n而不是i<sqrt(n)来限制i小于n的平方根。
4. [Music on the Street](https://www.hackerrank.com/contests/w26/challenges/street-parade-1)
换个方法描述题意就是，数轴被_n_个点分割为_n+1_个区间，拿一长度为m的线段贴合数轴，求线段起始位置，使得线段覆盖的所有区间（若部分覆盖则只算覆盖部分）的长度，都介于[_Hmin, Hmax_]
情景想象出来了，也就得出了等价的判断方法。如图，绿色区间能放下线段即可。
![](https://github.com/veslam/ImagesForBlog/raw/master/res/20161205_03_MusicStreet.png)
宽度小于Hmin的区间，情况简单，直接涂红。
宽度大于Hmax的区间，绿色部分要从左右分别侵入Hmax长度，表示线段可以覆盖到这两部分。
同理，绿色部分还要侵入最左侧点向左Hmax和最右侧点向右Hmax。
5. [Satisfactory Pairs](https://www.hackerrank.com/contests/w26/challenges/pairs-again)
给定n，求[Diophantine equation](https://en.wikipedia.org/wiki/Diophantine_equation) ax+by=n 有正数解的(a,b)组合数目。学到新定理：
1) 二元一次丢番图方程有解 <=> n % gcd(a,b) == 0
2) 若有解，记 d = gcd(a,b)，解的间距为(b/d, a/d)。
然而判定1)不能保证解为正数，目前思路是在(x=0,y)附近（b/d, a/d）区域找是否有正数解。然而超时，感觉肯定有更好的方法。
6. [Hard Homework](https://www.hackerrank.com/contests/w26/challenges/hard-homework)
给定n，x+y+z=n的前提下，sin(x)+sin(y)+sin(z)最大值？
据说是动态规划。发现讲动态规划很好的帖子：[什么是动态规划？动态规划的意义是什么？ - 回答作者: 徐凯强 Andy](http://zhihu.com/question/23995189/answer/35324479)
然而可能是我没有找到正确的状态定义，动态规划的复杂度并未降低，待解决。
7. [Tastes Like Winning](https://www.hackerrank.com/contests/w26/challenges/taste-of-win)
求满足下面条件的[Nim Game](https://en.wikipedia.org/wiki/Nim)先手获胜棋局总数：1) 各堆石子数各不相同，2) 堆最大石子数2^m。
[一篇很清晰的介绍](https://www.zhihu.com/question/29910524/answer/46075905) [lecture](https://www.mathcamp.org/gettoknowmathcamp/academics/archives/Alfonso-CGT-lectures.pdf) 百度百科中摘录定理：
1) P-position：面对者输 | N-position：面对者赢
(1).无法进行任何移动的局面（也就是terminal position）是P-position；
(2).可以移动到任意P-position的局面是N-position；
(3).所有移动都导致N-position的局面是P-position。
2) (Bouton's Theorem) 一个NimGame的局面(a1,a2,...,an)是P-position <=> a1^a2^...^an = 0
不会做…… 目前找到的最相近的参考：[POJ 2975 Nim 题解](http://www.hankcs.com/program/algorithm/poj-2975-nim.html)

---
最后是一点感想
+ 理解问题并有一个解题思路只是最基本要求，高一些的要求是实现最佳算法，之后还要针对数据范围进行优化。
+ 从分数来看，前面的几道题真不用纠结，加起来还没有后面多。
+ 看自己名次的变化过程也是挺有趣的。HackerRange在题目放出一天后才用完整的TestCases测试，此前只要你把已放出的TestCases全部通过，就会暂时得到满分。这就会造就一个分数虚高的时间段，看起来很爽，我曾经到过129/6644？然而最后还是会被打回原形。
Contest最终截止时间到后，还有一个最终名次的计算调整过程，我前进了大概160+，估计是一些人的虚高被挤掉了。。。

