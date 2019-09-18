---
layout: post
title: "[Algorithm]N+NN+NNN+NNNN... question"
description: ""
category: [Algorithm,C++]
---
# A solution for N+NN+NNN+NNNN... question


### 题目描述

##### Input:   字符串，合法内容为"N M",N和M为正整数，unsigned int

##### Output:  N+NN+NNN+NNNN...的值，M为相加的次数，每次被加数按照左边的格式增加位数

##### Example: "1 3" 的输出为 1 + 11 + 111 = 123，"11 3" 的输出为 11 + 1111 + 111111 = 112233；非法输入提示相应错误信息。


### 解决思路

* 需要对输入字符串的输入进行合法性验证
* 需要考虑相加结果可能的数据类型越界
* 需要考虑不同CPU架构不同操作系统平台上不同整型数的取值范围

#### 1. 声明下面的函数

{% highlight c++ %}
//Test
void TestCalculateSum(void);
//Checkout integeral types limits on current machine
void TypeLimit(void);
bool ValidateAndGetNums(const string &input, int &metaValue, int &sequenceValue, int &metaValueDigits);
unsigned long long Calculate(const int &metaValue, const int &sequenceValue, const int &metaValueDigit);
//Function entry for question one
void CalculateSum(const string &input);
{% endhighlight %}

#### 2. TDD推荐首先设计单元测试用例
{% highlight c++ %}
void TestCalculateSum(void)
{
	//Print the value range of integral types
	TypeLimit();

	CalculateSum("1 4");
	CalculateSum("2 3");
	CalculateSum("11 2");

	CalculateSum(""); //will output error message
	CalculateSum(" "); //will output error message
	CalculateSum("test"); //will output error message
	CalculateSum("9"); //will output error message
	CalculateSum("9 two"); //will output error message
	CalculateSum("-2 3"); //will output error message
	CalculateSum("11 100"); //will output error message
}
{% endhighlight %}

#### 3. 检查并打印出整型数据类型在当前环境的取值范围

{% highlight c++ %}
void TypeLimit(void)
{
	string formatter_int = "%-20s\t%-5d\t%-20d\t%-20d\n";
	string formatter_unsgined_int = "%-20s\t%-5d\t%-20u\t%-20u\n";
	string formatter_long = "%-20s\t%-5d\t%-20ld\t%-20ld\n";
	string formatter_unsgined_long = "%-20s\t%-5d\t%-20lu\t%-20lu\n";
	string formatter_long_long = "%-20s\t%-5d\t%-20lld\t%-20lld\n";
	string formatter_unsgined_long_long = "%-20s\t%-5d\t%-20llu\t%-20llu\n";

	printf("\n************************  sizeof and limits info  **************\n");
	printf("Type\t\t\tsizeof\tMax\t\t\tMin");
	printf(formatter_int.c_str(),
		"short", sizeof(short), numeric_limits<short>::max(), numeric_limits<short>::min());
	printf(formatter_int.c_str(),
		"int", sizeof(int), numeric_limits<int>::max(), numeric_limits<int>::min());
	printf(formatter_unsgined_int.c_str(),
		"unsigned int", sizeof(unsigned int), numeric_limits<unsigned int>::max(), numeric_limits<unsigned int>::min());
	printf(formatter_long.c_str(),
		"long", sizeof(long), numeric_limits<long>::max(), numeric_limits<long>::min());
	printf(formatter_unsgined_long.c_str(),
		"unsigned long", sizeof(unsigned long), numeric_limits<unsigned long>::max(), numeric_limits<unsigned long>::min());
	printf(formatter_long_long.c_str(),
		"long long", sizeof(long long), numeric_limits<long long>::max(), numeric_limits<long long>::min());
	printf(formatter_unsgined_long_long.c_str(),
		"unsigned long long", sizeof(unsigned long long), numeric_limits<unsigned long long>::max(), numeric_limits<unsigned long long>::min());
	printf("\n************************  sizeof and limits info  **************\n");
}
{% endhighlight %}

#### 4. 输入合法性验证的实现

{% highlight c++ %}
bool ValidateAndGetNums(const string &input, int &metaValue, int &sequenceValue, int &metaValueDigits)
{
	//Input validation
	if(input.empty())
	{
		printf("Input string is \"%s\", Invalid, the input string is empty\n", input.c_str());
		return false;
	}

	int spaceIndex = input.find(' ');
	if(spaceIndex <= 0)
	{
		printf("Input string is \"%s\", Invalid, the input string format should be like \"N M\", both N and M are positive integer.\n", input.c_str());
		return false;
	}
	string firstPart = input.substr(0,spaceIndex);
	metaValue = atoi(firstPart.c_str());
	if(metaValue <= 0)
	{
		printf("Input string is \"%s\", Invalid, the first number should be a positive integer.\n", input.c_str());
		return false;
	}

	string secondPart = input.substr(spaceIndex + 1);
	sequenceValue = atoi(secondPart.c_str());
	if(sequenceValue <= 0)
	{
		printf("Input string is \"%s\", Invalid, the second number should be a positive integer.\n", input.c_str());
		return false;
	}

	metaValueDigits = spaceIndex;
	return true;
}
{% endhighlight %}

#### 5. 相加算法的实现，这里需要注意的是

* 在计算过程中需要构造出每个被加数
* 在计算过程中需要确保每次计算不越界
* 用unsigned long long 来保存相加结果，由于unsigned long 和unsigned long long 在Ubuntu gcc4.8环境的取值范围一样，但是在Windows平台不同，所以使用unsigned long long 来保证相加结果的最大值在这两个平台是相同的。

{% highlight c++ %}
unsigned long long Calculate(const int &metaValue, const int &sequenceValue, const int &metaValueDigit)
{
	unsigned long long tempSum = metaValue;
	unsigned long long sum = 0;

	unsigned long long MAX = (numeric_limits<unsigned long long>::max)();

	unsigned long long decimalIndex = 1;
	for(int index = 1; index <= metaValueDigit; ++index)
	{
		if(MAX / 10 > decimalIndex)
			decimalIndex *= 10;
		else
			return 0;
	}

	for(int i = 1; i <= sequenceValue; i++)
	{
		if(i == 1)
			sum = tempSum;
		else
		{
			unsigned long long newValue = 1;
			for(int j = 1; j < i; j++)
			{
				if(MAX / decimalIndex >= newValue)
				{
					newValue *= decimalIndex;
				}
				else
					return 0;

			}
			if(MAX / newValue <= (unsigned long long)metaValue)
				return 0;

			if((MAX - newValue * metaValue) <= tempSum)
				return 0;

			tempSum = tempSum + metaValue * newValue;
			if(MAX - tempSum < sum)
				return 0;

			if(MAX - tempSum == sum && i < sequenceValue)
				return 0;

			sum += tempSum;
		}
	}

	return sum;
}
{% endhighlight %}

#### 6. 进行计算的入口函数实现

{% highlight c++ %}
void CalculateSum(const string &input)
{
	int metaValue = 0;
	int sequenceValue = 0;
	int metaValueDigits = 0;
	if(!ValidateAndGetNums(input, metaValue, sequenceValue, metaValueDigits))
		return;

	unsigned long long sum = Calculate(metaValue, sequenceValue, metaValueDigits);
	if(sum != 0)
		printf("%llu\n", sum);
	else
		printf("Input string is \"%s\", the result is out of the range of unsigned long long.\n", input.c_str());
}
{% endhighlight %}

#### 7. 在main函数中这样调用：

{% highlight c++ %}
int main()
{
	//Test question one
	TestCalculateSum();

	return 0;
}
{% endhighlight %}


