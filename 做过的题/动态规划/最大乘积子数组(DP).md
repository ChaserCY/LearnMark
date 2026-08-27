给你一个整数数组 `nums` ，请你找出数组中乘积最大的非空连续 子数组（该子数组中至少包含一个数字），并返回该子数组所对应的乘积。

测试用例的答案是一个 **32-位** 整数。

**请注意**，一个只包含一个元素的数组的乘积是这个元素的值。

**示例 1:**

**输入:** nums = \[2,3,-2,4]
**输出:** `6`
**解释:** 子数组\[2,3] 有最大乘积 6。

**示例 2:**

**输入:** nums = \[-2,0,-1]
**输出:** 0
**解释:** 结果不能为 2, 因为 \[-2,-1] 不是子数组。

**提示:**

- `1 <= nums.length <= 2 * 104`
- `-10 <= nums[i] <= 10`
- `nums` 的任何子数组的乘积都 **保证** 是一个 **32-位** 整数

# 题解
- 使用两个最值，来概括正负乘法情况
- 从左到右，新值只会与最近的左值相乘有关，所以用dp推导
```cpp
class Solution {
public:
    int maxProduct(vector<int>& nums) {
        int n = nums.size();
        if(n<=0)return 0;
        else if(n==1)return nums[0];
        int maxP = nums[0],p = nums[0],minP = nums[0];
        for(int i = 1;i < n;i++)
        {
            int t = maxP;
            maxP = max(max(t*nums[i],nums[i]),minP*nums[i]);
            minP = min(min(t*nums[i],nums[i]),minP*nums[i]);
            p = max(maxP,p);
        }
        return p;

    }
};
```