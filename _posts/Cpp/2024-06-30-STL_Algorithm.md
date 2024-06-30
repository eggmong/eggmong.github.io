---
title:  "STL 알고리즘 for 코딩테스트"

categories:
  - Cpp
tags:
  - [Cpp, C++, STL]

toc: true
toc_sticky: true
 
date: 2024-06-30
last_modified_at: 2024-06-30
---

<br>

# count() 함수로 횟수 세기

시간 복잡도는 O(N).

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

using namespace std;

int main()
{
    vector<int> v = {1, 4, 3, 4, 5,4, 5};
    int ret = count(v.begin(), v.end(), 5);
    // 5 라는 값이 벡터 v에 몇 번 나타나는지 세기
}
```

# sort() 함수로 정렬하기

시간 복잡도는 O(NlogN)

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

using namespace std;

int main()
{
    vector<int> v = {4, 2, 5, 3, 1};

    sort(v.begin(), v.end());
    // 벡터 v를 오름차순으로 정렬
    // 1, 2, 3, 4, 5

    sort(v.rbegin(), v.rend());
    // 내림차순으로 정렬
    // 5, 4, 3, 2, 1
}
```

## 사용자 정의 비교 함수 사용하기

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

using namespace std;

struct Point
{
    int x, y;
    Point(int x, int y) : x(x), y(y) {}
};

// 사용자 정의 비교 함수
bool compare(const Point &a, const Point &b)
{
    if (a.x == b.x)
    {
        return a.y < b.y;
        // x 좌표가 같으면, y 좌표가 작은 순서대로 정렬
    }

    return a.x < b.x;
    // x 좌표가 작은 순서대로 정렬
}

int main()
{
    sort (points.begin(), points.end(), compare);

    for (const Point &p : points)
    {
        cout << "(" << p.x << ", " << p.y << ") ";
    }
    // 출력값 : (1, 2), (2, 5), (3, 1), (3, 4)
}
```

# next_permutation() 함수로 순열 생성하기

가능한 모든 순열을 생성한다.  
단 데이터가 <b>사전순으로 정렬</b>된 상태여야 한다.  

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

using namespace std;

int main()
{
    vector<int> v = {1, 2, 3};

    do
    {
        for (int i : v)
        {
            cout << i << endl;
        }

        cout << endl;
    } while (next_permutation(v.begin(), v.end()));

    return 0;

    // 출력값
    // 1 2 3
    // 1 3 2
    // 2 1 3
    // 2 3 1
    // 3 1 2
    // 3 2 1
}
```

# unique() 함수로 중복 정리하기

컨테이너 내 중복되는 원소들을 뒤로 밀어내고 중복되지 않은 원소들만 남겨 새로운 범위의 끝 반복자를 반환한다.  
삭제하는 건 아님!  

시간복잡도는 O(N).

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

using namespace std;

int main()
{
    vector<int> v = {1, 2, 2, 3, 3, 3, 4, 4, 5, 5, 5};

    auto newEnd = unique(v.begin(), v.end());
    // unique 함수를 사용하여 중복 요소 제거

    for (auto it = v.begin(); it != newEnd; ++it)
    {
        cout << *it << "";
    }
    cout << endl;
    // 1, 2, 3, 4, 5

    cout << v.size() << endl;
    // 벡터의 크기 : 11
}
```

# binary_search() 함수로 이진 탐색하기

단 원소가 이미 정렬되어 있어야 동작한다.  
O(logN)  

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

using namespace std;

int main()
{
    vector<int> v = {1, 2, 3, 4, 5};

    cout << binary_search(v.begin(), v.end(), 3) << endl;  // 1
    cout << binary_search(v.begin(), v.end(), 7) << endl;  // 0

    return 0;
}
```