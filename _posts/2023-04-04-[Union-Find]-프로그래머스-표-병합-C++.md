---
title: '[Union-Find] 프로그래머스 표 병합 C++'
author: HJ
date: 2023-04-04 22:10:00 +0900
categories: [코딩테스트, Union-Find]
tags: [Union-Find]
math: true
mermaid: true
---

## Links
<https://school.programmers.co.kr/learn/courses/30/lessons/150366>

## C++
```c++
#include <string>
#include <vector>
#include <sstream>

using namespace std;

struct position {
    int r, c;
    
    bool operator == (const position &p){
        if(r != p.r) return false;
        if(c != p.c) return false;
        return true;
    }
};

string table[51][51]; // 병합위치 -> 문자열 str
position parent[51][51]; // 위치 -> 병합위치

vector<string> split(string str, char ch) {
    vector<string> answer;
    stringstream ss(str);
    string tmp;

    while(getline(ss, tmp, ch)) answer.push_back(tmp);
    return answer;
}

position find(position x) {
    if ((parent[x.r][x.c] == x)) return x;
    return parent[x.r][x.c] = find(parent[x.r][x.c]);
}

void unions(position x, position y) {
    x = find(x);
    y = find(y);

    if (x == y) return;
    parent[y.r][y.c] = x; // x 기준
}

vector<string> solution(vector<string> commands) {
    vector<string> answer;
     
    // 초기화
    for(int i=1; i<51; i++) {
        for(int j=1; j<51; j++)
            parent[i][j] = {i, j};
    }
    
    for(auto command : commands) {
        vector<string> v = split(command, ' ');
        
        // UPDATE
        if (v[0] == "UPDATE") {
            if (v.size() == 4) { // mp 삭제, table, mp 변경
                int r = stoi(v[1]);
                int c = stoi(v[2]);
                position p = find({r, c});
                string str = v[3];
                
                table[p.r][p.c] = str;
            }
            else { // 문자열로 위치 찾기, table 변경, mp 삭제, 변경
                string str = v[2];
                for(int i=1; i<51; i++) {
                    for(int j=1; j<51; j++) {
                        if (v[1] == table[i][j]) table[i][j] = str;
                    }
                }
            }
        }
        // MERGE : mp 삭제, merge 변경, unmerge 추가, table 변경
        else if (v[0] == "MERGE") {
            int r1 = stoi(v[1]);
            int c1 = stoi(v[2]);
            int r2 = stoi(v[3]);
            int c2 = stoi(v[4]);
            
            position p1 = find({r1, c1});
            position p2 = find({r2, c2});
            
            string str = table[p1.r][p1.c];
            if (str == "") str = table[p2.r][p2.c];
            table[p1.r][p1.c] = ""; // 초기화
            table[p2.r][p2.c] = "";
            unions(p1, p2);
            table[p1.r][p1.c] = str;
        }
        // UNMERGE : merge, unmerge 초기화
        else if (v[0] == "UNMERGE") {
            int r = stoi(v[1]);
            int c = stoi(v[2]);
            position p = find({r, c});
            string str = table[p.r][p.c];
            table[p.r][p.c] = "";
            table[r][c] = str;
            
            vector<position> vec;
            for(int i=1; i<51; i++) {
                for(int j=1; j<51; j++) {
                    position temp = find({i, j});
                    if (temp == p) vec.push_back({i, j});
                }
            }
            for(auto v : vec) parent[v.r][v.c] = {v.r, v.c}; // 저장 후 수정
        }
        // PRINT
        else {
            int r = stoi(v[1]);
            int c = stoi(v[2]);
            position p = find({r, c});
            string str = table[p.r][p.c];
            
            if (str == "") answer.push_back("EMPTY");
            else answer.push_back(str);
        }
    }

    return answer;
}
```