---
title: Minimum Number of Operations to Make Array XOR Equal to K
categories: [Algorithm]
tags: [array, xor, 배열, algorithm, bit manipulation]
---

### 생각할 포인트


### 접근방식
1. 일단 예제풀이
   - XOR 연산을 많이 안해봐서 일단 어떻게 해야 답을 도출할 수 있을지 예제를 풀어봤다. 풀다보니 1이 홀수개가 나오면 마지막이 1이 되는 것을 확인했다
2. 풀이과정
   - nums 배열의 값들을 모두 이진수로 바꾸고 다 더한다
   - 0이나 짝수는 0으로, 홀수는 1로 치환한다
   - 숫자들을 k의 이진수와 비교하며 다를때 answer(바꾼 횟수)++을 해준다

### 오답노트
1. 답은 맞았으나 시간이 너무 느리게 나왔다. 따라서 다른 풀이방법을 생각해봤다
2. 로직을 바꿔볼지 toBinaryString() 같은 함수를 직접 구현할지 고민했는데, 함수는 간단한거라 크게 걸릴 것 같지 않고 딱봐도 로직이 맘에 안들어서 로직을 바꿀 생각을 했다
3. 풀이과정 중 XOR이 교환법칙이 성립된다는 것을 알았다. 또 이항해도 된다는 것도 알았다. 일단 킵
4. XOR을 코딩하면서 써본적이 없어서 인터넷에 쳐봤다. ^을 사용하면 된단다. 아 본것같기도..?
5. toBinaryString()을 쓸 필요없이 ^로 모두 연산한 값(sum)만 k와 비교하면 됐다
6. 3번에서 해본 이항으로 sum도 sum ^ k로 만들어버렸다
7. sum ^ k에서 1의 개수만 카운트하면 됐다

### 전체코드

```java
class Solution {
    public int minOperations(int[] nums, int k) {
        int sum = getSum(nums);
        int answer = getAnswer(sum, k);
        return answer;
    }

    public int getSum(int[] nums){
        int sum = nums[0];
        int numsSize = nums.length;
        for(int i=1; i<numsSize; i++){
           sum = sum ^ nums[i];
        }
        return sum;
    }

    public int getAnswer(int sum, int k){
        int answer = 0;
        int num = sum ^ k;
        while(num/2 != 0){
            if(num%2==1) answer++;
            num /= 2;
        }
        if(num==1) answer++;
        return answer;
    }
}
```
