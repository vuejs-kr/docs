---
outline: deep
---
# 컴포지션 API FAQ {#composition-api-faq}

:::tip
이 FAQ는 Vue 2를 사용해본 경험이 있거나, 주로 옵션 API를 사용하고 있다는 것을 전제로 합니다.
:::

## 컴포지션 API란? {#what-is-composition-api}

<VueSchoolLink href="https://vueschool.io/lessons/introduction-to-the-vue-js-3-composition-api" title="무료 컴포지션 API 강좌"/>

컴포지션(Composition) API는 옵션을 선언하는 대신 `import`한 함수를 사용하여 Vue 컴포넌트를 작성할 수 있는 API 세트입니다.
이것은 아래 API를 다루는 포괄적인 용어입니다:

- [반응형(Reactivity) API](/api/reactivity-core): 예를 들어 `ref()` 및 `reactive()`를 사용하여 반응형 상태, 계산된 상태 및 감시자를 직접 생성할 수 있습니다.

- [생명주기 훅](/api/composition-api-lifecycle): 예를 들어 `onMounted()` 및 `onUnmounted()`를 사용하여 컴포넌트 생명주기에 프로그래밍 방식으로 연결할 수 있습니다.

- [의존성 주입(Dependency Injection)](/api/composition-api-dependency-injection): `provide()` 및 `inject()`를 사용하면 반응형 API를 사용하는 동안 Vue의 의존성 주입 시스템을 활용할 수 있습니다.


컴포지션 API는 Vue 3 및 [Vue 2.7](https://blog.vuejs.org/posts/vue-2-7-naruto)에 내장된 기능입니다. 이전 Vue 2 버전의 경우 공식적으로 유지 관리되는 [`@vue/composition-api`](https://github.com/vuejs/composition-api) 플러그인을 사용하십시오. Vue 3에서는 주로 싱글 파일 컴포넌트에서 [`<스크립트 설정>`](/api/sfc-script-setup) 구문과 함께 사용되기도 합니다. 다음은 컴포지션 API를 사용하는 컴포넌트의 기본 예시입니다:

```vue
<script setup>
import { ref, onMounted } from 'vue'

// 반응형 상태
const count = ref(0)

// 상태를 변경하고 업데이트를 트리거하는 함수
function increment() {
  count.value++
}

// 생명주기 훅
onMounted(() => {
  console.log(`숫자를 세기 위한 초기값은 ${count.value} 입니다.`)
})
</script>

<template>
  <button @click="increment">숫자 세기: {{ count }}</button>
</template>
```

함수 구성에 기반한 API 스타일에도 불구하고 **컴포지션 API는 함수형 프로그래밍이 아닙니다**.
컴포지션 API는 Vue의 변경 가능하고 세분화된 반응성 패러다임을 기반으로 하는 반면 기능적 프로그래밍은 불변성을 강조합니다.

Vue를 컴포지션 API와 함께 사용하는 방법을 배우고 싶다면 왼쪽 사이드바 상단의 토글을 사용하여 사이트 전체 API 환경 설정을 컴포지션 API로 설정한 다음 처음부터 가이드를 살펴보세요.

## 왜 컴포지션 API인가요? {#why-composition-api}

### 더 나은 로직 재사용성 {#better-logic-reuse}

컴포지션 API의 가장 큰 장점은 [컴포저블 함수](/guide/reusability/composables)의 형태로 깔끔하고 효율적인 로직 재사용이 가능하다는 것입니다.
옵션 API의 기본 로직 재사용 메커니즘인 [믹스인의 모든 단점](/guide/reusability/composables.html#vs-mixins)을 해결합니다.

컴포지션 API의 로직 재사용 기능은 컴포저블 유틸리티의 계속 성장하는 컬렉션인 [VueUse](https://vueuse.org/)와 같은 인상적인 커뮤니티 프로젝트를 탄생시켰습니다.
또한 상태 저장 타사 서비스 또는 라이브러리를 [불변 데이터](/guide/extras/reactivity-in-depth.html#immutable-data), [상태 머신](/guide/extras/reactivity-in-depth.html#state-machines) 및 [RxJS](/guide/extras/reactivity-in-depth#rxjs)와 같은 Vue의 반응형 시스템에 쉽게 통합하기 위한 깔끔한 메커니즘 역할을 합니다.

### 보다 유연한 코드 구성 {#more-flexible-code-organization}

많은 사용자는 기본적으로 옵션 API를 사용하여 조직화된 코드를 작성하는 것을 좋아합니다.
그러나 옵션 API는 단일 컴포넌트의 논리가 특정 복잡성 임계값을 초과하는 경우 심각한 제한을 가집니다.
이 제한은 여러 프로덕션 Vue 2 앱에서 직접 목격한 여러 **논리적 문제**를 처리해야 하는 컴포넌트에서 특히 두드러집니다.

Vue CLI의 GUI에서 폴더 탐색기 컴포넌트를 예로 들어 보겠습니다.
이 컴포넌트는 다음과 같은 논리적 문제를 야기합니다:

- 현재 폴더 상태 추적 및 내용 표시
- 폴더 탐색 처리(열기, 닫기, 새로 고침...)
- 새 폴더 생성 처리
- 즐겨찾기 폴더만 표시 전환
- 숨김 폴더 표시 전환
- 현재 작업 디렉터리 변경 처리

컴포넌트의 [원본 버전](https://github.com/vuejs/vue-cli/blob/a09407dd5b9f18ace7501ddb603b95e31d6d93c0/packages/@vue/cli-ui/src/components/folder/FolderExplorer.vue#L198-L404)은 옵션 API로 작성되었습니다.
다루는 논리적 문제를 기준으로 코드의 각 라인에 색상을 지정하면 다음과 같이 표시됩니다:

<img alt="folder component before" src="./images/options-api.png" width="129" height="500" style="margin: 1.2em auto">

동일한 논리적 문제를 처리하는 코드가 파일의 다른 부분에 분할어 있습니다.
수백 줄 길이의 컴포넌트에서 단일 논리적 문제를 이해하고 탐색하려면 파일을 지속적으로 위아래로 스크롤해야 하므로 어렵습니다.
또한 논리적 문제를 재사용 가능한 유틸리티로 추출하려는 경우,
파일의 다른 부분에서 올바른 코드 조각을 찾아 추출하는 데 상당한 노력이 필요합니다.

다음은 [Composition API로 리팩터링](https://gist.github.com/yyx990803/8854f8f6a97631576c14b63c8acd8f2e) 전후의 동일한 컴포넌트입니다:

![folder component after](./images/composition-api-after.png)


동일한 논리적 문제와 관련된 코드가 어떻게 그룹화 되었는지 보십시오.
특정 논리적 문제를 해결하는 동안 더 이상 다른 옵션 블록 사이를 이동할 필요가 없습니다.
또한, 추출을 위해 더 이상 코드를 섞을 필요가 없기 때문에 최소한의 노력으로 코드 그룹을 외부 파일로 이동할 수 있습니다.
리팩토링을 위한 소모 시간 감소는 대규모 코드베이스에서 장기적인 유지 관리의 핵심입니다.

### 더 나은 타입 추론 {#better-type-inference}

최근 몇 년 동안 점점 더 많은 프론트엔드 개발자들이 [TypeScript](https://www.typescriptlang.org/)를 채택하고 있습니다.
이는 우리가 보다 강력한 코드를 작성하고, 보다 자신 있게 변경하고, IDE 지원을 통해 뛰어난 개발 경험을 제공하는 데 도움이 되기 때문입니다.
그러나 원래 2013년에 구상된 옵션 API는 유형 추론을 염두에 두지 않고 설계되었습니다.
유형 추론이 옵션 API와 함께 작동하도록 하기 위해 일부 [터무니없이 복잡한 유형 구조](https://github.com/vuejs/core/blob/44b95276f5c086e1d88fa3c686a5f39eb5bb7821/packages/runtime-core/src/componentPublicInstance.ts#L132-L165)를 구현해야 했습니다.
이 모든 노력에도 불구하고 옵션 API에 대한 유형 추론은 여전히 믹스인 및 의존성 주입에 대해 분해될 수 있습니다.

이로 인해 TS와 함께 Vue를 사용하고자 하는 많은 개발자가 `vue-class-component`로 구동되는 클래스 API에 의존하게 되었습니다. 그러나 클래스 기반 API는 2019년 Vue 3 개발 당시 2단계 제안에 불과했던 언어 기능인 ES 데코레이터에 크게 의존합니다. 저희는 불안정한 제안을 기반으로 공식 API를 만드는 것은 너무 위험하다고 생각했습니다. 그 이후로 데코레이터 제안은 또 한 번의 전면적인 개편을 거쳐 2022년에 마침내 3단계에 도달했습니다. 또한 클래스 기반 API는 옵션 API와 마찬가지로 로직 재사용 및 구성 제한이 있습니다.


이에 비해 컴포지션 API는 기본적으로 유형 친화적인 일반 변수와 함수를 주로 사용합니다.
컴포지션 API로 작성된 코드는 수동 유형 힌트가 거의 필요 없이 전체 유형 추론을 즐길 수 있습니다.
대부분의 경우 컴포지션 API 코드는 TypeScript와 일반 JavaScript에서 거의 동일하게 보입니다.
이것은 또한 일반 JavaScript 사용자가 부분적인 유형 추론의 이점을 누릴 수 있도록 합니다.

### 더 작은 프로덕션 번들 및 더 적은 오버헤드 {#smaller-production-bundle-and-less-overhead}

컴포지션 API와 `<script setup>`으로 작성된 코드는 옵션 API에 비해 더 효율적이고 축소하기 쉽습니다.
이는 `<script setup>` 컴포넌트의 템플릿이 `<script setup>` 코드의 동일한 범위에 인라인된 함수로 컴파일되기 때문입니다.
`this`의 속성 접근과 달리 컴파일된 템플릿 코드는 인스턴스 프락시 없이 `<script setup>` 내부에 선언된 변수에 직접 접근할 수 있습니다.
이것은 모든 변수 이름을 안전하게 단축할 수 있기 때문에 더 나은 축소로 이어집니다.

## 옵션 API와의 관계 {#relationship-with-options-api}

### Trade-offs {#trade-offs}

옵션 API에서 넘어온 일부 사용자들는 그들의 컴포지션 API 코드가 덜 구성적이라 생각하고,
컴포지션 API가 코드 구성 측면에서 "더 나쁘다"고 결론짓습니다.
이러한 의견을 가진 사용자라면 해당 문제를 다른 관점에서 볼 것을 권장합니다.

컴포지션 API가 더 이상 코드를 해당 버킷에 넣도록 안내하는 "가드 레일"(`export default {}`)을 제공하지 않는다는 것입니다.
따라서 일반적인 JavaScript를 작성하는 것처럼 컴포넌트 코드를 작성할 수 있습니다.
그러므로 **일반적인 JavaScript를 작성할 때와 마찬가지로 모든 코드 구성 모범 사례를 컴포지션 API 코드에 적용할 수 있고 적용해야 합니다**.
잘 구성된 자바스크립트를 작성할 수 있다면 잘 구성된 컴포지션 API 코드도 작성할 수 있어야 합니다.

옵션 API를 사용하면 컴포넌트 코드를 작성할 때 "생각을 덜"할 수 있으므로 많은 사용자가 이 API를 좋아합니다.
정신적 피로도는 줄어들지만, 다양성 없이 규정된 코드 구성 패턴에 갇히게 되므로 대규모 프로젝트에서 코드 품질을 리팩토링하거나 개선하기 어려울 수 있습니다.
이와 관련하여 컴포지션 API는 더 나은 장기적인(거시적인) 확장성을 제공합니다.


### 컴포지션 API는 모든 사용 사례를 포괄합니까? {#does-composition-api-cover-all-use-cases}

상태 저장 로직 측면에서 그렇습니다.
컴포지션 API를 사용할 때 여전히 필요할 수 있는 옵션은 `props`, `emit`, `name` 및 `inheritAttrs`뿐입니다.
`<script setup>`을 사용하는 경우 일반적으로 `inheritAttrs`가 별도의 일반 `<script>` 블록을 필요로 하는 유일한 옵션입니다.

위에 나열된 옵션과 함께 컴포지션 API를 독점적으로 사용하려는 경우,
Vue에서 옵션 API 관련 코드를 삭제하는 [컴파일 타임 플래그](https://github.com/vuejs/core/tree/main/packages/vue#bundler-build-feature-flags)를 통해 프로덕션 번들에서 몇 kb를 줄일 수 있습니다.
이것은 의존성의 Vue 컴포넌트에도 영향을 미칩니다.

### 두 API를 함께 사용할 수 있습니까? {#can-i-use-both-apis-together}

예. 옵션 API 컴포넌트에서 [`setup()`](/api/composition-api-setup) 옵션을 통해 컴포지션 API를 사용할 수 있습니다.

그러나 컴포지션 API로 작성된 새 기능 또는 외부 라이브러리와 통합해야 하는 기존 옵션 API 코드베이스가 있는 경우에만 그렇게 하는 것이 좋습니다.

### 옵션 API가 더 이상 사용되지 않습니까? {#will-options-api-be-deprecated}

아니요, 그렇게 할 계획이 없습니다.
옵션 API는 Vue의 필수적인 부분이며 많은 개발자들이 Vue를 좋아하는 이유입니다.
또한 컴포지션 API의 많은 이점은 대규모 프로젝트에서만 나타나고, 옵션 API는 복잡성이 낮은 여러 시나리오에서 여전히 확실한 선택이라는 것을 알고 있습니다.

## 클래스 API와의 관계 {#relationship-with-class-api}

컴포지션 API가 추가 로직 재사용 및 코드 구성 이점과 함께 뛰어난 TypeScript 통합을 제공한다는 점을 감안할 때 Vue 3에서 Class API를 더 이상 사용하지 않는 것이 좋습니다.

## React 훅과의 비교 {#comparison-with-react-hooks}

컴포지션 API는 React 훅과 동일한 수준의 로직 구성 기능을 제공하지만 몇 가지 중요한 차이점이 있습니다.

React 훅은 컴포넌트가 업데이트될 때마다 반복적으로 호출됩니다.
이것은 노련한 React 개발자도 혼동할 수 있는 여러 가지 주의 사항을 만듭니다.
또한 개발 경험에 심각한 영향을 줄 수 있는 성능 최적화 문제로 이어집니다.
여기 몇 가지 예가 있습니다:

- Hooks are call-order sensitive and cannot be conditional.

- Variables declared in a React component can be captured by a hook closure and become "stale" if the developer fails to pass in the correct dependencies array. This leads to React developers relying on ESLint rules to ensure correct dependencies are passed. However, the rule is often not smart enough and over-compensates for correctness, which leads to unnecessary invalidation and headaches when edge cases are encountered.

- Expensive computations require the use of `useMemo`, which again requires manually passing in the correct dependencies array.

- Event handlers passed to child components cause unnecessary child updates by default, and require explicit `useCallback` as an optimization. This is almost always needed, and again requires a correct dependencies array. Neglecting this leads to over-rendering apps by default and can cause performance issues without realizing it.

- The stale closure problem, combined with Concurrent features, makes it difficult to reason about when a piece of hooks code is run, and makes working with mutable state that should persist across renders (via `useRef`) cumbersome.

이에 비해 Vue 컴포지션 API는 다음과 같습니다:

- Invokes `setup()` or `<script setup>` code only once. This makes the code align better with the intuitions of idiomatic JavaScript usage as there are no stale closures to worry about. Composition API calls are also not sensitive to call order and can be conditional.

- Vue's runtime reactivity system automatically collects reactive dependencies used in computed properties and watchers, so there's no need to manually declare dependencies.

- No need to manually cache callback functions to avoid unnecessary child updates. In general, Vue's fine-grained reactivity system ensures child components only update when they need to. Manual child-update optimizations are rarely a concern for Vue developers.

우리는 React 훅의 창의성을 인정하며 이는 컴포지션 API에 주요 영감의 원천입니다.
그러나 위에서 언급한 문제는 설계에 존재하며,
Vue의 반응형 모델이 이러한 문제를 해결할 수 있는 방법을 제공한다는 것을 알게 되었습니다.
