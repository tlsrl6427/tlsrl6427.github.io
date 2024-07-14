---
title: Minimum Number of Operations to Make Array XOR Equal to K
categories: [Algorithm]
tags: [array, xor, 배열, algorithm, bit manipulation]
---

[문제링크](https://leetcode.com/problems/minimum-number-of-operations-to-make-array-xor-equal-to-k/submissions/1317056678)
<br>

### 문제설명
주어지는 변수: int[] nums, int k
1. nums에 있는 수와 k를 이진수로 바꾼다
2. nums에 있는 수를 모두 XOR로 연산한 값이 k와 같아야한다
3. 이때 k와 같아지기 위해 nums의 이진수값을 자유롭게 0->1, 1->0으로 바꿀 수 있다
4. 최소한의 변경횟수가 답이다
<br>

### 생각할 포인트
1. nums 배열의 특정 수를 바꾸는 게 중요한 것이 아니라 전체적으로 최소 몇번 바꾸는지가 중요하다
<br>

### 접근방식
1. 일단 예제풀이
   - XOR 연산을 많이 안해봐서 일단 어떻게 해야 답을 도출할 수 있을지 예제를 풀어봤다. 풀다보니 1이 홀수개가 나오면 마지막이 1이 되는 것을 확인했다
2. 풀이과정
   - nums 배열의 값들을 모두 이진수로 바꾸고 다 더한다
   - 0이나 짝수는 0으로, 홀수는 1로 치환한다
   - 숫자들을 k의 이진수와 비교하며 다를때 answer(바꾼 횟수)++을 해준다<br>
![img1](/assets/img/2024-07-11-minimum-number-of-operations-to-make-array-xor-equal-to-k/img1.jpg)
<br>

### 오답노트
![img2](/assets/img/2024-07-11-minimum-number-of-operations-to-make-array-xor-equal-to-k/img2.png)
1. 답은 맞았으나 시간이 너무 느리게 나왔다. 일단 출력문들을 지워봤는데 큰 차이는 없었다. 따라서 다른 풀이방법을 생각해봤다
2. 로직을 바꿔볼지 toBinaryString() 같은 함수를 직접 구현할지 고민했는데, 함수는 간단한거라 크게 걸릴 것 같지 않고 딱봐도 로직이 맘에 안들어서 로직을 바꿀 생각을 했다
3. 풀이과정 중 XOR이 교환법칙이 성립된다는 것을 알았다. 또 이항해도 된다는 것도 알았다. 일단 킵
4. XOR을 코딩하면서 써본적이 없어서 인터넷에 쳐봤다. ^을 사용하면 된단다. 아 본것같기도..?
5. toBinaryString()을 쓸 필요없이 ^로 모두 연산한 값(sum)만 k와 비교하면 됐다
6. 3번에서 해본 이항으로 sum도 sum ^ k로 만들어버렸다
7. sum ^ k에서 1의 개수만 카운트하면 됐다
<br>

### 전체코드

```java
class Solution {
    public int minOperations(int[] nums, int k) {
        int sum = getSum(nums);
        int answer = getAnswer(sum, k);
        return answer;
    }

    // nums배열 모든 수 XOR한 값(sum) 리턴
    public int getSum(int[] nums){
        int sum = nums[0];
        int numsSize = nums.length;
        for(int i=1; i<numsSize; i++){
           sum = sum ^ nums[i];
        }
        return sum;
    }

    //sum ^ k 를 이진수로 바꾸면서 1이 들어간 횟수 카운트
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

### Best Solution과의 비교
```java
class Solution {
    public int minOperations(int[] nums, int k) {
        int finalXor = 0;
        // XOR of all integers in the array.
        for (int n : nums) {
            finalXor = finalXor ^ n;
        }
        
        int count = 0;
        // Keep iterating until both k and finalXor becomes zero.
        while (k > 0 || finalXor > 0) {
            // k % 2 returns the rightmost bit in k,
            // finalXor % 2 returns the rightmost bit in finalXor.
            // Increment counter, if the bits don't match.
            if ((k % 2) != (finalXor % 2)) {
                count++;
            }
            
            // Remove the last bit from both integers.
            k /= 2;
            finalXor /= 2;
        }
        
        return count;
    }
}
```
1. 크게 두 부분으로 나뉘어서 sum과 sum^k의 값을 구하는 것은 유사하다
2. sum을 구하는 부분에서 나는 시작값을 nums[0]으로 했는데 생각해보니 XOR 연산이니 0으로 시작해도 됐겠다
3. 아래 부분은 시간이 오래걸린 로직과 똑같은데, 두 값이 다르면 바꾸고 카운트해야된다는 소리이다. XOR 연산 자체가 같으면 0, 다르면 1을 나타내기 때문에 나는 저렇게 k와 finalXor을 따로 안하고 k ^ finalXor을 한 후에 판단했다. 연산차이는 크지 않을 것 같은데 뭐가 좋을지 굳이 따지자면 간결성은 내 것이 좋으나 다른 사람이 코드를 보고 이해를 할때는 이렇게 적는 것이 좋을 것 같다
<br>

* if문을 안쓴 이유
  
![img3](/assets/img/2024-07-11-minimum-number-of-operations-to-make-array-xor-equal-to-k/img3.png)
