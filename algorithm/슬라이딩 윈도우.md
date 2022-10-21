---
toc : true
title : "[Algorithm] 슬라이딩 윈도우"
category : 
    - Algorithm
---
슬라이딩 윈도우 알고리즘은 투 포인터와 비슷하게 구간을 훑으면서 지나간다는 공통점이 있으나 슬라이딩 윈도우 알고리즘의 경우 훑는 구간의 넓이가 동일하다는 차이가 있습니다.

슬라이딩 윈도우 알고리즘은 매번 처리되는 중복 요소를 재활용하므로써 낭비되는 계산을 하지 않음으로써 효율적인 처리가 가능하게 합니다.

배열 [2,4,7,10,8,4,5,6,7,1]에서 `길이가 3`인 서브배열의 합계가 가장 큰 서브배열을 찾는 문제로 예를 들어보겠습니다.

![sliding-window](/assets/images/algo/sliding-window-1.png)

길이가 3인 서브배열중 가장 큰 서브배열을 구하는 가장 단순한 방법은 각각 전체구간과 길이가 3인 구간을 훑는 두개의 for문을 중첩시켜 구현할 수 있습니다.

하지만 이 방법은 전체 배열의 길이가 n이고 구간이 m일때 O$($n x m) 시간이 걸리며 비효율적이라는 단점이 있습니다.

이때 우리가 발견할 수 있는것은 구간을 훑는 과정에서 공통요소가 존재한다는 것입니다.

![sliding-window](/assets/images/algo/sliding-window-2.png)

이러한 공통요소를 구간 합 계산시에 재활용 하면 보다 효율적인 처리가 가능합니다.

먼저 슬라이딩 윈도우를 활용하여 최대 서브배열합을 구하는 코드를 보겠습니다.

``` cpp
#include<iostream>
#include<algorithm>
using namespace std;

int max_sub_array(int (&arr)[6], int k)
{
    int window_sum = 0;
    int max_sum = 0;
    int window_start = 0;

    for(int window_end=0; window_end<7; window_end++)
    {
        window_sum += arr[window_end]; // 슬라이딩 인덱스 범위 요소 합산
        
        //슬라이딩 윈도우의 범위가 k 보다 커진 경우
        if(window_end >= (k - 1))
        {
            max_sum = max(max_sum, window_sum);
            window_sum -= arr[window_start];  // 슬라이드 윈도우 범위를 벗어난 요소를 합계에서 제거
            window_start += 1;  // 슬라이드 윈도우 시작 인덱스 증가
        }
    }
    return max_sum;
}

int main(void)
{
    int arr[]={2, 4, 7, 10, 8, 4};
    cout<<max_sub_array(arr, 3); // O/P : 25
    return 0;
}
```

max_sub_array 함수에서 사용된 변수들의 의미를 먼저 살펴보자면

- window_start: 슬라이딩 윈도우 시작 인덱스
- window_end: 슬라이딩 윈도우 끝 인덱스
- widdow_sum: 슬라이딩 윈도우 합계

`if(window_end >= (k-1))`에 의해 슬라이딩 윈도우의 구간


>[참고]:(https://blog.fakecoding.com/archives/algorithm-slidingwindow/)
