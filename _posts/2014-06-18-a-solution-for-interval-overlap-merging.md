---
layout: post
title: "[Algorithm]Interval overlap merging"
description: ""
category: [Algorithm,C++]
---
# A Solution for interval overlap merging


### 题目描述

##### Input:   字符串，合法内容为"[A,B] [C,D] [E,F]"，A,B,C,D,E,F为int类型，它们所表示的intervals都为闭区间。

##### Output:  将输入的首个interval看为一个集合，其余的intervals为另一个集合，查询第二个集合中和首个interval overlap的部分，并输出可能的intervals，升序并merge重合的部分。如果没有符合条件的interval，输出"Empty"

##### Example: "[2,9] [3,12] [-12,1]" 的输出为 "[3,9]"

### 解决思路
* 需要对输入字符串的输入进行合法性验证

#### 1. 声明下面的函数

{% highlight c++ %}
//Test
void TestIntervalFilter(void);
//finite closed interval struct
struct Interval
{
	Interval(const int &s, const int &e) : start(s), end(e){}
	int start;
	int end;
};
void CheckAndMergeIntervals(vector<Interval> &intervals);
bool ValidateAndGetIntervals(const string &input, Interval &seedInterval, vector<Interval> &intervals);
//Function entry for question two
void IntervalFilter(const string &input);
{% endhighlight %}

#### 2. TDD推荐首先设计单元测试用例

{% highlight c++ %}
void TestIntervalFilter(void)
{
	IntervalFilter("[6,27] [5,7] [21,34] [13,25]");
	IntervalFilter("[24,35] [3,20] [-2,9] [37,40]");

	//below test will output error message
	IntervalFilter("");
	IntervalFilter("[");
	IntervalFilter(" ");
	IntervalFilter("[]");
	IntervalFilter("[1,2]");
	IntervalFilter("[1,test]");
	IntervalFilter("[1,5] [2,10] ");

}
{% endhighlight %}

#### 3. 输入合法性验证的实现

{% highlight c++ %}
bool ValidateAndGetIntervals(const string &input, Interval &seedInterval, vector<Interval> &intervals)
{
	if(input.empty() || input[0] != '[' || input[input.length() - 1] != ']')
	{
		printf("invalid\n");
		return false;
	}

	string dataStr = input;
	int spaceIndex = dataStr.find_first_of(" ");
	if(spaceIndex <= 0)
	{
		printf("invalid\n");
		return false;
	}

	bool alreadyGetSeedInterval = false;
	while(spaceIndex > 0)
	{
		spaceIndex = dataStr.find_first_of(" ");
		string intervalStr = dataStr.substr(0, spaceIndex);
		if(intervalStr.length() < 3 || intervalStr[0] != '[' || intervalStr[intervalStr.length() - 1] != ']')
		{
			printf("invalid\n");
			return false;
		}
		int commaIndex = intervalStr.find(',');
		if(commaIndex <= 1)
		{
			printf("invalid\n");
			return false;
		}
		int start = atoi(intervalStr.substr(1,commaIndex).c_str());
		int end = atoi(intervalStr.substr(commaIndex + 1, intervalStr.length() - commaIndex - 1).c_str());
		if(start != 0 && end != 0 && start < end)
		{
			Interval currentInterval(start, end);
			if(!alreadyGetSeedInterval)
			{
				seedInterval = currentInterval;
				alreadyGetSeedInterval = true;
			}
			else
				intervals.push_back(currentInterval);
		}
		else
		{
			printf("invalid\n");
			return false;
		}

		if(spaceIndex < dataStr.length() - 1)
		{
			dataStr = dataStr.substr(spaceIndex + 1);
		}
		else
			break;
	}

	return true;
}
{% endhighlight %}

#### 4. 定义sort函数的比较算法

{% highlight c++ %}
bool Compare(Interval i1, Interval i2)
{
	return (i1.start < i2.start)? true: false;
}
{% endhighlight %}

#### 5. 对符合条件的intervals进行merge

{% highlight c++ %}
void CheckAndMergeIntervals(vector<Interval> &intervals)
{
	//Make sure the given intervals are not empty
	if (intervals.size() <= 0)
		return;

	vector<Interval> output;

	//Sort the given intervals according to their start number to asc order
	sort(intervals.begin(), intervals.end(), Compare);

	output.push_back(intervals[0]);

	for (unsigned int i = 1 ; i < intervals.size(); i++)
	{
		Interval last = output[output.size() - 1];

		//Add the current interval into the vector if it is not overlapping with the top last in it
		if (last.end < intervals[i].start)
		{
			output.push_back(intervals[i]);
		}

		//Update the last interval in the vector if necessary
		else if (last.end < intervals[i].end)
		{
			last.end = intervals[i].end;
			output[output.size() - 1] = last;
		}
	}

	for(vector<Interval>::const_iterator cit = output.begin(); cit != output.end(); ++cit)
	{
		Interval currInterval = *cit;
		printf("[%d,%d] ", currInterval.start, currInterval.end);
	}

	printf("\n");
}
{% endhighlight %}

#### 6. 解决问题的入口函数实现，从第二个intervals的集合中获取符合条件的intervals并调用上面的merge函数
{% highlight c++ %}
void IntervalFilter(const string &input)
{
	vector<Interval> intervals;
	Interval seedInterval(0,0);
	if(!ValidateAndGetIntervals(input, seedInterval, intervals))
		return;

	if(intervals.size() == 0)
		printf("invalid\n");

	vector<Interval> intervalsAfterSeedCompare;
	for(vector<Interval>::iterator it = intervals.begin(); it != intervals.end(); ++it)
	{
		Interval currentInterval = *it;

		if(currentInterval.end < seedInterval.start)
			continue;

		if(currentInterval.end == seedInterval.start)
		{
			currentInterval.start = currentInterval.end;
		}
		else if(currentInterval.end > seedInterval.start && currentInterval.end <= seedInterval.end)
		{
			if(currentInterval.start <= seedInterval.start)
				currentInterval.start = seedInterval.start;
		}
		else if(currentInterval.end > seedInterval.end)
		{
			if(currentInterval.start <= seedInterval.end)
			{
				currentInterval.end = seedInterval.end;
				if(currentInterval.start <= seedInterval.start)
					currentInterval.start = seedInterval.start;
			}
			else
				continue;
		}

		intervalsAfterSeedCompare.push_back(currentInterval);
	}

	if(intervalsAfterSeedCompare.size() > 0)
		CheckAndMergeIntervals(intervalsAfterSeedCompare);
	else
		printf("Empty\n");
}
{% endhighlight %}

#### 7. 在main函数中这样调用：

{% highlight c++ %}
int main()
{
	//Test question one
	TestIntervalFilter();

	return 0;
}
{% endhighlight %}




