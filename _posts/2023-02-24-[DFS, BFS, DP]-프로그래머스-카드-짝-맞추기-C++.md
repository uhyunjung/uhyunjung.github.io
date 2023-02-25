---
title: '[DFS+BFS+DP, BFS] 프로그래머스 카드 짝 맞추기 C++'
author: HJ
date: 2023-02-24 15:40:00 +0900
categories: [코딩테스트, 그래프|트리]
tags: [DFS, BFS, DP]
math: true
mermaid: true
---

## Links
<https://school.programmers.co.kr/learn/courses/30/lessons/72415>

## DFS+BFS+DP
```c++
#include <string>
#include <vector>
#include <queue>

using namespace std;

struct Position {
    int x, y;
};

int n, card;
int dx[4] = {0, 0, -1, 1};
int dy[4] = {-1, 1, 0, 0};
vector<vector<int>> Board;
vector<Position> idx[7];
bool check[7];

int bfs(Position s, Position e) {
    int answer = 1e9;
    int dist[n][n]; // visited는 dx, dy 순서에 따라 안 될 가능성O
    fill(&dist[0][0], &dist[n-1][n], 1e9);
    
    queue<Position> q;
    q.push(s);
    dist[s.y][s.x] = 1; // Enter 초기화
    while(!q.empty()) {
        Position now = q.front();
        q.pop();
        
        if ((now.x == e.x) && (now.y == e.y)) {
            answer = min(answer, dist[e.y][e.x]);
            continue;
        }
        
        for(int i=0; i<4; i++) {
            // 한칸 이동
            Position pos = {now.x + dx[i], now.y + dy[i]};
            if ((pos.x < 0) || (pos.y < 0) || (pos.x >= n) || (pos.y >= n)) continue;
            if (dist[pos.y][pos.x] < dist[now.y][now.x] + 1) continue;
            dist[pos.y][pos.x] = dist[now.y][now.x] + 1;
            q.push(pos);
            
            // ctrl 이동
            while(Board[pos.y][pos.x] == 0) {
                pos = {pos.x + dx[i], pos.y + dy[i]};
                if ((pos.x < 0) || (pos.y < 0) || (pos.x >= n) || (pos.y >= n)) {
                    pos = {pos.x - dx[i], pos.y - dy[i]};
                    break;
                }
            }
            if (dist[pos.y][pos.x] < dist[now.y][now.x] + 1) continue;
            dist[pos.y][pos.x] = dist[now.y][now.x] + 1;
            q.push(pos);
        }
    }
    if (answer == 1e9) return 0; // Segmentation Fault
    return answer;
}

int dfs(Position s) {
    int ret = 1e9;
    
    for(int i=1; i<=card; i++) {
        if (check[i]) continue;
        
        int one = bfs(s, idx[i][0]) + bfs(idx[i][0], idx[i][1]);
        int two = bfs(s, idx[i][1]) + bfs(idx[i][1], idx[i][0]);
        
        check[i] = true; // Board로 확인X
        Board[idx[i][0].y][idx[i][0].x] = 0;
        Board[idx[i][1].y][idx[i][1].x] = 0;
        
        ret = min(ret, one + dfs(idx[i][1]));
        ret = min(ret, two + dfs(idx[i][0]));
        
        check[i] = false;
        Board[idx[i][0].y][idx[i][0].x] = i;
        Board[idx[i][1].y][idx[i][1].x] = i;
    }
    
    if (ret == 1e9) return 0;
    return ret;
}

int solution(vector<vector<int>> board, int r, int c) {
    n = board.size();
    Board = board;
    
    for(int i=0; i<n; i++) {
        for(int j=0; j<n; j++) {
            if (board[i][j] != 0) {
                card = max(card, board[i][j]);
                idx[board[i][j]].push_back({j, i}); // 카드 번호 -> 인덱스
            }
        }
    }

    return dfs({c, r});
}
```

## BFS
```c++
{% raw %}
#include <string>
#include <vector>
#include <queue>
#include <set>

using namespace std;

struct info {
    string board;
    int r, c; // 커서 위치
    int y, x; // 카드 위치
    
    bool operator < (const info & v) const {
        if(r != v.r) return r < v.r;
        if(c != v.c) return c < v.c;
        if(y != v.y) return y < v.y;
        if(x != v.x) return x < v.x;
        if (board.compare(v.board) < 0) return true; // = 제외
        return false;
    }
};

string arrToStr(vector<vector<int>>& board) { // set 안의 vector 연산자 오버로딩 어려움
    string ret = "";
    for(int i=0; i<board.size(); i++) {
        for(int j=0; j<board[0].size(); j++) {
            ret += (board[i][j] + '0');
        }
    }
    return ret;
}

int solution(vector<vector<int>> board, int r, int c) {
    int answer = 1e9;
    
    int dx[4] = {0, 0, -1, 1};
    int dy[4] = {-1, 1, 0, 0};
    
    int n = board.size();
    
    queue<pair<info, int>> q;
    string boardStr = arrToStr(board);
    q.push({{boardStr, r, c, -1, -1}, 0});
    
    set<info> s;
    s.insert({boardStr, r, c, -1, -1});
    
    while(!q.empty()) {
        info now = q.front().first;
        int count = q.front().second;
        q.pop();
        
        for(int i=0; i<4; i++) {
            // 한칸 이동
            int newX = now.c + dx[i];
            int newY = now.r + dy[i];
            if ((newX < 0) || (newY < 0) || (newX >= n) || (newY >= n)) continue;
            
            info newInfo = {now.board, newY, newX, now.y, now.x};
            if (s.find(newInfo) == s.end()) {
                s.insert(newInfo);
                q.push({newInfo, count + 1});
            }
            
            // ctrl 이동
            while (now.board[4*newY+newX] == '0') {
                newX += dx[i];
                newY += dy[i];
                if ((newX < 0) || (newY < 0) || (newX >= n) || (newY >= n)) {
                    newX -= dx[i];
                    newY -= dy[i];
                    break;
                }
            }
            
            newInfo = {now.board, newY, newX, now.y, now.x};
            if (s.find(newInfo) != s.end()) continue;
            s.insert(newInfo);
            q.push({newInfo, count + 1});
        }
        
        if (now.board[4*now.r+now.c] == '0') continue;
        
        info newInfo;
        if ((now.x != -1) && (now.y != -1)) { // 카드 짝 맞추기
            if (((now.r != now.y) || (now.c != now.x)) && (now.board[4*now.r+now.c] == now.board[4*now.y+now.x])) { // ||, && 조심
                now.board[4*now.r+now.c] = '0'; // for문 이후
                now.board[4*now.y+now.x] = '0'; // now.board.replace(4*now.y+now.x, 1, "0");
                bool isFind = true;
                for(int i=0; i<n; i++) {
                    for(int j=0; j<n; j++) {
                        if (now.board[4*i+j] != '0') isFind = false;
                    }
                }
                if (isFind) answer = min(answer, count + 1); // 바로 리턴 가능
            }
            newInfo = {now.board, now.r, now.c, -1, -1};
        }
        else newInfo = {now.board, now.r, now.c, now.r, now.c}; // 카드 뒤집기

        if (s.find(newInfo) != s.end()) continue;
        s.insert(newInfo);
        q.push({newInfo, count + 1});
    }
    
    return answer;
}
{% endraw %}
```

## 참고
<https://velog.io/@hsw0194/%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%A8%B8%EC%8A%A4-%EC%B9%B4%EB%93%9C-%EC%A7%9D-%EB%A7%9E%EC%B6%94%EA%B8%B0-2021-%EC%B9%B4%EC%B9%B4%EC%98%A4-%EB%B8%94%EB%9D%BC%EC%9D%B8%EB%93%9C-%EA%B3%B5%EC%B1%84#%ED%92%80%EC%9D%B4%EC%BD%94%EB%93%9C%EF%BB%BF>