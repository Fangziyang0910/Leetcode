# [LCP 33.蓄水](https://leetcode.cn/problems/o8SXZn/) 

## 题目

给定 N 个无限容量且初始均空的水缸，每个水缸配有一个水桶用来打水，第 i 个水缸配备的水桶容量记作 bucket[i]。小扣有以下两种操作：

**升级水桶**：选择任意一个水桶，使其容量增加为 bucket[i]+1
**蓄水**：将全部水桶接满水，倒入各自对应的水缸
每个水缸对应最低蓄水量记作 vat[i]，返回小扣至少需要多少次操作可以完成所有水缸蓄水要求。

**注意：实际蓄水量 达到或超过 最低蓄水量，即完成蓄水要求。**

**示例 1：**

    输入：bucket = [1,3], vat = [6,8]
    输出：4
    解释：
    第 1 次操作升级 bucket[0]；
    第 2 ~ 4 次操作均选择蓄水，即可完成蓄水要求。
    

**示例 2：**

    输入：bucket = [9,0,1], vat = [0,2,2]
    输出：3
    解释：
    第 1 次操作均选择升级 bucket[1]
    第 2~3 次操作选择蓄水，即可完成蓄水要求。
    
## 方法：贪心 + 优先队列

### 思路与分析

题目中对应了两种操作，分别是[升级水桶]和[蓄水],我们可以贪心地将所有的升级水桶操作放在蓄水操作之后，这样所有的升级水桶所带来的收益就可以在后续的每一次蓄水中叠加。

多个水桶的蓄水次数取决于$$max \frac{vat[i]}{bucket[i]}$$,所以我们可以维护一个优先队列，队列中存放$$pair<int,int>$$其中第一个数是第i个水箱需要的蓄水次数，第二个数是i，也就是水箱的下标。
维护优先队列，将每一个水桶的初始信息写进去。需要注意的是每个水桶的蓄水次数应该向上取整。也就是$\lceil \frac{vat[i]}{bucket[i]} \rceil$。
我们首先需要对优先队列进行初始化，第一种情况是水缸的容量为0，那么此时水缸其实是完全不用加水的，我们可以将其忽略。第二种情况是水桶容量为0，这个时候我们应该将其进行[升级水桶的操作]。通过变量sum来记录升级操作的执行次数。


贪心策略的流程为，每次从优先队列中选取出需要蓄水次数最多的水桶对其进行升级。升级完之后再将升级之后更新的pair数据放回去顶堆之中。此时总的最小操作数res是sum和maxtry.top().first的和。
需要注意的是贪心的退出条件并不是最大蓄水次数不在减小的时候。举个例子，队列中有两个水缸都需要蓄水5次，并且最大的蓄水次数也是5，贪心法取出其中一个水缸，升级水缸之后这个水缸的蓄水次数变为4，但是总的蓄水次数还是5，因为还存在一个次数为5的水缸。
所以贪心的推出条件应该是升级水缸的操作已经不能减少**总操作次数**了，总操作数为sum和maxtry.top().first的和。但是我们无法预知总操作数res何时不再减少，我们可以放宽一点条件，我们知道每次贪心之后sum都会增加，当sum的次数已经增加得比res多的时候，此时肯定已经找不到更优的res了，这才是退出贪心循环的条件。
循环的另一个出口是最大蓄水次数已经为1了，此时不需要再升级水桶，只需要一次的蓄水操作就可以完成。



```
class Solution {
public:
    int storeWater(vector<int>& bucket, vector<int>& vat) {
        priority_queue<pair<int,int>> maxtry;
        int n = bucket.size();
        int sum = 0;
        for(int i = 0;i<n;i++)
        {
            int rate = bucket[i]==0?(vat[i]==0?0:INT_MAX):ceil((double)vat[i]/bucket[i]);
            maxtry.push({rate,i});
        } 
        
        int res = maxtry.top().first;
        while(res>sum&&maxtry.top().first!=1)
        {
            auto [rate,index] = maxtry.top();
            maxtry.pop();
            sum++;
            int newrate = ceil((double)vat[index]/(bucket[index]+1));
            maxtry.push({newrate,index});
            bucket[index]++;
            res = min(maxtry.top().first==INT_MAX?INT_MAX:sum+maxtry.top().first,res);
        }
        return res;
    }
};
```

### 复杂度分析

#### 时间复杂度

$ O(n * log n*M)$n为水桶的个数，最大堆中一次存操作的时间复杂度为$O(log n)$ 。M为水缸的最大蓄水次数。

#### 空间复杂度

$O(n)$其中n为水洞个数，主要是开辟优先队列的空间开销。