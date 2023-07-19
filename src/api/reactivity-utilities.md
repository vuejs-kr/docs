# 반응형 API: 유틸리티 {#reactivity-api-utilities}

## isRef() {#isref}

값이 ref 객체인지 확인합니다.

- **타입**:

  ```ts
  function isRef<T>(r: Ref<T> | unknown): r is Ref<T>
  ```

  반환 타입은 [type predicate](https://www.typescriptlang.org/docs/handbook/2/narrowing.html#using-type-predicates)이므로,
  `isRef`를 타입 가드로 사용할 수 있습니다.

  ```ts
  let foo: unknown
  if (isRef(foo)) {
    // foo의 타입은 Ref<unknown>으로 한정됨
    foo.value
  }
  ```

## unref() {#unref}

인자가 ref이면 내부 값을 반환하고, 그렇지 않으면 인자 자체를 반환합니다.
이것은 `val = isRef(val) ? val.value : val`과 같습니다.

- **타입**:

  ```ts
  function unref<T>(ref: T | Ref<T>): T
  ```

- **예제**

  ```ts
  function useFoo(x: number | Ref<number>) {
    const unwrapped = unref(x)
    // unwrapped는 이제 확실히 숫자 입니다
  }
  ```

## toRef() {#toref}

Can be used to normalize values / refs / getters into refs (3.3+).

반응형 객체의 속성에 대한 ref를 만드는 데 사용할 수 있습니다.
생성된 ref는 소스 속성과 동기화됩니다.
소스 속성을 변경하면 ref가 업데이트되고 그 반대의 경우도 마찬가지입니다.

- **타입**:

  ```ts
  // normalization signature (3.3+)
  function toRef<T>(
    value: T
  ): T extends () => infer R
    ? Readonly<Ref<R>>
    : T extends Ref
    ? T
    : Ref<UnwrapRef<T>>

  // object property signature
  function toRef<T extends object, K extends keyof T>(
    object: T,
    key: K,
    defaultValue?: T[K]
  ): ToRef<T[K]>

  type ToRef<T> = T extends Ref ? T : Ref<T>
  ```

- **예제**

  Normalization signature (3.3+):

  ```js
  // returns existing refs as-is
  toRef(existingRef)

  // creates a readonly ref that calls the getter on .value access
  toRef(() => props.foo)

  // creates normal refs from non-function values
  // equivalent to ref(1)
  toRef(1)
  ```

  Object property signature:

  ```js
  const state = reactive({
    foo: 1,
    bar: 2
  })

  // a two-way ref that syncs with the original property
  const fooRef = toRef(state, 'foo')

  // ref를 변경하면 원본도 업데이트 됨
  fooRef.value++
  console.log(state.foo) // 2

  // 원본을 변경하면 ref도 업데이트 됨
  state.foo++
  console.log(fooRef.value) // 3
  ```

  이것은 다음과 다름에 주의해야 합니다:

  ```js
  const fooRef = ref(state.foo)
  ```

  위의 ref는 `state.foo`와 **동기화되지 않습니다**.
  `ref()`가 일반 숫자 값을 수신하기 때문입니다.

  `toRef()`는 컴포저블 함수에 prop을 ref로 전달하려는 경우에 유용합니다:

  ```vue
  <script setup>
  import { toRef } from 'vue'
  
  const props = defineProps(/* ... */)

  // `props.foo`를 ref로 변환한 다음 컴포저블 함수에 전달
  useSomeFeature(toRef(props, 'foo'))
  
  // getter syntax - recommended in 3.3+
  useSomeFeature(toRef(() => props.foo))
  </script>
  ```

  `toRef`가 컴포넌트 props와 함께 사용되면,
  props 변경에 대한 일반적인 제한 사항이 계속 적용됩니다.
  ref에 새 값을 할당하려는 시도는 prop을 직접 수정하려는 것과 동일하며 허용되지 않습니다.
  이런 경우에는 [`computed()`](./reactivity-core.html#computed)에 `get`과 `set`을 선언하여 사용하는 것으로 구현할 수 있습니다.
  자세한 내용은 [컴포넌트를 `v-model`과 함께 사용하기](/guide/components/v-model) 가이드 참고.

  When using the object property signature, `toRef()` will return a usable ref even if the source property doesn't currently exist. This makes it possible to work with optional properties, which wouldn't be picked up by [`toRefs`](#torefs).

## toValue() <sup class="vt-badge" data-text="3.3+" /> {#tovalue}

Normalizes values / refs / getters to values. This is similar to [unref()](#unref), except that it also normalizes getters. If the argument is a getter, it will be invoked and its return value will be returned.

This can be used in [Composables](/guide/reusability/composables.html) to normalize an argument that can be either a value, a ref, or a getter.

- **Type**

  ```ts
  function toValue<T>(source: T | Ref<T> | (() => T)): T
  ```

- **Example**

  ```js
  toValue(1) //       --> 1
  toValue(ref(1)) //  --> 1
  toValue(() => 1) // --> 1
  ```

  Normalizing arguments in composables:

  ```ts
  import type { MaybeRefOrGetter } from 'vue'

  function useFeature(id: MaybeRefOrGetter<number>) {
    watch(() => toValue(id), id => {
      // react to id changes
    })
  }

  // this composable supports any of the following:
  useFeature(1)
  useFeature(ref(1))
  useFeature(() => 1)
  ```

## toRefs() {#torefs}

반응형 객체를 일반 객체로 변환하고,
변환된 일반 객체의 각 속성은 원본 객체(반응형 객체)의 속성이 ref된 것 입니다.
각 개별 ref는 [`toRef()`](#toref)를 사용하여 생성됩니다.

- **타입**:

  ```ts
  function toRefs<T extends object>(
    object: T
  ): {
    [K in keyof T]: ToRef<T[K]>
  }

  type ToRef = T extends Ref ? T : Ref<T>
  ```

- **예제**

  ```js
  const state = reactive({
    foo: 1,
    bar: 2
  })

  const stateAsRefs = toRefs(state)
  /*
  stateAsRefs의 타입: {
    foo: Ref<number>,
    bar: Ref<number>
  }
  */

  // 원본 속성이 ref와 "연결됨"
  state.foo++
  console.log(stateAsRefs.foo.value) // 2

  stateAsRefs.foo.value++
  console.log(state.foo) // 3
  ```

  `toRefs`는 컴포저블 함수에서 반응형 객체를 반환하면,
  이것을 사용하는 컴포넌트가 반응형을 잃지 않고 분해 할당 및 확장 할 수 있어 유용합니다.

  ```js
  function useFeatureX() {
    const state = reactive({
      foo: 1,
      bar: 2
    })

    // ...state를 사용하여 작동하는 로직

    // 반환할 때 refs로 변환
    return toRefs(state)
  }

  // 반응형을 잃지 않고 분해 할당 가능
  const { foo, bar } = useFeatureX()
  ```

  `toRefs`는 호출 시 소스 객체에서 열거 가능한 속성만 참조로 생성합니다.
  아직 존재하지 않을 수 있는 속성에 대한 참조를 생성하려면 [`toRef`](#toref)를 사용해야 합니다.

## isProxy() {#isproxy}

객체가 [`reactive()`](./reactivity-core.html#reactive), [`readonly()`](./reactivity-core.html#readonly), [`shallowReactive()`](./reactivity-advanced.html#shallowreactive) 또는 [`shallowReadonly()`](./reactivity-advanced.html#shallowreadonly)에 의해 생성된 프락시인지 확인합니다.

- **타입**:

  ```ts
  function isProxy(value: unknown): boolean
  ```

## isReactive() {#isreactive}

객체가 [`reactive()`](./reactivity-core.html#reactive) 또는 [`shallowReactive()`](./reactivity-advanced.html#shallowreactive)에 의해 생성된 프락시인지 확인합니다.

- **타입**:

  ```ts
  function isReactive(value: unknown): boolean
  ```

## isReadonly() {#isreadonly}

전달된 값이 읽기 전용 객체인지 확인합니다. 읽기 전용 객체의 속성은 변경할 수 있지만 전달된 객체를 통해 직접 할당할 수는 없습니다.

[`readonly()`](./reactivity-core.html#readonly) 및 [`shallowReadonly()`](./reactivity-advanced.html#shallowreadonly)로 생성된 프록시는 모두 읽기 전용으로 간주되며, `set` 함수가 없는 [`computed()`](./reactivity-core.html#computed) 참조와 마찬가지로 마찬가지입니다.


- **타입**:

  ```ts
  function isReadonly(value: unknown): boolean
  ```
