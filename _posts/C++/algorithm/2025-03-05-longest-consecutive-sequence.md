---
title: "[NeetCode] Longest Consecutive Sequence"

categories:
  - Algorithm
tags:
  - [C++, Arrays & Hashing]

toc: true
toc_sticky: true

date: 2025-03-05
last_modified_at: 2025-03-05
---

# 링크

[NeetCode: Longest Consecutive Sequence](https://neetcode.io/problems/longest-consecutive-sequence)

# 코드

```cpp
#include <vector>
#include <unordered_map>
#include <unordered_set>
#include <algorithm>

using namespace std;

class Solution {
public:
    int myLongestConsecutive(vector<int>& nums) {
        unordered_map<int, int> um;
        countingSort(nums);

        for (const int& n: nums) {
            if (um[n-1]) {
                um[n] = um[n-1]+1;
            } else {
                um[n] = 1;
            }
        }

        int answer = 0;
        for (const auto [_, v]: um) {
            answer = answer > v ? answer : v;
        }

        return answer;
    }

    int bestLongestConsecutive(vector<int>& nums) {
        unordered_set<int> numSet(nums.begin(), nums.end()); // O(n)

        int longest = 0;
        for (const int& num : numSet) {
            // 이전 숫자가 없으면 새로운 시퀀스 시작
            if (numSet.find(num - 1) == numSet.end()) {
                int currentNum = num;
                int currentStreak = 1;

                // 연속된 숫자가 존재하는지 확인
                while (numSet.find(currentNum + 1) != numSet.end()) {
                    currentNum += 1;
                    currentStreak += 1;
                }

                longest = max(longest, currentStreak);
            }
        }
        return longest;
    }

private:
    void countingSort(vector<int>& nums) {
        if (nums.empty()) return;

        // 1. 최댓값, 최솟값 찾기
        int minVal = *min_element(nums.begin(), nums.end());
        int maxVal = *max_element(nums.begin(), nums.end());
        int range = maxVal - minVal + 1;

        // 2. 카운트 배열 생성
        vector<int> count(range, 0);

        // 3. 각 숫자의 빈도 계산
        for (int num : nums) {
            count[num - minVal]++;
        }

        // 4. 정렬된 결과를 원본 벡터에 복사
        int index = 0;
        for (int i = 0; i < range; ++i) {
            while (count[i]-- > 0) {
                nums[index++] = i + minVal;
            }
        }
    }
};

```

# 성능 최적화

|                  | **기존 코드 O(n+k)**                              | **최적화 코드 O(n)**                           |
| ---------------- | ------------------------------------------------- | ---------------------------------------------- |
| 정렬             | Counting Sort을 사용 (O(n + k))                   | unordered_set을 사용하여 정렬 없이 해결 (O(n)) |
| 연속된 수열 탐색 | unordered_map을 활용 (O(n))                       | unordered_set으로 직접 탐색 (O(n))             |
| 최종 시간복잡도  | 최악의 경우 O(n + k)로 k가 클 경우 성능 저하 가능 | 항상 O(n)으로 동작                             |

## 현재 코드의 시간 복잡도 → O(n+k)

k가 O(n)일 경우 O(n)으로 동작하지만, 범위가 크면 성능이 저하 됨.

### **정렬 부분 (Counting Sort) → O(n+k)**

- countingSort() 함수는 **O(n + k)**의 시간 복잡도를 가진다.
  - n은 nums의 크기
  - k는 숫자의 범위(maxVal - minVal + 1)
- 최악의 경우 k가 O(n)에 비례하면, 전체 정렬 복잡도는 **O(n)**
- 하지만, k가 O(n)보다 훨씬 크다면 성능 저하가 발생할 수 있다.

### **연속 부분 수열 계산 부분 → O(n)**

- `unordered_map<int, int> um;` 사용
- `for (const int& n: nums)` 에서 각 숫자를 확인하며 **O(n)**
- `for (const auto [_, v]: um)` 에서 최댓값을 찾으며 **O(n)**

## 최적화된 코드

- unordered_set에 nums를 저장하는 데 **O(n)**
- 각 숫자에 대해 **최대 O(1) 연산**을 수행하며, 전체적으로 **O(n)**
- 정렬 없이 연속된 수열을 탐색할 수 있으므로 **정렬 없이 O(n)** 유지 가능

# unordered_set

unordered_set은 내부적으로 **해시 테이블(Hash Table)** 을 사용하여 원소를 저장하고 검색한다. 해시 테이블을 기반으로 동작하기 때문에 평균적으로 **삽입, 삭제, 검색 연산이 O(1)** 의 시간 복잡도를 가진다. 따라서 전체적으로 unordered_set을 사용한 풀이가 **O(n)** 으로 동작할 수 있다.

## **주요 연산과 시간 복잡도**

| **연산**           | **시간 복잡도 (평균)** | **시간 복잡도 (최악)**      |
| ------------------ | ---------------------- | --------------------------- |
| 원소 삽입 (insert) | **O(1)**               | **O(n)** (충돌이 많을 경우) |
| 원소 삭제 (erase)  | **O(1)**               | **O(n)** (충돌이 많을 경우) |
| 원소 탐색 (find)   | **O(1)**               | **O(n)** (충돌이 많을 경우) |

# 해시 테이블(Hash Table)

- 해시 테이블은 해시 함수를 사용하여 데이터를 O(1)에 가깝게 저장 및 검색 가능하다.
- `unordered_map`과 `unordered_set`은 **체이닝 방식**을 사용해 충돌을 해결한다.
- 해시 충돌이 많으면 최악의 경우 O(n)으로 느려질 수 있다.
- 정렬이 필요하면 `map`, 빠른 접근이 필요하면 `unordered_map`을 사용한다.

## **1. 개념**

해시 테이블은 **키(Key)와 값(Value)**을 저장하는 **데이터 구조**로, **빠른 검색, 삽입, 삭제**를 수행할 수 있다.

일반적으로 **해시 함수(Hash Function)**를 사용하여 **키를 특정 위치(해시 인덱스)로 변환하여 저장**하는 방식으로 동작한다.

### **장점**

- **빠른 속도** (평균적으로 검색, 삽입, 삭제 연산이 O(1)에 가깝다)
- **대량의 데이터**를 빠르게 찾을 때 유용하다.
- **STL 컨테이너**가 내부적으로 해시 테이블을 사용(`unordered_map`, `unordered_set` 등)

### **단점**

- 해시 충돌(Hash Collision)이 발생 가능 (이를 해결하는 기법이 필요하다)
- **메모리를 추가 사용** (빈 공간이 많아질 수 있다)
- **순서를 유지하지 않음** (일반적인 해시 테이블은 삽입 순서를 보장하지 않는다)

## **2. 해시 테이블의 기본 동작 원리**

### **i. 해시 함수(Hash Function) 적용**

- 주어진 키를 **해시 함수**에 넣어 특정 **인덱스(Index)** 를 얻는다.
- "apple"을 저장하는 경우

```
hash("apple") → 5 (배열의 5번 인덱스에 저장)
```

### **ii. 해시 테이블(Array or List)에 저장**

- 해시 함수가 반환한 인덱스 위치에 데이터를 저장한다.

### **iii. 검색, 삭제 연산**

- 검색할 때, **같은 해시 함수를 적용하여 인덱스를 찾고** 데이터를 반환
- 삭제할 때, **해시 함수로 인덱스를 찾아 제거**

## **3. 해시 충돌(Hash Collision) 해결 방법**

### 개념

- 서로 다른 키가 **같은 해시 인덱스를 가지는 경우**
- 예를 들어, "apple"과 "banana"가 해시 함수에 의해 같은 인덱스 5를 반환하는 경우

### 해결 방법

1. **체이닝(Chaining) 방식**
   - 같은 인덱스에 여러 개의 값이 저장될 경우, 연결 리스트(Linked List)로 관리한다.
   - 같은 위치에 여러 개가 들어가도, 연결된 리스트에서 탐색
   - `unordered_map`, `unordered_set`에서 보통 체이닝 방식을 사용한다.
2. **개방 주소법(Open Addressing) 방식**
   - 충돌이 발생하면 다른 빈 공간을 찾아 저장한다.
   - 대표적인 방식:
     - **선형 탐색(Linear Probing)** → 다음 빈 슬롯을 찾는다.
     - **이차 탐색(Quadratic Probing)** → 점프하면서 빈 슬롯을 찾는다.

### **4. 해시 테이블의 시간 복잡도**

| **연산**      | **평균 시간 복잡도** | **최악의 시간 복잡도 (충돌 심할 때)** |
| ------------- | -------------------- | ------------------------------------- |
| 삽입 (Insert) | **O(1)**             | **O(n)**                              |
| 검색 (Find)   | **O(1)**             | **O(n)**                              |
| 삭제 (Erase)  | **O(1)**             | **O(n)**                              |

- **평균적으로 O(1)** 에 가깝지만, 해시 충돌이 많아지면 **O(n)** 으로 느려질 수 있다.
- 따라서, **적절한 해시 함수 선택**과 **적절한 해시 테이블 크기 조절(리사이징, Rehashing)**이 중요하다.

### **5. C++에서 해시 테이블 사용 방법**

- `unordered_map<Key, Value>` → 키-값 저장
- `unordered_set<Key>` → 키만 저장

# O(n) 정렬 방법

일반적으로 sort()(O(n log n))가 가장 범용적이며, 특정 조건에서 O(n) 알고리즘이 유리하다.

| **정렬 알고리즘** | **시간 복잡도**                | **공간 복잡도** | **특징**                         |
| ----------------- | ------------------------------ | --------------- | -------------------------------- |
| Counting Sort     | O(n + k)                       | O(k)            | 작은 정수 범위에서 가장 빠름     |
| Radix Sort        | O(d \* (n + k))                | O(n + k)        | 정수 정렬에 적합, 비교 연산 없음 |
| Bucket Sort       | O(n) (평균), O(n log n) (최악) | O(n)            | 실수(float) 정렬에 최적          |

- 정수 범위가 작으면 Counting Sort
- 큰 정수를 정렬할 때는 Radix Sort
- 0~1 사이의 실수 정렬에는 Bucket Sort

## **1. Counting Sort (계수 정렬)**

### **조건**

- 데이터의 범위(max - min)가 크지 않아야 한다.
- 음수가 포함되면 offset을 적용해야 한다.

### **시간 복잡도**

- **O(n + k)**, 여기서 k는 숫자의 범위 (max - min)

### **공간 복잡도**

- **O(k)**, 추가적인 count 배열이 필요하다.

### **코드**

```cpp
#include <iostream>
#include <vector>

using namespace std;

void countingSort(vector<int>& nums) {
    if (nums.empty()) return;

    // 1. 최댓값, 최솟값 찾기
    int minVal = *min_element(nums.begin(), nums.end());
    int maxVal = *max_element(nums.begin(), nums.end());
    int range = maxVal - minVal + 1;

    // 2. 카운트 배열 생성
    vector<int> count(range, 0);

    // 3. 각 숫자의 빈도 계산
    for (int num : nums) {
        count[num - minVal]++;
    }

    // 4. 정렬된 결과를 원본 벡터에 복사
    int index = 0;
    for (int i = 0; i < range; ++i) {
        while (count[i]-- > 0) {
            nums[index++] = i + minVal;
        }
    }
}

int main() {
    vector<int> nums = {5, 3, 2, 8, 7, 3, 1, 6, 5, 8};
    countingSort(nums);

    for (int num : nums) {
        cout << num << " ";
    }
}
```

- 정수 범위가 작을 때 빠른 정렬 가능
- 음수가 포함된 경우도 minVal을 기준으로 보정하여 처리 가능

## **2. Radix Sort (기수 정렬) - O(n)**

### **조건**

- 비교 기반이 아닌 정렬 방법이다.
- 데이터가 정수이거나, 일정한 길이의 문자열일 때 사용 가능하다.
- 음수를 처리하려면 **부호를 따로 분리해서 처리**해야 한다.

### **시간 복잡도**

- **O(d \* (n + k))** (d: 자릿수, k: 숫자의 기수, 보통 10)
- d가 상수라면 **O(n)**

### **공간 복잡도**

- **O(n + k)**, 추가적인 bucket 필요

### **코드**

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

using namespace std;

// 계수 정렬을 이용한 자릿수별 정렬
void countingSortRadix(vector<int>& nums, int exp) {
    vector<int> output(nums.size());
    vector<int> count(10, 0);

    // 자릿수별 빈도 계산
    for (int num : nums) {
        count[(num / exp) % 10]++;
    }

    // 누적 합을 이용한 정렬 인덱스 계산
    for (int i = 1; i < 10; i++) {
        count[i] += count[i - 1];
    }

    // 정렬된 결과를 임시 배열에 저장
    for (int i = nums.size() - 1; i >= 0; i--) {
        output[count[(nums[i] / exp) % 10] - 1] = nums[i];
        count[(nums[i] / exp) % 10]--;
    }

    // 원래 배열에 정렬된 값 복사
    nums = output;
}

// 기수 정렬
void radixSort(vector<int>& nums) {
		// 가장 큰 숫자의 자릿수를 구함
    int maxNum = *max_element(nums.begin(), nums.end());

    // 자릿수마다 정렬 수행
    for (int exp = 1; maxNum / exp > 0; exp *= 10) {
        countingSortRadix(nums, exp);
    }
}

int main() {
    vector<int> nums = {170, 45, 75, 90, 802, 24, 2, 66};
    radixSort(nums);

    for (int num : nums) {
        cout << num << " ";
    }
}
```

- 10진수 기반으로 숫자를 정렬하므로 **O(n)** 성능 보장
- 자릿수를 기준으로 정렬하므로 데이터 크기가 커도 빠름
- 비교 연산 없이 진행되므로 안정적인 정렬 가능

## **3. Bucket Sort (버킷 정렬) - O(n)**

### **조건**

- 입력값이 특정 범위 내에서 **균등하게 분포**되어 있을 때 최적
- 실수(float) 값을 정렬할 때 자주 사용됨

### **시간 복잡도**

- 최선: **O(n)**
- 최악: **O(n log n)** (버킷 내부에서 sort() 호출 시)

### **공간 복잡도**

- **O(n + k)**, 추가적인 버킷 필요

### **코드**

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

using namespace std;

void bucketSort(vector<float>& nums) {
    int n = nums.size();
    vector<vector<float>> buckets(n);

    // 1. 숫자를 적절한 버킷에 분배
    for (float num : nums) {
        int idx = num * n; // 0~1 사이 값이라 가정
        buckets[idx].push_back(num);
    }

    // 2. 각 버킷 내부 정렬
    for (auto& bucket : buckets) {
        sort(bucket.begin(), bucket.end());
    }

    // 3. 정렬된 값을 하나로 합치기
    int index = 0;
    for (auto& bucket : buckets) {
        for (float num : bucket) {
            nums[index++] = num;
        }
    }
}

int main() {
    vector<float> nums = {0.42, 0.32, 0.23, 0.52, 0.25, 0.47, 0.51};
    bucketSort(nums);

    for (float num : nums) {
        cout << num << " ";
    }
}
```

- 0~1 범위 실수 정렬에 최적
- 입력값이 균등 분포하면 O(n) 성능 보장
- 정렬 후에도 원래 순서를 유지하여 안정 정렬 가능
