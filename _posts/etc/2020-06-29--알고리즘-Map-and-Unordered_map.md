---
layout: post
title: 'Map and Unordered_map'
tags: [etc]
comments: true
description: >
  STL map 그리고 unordered_map 비교
sitemap :
  changefreq: weekly
  priority : 1.0
---
std::map은 Key와 value를 갖는 데이터 구조이다. std::map의 경우 key값에 따라서 정렬이 되지만, std::unordered_map은 hash_map과 동일하게 내부적으로 key 값에 의해 정렬되지 않는다. std::unordered_map은 c++11에서 STL 표준으로 추가되었다. 

std::unordered_map의 장점은 빠른 탐색속도이다. N개의 데이터 쌍을 갖는 std::map의 경우에는 O(logN)의 탐색속도를 갖는 반면, std::unordered_map은 O(1)의 탐색속도를 갖는다. 

### Map

map은 기본적으로 red-black tree 기반으로 되어 있다. 따라서 모든 데이터는 key 값을 기준으로 정렬되어 저장되어 진다. map의 경우 입력되는 key 값의 분포가 고르지 못할 경우 balancing에 대한 비용이 계속 들어가기 때문에 성능이 저하된다. (탐색속도 O(logN)은 보장된다)

### Unordered_map

unordered_map은 hash_table 기반의 hash_container이다. 

node들을 정렬할 필요가 없기 때문에 탐색에서 꾸준한 성능 (O(1))의 성능을 보장한다. 

### Example

다음과 같은 알고리즘 문제에서 탐색을 위해 unordered map을 사용하면 빠르게 탐색이 가능하다

1. Two Sum

Given an array of integers, return **indices** of the two numbers such that they add up to a specific target.

You may assume that each input would have ***exactly*** one solution, and you may not use the *same* element twice.

**Example:**

```
Given nums = [2, 7, 11, 15], target = 9,

Because nums[0] + nums[1] = 2 + 7 = 9,
return [0, 1].
```

2. Solution

```bash
class Solution {
public:
    vector<int> twoSum(vector<int>& nums, int target) {

        unordered_map<int,int> map;
        vector<int> result;
        for(int i = 0 ; i < nums.size() ; ++i){
            int complement = target - nums[i];
            if(map.find(complement)!= map.end() && map[complement] != i){
                
                result.push_back(i);
                result.push_back(map[complement]);
                return result;
            }else{
                map[nums[i]] = i;
            }
        }
        return result;
    }
};
```

### Reference

[[C++] map vs hash_map(unordered_map)](https://gracefulprograming.tistory.com/3)
