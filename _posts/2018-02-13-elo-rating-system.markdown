---
layout: post
title:  "Elo rating system and match making rating (MMR) system."
date:   2018-02-13 18:08:32 +0800
categories: mypost
---
# Introduction

Yesterday I wrote about how to reference equations correctly with Microsoft Office 2013 and 2016. Today I'll write about how to set up MathJax, a beautiful equation rendering engine written in JavaScript, which handles LaTeX and MathML (and ASCIIMathML) to your Jekyll site on GitHub (and everywhere else on the Web of course). I found a handful of descriptions about it ([this][mmr-distribution], [this][elo-rating] and [this][tobanwiebe] - [this][haixing-hu] could also be helpful), but those are not working for me. At the end, I got help from [here][pages-gem]. Thanks again, **hugomilan**! :)

# How to do it?
The recipe is simple.

* Go to your \_includes/head.html in your Jekyll folder structure.

* Put the following code snippet there (before the </head> tag).
    {% highlight html %}
       <head>
       ...
       <script type="text/x-mathjax-config"> MathJax.Hub.Config({ TeX: { equationNumbers: { autoNumber: "all" } } }); </script>
       <script type="text/x-mathjax-config">
         MathJax.Hub.Config({
           tex2jax: {
             inlineMath: [ ['$','$'], ["\\(","\\)"] ],
             processEscapes: true
           }
         });
       </script>
       <script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>
       </head>
    {% endhighlight %}

![elo-dis]({{ site.github.url }}/assets/2018-02/elo-dis1.jpg)
![elo-dis]({{ site.github.url }}/assets/2018-02/elo-dis2.jpg)
![elo-dis]({{ site.github.url }}/assets/2018-02/elo-dis3.jpg)
![elo-dis]({{ site.github.url }}/assets/2018-02/elo-dis4.jpg)

* And you are done! These three lines will do three things. It will allow you to use equations through the entire site, everywhere you want. The second thing is, that you can use equations with auto numbering using two dollar signs. For example, this equation <code>$$ E = m\cdot c^2 \label{eq:mc2}$$</code> will look like the equation below and you can refer to it as <code>\ref{eq:mc2}</code> (which will render to this: \ref{eq:mc2}).

    $$ E = M\cdot c^2 \label{eq:mc2} $$

    But you can use inline equations too (this is the third thing), with one dollar sign, like this: <code>$ J(x) = \int{L(t, x, \dot{x}) dt} \$</code>. The equation above will render to this: $ J(x) = \int{L(t, x, \dot{x}) dt} $.

That's all folks! (For today.)

相信接触过竞技游戏的玩家对下面这个东西不会陌生


不管遭遇过怎样坑爹的匹配经历现在都先放下，这一次我想要给大家分享的就是这个匹配系统背后一些比较核心的东西：包括排位积分的计算方式—ELO积分的推导原理， ELO积分在竞技游戏中应用会遇到的困难，以及使用ELO积分的利和弊等等。



那么开始还是从理论基础开篇（可能有点晦涩，我尽量只讲公式的合理性，公式大概的一个推导过程，尽量不讲具体的公式换算）

1 ELO积分基本情况介绍
Ø ELO（埃洛积分）等级分的由来：

elo等级分是根据它的创立者埃洛教授名字命名的，最早用来评估国际象棋选手的实力。在埃洛以前，关于国际象棋选手排名体系已经有两套（分别是欧洲的INGO系统和美国象棋协会提出的肯尼斯·哈克尼斯等级分系统），但是这两套系统在选手实力的表现上都不够直观，所以最后才被由ELO提出的等级分系统取代！

关于ELO分的基本印象只要记住下面三点：最早在国际象棋评分中出现，作者是搞物理的埃洛，然后公式的来源是选手胜率评估（这点我会在后面详细解释）




2 公式的理论解释
ELO在解释选手对战情况时，分为三个部分。第一部分是根据战力差来计算对战胜率情况；多少分差对应多少胜率； 第二部分是选手根据对手情况和战斗结果所表现出来的水平分；第三部分是选手完成对局后实际获得的积分增长。

2.1 预期胜率计算公式
大概解释一下ELO积分预期胜率计算公式为什么是这么一个样子：




其中D代表两个选手的排位分差，大于零表示对手更强。p(D)代表战胜分差为D的玩家的概率

这公式怎么来的？

我们知道，强手未必总是胜过弱手，实际上任何一名选手的即时表现都应该是符合正态分布的（围绕某一个水平上下波动）。

每个选手的表现都应该符合正态分布函数为：




其中U代表选手的平均水平，δ代表稳定性（表现分值的方差）。


这东西有点茫然，画个曲线分布图来形象解释一下：

选手A：平均表现应该是1800分，但是不是很稳定，有200分左右的上下波动（可以出现更大的偏移，但是概率极小）；

选手B：平均表现2000分，选手状态稳定，波动分值在100左右；

他们的表现符合下面两个正态分布：B和A的表现大概如下




那么在有了概率分布了，我们可以选取任何一个特殊情况分析（这里假设是1880分处）B能战胜A的条件：


结合概率分布函数图像，以1880分为分割点B能战胜A的条件是：A得1880时，B的表现分高于1880。

那么我们只需要得到A得分等于1880的概率，B得分高于1880的概率，就能计算出1880分处B获胜的概率；

假设A得分等于1880的概率为Pa(1880)

假设B得分大于1880的概率为Pb(＞1880)

那么在1880处B能战胜A的概率Pba(1880)=Pa(1880)* Pb(＞1880)


在统计学中，A表现等于1880的概率可以直接通过概率分布函数得到；B得分＞1880则可以通过对每一个点的概率进行累加（积分求和）得到。

B能战胜A就是B的得分一定要在A的右边（比A得分高）。即A、B的散点分布一定是下面灰色和红色斜线两个区域。


而像1880这样的分割点一共有无数个，所以B能战胜A的最终概率，应该是这无数个分割点B战胜A的概率的一个求和，也即是积分运算。

综合这几个步骤，就可以得出那个奇奇怪怪的积分公式ELO积分。




不过考虑到应用方便，实际的运用中并没有直接用它，而是利用了最小二乘，得出了和它函数趋向相近的另外的一个公式，这也是我们实际运用该公式时常用的公式：


到这里我已经尽量的通俗的去解释ELO公式是怎么来的的（这个公式不是随便写的，是符合统计规则的）有了这个公式后，我们就能根据自己和对手的分差来进行胜负模拟：如下（这张表后面的计算中需要用到）




2.2 表现等级分计算公式
这是ELO体系中比较重要的一个公式，他的形式是这样的：


其中：Rp是玩家自己的表现分；Rc是对手排位分，如果对手不止一个就要取平均值；D(p)是胜率计算函数的反函数，p表示参考局数内玩家的胜率；（如果只看一局的表现，那么他的胜率就只有100%和0%）举个例子如下：


D（p）的得分查胜率和积分差对应表即可，也可以根据函数去计算：（求下面这个函数的反函数即可，不再推导了）




2.3 积分迭代公式（这个才是我们实际要用到的公式）
上面的公式只能反应玩家一定局数内的表现水平，但是实际上我们用来衡量玩家水平高低的评价分数应该是无数个短局表现分的综合结果，所以并不能直接用短期内的表现分直接取取代玩家原有的分值。这就是经过修正后的ELO体系积分迭代公式，形式如下：


其中：

Rn是玩家比赛结束后的新的排位分值；

Ro是比赛前玩家的排位分

K是一个加成系数，由玩家当前分值水平决定（分值越高K越小）


G是玩家实际对局得分（赢得1分，输得0.5分）

Ge是原排位分基础上玩家的预期得分（根据胜率来算，多名对手情况就是和多名对手对战的胜率求和）

举个例子来说明该公式的运用：


3 ELO积分的应用分析

3.1 特点（在实用时要十分注重）
任何的算法体系都有优缺点，ELO积分也是，而且ELO积分的优缺点都还比较明显。

离散性： ELO积分的计算不需要获取太多的信息，只需要知道三个要素即可对积分进行迭代：选手赛前积分，对手赛前积分，比赛结果。


初期不精确性：ELO积分在达到合理之前需要一个收敛过程，比如一个2000分玩家玩小号，遇到的对手大概都是从1200分开始；这时候ELO分是不能精确反应他的真实实力的，这需要一个过程，也就是ELO积分的收敛过程


对所有比赛一视同仁：ELO分只会根据同一评分体系下的积分进行迭代，积分增长情况和赛事重要性无关。




3.2 在竞技游戏中的适用性
3.2.1 比赛重要性差异适配
和网球，象棋等相比，网络游戏中所有的玩家大多都是为了娱乐和休闲，所以在比赛和比赛之间的重要性差异，不会影响玩家在游戏中的评分，这一点恰恰规避了ELO积分将所有比赛看做一样的缺点。

3.2.2 收敛期解决方案
前面在算法的特点中我有提到，ELO积分有一个比较显著的特点就是需要一个收敛过程来对其进行大致的定位。

这一点在游戏中是有很好的体现的：（LOL引入定级赛就是出于这方面的考虑）


除此之外，为了尽量缩短玩家的收敛周期（达到真实水平所用的场次），LOL和王者荣耀都采取了段位继承机制

段位继承的实现并不困难，只需要在定位期加分算法中考虑到玩家上赛季得分就可，具体用什么比例，看需求而定。




3.2.3 1对1到多对多的拓展
ELO积分初期是针对象棋选手而定的，所以考虑的都是象棋选手1V1的情况。但是在5V5排位赛中，有几个要素是不能直接用定义来计算的。


这个公式中Ro，K,G值是不用变化的，但是Ge的计算是需要特殊处理的。

Ge的计算需要用到对手信息（Ge怎么计算要回到前面的公式和算法去），在LOL，王者荣耀等游戏中玩家一局游戏遇到的对手等级分实际上都不一致。所以，这时候在对手排位分到底用多少需要商榷。这里我提几个可行的方案（并没有经过验证）

方案1：直接用对手的分值平均值作为对手等级分

该方案实际上有明显的缺点：显然出现带妹队伍的时候，如果按照对手分取均值的话，妹子的分值就明显增长过快了。所以5V5情况下，对手的分值应该要加入自身的要素（因为在己方团体中每个个体对排位总分的贡献是不一致的，所以要引入权重的概念。）


方案2：于是我又提出了一个新的方案，在5V5对战中，计算玩家A的迭代积分时，玩家A的对手分值应该这样去算：


这样Ge中要用的自己和对手的分值差D就应该这样计算：




4 思考：排位分直接用ELO分显示有什么不好？
为什么LOL要用段位来替代之前的ELO积分？

为什么王者荣耀在此基础上做了更大的变化？

在英雄联盟和王者荣耀中分值还有用吗？

答：

1、 ELO分不会消失，从显示变成了隐藏，但是匹配对手时还是以此为基础来寻找对少的。（将玩家真实水平隐藏）

2、 因为ELO积分对玩家水平的精确指示会打击玩家的信心，分值收敛结束后，再进行更多的对局几乎不会有任何提高，如非自我进阶。（一入天梯深似海，从此DOTA是路人）


3、 段位的出现是对人性考量的结果！永远不要忘记MOBA类游戏盛行的至理：超神都是自己的！锅都是队友的！上不去是队友都是傻逼的！所以不需要在分值上对其水平进行太过精确地显示。你看，下面这样就舒服了！


4、 ELO积分不会消失，他对选手水平的精确指示性是用其作为匹配的重要参考依据保证玩家的游戏体验的根本。


段位并不是玩家真实水平的反应，段位对努力的选手有很多的鼓励，所以用段位来给玩家匹配对手会让女性玩家怀疑人生。（她们的真实水平实际并没有段位显示的那么高）

所以在进行匹配的时候，还是需要以ELO分为准！！


至于代练导致ELO分值上涨，然后开启连跪模式的亲们，这就十分抱歉了~~这种情况没得解！

Mathematical details
Performance isn't measured absolutely; it is inferred from wins, losses, and draws against other players. Players' ratings depend on the ratings of their opponents, and the results scored against them. The difference in rating between two players determines an estimate for the expected score between them. Both the average and the spread of ratings can be arbitrarily chosen. Elo suggested scaling ratings so that a difference of 200 rating points in chess would mean that the stronger player has an expected score (which basically is an expected average score) of approximately 0.75, and the USCF initially aimed for an average club player to have a rating of 1500.

A player's expected score is their probability of winning plus half their probability of drawing. Thus an expected score of 0.75 could represent a 75% chance of winning, 25% chance of losing, and 0% chance of drawing. On the other extreme it could represent a 50% chance of winning, 0% chance of losing, and 50% chance of drawing. The probability of drawing, as opposed to having a decisive result, is not specified in the Elo system. Instead a draw is considered half a win and half a loss.

If Player A has a rating of {\displaystyle R_{A}} R_{A} and Player B a rating of {\displaystyle R_{B}} R_{B}, the exact formula (using the logistic curve)[12] for the expected score of Player A is

{\displaystyle E_{A}={\frac {1}{1+10^{(R_{B}-R_{A})/400}}}.} E_{A}={\frac {1}{1+10^{(R_{B}-R_{A})/400}}}.
Similarly the expected score for Player B is

{\displaystyle E_{B}={\frac {1}{1+10^{(R_{A}-R_{B})/400}}}.} E_{B}={\frac {1}{1+10^{(R_{A}-R_{B})/400}}}.
This could also be expressed by

{\displaystyle E_{A}={\frac {Q_{A}}{Q_{A}+Q_{B}}}} E_{A}={\frac {Q_{A}}{Q_{A}+Q_{B}}}
and

{\displaystyle E_{B}={\frac {Q_{B}}{Q_{A}+Q_{B}}},} E_{B}={\frac {Q_{B}}{Q_{A}+Q_{B}}},
where {\displaystyle Q_{A}=10^{R_{A}/400}} Q_{A}=10^{R_{A}/400} and {\displaystyle Q_{B}=10^{R_{B}/400}} Q_{B}=10^{R_{B}/400}. Note that in the latter case, the same denominator applies to both expressions. This means that by studying only the numerators, we find out that the expected score for player A is {\displaystyle Q_{A}/Q_{B}} Q_{A}/Q_{B} times greater than the expected score for player B. It then follows that for each 400 rating points of advantage over the opponent, the expected score is magnified ten times in comparison to the opponent's expected score.

Also note that {\displaystyle E_{A}+E_{B}=1} E_{A}+E_{B}=1. In practice, since the true strength of each player is unknown, the expected scores are calculated using the player's current ratings.

When a player's actual tournament scores exceed their expected scores, the Elo system takes this as evidence that player's rating is too low, and needs to be adjusted upward. Similarly when a player's actual tournament scores fall short of their expected scores, that player's rating is adjusted downward. Elo's original suggestion, which is still widely used, was a simple linear adjustment proportional to the amount by which a player overperformed or underperformed their expected score. The maximum possible adjustment per game, called the K-factor, was set at K = 16 for masters and K = 32 for weaker players.

Supposing Player A was expected to score {\displaystyle E_{A}} E_{A} points but actually scored {\displaystyle S_{A}} S_{A} points. The formula for updating their rating is

{\displaystyle R_{A}^{\prime }=R_{A}+K(S_{A}-E_{A}).} R_{A}^{\prime }=R_{A}+K(S_{A}-E_{A}).
This update can be performed after each game or each tournament, or after any suitable rating period. An example may help clarify. Suppose Player A has a rating of 1613, and plays in a five-round tournament. He or she loses to a player rated 1609, draws with a player rated 1477, defeats a player rated 1388, defeats a player rated 1586, and loses to a player rated 1720. The player's actual score is (0 + 0.5 + 1 + 1 + 0) = 2.5. The expected score, calculated according to the formula above, was (0.51 + 0.69 + 0.79 + 0.54 + 0.35) = 2.88. Therefore, the player's new rating is (1613 + 32(2.5 − 2.88)) = 1601, assuming that a K-factor of 32 is used.

Note that while two wins, two losses, and one draw may seem like a par score, it is worse than expected for Player A because his or her opponents were lower rated on average. Therefore, Player A is slightly penalized. If Player A had scored two wins, one loss, and two draws, for a total score of three points, that would have been slightly better than expected, and the player's new rating would have been (1613 + 32(3 − 2.88)) = 1617.

This updating procedure is at the core of the ratings used by FIDE, USCF, Yahoo! Games, the Internet Chess Club (ICC) and the Free Internet Chess Server (FICS). However, each organization has taken a different route to deal with the uncertainty inherent in the ratings, particularly the ratings of newcomers, and to deal with the problem of ratings inflation/deflation. New players are assigned provisional ratings, which are adjusted more drastically than established ratings.

The principles used in these rating systems can be used for rating other competitions—for instance, international football matches.

Elo ratings have also been applied to games without the possibility of draws, and to games in which the result can also have a quantity (small/big margin) in addition to the quality (win/loss). See Go rating with Elo for more.

See also: Hubbert curve


[mmr-distribution]: https://zhuanlan.zhihu.com/p/25207214
[elo-rating]: https://zhuanlan.zhihu.com/p/28190267
[wikipeida-elo-rating-system]: https://en.wikipedia.org/wiki/Elo_rating_system

[mmr-distribution]: https://zhuanlan.zhihu.com/p/25207214
[elo-rating]: https://zhuanlan.zhihu.com/p/28190267
