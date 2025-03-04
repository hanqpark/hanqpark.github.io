---
title: "[NeetCode] Encode and Decode Strings"

categories:
  - Algorithm
tags:
  - [C++, Arrays & Hashing]

toc: true
toc_sticky: true

date: 2025-03-03
last_modified_at: 2025-03-04
---

# 링크

[NeetCode: Encode and Decode Strings](https://neetcode.io/problems/string-encode-and-decode)

# 코드

```cpp
#include <string>
#include <vector>
#include <sstream>
#include <numeric>

using namespace std;

class Solution {
public:
    string myEncode(vector<string>& strs) {
        string answer;
        for (const string& str: strs) {
            answer += str;
            answer += "-";
        }
        return answer;
    }

    string bestEncode(vector<string>& strs) {
        string answer;
        answer.reserve(accumulate(strs.begin(), strs.end(), 0, [](int sum, const string& str) { return sum + str.size() + 1; }));

        for (const string& str : strs) {
            answer.append(str);
            answer.push_back('-');
        }
        return answer;
    }

    vector<string> myDecode(string s) {
        string str;
        stringstream ss(s);
        vector<string> answer;

        while (getline(ss, str, '-')) answer.push_back(str);

        return answer;
    }

    vector<string> bestDecode(string s) {
        vector<string> answer;
        size_t start = 0, end;

        while ((end = s.find('-', start)) != string::npos) {
            answer.emplace_back(s.substr(start, end - start));
            start = end + 1;
        }
        return answer;
    }
};
```

# 성능 최적화

| **함수** | **기존 코드 시간 복잡도** | **개선 코드 시간 복잡도** |
| -------- | ------------------------- | ------------------------- |
| encode   | O(n + m)                  | O(n + m)                  |
| decode   | O(m)                      | O(m)                      |
| **전체** | **O(n + m)**              | **O(n + m)**              |

- 시간 복잡도는 동일하지만, 개선 코드가 더 효율적이다.
- 메모리 재할당과 불필요한 연산을 줄여 성능을 향상한다.

## 현재 코드의 시간 복잡도

### encode 함수: **O(n + m)**

- 주어진 `vector<string>& strs`를 순회하며 answer 문자열에 각 문자열을 추가하고 "-"를 붙인다.
- strs의 원소 개수를 n, 모든 문자열의 총 길이를 m이라 하면
  1. `answer += str` 수행 시 문자열이 계속 증가하면서 재할당이 발생 → **O(m)**
  2. `answer += "-"` 역시 같은 이유로 **O(n)**

### **decode 함수: O(m)**

- `stringstream ss(s)`를 이용해 "-" 기준으로 문자열을 나눈다.
- 입력 문자열의 길이를 m이라 하면
  1. `getline(ss, str, '-')`는 문자열을 탐색하면서 분할 → **O(m)**
  2. 벡터에 `push_back(str)` → **O(n)** (최대 n개의 원소)

**최종적으로 전체 코드의 시간 복잡도는 O(n + m)** 이다.

## 최적화 방법

### encode 함수

- `answer.reserve(...)`를 사용하여 문자열의 총 길이를 미리 할당하여 **메모리 재할당을 최소화** 하였다.
  - **메모리 복사 비용 감소**
- `answer.append(str) + answer.push_back('-')` 사용하여 **문자열 연산을 효율적으로 수행**하였다.

### **decode 함수**

- stringstream 대신 `s.find('-')`와 `substr()`을 사용하여 **불필요한 stringstream 연산을 제거**하였다.
- `emplace_back()` 사용하여 **벡터 내부 복사 비용 감소 →** `push_back()`보다 빠르다.

# StringStream

문자열을 파싱할 때 `std::stringstream`과 `std::string::find() + substr()` 중 어느 것이 더 성능이 좋은지 확인해 보자.

## **1. 두 방식의 차이**

### **`std::stringstream`을 이용한 문자열 파싱**

```cpp
std::stringstream ss("apple,banana,grape");
std::string token;
while (std::getline(ss, token, ',')) {
    std::cout << token << std::endl;
}
```

- `std::stringstream`은 내부적으로 문자열을 버퍼(`token`)에 저장한다.
- `std::getline()`을 호출하면, 내부적으로 버퍼에서 `,`를 찾고 문자열을 분리한다.

### **`std::string::find() + substr()`을 이용한 파싱**

```cpp
std::string s = "apple,banana,grape";
size_t start = 0, end;

while ((end = s.find(',', start)) != std::string::npos) {
    std::cout << s.substr(start, end - start) << std::endl;
    start = end + 1;
}
std::cout << s.substr(start) << std::endl; // 마지막 토큰 출력
```

- `s.find(',')`로 구분자의 위치를 찾는다.
- `substr(start, end - start)`로 부분 문자열을 생성한다.
- `start`를 이동하여 반복한다.

## **2. 시간 복잡도 비교**

| **연산**         | std::stringstream           | std::string::find() **+** substr() |
| ---------------- | --------------------------- | ---------------------------------- |
| 구분자 찾기      | O(n) (내부적으로 find 사용) | O(n) (find() 직접 호출)            |
| 문자열 복사      | O(k)                        | O(k)                               |
| 전체 시간 복잡도 | **O(n)**                    | **O(n)**                           |

- 이론적으로는 큰 차이가 없다.
- `std::stringstream`도 내부적으로 `std::string::find()` 같은 방식으로 작동한다.

## **3. 실제 성능 차이**

이론적으로는 비슷하지만, 실제 구현에서 차이가 발생하는 이유는 아래와 같다.

1. **메모리 할당 (Memory Allocation)**
   - `std::stringstream`은 내부적으로 동적 버퍼를 사용하기 때문에 **메모리 재할당**이 발생할 가능성이 있다.
   - `std::string::find() + substr()`은 원본 문자열을 직접 조작하여 메모리 할당 비용이 줄어든다.
2. **캐시 효율성 (Cache Efficiency)**
   - `std::string::find()`는 원본 문자열을 직접 검색하기 때문에 CPU 캐시에 더 효율적일 가능성이 크다.
   - `std::stringstream`은 내부적으로 버퍼를 관리하면서 추가적인 연산이 필요하므로 성능이 저하될 수 있다.

## **4. 성능 벤치마크**

실제 C++ 코드로 실행하여 비교해 보면 `std::string::find() + substr()`이 `std::stringstream`보다 빠른 경우가 많다.

### **실험 코드**

```cpp
#include <iostream>
#include <sstream>
#include <vector>
#include <string>
#include <chrono>

using namespace std;
using namespace chrono;

void test_stringstream(const string& s) {
    stringstream ss(s);
    string token;
    vector<string> tokens;
    while (getline(ss, token, ',')) {
        tokens.push_back(token);
    }
}

void test_find_substr(const string& s) {
    size_t start = 0, end;
    vector<string> tokens;
    while ((end = s.find(',', start)) != string::npos) {
        tokens.push_back(s.substr(start, end - start));
        start = end + 1;
    }
    tokens.push_back(s.substr(start));
}

int main() {
    string s;
    for (int i = 0; i < 100000; ++i) {
        s += "word,";
    }
    s.pop_back(); // 마지막 콤마 제거

    auto start1 = high_resolution_clock::now();
    test_stringstream(s);
    auto end1 = high_resolution_clock::now();
    cout << "stringstream: " << duration_cast<milliseconds>(end1 - start1).count() << "ms" << endl;

    auto start2 = high_resolution_clock::now();
    test_find_substr(s);
    auto end2 = high_resolution_clock::now();
    cout << "find + substr: " << duration_cast<milliseconds>(end2 - start2).count() << "ms" << endl;

    return 0;
}
```

### **실험 결과**

```
stringstream: 38ms
find + substr: 19ms
```

## **5. 결론**

| **방법**                       | **장점**                | **단점**                             | **추천 사용 경우**               |
| ------------------------------ | ----------------------- | ------------------------------------ | -------------------------------- |
| std::stringstream              | 코드가 간결하고 직관적  | 속도가 느릴 수 있음 (동적 버퍼 관리) | 소규모 문자열 처리 시            |
| std::string::find() + substr() | 메모리 할당이 적고 빠름 | 코드가 길어질 수 있음                | 대량의 문자열을 빠르게 처리할 때 |

- **`std::stringstream`은 코드 가독성 및 간결한 코드가 필요할 때 사용**하자.
- **`std::string::find() + substr()`은 성능이 중요한 경우** 혹은 **최적화**를 원할 때 사용하자**.**
