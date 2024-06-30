---
title: Add Two numbers
categories: [Algorithm]
tags: [linked list, 링크드 리스트]
---

[문제링크](https://leetcode.com/problems/add-two-numbers/submissions/1300709064)

## 생각할 포인트
1. 링크드리스트이기 때문에 처음부터 총 몇개인지를 알 수 없어 while문을 써야겠다는 생각을 했다
2. 두 노드를 더한 값이 있는 새 노드를 만들돼, 더한 값이 10이 넘는다면 기억해뒀다가 다음값에 추가한다
3. 두 리스트의 길이가 다르기 때문에 끊긴 쪽은 null값 처리한다
4. while문을 탈출할 조건을 생각한다(두 리스트가 모두 null일 때 탈출)


## 접근방식
![images1](/assets/img/2024-06-26-add-two-numbers/images1.jpg){: w="600" h="600" }
- 제출용 변수와 실제로 움직일 변수를 선언한다
- 새 노드를 생성한 후 더한 값을 넣고, 자릿수가 넘어가는 값은 따로 저장한다


![images2](/assets/img/2024-06-26-add-two-numbers/images2.jpg){: w="600" h="600" }
- 새 노드를 생성하고, 전 노드의 next에 넣은 후 더한 값을 넣는다
- 자릿수가 넘어갔으니 따로 저장한다


![images3](/assets/img/2024-06-26-add-two-numbers/images3.jpg){: w="600" h="600" }
- 넘어간 자릿수를 추가로 더한다

## 오답노트
1. 마지막에 자릿수가 1이 남으면 추가로 노드를 하나 더 생성해서 붙여야했다


## 전체코드
```java
class Solution {
    public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
        int expandedNum = 0;
        ListNode listNodeA = l1;
        ListNode listNodeB = l2;
        ListNode node = new ListNode();
        ListNode answer = node;

        while(true){
            
            int l1Val = listNodeA != null ? listNodeA.val : 0;
            int l2Val = listNodeB != null ? listNodeB.val : 0;
            int totalNum = l1Val + l2Val + expandedNum;

            if(totalNum>=10){
                expandedNum = 1;
                totalNum -= 10;
            }else expandedNum = 0;
            node.val = totalNum;

            if(listNodeA != null && listNodeA.next != null) listNodeA = listNodeA.next;
            else listNodeA = null;
            if(listNodeB != null && listNodeB.next != null) listNodeB = listNodeB.next;
            else listNodeB = null;

            if(listNodeA==null && listNodeB==null) break;
            else{
                node.next = new ListNode();
                node = node.next;
            }
        }

        if(expandedNum==1) node.next = new ListNode(1);

        return answer;
    }
}
```


## Best Solution과의 비교
- 속도와 저장공간은 비슷했다
-  while 바로 옆 조건절에 두 노드 모두 끝이면 탈출하는 구문과, 오답노트에 있던 자릿수가 남아있을 경우에 대한 구문을 추가해서 간소화시켰다
