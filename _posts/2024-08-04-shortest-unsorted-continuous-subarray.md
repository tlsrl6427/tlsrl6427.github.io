---
title: Shortest Unsorted Continuous Subarray
categories: [Algorithm]
tags: [array, greedy, two pointers, sorting]
---

[문제링크](https://leetcode.com/problems/shortest-unsorted-continuous-subarray/description/)

### 문제설명
Input: int[] nums<br>
Output: 바꿔야할 요소 개수<br>
<br>
nums 배열이 오름차순이 안되도록 하는 배열의 요소 개수를 리턴하면 된다.<br>
ex. [2,6,4,8,10,9,15] -> [6,4,8,10,9]가 오름차순이 된다면 nums 전체가 오름차순이 되기 때문에 5를 리턴한다.<br>
<br>

### 생각할 포인트
1. 같은 숫자에 대한 생각이 필요할 것 같다. (ex. [2,1,1,1])<br>
2. for문 하나에 끝낼 수 있을 것 같기도..?<br>
3. 개수를 구할때 오름차순으로 바꿔야하는 subarray의 시작 index와 끝 index를 통해 구할려고 한다. 즉, 투 포인터를 활용한다.<br>
<br>

### 접근방식
![img1](/assets/img/2024-08-04-shortest-unsorted-continuous-subarray/img1.jpg)
1. 시작하는 부분 즉, 내림차순이 시작되는 부분을 찾는다.<br>
2. 시작하는 부분부터 끝까지 가며 끝 인덱스의 값을 계속 바꾼다.<br>
&emsp;   2-1. nums[i-1] > nums[i] 일 경우<br>
&emsp;   2-2. 내림차순 subarray의 최대값보다 nums[i]가 작을 경우<br>
3. 시작하는 부분부터 처음까지 가며 처음 인덱스의 값을 바꾼다.<br>
&emsp;   3-1. nums[i-1] == nums[i] 일 경우<br>
&emsp;   3-2. 내림차순 subarray의 최소값보다 nums[i]가 클 경우<br>
<br>

### 오답노트
1. 접근방식 2-2를 고려하지 않았다.(ex. [5,2,2,2,3])
2. 접근방식 3을 고려하지 않았다.(ex. [2,2,2,1,3])
<br>

### 전체코드
```java
class Solution {
    public int findUnsortedSubarray(int[] nums) {
        int size = nums.length;

        if(size==1) return 0;

        int startIdx = -1;
        int endIdx = 0;
        int maxNum = -10001;
        int minNum = 10001;
        for(int i=1; i<size; i++){
            if(nums[i] < nums[i-1]){
                startIdx = i-1;
                endIdx = i;
                maxNum = nums[i-1];
                minNum = nums[i];
                break;
            }
        } 

        if(startIdx == -1) return 0;

        for(int i=endIdx + 1; i<size; i++){
            if(maxNum < nums[i]) maxNum = nums[i];
            if(minNum > nums[i]) minNum = nums[i];
            if(nums[i] < nums[i-1]){
                endIdx = i;
            }else{
                if(maxNum > nums[i]) endIdx = i;
            }
        }

        for(int i=startIdx; i>=1; i--){
            if(nums[i] == nums[i-1] || nums[i-1] > minNum){
                startIdx = startIdx - 1;
            }else break;
        }

        System.out.println("startIdx: " + startIdx);
        System.out.println("endIdx: " + endIdx);
        System.out.println("maxNum: " + maxNum);

        return endIdx - startIdx + 1;
    }
}
```

딱봐도 직관적이지도 않고 속도와 저장공간의 효율도 좋지 않아서 후다닥 Best Solution을 보러갔다.<br>
<br>

### Best Solution과의 비교

1. 직관성 goat( O(NlogN) )
   
```java
class Solution {
    public int findUnsortedSubarray(int[] nums) {
        int[] arr=new int[nums.length];
        System.arraycopy(nums,0,arr,0,nums.length); // 1. 배열 복사
        Arrays.sort(arr); // 2. 복사한 배열 정렬

        int start=0, end=nums.length-1;
        for(;start<nums.length;start++){
            if(nums[start]!=arr[start])break; // 3. 처음부터 끝까지 nums와 정렬한 배열 비교해서 달라지는 곳 찾기
        }
        if(start>=nums.length-1) return 0; // 4. 끝까지 비교했는데 똑같으면 return 0;
        for(;end>=0;end--){
            if(nums[end]!=arr[end])break; // 5. 끝부터 처음까지 nums와 정렬한 배열 비교해서 달라지는 곳 찾기
        }
        return end-start+1;
    }
}
```

&nbsp;간단하게 어차피 오름차순 정렬이 되어야하는 거니까 오름차순 정렬을 시켜보고, 다른 부분을 찾는 것이다.<br>
정렬하는데에는 O(NlogN)의 시간이 들고 비교하는데에는 O(N)의 시간이 들기 때문에 결론적으로 O(NlogN)의 시간이 들지만, 직관성이 좋은 접근이다.<br>
[정답 링크](https://leetcode.com/problems/shortest-unsorted-continuous-subarray/solutions/3002154/easy-solution-well-explained-o-n-o-1-microsoft/)<br>
<br>

2. 효율성 goat( O(N) )

```java
public int findUnsortedSubarray(int[] A) {
    int n = A.length, beg = -1, end = -2, min = A[n-1], max = A[0];
    for (int i=1;i<n;i++) {
      max = Math.max(max, A[i]);
      min = Math.min(min, A[n-1-i]);
      if (A[i] < max) end = i;
      if (A[n-1-i] > min) beg = n-1-i; 
    }
    return end - beg + 1;
}
```

처음 봤을때는 조금 헷갈렸는데 for문 안의 max, min부분을 나누면 조금 보기 편했다.<br>

```java
// 처음부터 끝까지 가며 end를 조정
for (int i=1; i<n; i++) {
    max = Math.max(max, A[i]);
    if (A[i] < max) end = i;
}

// 끝부터 처음까지 가며 beg를 조정
for (int i=n-1; i>=0; i--) {
    min = Math.min(min, A[i]);
    if (A[i] > min) beg = i; 
}
```

아래 for문의 인덱스를 i -> n-1-i로 바꿈으로써 하나의 for문으로 합친 것이다. 어렴풋이 for문 하나로도 될 것 같기는 했는데 이 방법을 끝까지 생각못해서 내껀 코드가 조금 더러워졌다.<br>
이 해답의 접근은 max쪽은 max를 갱신하며 max보다 작으면 end에 i값을 넣는 것이고, min쪽은 min을 갱신하며 min보다 작으면 beg에 n-1-i를 넣는 것이다.<br>
max값의 의미는<br>
1. 전 인덱스가 크면(내림차순이면) end값을 갱신한다의 의미와<br>
2. 지금 당장은 오름차순이어도 전전전전전전 인덱스로 거슬러 올라갔을 때 지금보다 큰 숫자가 있다면 크게 내림차순으로 바꿔야하므로 max값을 기억해놓고 있는 것이다.<br>

min값도 반대로 동일하다.<br>
<br>
또한 초기값을 beg는 -1, end는 -2로 해놓은 이유는 이미 오름차순으로 정렬이 되어있을때 둘의 값은 변화가 없을 것이고, return값인 end - beg + 1이 0이 되게 하기 위함이다.<br>
<br>
결과적으로 for문 하나만 돌리기 때문에 속도는 O(N)이 나온다.<br>
[정답 링크](https://leetcode.com/problems/shortest-unsorted-continuous-subarray/solutions/103057/java-o-n-time-o-1-space/)<br>
<br>
뭔가 더 깨끗히 만들 수 있을 줄은 알았는데 문제푸는 도중에 머리가 한번 꼬여버리고 코어로직이 확신이 없어서 Best Solution에 접근하지 못했다.<br>
골머리를 더 썩히는 것보다 답을 보는게 나을 것 같아서 봤는데 이번 같은 경우에는 보는게 정신건강에 좋았던 것 같다. 분명 예전에는 깔끔하게 풀었을 것 같은데 이번에는 그러지 못했고, 나름 시간을 더 줬는데도 만족할만한 결과는 안나왔다.
상심하지말고 이번에 알았으니 다음에 비슷한 유형의 문제가 나왔을 때 깔끔하게 푸는 것을 목표로 하고 넘어가야겠다.
