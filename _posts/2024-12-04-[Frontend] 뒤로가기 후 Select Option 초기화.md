---
title: '[Frontend] 뒤로가기 후 Select Option 초기화'
date: 2024-12-04 10:30:00 +0900
categories: [프론트엔드, Frontend]
tags: [Frontend]
math: true
mermaid: true
---

## 문제
- Select Option을 선택한 후, 다른 페이지로 이동하고 뒤로가기로 해당 페이지에 돌아올 시 Select Option이 초기화되지 않음

## 해결 과정
```javascript
init() {
    window.addEventListener('pageshow', function(event) {
        const selectElement = document.getElementById('type');
        if (event.persisted || performance.getEntriesByType("navigation")[0].type === 'back_forward') {
            selectElement.selectedIndex = 0; // 기본 옵션으로 초기화
        }
    });
}
```