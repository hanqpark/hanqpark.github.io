---
title: "[NeetCode] Top K Frequent Elements"

categories:
  - Algorithm
tags:
  - [C++, Array & Hash]

toc: true
toc_sticky: true

date: 2025-02-26
last_modified_at: 2025-02-26
---

# 링크

[NeetCode: Top K Frequent Elements](https://neetcode.io/problems/top-k-elements-in-list)

# 코드

```cpp
#include <vector>
#include <unordered_map>
#include <queue>
#include <algorithm>
#include <iostream>

using namespace std;

class Solution {
public:
    vector<int> myAnswer(vector<int>& nums, int k) {
        unordered_map<int, int> m;
        for (const int& n: nums) {
            ++m[n];
        }

        vector<pair<int, int>> v(m.begin(), m.end());

        sort(v.begin(), v.end(), [](const pair<int, int>& a, const pair<int, int>& b) { return a.second > b.second; });

        int flag = 0;
        vector<int> sortedKeys;
        for (const auto& [key, val]: v) {
            if (flag++ == k) break;
            sortedKeys.push_back(std::move(key));
        }

        return sortedKeys;
    }

    vector<int> bestAnswer(vector<int>& nums, int k) {
        unordered_map<int, int> freqMap;
        for (const int& num : nums) {
            ++freqMap[num];
        }

        // Min-Heap (최소 힙) 사용하여 상위 K개만 유지
        priority_queue<pair<int, int>, vector<pair<int, int>>, greater<>> pq;

        for (const auto& [num, freq] : freqMap) {
            pq.push({freq, num});  // 힙에 (빈도, 숫자) 저장
            if (pq.size() > k) pq.pop(); // k개 초과 시 가장 작은 빈도를 제거
        }

        // 최종 k개의 요소를 결과 벡터에 저장
        vector<int> result;
        while (!pq.empty()) {
            result.push_back(pq.top().second);
            pq.pop();
        }

        reverse(result.begin(), result.end()); // 높은 빈도 순으로 정렬

        return result;
    }
};
```

# 성능 최적화

| **방법**                       | **시간 복잡도** | **공간 복잡도** |
| ------------------------------ | --------------- | --------------- |
| **기존 코드 (sort)**           | O(N log N)      | O(N)            |
| **개선 코드 (priority_queue)** | O(N log K)      | O(N + K)        |

## 현재 코드의 **시간 복잡도**

1. **빈도수 카운트 (unordered_map 이용)** → O(N)
2. **맵을 벡터로 변환** → O(N)
3. **벡터 정렬 (sort 이용, O(N log N))** → O(N log N)
4. **상위 k개 요소 선택** → O(K)

즉, 전체 **시간 복잡도는** O(N log N)이다.

## **최적화 방법**

1. sort(v.begin(), v.end(), ...)을 O(N log N) 시간에 수행하는 대신,
   - **최대 힙 (priority queue, O(N log K))**을 사용하면 더 효율적이다.
   - **최대 힙을 사용하면** 정렬이 필요 없고, O(N log K)에 k개의 최빈 원소를 추출할 수 있다.
2. 시간복잡도

   - **기존 코드**는 O(N log N)으로 전체 정렬 수행 → N이 크면 비효율적이다.
   - **개선 코드**는 O(N log K)로 상위 K개만 유지하는 **Min Heap**을 사용한다.

   → N이 커질수록 상대적으로 훨씬 빠름 (K가 작을 때 특히 효과적)

3. 공간복잡도
   - 개선 코드는 K만큼의 공간을 더 사용해서 기존 코드보다 조금 더 비효율적이다.

# Priority Queue

priority_queue는 C++ STL에서 제공하는 자료구조로, 우선순위 큐를 구현할 때 사용된다. 우선순위 큐는 **우선순위가 높은 요소를 먼저 처리하는 큐**다. 기본적으로 **최대 힙(Max Heap)**으로 동작하지만, greater나 less를 이용하여 **최소 힙(Min Heap)**으로도 사용할 수 있다.

## **특징**

- Heap
  - priority_queue는 힙을 기반으로 구현되며, 이는 **완전 이진 트리**로 구현되어 있다.
  - 부모 노드는 자식 노드보다 우선순위가 높다.

## **기본 구조**

priority_queue는 기본적으로 **최대 힙**으로 동작한다. 즉, 가장 큰 값이 우선순위가 높아 먼저 큐에서 제거됩니다. 기본적으로 다음과 같은 조건을 만족하는 자료구조이다.

- **최대 힙**: `priority_queue<int>`처럼 사용하면 가장 큰 값이 먼저 나온다.
- **최소 힙**: `priority_queue<int, vector<int>, greater<int>>`처럼 greater를 사용하면 가장 작은 값이 먼저 나온다.

## **사용 예시**

### **1. 최대 힙 (기본 동작)**

```cpp
#include <iostream>
#include <queue>
using namespace std;

int main() {
    priority_queue<int> pq;

    // 원소 삽입
    pq.push(10);
    pq.push(20);
    pq.push(15);

    // 우선순위가 높은 원소부터 출력
    while (!pq.empty()) {
        cout << pq.top() << " ";  // 가장 큰 값 출력
        pq.pop();  // 제거
    }
    // 출력: 20 15 10
}
```

위 예시에서 `priority_queue<int>`는 **최대 힙**으로 동작하므로, 가장 큰 값인 20이 먼저 출력된다.

### **2. 최소 힙 (greater 사용)**

```cpp
#include <iostream>
#include <queue>
using namespace std;

int main() {
    priority_queue<int, vector<int>, greater<int>> pq;

    // 원소 삽입
    pq.push(10);
    pq.push(20);
    pq.push(15);

    // 우선순위가 높은 원소부터 출력
    while (!pq.empty()) {
        cout << pq.top() << " ";  // 가장 작은 값 출력
        pq.pop();  // 제거
    }
    // 출력: 10 15 20
}
```

위 예시에서는 `priority_queue<int, vector<int>, greater<int>>`를 사용하여 **최소 힙**을 구현했다. 그 결과, 가장 작은 값인 10이 먼저 출력된다.

### **3. 사용자 정의 타입과 함께 사용**

priority_queue는 **사용자 정의 타입**을 저장할 수 있다. 이때, 우선순위를 어떻게 비교할지를 정의해줘야 한다. 예를 들어, **사람의 나이**를 기준으로 우선순위를 정하고 싶다면 다음과 같이 할 수 있다:

```cpp
#include <iostream>
#include <queue>
#include <vector>
using namespace std;

struct Person {
    string name;
    int age;

    // 우선순위 기준: 나이가 많은 사람이 먼저 오게
    bool operator<(const Person& other) const {
        return age < other.age;
    }
};

int main() {
    priority_queue<Person> pq;

    pq.push({"Alice", 30});
    pq.push({"Bob", 25});
    pq.push({"Charlie", 35});

    while (!pq.empty()) {
        cout << pq.top().name << " " << pq.top().age << endl;
        pq.pop();
    }
    // 출력: Charlie 35, Alice 30, Bob 25
}
```

Person 구조체에서 `operator<`를 정의하여 우선순위를 **나이가 많은 순서대로** 정리하도록 구현했다.

## **주요 메서드**

- `push(x)`: x 값을 큐에 삽입
- `pop()`: 큐에서 가장 우선순위가 높은 원소를 제거
- `top()`: 큐에서 가장 우선순위가 높은 원소를 반환 (제거는 하지 않음)
- `empty()`: 큐가 비었는지 여부를 반환
- `size()`: 큐에 있는 원소의 개수를 반환

## **시간 복잡도**

- **삽입** (push): O(log N)
- **삭제** (pop): O(log N)
- **가장 큰 값 확인** (top): O(1)

priority_queue는 삽입과 삭제 시에 **힙 구조를 유지하기 위해** 트리 구조를 재조정해야 하므로 삽입 및 삭제의 시간 복잡도가 O(log N)이다.
