---
title: '[기타] Github Jekyll 블로그 세팅'
date: 2024-12-03 08:10:00 +0900
categories: [기타, Github]
tags: [Github]
math: true
mermaid: true
---

## 테마 선택
- <https://github.com/cotes2020/jekyll-theme-chirpy>

## Github Settings
- General > Default branchbranch > main 선택
- Pages > GitHub Pages > Build and deployment Source > Github Actions 선택

## Source Code 수정
- _config.yml 수정

```yml
title: title
tagline: subtitle

social:
    name: username
    email: email
links:
    - profile link
cdn: 주석 처리
avatar: "assets/img/favicons/IMG.jpg"
future: true # 미래 날짜 포스팅 허용
```
- .github/workflows/jekyll.yml 수정

```yml
jobs:
    build:
        steps:
            - name: npm build
              run: npm install && npm run build
```

## Ruby Install
- <https://rubyinstaller.org/downloads/>

## Terminal Command
```bash
bundle install
bundle exec jekyll serve
```

## 루트 디렉토리 파일 추가
- robots.txt 추가

```
User-agent: *
Allow: /

Sitemap: https://uhyunjung.github.io/sitemap.xml
```
- sitemap.xml 추가
    - Google Sitemap 설정 : <https://www.google.com/webmasters/tools/sitemap-list>

## 구조
- _tabs : 사이드바
- _site : Build 결과
- _layout : home.html, post.html 등
- _includes : head.html, footer.html

## 참고) Jekyll CSS 적용 안 될 때
```
--- layout: home # Index page ---
```

- Setting > Pages > Source를 Github Actions로 놓고, Jekyll configure를 클릭해 jekyll.yml 생성