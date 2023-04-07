# E2E 테스트 도구

## Playwright

Playwright 이전에 Puppeteer 라는 라이브러리가 있었다. Puppeteer는 Headless 모드에서 Chrome, Chrominum을 제어할 수 있게 하는 API다. 브라우저에서 수동으로 할 수 있는 대부분의 작업을 할 수 있기 때문에 [End-to-End(E2E) 테스트](../week1/testing-library.md#e2e-테스트end-to-end-test--ui-test)를 할 때 Jest와 같이 사용되곤 했다.

{% hint style="info" %}
Headless : 브라우저 창을 띄우지 않고도 브라우저 엔진을 사용하여 웹을 렌더링 할 수 있는 기능을 제공한다. UI 테스트에서 이를 이용할 시 실제로 사용자가 화면을 통해 사용한 것처럼 테스트할 수 있다.
{% endhint %}

Puppeteer를 만든 팀이 마이크로소프트로 이적하여 Playwright 팀에 합류됐고, Playwright은 Puppeteer보다 한 단계 발전했다. 둘은 거의 비슷하나 Puppeteer와 비교했을 때 Playwright이 조금 더 테스트에 특화되어 있는 느낌이다.

Playwright는 거의 모든 브라우저를 지원하고 있어 크로스 브라우징 테스트가 가능하고, _@playwright/test_ 와 같은 자체적인 Test runner도 지원한다.

### 설치 및 세팅

```bash
npm i -D @playwright/test eslint-plugin-playwright
```

```js
// playwright.config.ts
import { PlaywrightTestConfig } from '@playwright/test';

const config: PlaywrightTestConfig = {
    testDir: './tests', // 테스트 코드 위치
    retries: 0, // 테스트 실패 시 재시도 횟수
    use: {
        channel : "chrome", // 테스트 할 브라우저
        baseURL: 'http://localhost:8080', // npm start로 띄워놓을 서버 주소
        headless: !!process.env.CI, // CI가 true일 경우 headless 모드로 실행
        screenshot: 'only-on-failure', // 실패 시에만 스크린샷
    },
};

export default config;
```

참고로 `process.env.CI`의 CI 환경 변수는 CI/CD 시스템에서 설정하며, 테스트를 자동화할 때 브라우저를 Headless 모드로 실행할 때 주로 사용된다.

우리가 일반적으로 Headless 모드로 테스트할 때는 CI를 직접 줄 수도 있다. !!를 사용하여 진리값으로 변경되기 때문에 CI에 아무 truthy한 값을 주면 된다. ([적용한 예시](#테스트-코드-작성))

```js
// tests/.eslintrc.js
module.exports = {
    env: {
        jest: false,
    },
    extends: ['plugin:playwright/playwright-test'],
    rules: {
        'import/no-extraneous-dependencies': 'off',
    },
};
```

### 테스트 코드 작성

```js
import { test, expect } from '@playwright/test';

test('Filter products', async ({ page }) => {
    await page.goto('/');

    await expect(page.getByText('Apple')).toBeVisible();

    const searchInput = page.getByLabel('Search');

    await searchInput.fill('a');

    await expect(page.getByText('Apple')).toBeVisible();

    await searchInput.fill('aa');

    await expect(page.getByText('Apple')).toBeHidden();
});

test('Click the “Increase” button', async ({ page }) => {
    await page.goto('/');

    const count = 13;

    await Promise.all((
        [...Array(count)].map(async () => {
            await page.getByText('Increase').click();
        })
    ));

    await expect(page.getByText(`${count}`)).toBeVisible();
});
```

코드에서 보다시피 await를 통해 순차적으로 테스트 코드를 실행하며, 이전 await 문이 체크돼야 다음으로 넘어간다.

테스트 함수의 두번째 인자에서 `page` 객체를 넘겨주어 이를 통해 브라우저를 조작하고, `expect` 함수로 동작을 검증할 수 있다.

`page`와 함께 사용되는 메서드, 프로퍼티는 [공식 문서](https://playwright.dev/docs/api/class-page)에서 확인할 수 있다.

### 테스트 실행

```bash
npx playwright test
```

Headless 모드로 실행하기

```bash
CI=true npx playwright test
```

## CodeceptJS

좀 더 간단하게 E2E 테스트(UI 테스트)를 진행하려면 CodeceptJS를 사용할 수도 있다. CodeceptJS는 테스트 코드를 잘 모르는 사람도 내용을 파악할 수 있을 정도로 인간친화적 언어를 사용한다.

### 설치 및 세팅

```bash
npm install -g codeceptjs
```

이 작업을 하면 `package.json` 파일이 자동으로 들여쓰기를 스페이스 4칸으로 변경한다. 원상복구 해주자.

```bash
npx codeceptjs init
```

`npx codeceptjs init`를 입력하면 여러가지 질문을 하는데, 상황에 맞게 답변하면 된다.

### 테스트 코드 작성

테스트 파일은 `*_test.js` 형태로 네이밍해야 한다.

```js
Feature('Google 검색');

Scenario('검색어 입력하기', (I) => {
  I.amOnPage('https://www.google.com/');
  I.fillField('q', 'CodeceptJS');
  I.click('Google 검색');
  I.see('CodeceptJS');
});
```

`I` 와 함께 쓸 수 있는 메서드는 [공식 문서](https://codecept.io/playwright/#actions)에서 확인할 수 있다. 참고로 playwright과 같이 쓸 때 사용할 수 있는 메서드를 알려주는 페이지다.

링크된 공식 문서에 들어가보면 알겠지만, 지원되는 메서드가 별로 없어 정말 간단하게 테스트할 때 사용하기에 좋다.

### 테스트 실행

```bash
npm run codeceptjs

npm run codeceptjs:headless # 브라우저를 띄우지 않고 실행

npm run codeceptjs:ui # CodeceptJS 자체 인터페이스에서 테스트 실행
```
