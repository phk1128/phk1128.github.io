---
title: fatal could not read Username for 'https://github.com' terminal prompts disabled
date: 2024-09-14
categories: [ TroubleShooting ]
tags: [ github actions, github 인증에러 ]
image: assets/posts/img_15.png
---

## 🎬 Intro
> GitHub Actions에서 발생한 인증 오류를 해결해보겠습니다.

---

### ✅ 문제
```yaml
    steps:
      - name: 레포지토리 체크아웃
        uses: actions/checkout@v4
        with:
          token: ${{ secrets.SUBMODULE_TOKEN }}
          submodules: true
```

현재 CI workflow에는 설정파일을 서브모듈에서 가져오도록 되어 있습니다. 따라서 CI를 위해서는 반드시 유효한 github 토큰이 요구됩니다.


![img.png](/assets/posts/img_24.png)

이 에러는 GitHub Actions에서 발생한 인증 문제와 관련이 있습니다. 구체적으로 "fatal: could not read Username for 'https://github.com': terminal prompts disabled"라는 메시지는 Git이 GitHub에 접근할 때 사용자 이름과 비밀번호를 입력해야 하지만, GitHub Actions는 터미널 입력을 받을 수 없어서 발생하는 오류입니다.

### ✅ 해결과정
![img_2.png](/assets/posts/img_26.png)

![img_1.png](/assets/posts/img_25.png)

- workflow에서 사용하는 토큰이 만료되어있는지 확인하고 만료가 되었다면 새로 생신을 해줍니다. 저의 경우 submodule 토큰이 만료가 되었어서 오류가 발생했기 때문에 새로 갱신을 해주었습니다.
- github actions 동작 시 사용하는 서브모듈 토큰 secret 값을 갱신해주면 됩니다.

---


## ✨Summary
#### github actions 동작 중 인증오류가 발생하면 secret 값들이 잘못되어 있지는 않은지 점검해보면 됩니다.
