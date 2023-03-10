---
description: CRA 없이 프로젝트 세팅하기
---

# 개발 환경 세팅

### Node.js 설치, 버전 확인

- LTS 최신버전 권장

{% hint style="info" %}
**Fast Node Manager**를 사용하면 프로젝트마다 다른 Node 버전을 사용할 때 용이하다.
{% endhint %}

### npm 프로젝트 생성

- package.json 생성

```typescript
npm init -y // 모든 조건에 yes
```

### .gitignore 파일 수동 생성

* Git에 업로드하지 않을 폴더, 파일 목록을 기재하는 문서

{% hint style="info" %}
나에게 필요한 환경에 맞춰 .gitignore를 생성해주는 서비스를 이용해도 좋다. (구글링!)
{% endhint %}

### 필요한 개발 프로그램 및 도구 설치

#### typescript 설치 및 설정 파일 생성

```terminal
npm i -D typescript
npx tsc --init
```

#### ESLint 설치 및 설정 파일 생성

```terminal
npm i -D eslint
npx eslint --init
```

추가로, 파일 수정 후 저장할 때마다 ESLint를 적용하고 싶다면 vscode 기본 설정에서 아래 코드를 추가하면 된다.

```json
{
    "editor.codeActionsOnSave": {
        "source.fixAll.eslint": true
    }
}
```

`.eslintignore` 파일도 `.gitignore`과 동일하게 작성해준다.

#### React와 React type 정보 설치

```terminal
npm i react react-dom
npm i -D @types/react @types/react-dom
```

#### 테스팅 도구 설치 / jest.config.js 파일 수동 생성

```terminal
npm i -D jest @types/jest @swc/core @swc/jest \
         jest-environment-jsdom \
         @testing-library/react @testing-library/jest-dom
```

#### Parcel 설치

```terminal
npm i -D parcel
```

추가 작업으로

* `package.json`에 `"source": "./index.html"` 추가하여 메인 페이지 설정하기
* `parcel-reporter-static-files-copy` 설치하고 정적 파일 업로드 할 `static` 폴더 만들기&#x20;

```terminal
npm install -D parcel-reporter-static-files-copy
```



### `package.json` "script" 수정

* 설치한 개발 도구를 편리하게 사용하기 위해 "script" 항목 추가 및 수정



### 애플리케이션 기본 파일 생성 및 코드 작성

* `index.html`
* `src/main.tsx`
* `src/main.test.tsx`
* `src/App.tsx`
* `src/App.test.tsx`
* `src/components/Greeting.test.tsx`
* `src/components/Greeting.tsx`
