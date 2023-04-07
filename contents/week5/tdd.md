# TDD

TDD(test-driven development, 테스트 주도 개발)는 자동화된 **[단위 테스트](../week1/testing-library.md#테스트-종류) 코드를 먼저 작성**하여 테스트가 개발을 이끌어가도록 하는 **방식** 자체를 뜻한다.

## TDD 기대 효과

- 깔끔한 코드 작성 가능하다.
  - TDD의 개발 단계 자체에 리팩토링이 있어 중복된 코드를 제거하고 복잡한 코드를 정리하게 된다.
- 장기적으로 개발 비용 절감할 수 있다.
- 개발이 끝나고 나면 테스트 코드를 작성하는 것이 매우 귀찮다.

## TDD Cycle

1. RED : 실패하는 작은 단위 테스트 코드를 작성한다. (인터페이스와 스펙에 집중)
2. GREEN : 재빨리 테스트를 통과시킨다. 정답이 아닌 가짜 구현이어도 괜찮다. 어렵다면 다시 1번으로 돌아가 더 작고 쉬운 문제로 정의해보자.
3. Refactor : 중복을 제거하는 등의 리팩터링을 통해 코드를 올바르게 만든다.

## 예시

참고 : [TDD FAQ](https://github.com/ahastudio/til/blob/main/blog/2016/12-03-tdd-faq.md), [Jest에서 TDD 예제](https://github.com/ahastudio/til/blob/main/jest/20201204-simple-tdd-example.md)
