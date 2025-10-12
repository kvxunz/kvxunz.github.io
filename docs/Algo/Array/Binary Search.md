# Binary Search
&emsp;&emsp; **_Given an array of integers nums which is sorted in ascending order, and an integer target, write a function to search target in nums. If target exists, then return its index. Otherwise, return -1. 
You must write an algorithm with O(log n) runtime complexity._**

**Example 1:**

    Input: nums = [-1,0,3,5,9,12], target = 9
    Output: 4
    Explanation: 9 exists in nums and its index is 4

**Example 2:**

    Input: nums = [-1,0,3,5,9,12], target = 2
    Output: -1
    Explanation: 2 does not exist in nums so return -1

**Constraints:**

    1 <= nums.length <= 104
    -104 < nums[i], target < 104
    All the integers in nums are unique.
    nums is sorted in ascending order.
## **Ideas**
&emsp;&emsp; 二分搜索是一种在有序数组中搜索某个特定元素的算法。它的基本思路是，将数组分成两半，先比较中间的元素和要搜索的元素，如果中间的元素正好是要搜索的元素，那么搜索就结束了；如果中间的元素比要搜索的元素大，就在数组的左半部分继续搜索；如果中间的元素比要搜索的元素小，就在数组的右半部分继续搜索。

&emsp;&emsp; 代码中，我们使用两个指针 `left` 和 `right` 分别指向数组的左端和右端，每次取 `mid=left+(right-left)/2` 计算出数组的中间位置。然后根据中间位置的数字和要搜索的数字的大小关系，更新左右端点的位置，直到找到要搜索的数字或者 `left>right` 为止。

&emsp;&emsp; 二分搜索的时间复杂度为 `O(log n)`，因此它是一种非常高效的搜索算法。

## **Conditions**
1. 数组中无重复元素
2. 数组为有序数组



## **Function**

### _Method-1_ 

   - 定义 `target` 是在一个在左闭右闭的区间里，也就是`[left, right]`

   - `while (left <= right)` 要使用 `<=` ，因为`left == right`是有意义的，所以使用 `<= `

   - `if (nums[middle] > target) right` 要赋值为 `middle - 1`，因为当前这个`nums[middle]`一定不是`target`，那么接下来要查找的左区间结束下标位置就是 `middle - 1`

``` 
// 版本一
class Solution {
public:
    int search(vector<int>& nums, int target) {
        int left = 0;
        int right = nums.size() - 1; // 定义target在左闭右闭的区间里，[left, right]
        while (left <= right) { // 当left==right，区间[left, right]依然有效，所以用 <=
            int middle = left + ((right - left) / 2);// 防止溢出 等同于(left + right)/2
            if (nums[middle] > target) {
                right = middle - 1; // target 在左区间，所以[left, middle - 1]
            } else if (nums[middle] < target) {
                left = middle + 1; // target 在右区间，所以[middle + 1, right]
            } else { // nums[middle] == target
                return middle; // 数组中找到目标值，直接返回下标
            }
        }
        // 未找到目标值
        return -1;
    }
};
``` 

### *Method-2* 

   - `while (left < right)`，这里使用 `< `,因为`left == right`在区间`[left, right)`是没有意义的

   - `if (nums[middle] > target)` `right` 更新为 `middle`，因为当前`nums[middle]`不等于`target`，去左区间继续寻找，而寻找区间是左闭右开区间，所以`right`更新为`middle`，即：下一个查询区间不会去比较`nums[middle]`

```
// 版本二
class Solution {
public:
    int search(vector<int>& nums, int target) {
        int left = 0;
        int right = nums.size(); // 定义target在左闭右开的区间里，即：[left, right)
        while (left < right) { // 因为left == right的时候，在[left, right)是无效的空间，所以使用 <
            int middle = left + ((right - left) >> 1);
            if (nums[middle] > target) {
                right = middle; // target 在左区间，在[left, middle)中
            } else if (nums[middle] < target) {
                left = middle + 1; // target 在右区间，在[middle + 1, right)中
            } else { // nums[middle] == target
                return middle; // 数组中找到目标值，直接返回下标
            }
        }
        // 未找到目标值
        return -1;
    }
};
``` 


         

## **Test**
    
    #include <iostream>
    #include <vector>
    using namespace std;

    int main() {

        Solution s;
        vector<int> nums = {1, 2, 3, 4, 5, 6, 7, 8, 9};
        int target = 5;
        int index = s.search(nums, target);
        if (index != -1) {
            cout << "找到了！下标为：" << index << endl;
        } else {
            cout << "没有找到。" << endl;
        }
        return 0;
    }