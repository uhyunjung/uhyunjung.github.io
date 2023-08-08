---
title: '[다익스트라, DFS, BFS, DP] 프로그래머스 경주로 건설 C++'
author: HJ
date: 2023-03-03 15:40:00 +0900
categories: [코딩테스트, 그래프|트리]
tags: [다익스트라, DFS, BFS, DP]
math: true
mermaid: true
---

## Links
<https://school.programmers.co.kr/learn/courses/30/lessons/67259>

## 다익스트라
```c++
#include <vector>
#include <queue>

using namespace std;

int N, answer;
vector<vector<int>> Board;
int dx[4] = {0, 0, -1, 1}; // 상하좌우
int dy[4] = {-1, 1, 0, 0};

void dijkstra() {
    int dist[N][N][4]; // 방향별로 최소 비용 기록
    fill(&dist[0][0][0], &dist[N-1][N-1][4], 1e9);
    
    priority_queue<vector<int>, vector<vector<int>>, greater<>> pq; // 오름차순
    pq.push({0, 0, 0, 1}); // cost, r, c, dir
    pq.push({0, 0, 0, 3});
    while(!pq.empty()) {
        vector<int> now = pq.top();
        pq.pop();

        if (dist[now[1]][now[2]][now[3]] < now[0]) continue;
        if ((now[1] == N-1) && (now[2] == N-1)) {
            answer = min(answer, now[0]); // return X
            continue;
        }
        
        for(int i=0; i<4; i++) {
            int newX = now[2] + dx[i];
            int newY = now[1] + dy[i];
            if ((newX < 0) || (newY < 0) || (newX >= N) || (newY >= N)) continue;
            if (Board[newY][newX]) continue;
            
            int dir = now[3]; // 이전 방향
            int cost = (i == dir)? now[0] + 100 : now[0] + 600; // 방향 전환 여부
            if (dist[newY][newX][i] < cost) continue;
            dist[newY][newX][i] = cost;
            
            pq.push({dist[newY][newX][i], newY, newX, i});
        }
    }
}

int solution(vector<vector<int>> board) {
    answer = 1e9;
    N = board.size();
    Board = board;
    
    dijkstra();
    
    return answer;
}
```

## DFS+DP
```c++
#include <vector>

using namespace std;

int N, answer;
vector<vector<int>> Board;
bool visited[25][25];
int dist[25][25][4];
int dx[4] = {1, -1, 0, 0};
int dy[4] = {0, 0, 1, -1};

void dfs(int cost, int r, int c, int dir) {
    if ((dir != -1) && (dist[r][c][dir] < cost)) return;
    if ((r == N-1) && (c == N-1)) {
        answer = min(answer, cost);
        return;
    }
    
    for(int i=0; i<4; i++) {
        int newX = c + dx[i];
        int newY = r + dy[i];
        if ((newX < 0) || (newY < 0) || (newX >= N) || (newY >= N)) continue;
        if (visited[newY][newX]) continue;
        if (Board[newY][newX]) continue;

        int before = (dir == -1)? i : dir;
        int newCost = (i == before)? cost+100 : cost+600;
        if (dist[newY][newX][i] < newCost) continue;
        
        visited[newY][newX] = true;
        dist[newY][newX][i] = newCost;
        dfs(newCost, newY, newX, i);
        visited[newY][newX] = false;
    }
}

int solution(vector<vector<int>> board) {
    answer = 1e9;
    N = board.size();
    Board = board;
    fill(&dist[0][0][0], &dist[N-1][N-1][4], 1e9);
    
    dfs(0, 0, 0, -1);
    
    return answer;
}
```

## BFS+DP
```c++
#include <vector>
#include <queue>

using namespace std;

int N, answer;
vector<vector<int>> Board;
int dx[4] = {0, 0, -1, 1}; // 상하좌우
int dy[4] = {-1, 1, 0, 0};

void bfs() {
    int dist[N][N][4];
    fill(&dist[0][0][0], &dist[N-1][N-1][4], 1e9);
    
    queue<vector<int>> q;
    q.push({0, 0, 0, -1}); // cost, r, c, dir
    while(!q.empty()) {
        vector<int> now = q.front();
        q.pop();
        
        if ((now[3] != -1) && (dist[now[1]][now[2]][now[3]] < now[0])) continue;
        if ((now[1] == N-1) && (now[2] == N-1)) {
            answer = min(answer, now[0]); // return X
            continue;
        }
        
        for(int i=0; i<4; i++) {
            int newX = now[2] + dx[i];
            int newY = now[1] + dy[i];
            if ((newX < 0) || (newY < 0) || (newX >= N) || (newY >= N)) continue;
            if (Board[newY][newX]) continue;
            
            int dir = (now[3] == -1)? i: now[3]; // 시작점
            int cost = (i == dir)? now[0] + 100 : now[0] + 600; // 방향 전환 여부
            if (dist[newY][newX][i] < cost) continue;
            
            dist[newY][newX][i] = cost;
            q.push({cost, newY, newX, i});
        }
    }
}

int solution(vector<vector<int>> board) {
    answer = 1e9;
    N = board.size();
    Board = board;
    
    bfs();
    
    return answer;
}
```