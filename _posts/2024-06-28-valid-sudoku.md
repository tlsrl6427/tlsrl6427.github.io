---
title: Valid Sudoku
categories: [Algorithm]
tags: [array, matrix, 배열]
---

[문제링크](https://leetcode.com/problems/valid-sudoku/submissions/1302676428)

### 생각할 포인트
1. 중복확인을 위해 Set을 사용한다
2. 중복을 만날 경우 바로 false를 return한다

### 접근방식
![image1](/assets/img/2024-06-28-valid-sudoku/image1.jpg)
- 행, 열, 정사각형에 대해 각각 for문을 돌린다
- 값을 Set에 넣으며 중복을 비교한다

### 오답노트
1. "."에 대한 중복처리를 하지 않아서 중복이 아닌 경우도 중복처리 되었다

### 전체코드
```java
class Solution {
    public boolean isValidSudoku(char[][] board) {
        
        //행
        Set s = new HashSet();
        for(int i=0; i<9; i++){
            for(int j=0; j<9; j++){
                char v = board[i][j];
                if(v=='.') continue;
                if(!s.contains(v))
                    s.add(v);
                else return false;
            }
            s.clear();
        }

        //열
        for(int i=0; i<9; i++){
            for(int j=0; j<9; j++){
                char v = board[j][i];
                if(v=='.') continue;
                if(!s.contains(v))
                    s.add(v);
                else return false;
            }
            s.clear();
        }

        //정사각형
        //00 03 06
        //30 33 36
        //60 63 66
        for(int r=0; r<9; r+=3){
            for(int c=0; c<9; c+=3){

                for(int i=r; i<r+3; i++){
                    for(int j=c; j<c+3; j++){
                        char v = board[i][j];
                        if(v=='.') continue;
                        if(!s.contains(v))
                            s.add(v);
                        else return false;
                    }
                }
                s.clear();
            }
        }
        
        return true;
    }
}
```

### Best Solution과의 비교
```java
public boolean isValidSudoku(char[][] board) {
    Set seen = new HashSet();
    for (int i=0; i<9; ++i) {
        for (int j=0; j<9; ++j) {
            char number = board[i][j];
            if (number != '.')
                if (!seen.add(number + " in row " + i) ||
                    !seen.add(number + " in column " + j) ||
                    !seen.add(number + " in block " + i/3 + "-" + j/3))
                    return false;
        }
    }
    return true;
}
```
- Runtime은 내 것이 3ms, Best Soultion이 13ms로 Best Solution이 더 오래걸리지만 코드가 압도적으로 간소화되었다
- 이중 for문을 돌리며 String Set을 이용해 중복체크를 하고 있다.
- 나는 contains로 이미 수가 존재하는지 확인했는데, add()의 반환값으로 오는 boolean으로 if문을 확인하고 있다. 이를 통해 코드를 조금 더 간소화시킬 수 있을 것 같다

