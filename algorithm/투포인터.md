---
toc : true
title : "[Algorithm] 투 포인터"
category :
    - Algorithm
---
`투 포인터 알고리즘`이란 1차원 배열의 각기 다른 원소를 가리키는 두개의 포인터를 조작해가며 원하는 값을 얻는 알고리즘을 말합니다.

예를 들어 N개의 수로 된 수열 A[1], A[2], ... A[N]이 있고 이 수열의 i번째 부터 j번째 수 까지의 합 A[i] + A[i+1] + … + A[j-1] + A[j]가 M이 되는 경우의 수를 구하는 프로그램을 작성해 보겠습니다. 

이러한 문제를 모든 경우를 확인 하는 중첩 for문을 사용하여 해결할 경우 시간 복잡도는 O$($N<sup>2</sup>)이 된다. 하지만 투 포인터 알고리즘을 사용하여 문제를 해결하면 O$($N) 시간 안에 문제를 해결할 수 있습니다.

### 투 포인터 활용 과정
N=10, M=5
10개의 수로 이루어진 수열이 있고 이 수열의 i부터 j번째 수까지의 합이 5인 경우가 몇가지인지 투 포인터 알고리즘을 사용하여 알아보겠습니다.

맨 처음 배열의 첫번째 인덱스를 가리키는 두개의 포인터 s, e를 정의 합니다. 
1. s에서 e까지의 부분합이 M$($5)보다 작다면 `e를 오른쪽으로 이동시켜 부분합을 증가시켜 주고(e++)` 
<br>

2. s에서 e까지의 부분합이 M$($5) 이상이며 e가 맨 마지막 원소에 도달 했다면$($e==N) s를 오른쪽으로 이동 시켜줘서 부분합을 줄여줍니다.
<br>

3. 만약 현재 s에서 e까지의 부분합이 M$($5)과 같다면 ans++

위의 1~3 과정을 s < N 일 때까지 반복해 주면 됩니다.

![투포인터](/assets/images/algo/twoPointer-1.png)

초기 상태입니다. s에서 e까지의 부분합인 e가 오른쪽으로 움직이면 새로 s와 e사이의 구간에 들어온 원소값을 sum에 더해주고 s가 오른쪽으로 이동하면 이동하면서 구간에서 제외된 원소값을 sum에서 빼주는 방식으로 sum을 매번 구할 수 있습니다.

![투포인터](/assets/images/algo/twoPointer-2.png)

sum < M 이므로 e를 계속 오른쪽으로 이동하다 보면 아래와 같이 sum = 6 이 됩니다.

![투포인터](/assets/images/algo/twoPointer-3.png)

sum >= M 이 되었으므로 s를 오른쪽으로 이동시켜 줍니다. $($s++)

![투포인터](/assets/images/algo/twoPointer-4.png)

sum == M$($5)인 경우를 만났으므로 결과값 ans를 1증가 시켜줍니다.

![투포인터](/assets/images/algo/twoPointer-5.png)

위와 같은 방법을 반복하다가 s == N이 되었을 때 반복을 종료 시켜주고 결과값을 출력해 주면 됩니다.

![투포인터](/assets/images/algo/twoPointer-6.png)

### 소스코드

``` cpp
#include<iostream>
using namespace std;

int main(void)
{
    ios::sync_with_stdio(false); cin.tie(0); cout.tie(0);
    int n, m, s, e, ans, sum;
    int arr[10001];
    cin>>n>>m;
    for(int i=0; i<n; i++) cin>>arr[i];
    s=e=ans=sum=0;
    while(true)
    {
        if(sum>=m) sum-=arr[s++];
        else if(e==n) break;
        else sum+=arr[e++];
        if(sum==m) ans++;
    }
    cout<<ans;
}
```

### 풀어보면 좋은 문제
- [수들의 합2 $($BOJ_2003)](https://www.acmicpc.net/problem/2003)