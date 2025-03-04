---
title: "[NeetCode] Product of Array Except Self"

categories:
  - Algorithm
tags:
  - [C++, Arrays & Hashing]

toc: true
toc_sticky: true

date: 2025-03-04
last_modified_at: 2025-03-04
---

# 링크

[NeetCode: Product of Array Except Self](https://neetcode.io/problems/products-of-array-discluding-self)

# 코드

```cpp

class Solution {
public:
    vector<int> myProductExceptSelf(vector<int>& nums) {
        vector<int> answer;
        deque<int> dq(nums.begin(), nums.end());

        for (int i = 0; i < dq.size(); ++i) {

            const int& num = dq.front();
            dq.pop_front();
            int val = std::accumulate(dq.begin(), dq.end(), 1, std::multiplies<int>());
            answer.emplace_back(val);
            dq.push_back(num);
        }

        return answer;
    }
    vector<int> bestProductExceptSelf(vector<int>& nums) {
        int n = nums.size();
        vector<int> answer(n, 1);

        // 왼쪽 곱셈 배열 채우기
        int left_product = 1;
        for (int i = 0; i < n; ++i) {
            answer[i] = left_product;
            left_product *= nums[i];
        }

        // 오른쪽 곱셈 값을 이용하여 최종 결과 계산
        int right_product = 1;
        for (int i = n - 1; i >= 0; --i) {
            answer[i] *= right_product;
            right_product *= nums[i];
        }

        return answer;
    }
};

```

# 성능 최적화

| **기존 코드 (O(n²))**              | **개선된 코드 (O(n))**                          |
| ---------------------------------- | ----------------------------------------------- |
| std::accumulate() 사용 (O(n) 반복) | **왼쪽 곱셈, 오른쪽 곱셈을 사용하여 O(n) 해결** |
| deque 사용                         | **vector만 사용하여 불필요한 연산 제거**        |
| pop_front()와 push_back() 사용     | **한 번의 순회로 해결하여 최적화**              |

## 현재 코드의 시간 복잡도 → **O(n²)**

1. **`deque<int> dq(nums.begin(), nums.end());` → O(n)**
   - nums의 요소를 deque에 복사하는 작업이므로 O(n)
2. **반복문 → O(n²)**
   - for 루프가 n번 반복.
   - 각 반복에서 `std::accumulate(dq.begin(), dq.end(), 1, std::multiplies<int>())`가 실행되는데, 이는 O(n)이 걸림.

## **최적화된 코드 → O(n)**

| **단계**                  | **연산 내용**                 | **시간 복잡도** |
| ------------------------- | ----------------------------- | --------------- |
| vector<int> answer(n, 1); | 결과 배열 초기화              | **O(n)**        |
| 첫 번째 for 루프          | 왼쪽 곱셈 값 계산             | **O(n)**        |
| 두 번째 for 루프          | 오른쪽 곱셈 값 적용           | **O(n)**        |
| **전체 복잡도**           | **O(n) + O(n) + O(n) = O(n)** |                 |

- 각 숫자를 제외한 곱셈을 직접 계산하는 방법으로 최적화한다.
- **왼쪽 곱셈(left_product)과 오른쪽 곱셈(right_product)을 이용한 DP 방식**을 사용한다.

## 왼쪽 곱셈 + 오른쪽 곱셈

각 원소 `answer[i]`를 계산하기 위해 **왼쪽 곱셈과 오른쪽 곱셈을 따로 저장**하는 방법을 사용.

- **왼쪽 곱셈(left product)**: answer[i]가 되기 위해 nums[i]의 **왼쪽**에 있는 모든 요소의 곱을 저장.
- **오른쪽 곱셈(right product)**: nums[i]의 **오른쪽**에 있는 모든 요소의 곱을 저장.

## 동작 과정

```cpp
nums = [1, 2, 3, 4]
```

### 왼쪽 곱셈

| **i** | **nums[i]** | **leftProduct** | **answer[i] (왼쪽 곱셈 저장)** |
| ----- | ----------- | --------------- | ------------------------------ |
| 0     | 1           | 1               | 1                              |
| 1     | 2           | 1×1 = 1         | 1                              |
| 2     | 3           | 1×2 = 2         | 2                              |
| 3     | 4           | 2×3 = 6         | 6                              |

첫 번째 for문을 순회한 후, answer 배열의 상태

```cpp
nums = [1, 1, 2, 6]
```

### 오른쪽 곱셈

| **i** | **nums[i]** | **rightProduct** | **answer[i] × rightProduct** |
| ----- | ----------- | ---------------- | ---------------------------- |
| 3     | 4           | 1                | 6 × 1 = 6                    |
| 2     | 3           | 1×4 = 4          | 2 × 4 = 8                    |
| 1     | 2           | 4×3 = 12         | 1 × 12 = 12                  |
| 0     | 1           | 12×2 = 24        | 1 × 24 = 24                  |

두 번째 for문을 순회한 후, answer 배열의 상태

```cpp
nums = [24, 12, 8, 6]
```
