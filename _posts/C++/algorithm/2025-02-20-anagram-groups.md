---
title: "[NeetCode] Anagram Groups"

categories:
  - Algorithm
tags:
  - [C++, Arrays & Hashing]

toc: true
toc_sticky: true

date: 2025-02-20
last_modified_at: 2025-02-20
---

# 링크

[NeetCode: Anagram Groups](https://neetcode.io/problems/anagram-groups)

# 코드

```cpp
class Solution {
public:
    vector<vector<string>> myAnswer(vector<string>& strs) {
        map<string, vector<string>> m;
        for (const string& str: strs) {
            string s = str;
            sort(s.begin(), s.end());
            m[s].push_back(str);
        }

        vector<vector<string>> answer;
        for (const auto& [key, value]: m) {
            answer.push_back(value);
        }

        return answer;
    }

    vector<vector<string>> bestAnswer(vector<string>& strs) {
        unordered_map<string, vector<string>> m;

        for (const string& str : strs) {
            string key = getAnagramKey(str);
            m[key].push_back(str);
        }

        vector<vector<string>> answer;
        answer.reserve(m.size());

        for (auto& [_, value] : m) {
            answer.push_back(std::move(value));
        }

        return answer;
    }

private:
    string getAnagramKey(const string& s) {
        array<int, 26> count = {0};
        for (char c : s) {
            count[c - 'a']++;
        }

        string key;
        key.reserve(52);
        for (int i = 0; i < 26; ++i) {
            if (count[i] > 0) {
                key += 'a' + i;
                key += to_string(count[i]);
            }
        }
        return key;
    }
};
```

`myAnswer`가 내가 작성한 코드, `bestAnswer`가 ChatGPT가 성능을 최적화한 코드이다.

# 성능 최적화

|                                   | **기존 코드**    | **개선된 코드**       |
| --------------------------------- | ---------------- | --------------------- |
| 탐색 (`map` vs `unordered_map`)   | **O(log N)**     | **O(1)** (평균적으로) |
| 정렬 (`sort` vs counting sort)    | **O(K log K)**   | **O(K)**              |
| 최종 복사 (`push_back` vs `move`) | **O(N)**         | **O(N)**              |
| **총 시간복잡도**                 | **O(N K log K)** | **O(N K)**            |

## 1. **`unordered_map` 사용**

- key, value 자료구조를 사용할 때는 map보다는 unordered_map을 사용하도록 하자. 탐색 속도가 훨씬 많이 개선된다.

## 2. `sort()`의 시간복잡도 고려

- `std::sort()`는 O(N log N)의 시간복잡도를 가지므로, 더 빠른 정렬 방법이 있으면 그것을 사용하거나 구현하는 것이 좋다.

## 3. **`reserve()`로 벡터 성능 최적화**

- counting sort로 시간복잡도 최적화 하였다.
  입력값이 apple이라면,
  1. 내가 짠 코드는 aelpp로 정렬하였지만,
  2. ChatGPT는 a1e1l1p2 이런 key를 제공한다.
- `answer.reserve(m.size());`
  answer 벡터의 크기를 미리 예약하면, **동적 재할당을 방지**하여 push_back() 성능을 향상한다.
- `key.reserve(52);`
  key의 최대 길이는 26 \* 2 = 52 이므로 미리 공간을 확보하여 동적 할당을 줄인다.

## 4. **`move(value)` 사용**

- `answer.push_back(move(value));`
  - vector<string>을 복사하지 않고 **이동(move)하여 불필요한 메모리 복사를 방지한다.**
  - value는 for loop 이후로 더 사용하지 않아, 복사 보단 이동이 합리적이다.
- 사용하면 안되는 상황 (첫 번째 for loop)
  ```cpp
  for (const string& str : strs) {
      string key = getAnagramKey(str);
      m[key].push_back(str);
  }
  ```
  1. str은 **const string& (읽기 전용 참조)** 로 받아오기 때문에, std::move(str)를 사용하면 **컴파일 에러**가 발생한다.
  2. str을 **다른 곳에서 계속 사용할 가능성이 있을 때 그대로 복사**하는 것이 안전하다.
