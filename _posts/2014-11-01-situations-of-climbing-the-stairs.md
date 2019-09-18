---
layout: post
title: "爬楼梯和斐波拉契数列"
description: ""
category: [Algorithm, C++]
---


## 爬楼梯的选择

假设有一个台阶数为 M 的楼梯，上楼梯的时候，每一步只能上一级或者两级台阶，那么有多少种选择可以上去？

## 两种思路

1. 最直接的思路是穷举法：
	- 每次只走一个台阶算一种情况
	- 当台阶数大于 2 时（假设为 m），算出可能的一步走两个台阶的最大次数 n，然后计算从 1 到 n 次一步上两个台阶的时候，这些一步上两个台阶情况（假设为 k）在 k + m - k * 2 步数中的排列组合可能性，并相加。
	- 对于总台阶数为偶数时，还需要把所有步数都一次上两个台阶这种情况单独处理，因为这只算一种走法。
2. 数学归纳法。先看看有没有规律。
	- 当台阶数为 1 时，总共有 1 种走法。
	- 当台阶数为 2 时，总共有 2 种走法。
	- 当台阶数为 3 时，总共有 3 种走法。
	- 当台阶数为 4 时，总共有 5 种走法。
	- 当台阶数为 5 时，总共有 8 种走法。

很熟悉啊，这不就是斐波拉契数列吗？但是怎么解释呢。当有 m （m > 2）个台阶时，总的走法为 __f(m)__，那么由于每一步只能走一个台阶或者两个台阶，那么：


> 总的走法 = 最后一步上了一级台阶的走法 + 最后一步上了两级台阶的走法

而 “最后一步上了一级台阶的走法” 其实就是 __f(m - 1)__， “最后一步上了两级台阶的走法” 也就是 __f(m - 2)__, 所以：

> f(m) = f(m - 1) + f(m - 2)

## 编码实现

- 上面所说的第二种思路其实就是计算斐波拉契数列的第 m + 1 项。（斐波拉契数列从两个 1 开始。m 为台阶总数），我们先实现这个

{% highlight c++ %}

typedef struct MyStruct
{
	unsigned long long content[];
} ms, *pms;

unsigned long long Solution::SimpiestWay() 
{
	// 最后一步上了一级台阶就是content[i-1], 最后一步上了两级台阶就是content[i-2]
	// 所以 content[i] = content[i-1] + content[i-2]
	pms stairsPMS = (pms)malloc(stairs_ * sizeof(unsigned long long));
	if(stairsPMS == NULL)
	{
		printf("No enough memory when calculating " + stairs_);
	}
	stairsPMS->content[0] = 1;
	stairsPMS->content[1] = 2;
	for(int i = 2; i < stairs_; i++)
		stairsPMS->content[i] = stairsPMS->content[i-1] + stairsPMS->content[i-2];

	unsigned long long result = stairsPMS->content[stairs_-1];

	free(stairsPMS);
	return result;
}
{% endhighlight %}

这里使用柔性数组主要是为了动态测试不同台阶数的方便。

- 对于上面的第一个思路，我们需要知道排列组合的一些计算方法：

	![comb1][2]

	![comb2][3]

- 由于计算过程中可能会产生很大的值，为了避免 unsigned long long 越界，需要在循环中做处理，具体措施是在确定被除数后，每次相乘运算前检查是否可以无余数除以被除数，如果可以就先除再乘。

先看一下蛋疼的实现：

{% highlight c++ %}
unsigned long long Solution::CombinationWay()
{
	if(stairs_ == 1)
		return 1;

	if(stairs_ == 2)
		return 2;

	//全是1的情况
	unsigned long long result = 1;

	//可以有一次两个台阶的次数
	int countOf2 = stairs_ / 2;

	if(stairs_ % 2 != 0)
	{
		//分别计算从 1 到 countOf2  时，可能的情况并相加
		for(int currentCountOf2 = 1; currentCountOf2 <= countOf2; ++currentCountOf2)
		{
			//这种情况下总的步数
			int countOfSteps = currentCountOf2 + stairs_ - currentCountOf2 * 2;

			//只有一次一步两个台阶的情况
			if(currentCountOf2 == 1)
				result += countOfSteps;
			else
				result += internalCalculate(currentCountOf2, countOfSteps);

			//printf("result = %20llu, currentCountOf2 = %d, countOfSteps = %d\n", result, currentCountOf2, countOfSteps);
		}
	}
	else
	{
		//除去每次都两个台阶的情况
		//分别计算从 1 到 countOf2 - 1  时，可能的情况并相加
		for(int currentCountOf2 = 1; currentCountOf2 <= countOf2 - 1; ++currentCountOf2)
		{
			//这种情况下总的步数
			int countOfSteps = currentCountOf2 + stairs_ - currentCountOf2 * 2;

			//只有一次一步两个台阶的情况
			if(currentCountOf2 == 1)
				result += countOfSteps;
			else
				result += internalCalculate(currentCountOf2, countOfSteps);

			//printf("result = %20llu, currentCountOf2 = %d, countOfSteps = %d\n", result, currentCountOf2, countOfSteps);
		}

		//加上全部都是一次两个台阶的情况
		++result;
	}
	return result;
}
{% endhighlight %}

其中 internalCalculate 函数的实现为：

{% highlight c++ %}
unsigned long long Solution::internalCalculate(int count, int initNum)
{
	if(count > initNum / 2)
		count = initNum - count;

	unsigned long long result = 1;
	unsigned long long dummyCount = count;
	unsigned long long countResult = 1;

	set<int> set_steps;
	for(int step = 2; step <= count; ++step)
	{
		set_steps.insert(step);
	}

	for(int index = 0; index < count; ++index)
	{
		result *= initNum;
		if(set_steps.size() > 0)
		{
			for(int step = 2; step <= count; ++step)
			{
				if(set_steps.find(step) != set_steps.end())
				{
					if(result % step == 0)
					{
						result /= step;
						set_steps.erase(step);
					}
				}
			}
		}
		--initNum;
	}
	return result;
}
{% endhighlight %}

为了更好地对比两个思路指导下实现的性能，格式化一下输出：

{% highlight c++ %}
void StartCounter(void) 
{     
	LARGE_INTEGER li;
	if(!QueryPerformanceFrequency(&li))         
		printf( "QueryPerformanceFrequency failed!\n"); 

	PCFreq = DOUBLE(li.QuadPart)/1000000000.0;      
	QueryPerformanceCounter(&li);     
	CounterStart = li.QuadPart; 
} 
unsigned long long StopAndGetCounter(void) 
{     
	LARGE_INTEGER li;
	if(!QueryPerformanceCounter(&li))         
		printf( "QueryPerformanceCounter failed!\n");  

	return (unsigned long long)(li.QuadPart-CounterStart)/PCFreq; 
}  

void TestCombination(const int stairs)
{
	printf("%-5s%-25s%-25s%-25s%-25s\n", "Num",
		"S_Result",   "C_Result",
		"S_Duration", "C_Duration");

	const char *formatStr = "%-5d%-25llu%-25llu%-25llu%-25llu\n";


	unsigned long long resultSimpiestWay = 0;
	unsigned long long durationSimpiestWay = 0;
	unsigned long long resultCombinationWay = 0;
	unsigned long long durationCombinationWay = 0;

	for(int stair = 1; stair <= stairs; ++stair)
	{
		Solution sol(stair);

		StartCounter();
		resultSimpiestWay = sol.SimpiestWay();
		durationSimpiestWay = StopAndGetCounter();

		CounterStart = 0;

		StartCounter();
		resultCombinationWay = sol.CombinationWay();
		durationCombinationWay = StopAndGetCounter();

		printf(formatStr, 
			stair, 
			resultSimpiestWay,   resultCombinationWay, 
			durationSimpiestWay, durationCombinationWay);
	}
}

int main()
{
	TestCombination(92);

	getchar();
	return 0;
}
{% endhighlight %}

运行的结果如下，和我们期待的一样，想明白问题本身之后，求斐波拉契数列的耗时很少，但是穷举法就相对耗时多了，而且在台阶数到92后，运算结果已经出错了，中间数据越界了。

{% highlight text %}

Num  S_Result                 C_Result                 S_Duration               C_Duration
1    1                        1                        3157                     394
2    2                        2                        1578                     0
3    3                        3                        1578                     394
4    5                        5                        2368                     0
5    8                        8                        1578                     37498
6    13                       13                       1578                     23683
7    21                       21                       1184                     29209
8    34                       34                       1184                     41840
9    55                       55                       1184                     63156
10   89                       89                       1973                     82497
11   144                      144                      1184                     118417
12   233                      233                      1184                     161442
13   377                      377                      1973                     194599
14   610                      610                      1973                     223809
15   987                      987                      1973                     279860
16   1597                     1597                     1973                     326437
17   2584                     2584                     1973                     387620
18   4181                     4181                     1973                     468144
19   6765                     6765                     1973                     598403
20   10946                    10946                    1973                     615771
21   17711                    17711                    1973                     677743
22   28657                    28657                    1578                     792213
23   46368                    46368                    1578                     890105
24   75025                    75025                    1973                     960761
25   121393                   121393                   2368                     1156939
26   196418                   196418                   1973                     1205490
27   317811                   317811                   1973                     1466798
28   514229                   514229                   2368                     1556401
29   832040                   832040                   2763                     1702844
30   1346269                  1346269                  2368                     1898628
31   2178309                  2178309                  2368                     2060070
32   3524578                  3524578                  1973                     2217171
33   5702887                  5702887                  2763                     2424007
34   9227465                  9227465                  2368                     2642290
35   14930352                 14930352                 2763                     3868701
36   24157817                 24157817                 1973                     3124250
37   39088169                 39088169                 2763                     3342138
38   63245986                 63245986                 1973                     3664628
39   102334155                102334155                1973                     3966988
40   165580141                165580141                2368                     4274084
41   267914296                267914296                2763                     5762988
42   433494437                433494437                2763                     4949459
43   701408733                701408733                2368                     6975978
44   1134903170               1134903170               2763                     5464970
45   1836311903               1836311903               2368                     5611413
46   2971215073               2971215073               1973                     5961929
47   4807526976               4807526976               1973                     6260736
48   7778742049               7778742049               2368                     6835456
49   12586269025              12586269025              2368                     7310705
50   20365011074              20365011074              3552                     7872399
51   32951280099              32951280099              2763                     8281334
52   53316291173              53316291173              2763                     8892764
53   86267571272              86267571272              2368                     9084600
54   139583862445             139583862445             2368                     9731950
55   225851433717             225851433717             1973                     9999573
56   365435296162             365435296162             1973                     11742285
57   591286729879             591286729879             10657                    12064775
58   956722026041             956722026041             11052                    12050960
59   1548008755920            1548008755920            12631                    15250208
60   2504730781961            2504730781961            11052                    14133136
61   4052739537881            4052739537881            11841                    13779462
62   6557470319842            6557470319842            11052                    18043283
63   10610209857723           10610209857723           13420                    16156892
64   17167680177565           17167680177565           12236                    26199490
65   27777890035288           27777890035288           13025                    17520273
66   44945570212853           44945570212853           12631                    17518694
67   72723460248141           72723460248141           11841                    19660473
68   117669030460994          117669030460994          12631                    26823156
69   190392490709135          190392490709135          15789                    23508649
70   308061521170129          308061521170129          12236                    33818871
71   498454011879264          498454011879264          13815                    36742601
72   806515533049393          806515533049393          12236                    28446266
73   1304969544928657         1304969544928657         13025                    31949058
74   2111485077978050         2111485077978050         11841                    25903052
75   3416454622906707         3416454622906707         12631                    26626978
76   5527939700884757         5527939700884757         5920                     29768201
77   8944394323791464         8944394323791464         13420                    28171143
78   14472334024676221        14472334024676221        14210                    31245263
79   23416728348467685        23416728348467685        14604                    30469628
80   37889062373143906        37889062373143906        11841                    30283712
81   61305790721611591        61305790721611591        13815                    32778770
82   99194853094755497        99194853094755497        11052                    34393591
83   160500643816367088       160500643816367088       13815                    42158231
84   259695496911122585       259695496911122585       13815                    41154050
85   420196140727489673       420196140727489673       12236                    36757600
86   679891637638612258       679891637638612258       12236                    37958749
87   1100087778366101931      1100087778366101931      11447                    47214266
88   1779979416004714189      1779979416004714189      16973                    49950501
89   2880067194370816120      2880067194370816120      13025                    48083451
90   4660046610375530309      4660046610375530309      13420                    51594531
91   7540113804746346429      7540113804746346429      13420                    44508030
92   12200160415121876738     12959737889368172866     15394                    46955721
{% endhighlight %}

这个问题本身不难，但是分析的过程很有意思，如何在标准库基本数据类型取值范围限制下计算排列组合的值很有意思。至于更大的数，有[NTL][1]来处理，本文就不涉及了。

[1]: http://www.shoup.net/ntl/
[2]: /images/comb1.gif
[3]: /images/comb2.gif


