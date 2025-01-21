---
title: '[개발환경] VSCode Ollama 자동완성 기능'
date: 2025-01-21 18:10:00 +0900
categories: [정리, 개발환경]
tags: [개발환경]
math: true
mermaid: true
published: false
---

## Ollama 다운로드
<https://ollama.com/download/windows>

## VSCode Continue Extension 설치
<https://marketplace.visualstudio.com/items?itemName=Continue.continue>

## 모델 다운로드
```bash
ollama pull qwen2.5-coder:1.5b
# ollama run qwen2.5-coder:1.5b # 실행
```
## Ollama 작동 확인
- Git Bash에서 실행
```
curl -v http://127.0.0.1:11434/api/generate
```

## config 수정
- `C:\Users\.continue\config.json`에 아래 항목 추가
```
"tabAutocompleteModel": {
    "title": "qwen2.5-coder:1.5b",
    "provider": "ollama",
    "apiBase": "http:/localhost:11434",
    "model": "qwen2.5-coder:1.5b-base"
}
```

## Enable Autocomplete
- 오른쪽 아래 Continue > Enable Autocomplete 선택