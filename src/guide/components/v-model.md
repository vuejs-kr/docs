# Component v-model {#component-v-model}


`v-model`을 컴포넌트에서 사용하여 양방향 바인딩을 구현할 수 있습니다.

먼저 네이티브 엘리먼트에서 v-model이 어떻게 사용되는지 다시 살펴봅시다:

```vue-html
<input v-model="searchText" />
```

내부적으로 템플릿 컴파일러는 `v-model`을 좀 더 자세한 표현으로 확장합니다. 따라서 위의 코드는 다음과 같은 작업을 수행합니다:

```vue-html
<input
  :value="searchText"
  @input="searchText = $event.target.value"
/>
```

컴포넌트에 사용하면 `v-model`이 대신 이렇게 확장됩니다:

```vue-html
<CustomInput
  :modelValue="searchText"
  @update:modelValue="newValue => searchText = newValue"
/>
```

하지만 이 기능이 실제로 작동하려면 `<CustomInput>` 컴포넌트가 두 가지 작업을 수행해야 합니다:

1. 네이티브 `<input>` 앨리먼트의 `value` 속성을 `modelValue` 프로퍼티에 바인딩합니다.
2. 네이티브 `input` 이벤트가 트리거되면 새 값으로 `update:modelValue` 사용자 지정 이벤트를 내보냅니다.

실제로 작동하는 모습은 다음과 같습니다:

<div class="options-api">

```vue
<!-- CustomInput.vue -->
<script>
export default {
  props: ['modelValue'],
  emits: ['update:modelValue']
}
</script>

<template>
  <input
    :value="modelValue"
    @input="$emit('update:modelValue', $event.target.value)"
  />
</template>
```

</div>
<div class="composition-api">

```vue
<!-- CustomInput.vue -->
<script setup>
defineProps(['modelValue'])
defineEmits(['update:modelValue'])
</script>

<template>
  <input
    :value="modelValue"
    @input="$emit('update:modelValue', $event.target.value)"
  />
</template>
```

</div>

이제 `v-model`이 이 컴포넌트와 완벽하게 작동합니다:

```vue-html
<CustomInput v-model="searchText" />
```

<div class="options-api">

[Try it in the Playground](https://play.vuejs.org/#eyJBcHAudnVlIjoiPHNjcmlwdD5cbmltcG9ydCBDdXN0b21JbnB1dCBmcm9tICcuL0N1c3RvbUlucHV0LnZ1ZSdcblxuZXhwb3J0IGRlZmF1bHQge1xuICBjb21wb25lbnRzOiB7IEN1c3RvbUlucHV0IH0sXG4gIGRhdGEoKSB7XG4gICAgcmV0dXJuIHtcbiAgICAgIG1lc3NhZ2U6ICdoZWxsbydcbiAgICB9XG4gIH1cbn1cbjwvc2NyaXB0PlxuXG48dGVtcGxhdGU+XG4gIDxDdXN0b21JbnB1dCB2LW1vZGVsPVwibWVzc2FnZVwiIC8+IHt7IG1lc3NhZ2UgfX1cbjwvdGVtcGxhdGU+IiwiaW1wb3J0LW1hcC5qc29uIjoie1xuICBcImltcG9ydHNcIjoge1xuICAgIFwidnVlXCI6IFwiaHR0cHM6Ly9zZmMudnVlanMub3JnL3Z1ZS5ydW50aW1lLmVzbS1icm93c2VyLmpzXCJcbiAgfVxufSIsIkN1c3RvbUlucHV0LnZ1ZSI6IjxzY3JpcHQ+XG5leHBvcnQgZGVmYXVsdCB7XG4gIHByb3BzOiBbJ21vZGVsVmFsdWUnXSxcbiAgZW1pdHM6IFsndXBkYXRlOm1vZGVsVmFsdWUnXVxufVxuPC9zY3JpcHQ+XG5cbjx0ZW1wbGF0ZT5cbiAgPGlucHV0XG4gICAgOnZhbHVlPVwibW9kZWxWYWx1ZVwiXG4gICAgQGlucHV0PVwiJGVtaXQoJ3VwZGF0ZTptb2RlbFZhbHVlJywgJGV2ZW50LnRhcmdldC52YWx1ZSlcIlxuICAvPlxuPC90ZW1wbGF0ZT4ifQ==)

</div>
<div class="composition-api">

[Try it in the Playground](https://play.vuejs.org/#eyJBcHAudnVlIjoiPHNjcmlwdCBzZXR1cD5cbmltcG9ydCB7IHJlZiB9IGZyb20gJ3Z1ZSdcbmltcG9ydCBDdXN0b21JbnB1dCBmcm9tICcuL0N1c3RvbUlucHV0LnZ1ZSdcbiAgXG5jb25zdCBtZXNzYWdlID0gcmVmKCdoZWxsbycpXG48L3NjcmlwdD5cblxuPHRlbXBsYXRlPlxuICA8Q3VzdG9tSW5wdXQgdi1tb2RlbD1cIm1lc3NhZ2VcIiAvPiB7eyBtZXNzYWdlIH19XG48L3RlbXBsYXRlPiIsImltcG9ydC1tYXAuanNvbiI6IntcbiAgXCJpbXBvcnRzXCI6IHtcbiAgICBcInZ1ZVwiOiBcImh0dHBzOi8vc2ZjLnZ1ZWpzLm9yZy92dWUucnVudGltZS5lc20tYnJvd3Nlci5qc1wiXG4gIH1cbn0iLCJDdXN0b21JbnB1dC52dWUiOiI8c2NyaXB0IHNldHVwPlxuZGVmaW5lUHJvcHMoWydtb2RlbFZhbHVlJ10pXG5kZWZpbmVFbWl0cyhbJ3VwZGF0ZTptb2RlbFZhbHVlJ10pXG48L3NjcmlwdD5cblxuPHRlbXBsYXRlPlxuICA8aW5wdXRcbiAgICA6dmFsdWU9XCJtb2RlbFZhbHVlXCJcbiAgICBAaW5wdXQ9XCIkZW1pdCgndXBkYXRlOm1vZGVsVmFsdWUnLCAkZXZlbnQudGFyZ2V0LnZhbHVlKVwiXG4gIC8+XG48L3RlbXBsYXRlPiJ9)

</div>

이 컴포넌트 내에서 `v-model`을 구현하는 또 다른 방법은 getter와 setter가 모두 있는 쓰기 가능한 `computed` 프로퍼티를 사용하는 것입니다. `get` 메서드는 `modelValue` 프로퍼티를 반환하고 `set` 메서드는 해당 이벤트를 발생시켜야 합니다:

<div class="options-api">

```vue
<!-- CustomInput.vue -->
<script>
export default {
  props: ['modelValue'],
  emits: ['update:modelValue'],
  computed: {
    value: {
      get() {
        return this.modelValue
      },
      set(value) {
        this.$emit('update:modelValue', value)
      }
    }
  }
}
</script>

<template>
  <input v-model="value" />
</template>
```

</div>
<div class="composition-api">

```vue
<!-- CustomInput.vue -->
<script setup>
import { computed } from 'vue'

const props = defineProps(['modelValue'])
const emit = defineEmits(['update:modelValue'])

const value = computed({
  get() {
    return props.modelValue
  },
  set(value) {
    emit('update:modelValue', value)
  }
})
</script>

<template>
  <input v-model="value" />
</template>
```

</div>

## `v-model` arguments {#v-model-arguments}

기본적으로 컴포넌트의 `v-model`은 `modelValue`를 프로퍼티로, `update:modelValue`를 이벤트로 사용합니다. `v-model`에 인자를 전달하여 이러한 이름을 수정할 수 있습니다:

```vue-html
<MyComponent v-model:title="bookTitle" />
```

이 경우 자식 컴포넌트는 `title` 프로퍼티를 예상하고 부모 값을 업데이트하는 `update:title` 이벤트를 발생시켜야 합니다:

<div class="composition-api">

```vue
<!-- MyComponent.vue -->
<script setup>
defineProps(['title'])
defineEmits(['update:title'])
</script>

<template>
  <input
    type="text"
    :value="title"
    @input="$emit('update:title', $event.target.value)"
  />
</template>
```

[Try it in the Playground](https://play.vuejs.org/#eyJBcHAudnVlIjoiPHNjcmlwdCBzZXR1cD5cbmltcG9ydCB7IHJlZiB9IGZyb20gJ3Z1ZSdcbmltcG9ydCBNeUNvbXBvbmVudCBmcm9tICcuL015Q29tcG9uZW50LnZ1ZSdcbiAgXG5jb25zdCB0aXRsZSA9IHJlZigndi1tb2RlbCBhcmd1bWVudCBleGFtcGxlJylcbjwvc2NyaXB0PlxuXG48dGVtcGxhdGU+XG4gIDxoMT57eyB0aXRsZSB9fTwvaDE+XG4gIDxNeUNvbXBvbmVudCB2LW1vZGVsOnRpdGxlPVwidGl0bGVcIiAvPlxuPC90ZW1wbGF0ZT4iLCJpbXBvcnQtbWFwLmpzb24iOiJ7XG4gIFwiaW1wb3J0c1wiOiB7XG4gICAgXCJ2dWVcIjogXCJodHRwczovL3NmYy52dWVqcy5vcmcvdnVlLnJ1bnRpbWUuZXNtLWJyb3dzZXIuanNcIlxuICB9XG59IiwiTXlDb21wb25lbnQudnVlIjoiPHNjcmlwdCBzZXR1cD5cbmRlZmluZVByb3BzKFsndGl0bGUnXSlcbmRlZmluZUVtaXRzKFsndXBkYXRlOnRpdGxlJ10pXG48L3NjcmlwdD5cblxuPHRlbXBsYXRlPlxuICA8aW5wdXRcbiAgICB0eXBlPVwidGV4dFwiXG4gICAgOnZhbHVlPVwidGl0bGVcIlxuICAgIEBpbnB1dD1cIiRlbWl0KCd1cGRhdGU6dGl0bGUnLCAkZXZlbnQudGFyZ2V0LnZhbHVlKVwiXG4gIC8+XG48L3RlbXBsYXRlPiJ9)

</div>
<div class="options-api">

```vue
<!-- MyComponent.vue -->
<script>
export default {
  props: ['title'],
  emits: ['update:title']
}
</script>

<template>
  <input
    type="text"
    :value="title"
    @input="$emit('update:title', $event.target.value)"
  />
</template>
```

[Try it in the Playground](https://play.vuejs.org/#eyJBcHAudnVlIjoiPHNjcmlwdD5cbmltcG9ydCBNeUNvbXBvbmVudCBmcm9tICcuL015Q29tcG9uZW50LnZ1ZSdcblxuZXhwb3J0IGRlZmF1bHQge1xuICBjb21wb25lbnRzOiB7IE15Q29tcG9uZW50IH0sXG4gIGRhdGEoKSB7XG4gICAgcmV0dXJuIHtcbiAgICAgIHRpdGxlOiAndi1tb2RlbCBhcmd1bWVudCBleGFtcGxlJ1xuICAgIH1cbiAgfVxufVxuPC9zY3JpcHQ+XG5cbjx0ZW1wbGF0ZT5cbiAgPGgxPnt7IHRpdGxlIH19PC9oMT5cbiAgPE15Q29tcG9uZW50IHYtbW9kZWw6dGl0bGU9XCJ0aXRsZVwiIC8+XG48L3RlbXBsYXRlPiIsImltcG9ydC1tYXAuanNvbiI6IntcbiAgXCJpbXBvcnRzXCI6IHtcbiAgICBcInZ1ZVwiOiBcImh0dHBzOi8vc2ZjLnZ1ZWpzLm9yZy92dWUucnVudGltZS5lc20tYnJvd3Nlci5qc1wiXG4gIH1cbn0iLCJNeUNvbXBvbmVudC52dWUiOiI8c2NyaXB0PlxuZXhwb3J0IGRlZmF1bHQge1xuICBwcm9wczogWyd0aXRsZSddLFxuICBlbWl0czogWyd1cGRhdGU6dGl0bGUnXVxufVxuPC9zY3JpcHQ+XG5cbjx0ZW1wbGF0ZT5cbiAgPGlucHV0XG4gICAgdHlwZT1cInRleHRcIlxuICAgIDp2YWx1ZT1cInRpdGxlXCJcbiAgICBAaW5wdXQ9XCIkZW1pdCgndXBkYXRlOnRpdGxlJywgJGV2ZW50LnRhcmdldC52YWx1ZSlcIlxuICAvPlxuPC90ZW1wbGF0ZT4ifQ==)

</div>

## Multiple `v-model` bindings {#multiple-v-model-bindings}

앞서 배운 것처럼 특정 prop과 이벤트를 타깃팅하는 기능을 [`v-model` 인자](#v-model-arguments)로 활용하면 이제 단일 컴포넌트 인스턴스에 여러 개의 `v-model` 바인딩을 생성할 수 있습니다.

각 `v-model`은 컴포넌트에서 추가 옵션 없이도 다른 prop에 동기화됩니다:

```vue-html
<UserName
  v-model:first-name="first"
  v-model:last-name="last"
/>
```

<div class="composition-api">

```vue
<script setup>
defineProps({
  firstName: String,
  lastName: String
})

defineEmits(['update:firstName', 'update:lastName'])
</script>

<template>
  <input
    type="text"
    :value="firstName"
    @input="$emit('update:firstName', $event.target.value)"
  />
  <input
    type="text"
    :value="lastName"
    @input="$emit('update:lastName', $event.target.value)"
  />
</template>
```

[Try it in the Playground](https://play.vuejs.org/#eyJBcHAudnVlIjoiPHNjcmlwdCBzZXR1cD5cbmltcG9ydCB7IHJlZiB9IGZyb20gJ3Z1ZSdcbmltcG9ydCBVc2VyTmFtZSBmcm9tICcuL1VzZXJOYW1lLnZ1ZSdcblxuY29uc3QgZmlyc3QgPSByZWYoJ0pvaG4nKVxuY29uc3QgbGFzdCA9IHJlZignRG9lJylcbjwvc2NyaXB0PlxuXG48dGVtcGxhdGU+XG4gIDxoMT57eyBmaXJzdCB9fSB7eyBsYXN0IH19PC9oMT5cbiAgPFVzZXJOYW1lXG4gICAgdi1tb2RlbDpmaXJzdC1uYW1lPVwiZmlyc3RcIlxuICAgIHYtbW9kZWw6bGFzdC1uYW1lPVwibGFzdFwiXG4gIC8+XG48L3RlbXBsYXRlPiIsImltcG9ydC1tYXAuanNvbiI6IntcbiAgXCJpbXBvcnRzXCI6IHtcbiAgICBcInZ1ZVwiOiBcImh0dHBzOi8vc2ZjLnZ1ZWpzLm9yZy92dWUucnVudGltZS5lc20tYnJvd3Nlci5qc1wiLFxuICAgIFwidnVlL3NlcnZlci1yZW5kZXJlclwiOiBcImh0dHBzOi8vc2ZjLnZ1ZWpzLm9yZy9zZXJ2ZXItcmVuZGVyZXIuZXNtLWJyb3dzZXIuanNcIlxuICB9XG59IiwiVXNlck5hbWUudnVlIjoiPHNjcmlwdCBzZXR1cD5cbmRlZmluZVByb3BzKHtcbiAgZmlyc3ROYW1lOiBTdHJpbmcsXG4gIGxhc3ROYW1lOiBTdHJpbmdcbn0pXG5cbmRlZmluZUVtaXRzKFsndXBkYXRlOmZpcnN0TmFtZScsICd1cGRhdGU6bGFzdE5hbWUnXSlcbjwvc2NyaXB0PlxuXG48dGVtcGxhdGU+XG4gIDxpbnB1dFxuICAgIHR5cGU9XCJ0ZXh0XCJcbiAgICA6dmFsdWU9XCJmaXJzdE5hbWVcIlxuICAgIEBpbnB1dD1cIiRlbWl0KCd1cGRhdGU6Zmlyc3ROYW1lJywgJGV2ZW50LnRhcmdldC52YWx1ZSlcIlxuICAvPlxuICA8aW5wdXRcbiAgICB0eXBlPVwidGV4dFwiXG4gICAgOnZhbHVlPVwibGFzdE5hbWVcIlxuICAgIEBpbnB1dD1cIiRlbWl0KCd1cGRhdGU6bGFzdE5hbWUnLCAkZXZlbnQudGFyZ2V0LnZhbHVlKVwiXG4gIC8+XG48L3RlbXBsYXRlPiJ9)

</div>
<div class="options-api">

```vue
<script>
export default {
  props: {
    firstName: String,
    lastName: String
  },
  emits: ['update:firstName', 'update:lastName']
}
</script>

<template>
  <input
    type="text"
    :value="firstName"
    @input="$emit('update:firstName', $event.target.value)"
  />
  <input
    type="text"
    :value="lastName"
    @input="$emit('update:lastName', $event.target.value)"
  />
</template>
```

[Try it in the Playground](https://play.vuejs.org/#eyJBcHAudnVlIjoiPHNjcmlwdD5cbmltcG9ydCBVc2VyTmFtZSBmcm9tICcuL1VzZXJOYW1lLnZ1ZSdcblxuZXhwb3J0IGRlZmF1bHQge1xuICBjb21wb25lbnRzOiB7IFVzZXJOYW1lIH0sXG4gIGRhdGEoKSB7XG4gICAgcmV0dXJuIHtcbiAgICAgIGZpcnN0OiAnSm9obicsXG4gICAgICBsYXN0OiAnRG9lJ1xuICAgIH1cbiAgfVxufVxuPC9zY3JpcHQ+XG5cbjx0ZW1wbGF0ZT5cbiAgPGgxPnt7IGZpcnN0IH19IHt7IGxhc3QgfX08L2gxPlxuICA8VXNlck5hbWVcbiAgICB2LW1vZGVsOmZpcnN0LW5hbWU9XCJmaXJzdFwiXG4gICAgdi1tb2RlbDpsYXN0LW5hbWU9XCJsYXN0XCJcbiAgLz5cbjwvdGVtcGxhdGU+IiwiaW1wb3J0LW1hcC5qc29uIjoie1xuICBcImltcG9ydHNcIjoge1xuICAgIFwidnVlXCI6IFwiaHR0cHM6Ly9zZmMudnVlanMub3JnL3Z1ZS5ydW50aW1lLmVzbS1icm93c2VyLmpzXCIsXG4gICAgXCJ2dWUvc2VydmVyLXJlbmRlcmVyXCI6IFwiaHR0cHM6Ly9zZmMudnVlanMub3JnL3NlcnZlci1yZW5kZXJlci5lc20tYnJvd3Nlci5qc1wiXG4gIH1cbn0iLCJVc2VyTmFtZS52dWUiOiI8c2NyaXB0PlxuZXhwb3J0IGRlZmF1bHQge1xuICBwcm9wczoge1xuXHQgIGZpcnN0TmFtZTogU3RyaW5nLFxuICBcdGxhc3ROYW1lOiBTdHJpbmdcblx0fSxcbiAgZW1pdHM6IFsndXBkYXRlOmZpcnN0TmFtZScsICd1cGRhdGU6bGFzdE5hbWUnXVxufVxuPC9zY3JpcHQ+XG5cbjx0ZW1wbGF0ZT5cbiAgPGlucHV0XG4gICAgdHlwZT1cInRleHRcIlxuICAgIDp2YWx1ZT1cImZpcnN0TmFtZVwiXG4gICAgQGlucHV0PVwiJGVtaXQoJ3VwZGF0ZTpmaXJzdE5hbWUnLCAkZXZlbnQudGFyZ2V0LnZhbHVlKVwiXG4gIC8+XG4gIDxpbnB1dFxuICAgIHR5cGU9XCJ0ZXh0XCJcbiAgICA6dmFsdWU9XCJsYXN0TmFtZVwiXG4gICAgQGlucHV0PVwiJGVtaXQoJ3VwZGF0ZTpsYXN0TmFtZScsICRldmVudC50YXJnZXQudmFsdWUpXCJcbiAgLz5cbjwvdGVtcGxhdGU+In0=)

</div>

## `v-model` 수정자 처리하기 {#handling-v-model-modifiers}

Form 양식 입력 바인딩에 대해 배울 때 `v-model`에 `.trim`, `.number` 및 `.lazy`와 같은 [내장 수정자](/guide/essentials/forms.html#modifiers)가 있다는 것을 알았습니다. 경우에 따라 사용자 정의 입력 컴포넌트에서 `v-model`이 사용자 정의 수정자를 지원하도록 할 수도 있습니다.

`v-model` 바인딩에서 제공하는 문자열의 첫 글자를 대문자로 표시하는 사용자 지정 수정자 예제인 `capitalize`를 만들어 보겠습니다:

```vue-html
<MyComponent v-model.capitalize="myText" />
```


`v-model` 컴포넌트에 추가되는 수정자는 `modelModifiers` 프로퍼티를 통해 컴포넌트에 제공됩니다. 아래 예시에서는 기본적으로 빈 객체로 설정된 `modelModifiers` 프로퍼티를 포함하는 컴포넌트를 생성했습니다:

<div class="composition-api">

```vue{4,9}
<script setup>
const props = defineProps({
  modelValue: String,
  modelModifiers: { default: () => ({}) }
})

defineEmits(['update:modelValue'])

console.log(props.modelModifiers) // { capitalize: true }
</script>

<template>
  <input
    type="text"
    :value="modelValue"
    @input="$emit('update:modelValue', $event.target.value)"
  />
</template>
```

</div>
<div class="options-api">

```vue{11}
<script>
export default {
  props: {
    modelValue: String,
    modelModifiers: {
      default: () => ({})
    }
  },
  emits: ['update:modelValue'],
  created() {
    console.log(this.modelModifiers) // { capitalize: true }
  }
}
</script>

<template>
  <input
    type="text"
    :value="modelValue"
    @input="$emit('update:modelValue', $event.target.value)"
  />
</template>
```

</div>

컴포넌트의 `modelModifiers` 프로퍼티에 `capitalize`가 포함되어 있고 그 값은 `v-model` 바인딩 `v-model.capitalize="myText"`에 설정되어 있기 때문에 `true`인 것을 알 수 있습니다.


이제 prop이  설정되었으므로 `modelModifiers` 객체 키를 확인하고 발신된 값을 변경하는 핸들러를 작성할 수 있습니다. 아래 코드에서는 `<input />` 앨리먼트가 `input` 이벤트를 실행할 때마다 문자열을 대문자로 표시합니다.

<div class="composition-api">

```vue{11-13}
<script setup>
const props = defineProps({
  modelValue: String,
  modelModifiers: { default: () => ({}) }
})

const emit = defineEmits(['update:modelValue'])

function emitValue(e) {
  let value = e.target.value
  if (props.modelModifiers.capitalize) {
    value = value.charAt(0).toUpperCase() + value.slice(1)
  }
  emit('update:modelValue', value)
}
</script>

<template>
  <input type="text" :value="modelValue" @input="emitValue" />
</template>
```

[Try it in the Playground](https://play.vuejs.org/#eyJBcHAudnVlIjoiPHNjcmlwdCBzZXR1cD5cbmltcG9ydCB7IHJlZiB9IGZyb20gJ3Z1ZSdcbmltcG9ydCBNeUNvbXBvbmVudCBmcm9tICcuL015Q29tcG9uZW50LnZ1ZSdcbiAgXG5jb25zdCBteVRleHQgPSByZWYoJycpXG48L3NjcmlwdD5cblxuPHRlbXBsYXRlPlxuICBUaGlzIGlucHV0IGNhcGl0YWxpemVzIGV2ZXJ5dGhpbmcgeW91IGVudGVyOlxuICA8TXlDb21wb25lbnQgdi1tb2RlbC5jYXBpdGFsaXplPVwibXlUZXh0XCIgLz5cbjwvdGVtcGxhdGU+IiwiaW1wb3J0LW1hcC5qc29uIjoie1xuICBcImltcG9ydHNcIjoge1xuICAgIFwidnVlXCI6IFwiaHR0cHM6Ly9zZmMudnVlanMub3JnL3Z1ZS5ydW50aW1lLmVzbS1icm93c2VyLmpzXCJcbiAgfVxufSIsIk15Q29tcG9uZW50LnZ1ZSI6IjxzY3JpcHQgc2V0dXA+XG5jb25zdCBwcm9wcyA9IGRlZmluZVByb3BzKHtcbiAgbW9kZWxWYWx1ZTogU3RyaW5nLFxuICBtb2RlbE1vZGlmaWVyczogeyBkZWZhdWx0OiAoKSA9PiAoe30pIH1cbn0pXG5cbmNvbnN0IGVtaXQgPSBkZWZpbmVFbWl0cyhbJ3VwZGF0ZTptb2RlbFZhbHVlJ10pXG5cbmZ1bmN0aW9uIGVtaXRWYWx1ZShlKSB7XG4gIGxldCB2YWx1ZSA9IGUudGFyZ2V0LnZhbHVlXG4gIGlmIChwcm9wcy5tb2RlbE1vZGlmaWVycy5jYXBpdGFsaXplKSB7XG4gICAgdmFsdWUgPSB2YWx1ZS5jaGFyQXQoMCkudG9VcHBlckNhc2UoKSArIHZhbHVlLnNsaWNlKDEpXG4gIH1cbiAgZW1pdCgndXBkYXRlOm1vZGVsVmFsdWUnLCB2YWx1ZSlcbn1cbjwvc2NyaXB0PlxuXG48dGVtcGxhdGU+XG4gIDxpbnB1dCB0eXBlPVwidGV4dFwiIDp2YWx1ZT1cIm1vZGVsVmFsdWVcIiBAaW5wdXQ9XCJlbWl0VmFsdWVcIiAvPlxuPC90ZW1wbGF0ZT4ifQ==)

</div>
<div class="options-api">

```vue{13-15}
<script>
export default {
  props: {
    modelValue: String,
    modelModifiers: {
      default: () => ({})
    }
  },
  emits: ['update:modelValue'],
  methods: {
    emitValue(e) {
      let value = e.target.value
      if (this.modelModifiers.capitalize) {
        value = value.charAt(0).toUpperCase() + value.slice(1)
      }
      this.$emit('update:modelValue', value)
    }
  }
}
</script>

<template>
  <input type="text" :value="modelValue" @input="emitValue" />
</template>
```

[Try it in the Playground](https://play.vuejs.org/#eyJBcHAudnVlIjoiPHNjcmlwdD5cbmltcG9ydCBNeUNvbXBvbmVudCBmcm9tICcuL015Q29tcG9uZW50LnZ1ZSdcbiAgXG5leHBvcnQgZGVmYXVsdCB7XG4gIGNvbXBvbmVudHM6IHsgTXlDb21wb25lbnQgfSxcbiAgZGF0YSgpIHtcbiAgICByZXR1cm4ge1xuICAgICAgbXlUZXh0OiAnJ1xuICAgIH1cbiAgfVxufVxuPC9zY3JpcHQ+XG5cbjx0ZW1wbGF0ZT5cbiAgVGhpcyBpbnB1dCBjYXBpdGFsaXplcyBldmVyeXRoaW5nIHlvdSBlbnRlcjpcbiAgPE15Q29tcG9uZW50IHYtbW9kZWwuY2FwaXRhbGl6ZT1cIm15VGV4dFwiIC8+XG48L3RlbXBsYXRlPiIsImltcG9ydC1tYXAuanNvbiI6IntcbiAgXCJpbXBvcnRzXCI6IHtcbiAgICBcInZ1ZVwiOiBcImh0dHBzOi8vc2ZjLnZ1ZWpzLm9yZy92dWUucnVudGltZS5lc20tYnJvd3Nlci5qc1wiXG4gIH1cbn0iLCJNeUNvbXBvbmVudC52dWUiOiI8c2NyaXB0PlxuZXhwb3J0IGRlZmF1bHQge1xuICBwcm9wczoge1xuICAgIG1vZGVsVmFsdWU6IFN0cmluZyxcbiAgICBtb2RlbE1vZGlmaWVyczoge1xuICAgICAgZGVmYXVsdDogKCkgPT4gKHt9KVxuICAgIH1cbiAgfSxcbiAgZW1pdHM6IFsndXBkYXRlOm1vZGVsVmFsdWUnXSxcbiAgbWV0aG9kczoge1xuICAgIGVtaXRWYWx1ZShlKSB7XG4gICAgICBsZXQgdmFsdWUgPSBlLnRhcmdldC52YWx1ZVxuICAgICAgaWYgKHRoaXMubW9kZWxNb2RpZmllcnMuY2FwaXRhbGl6ZSkge1xuICAgICAgICB2YWx1ZSA9IHZhbHVlLmNoYXJBdCgwKS50b1VwcGVyQ2FzZSgpICsgdmFsdWUuc2xpY2UoMSlcbiAgICAgIH1cbiAgICAgIHRoaXMuJGVtaXQoJ3VwZGF0ZTptb2RlbFZhbHVlJywgdmFsdWUpXG4gICAgfVxuICB9XG59XG48L3NjcmlwdD5cblxuPHRlbXBsYXRlPlxuICA8aW5wdXQgdHlwZT1cInRleHRcIiA6dmFsdWU9XCJtb2RlbFZhbHVlXCIgQGlucHV0PVwiZW1pdFZhbHVlXCIgLz5cbjwvdGVtcGxhdGU+In0=)

</div>


인자와 수정자가 모두 있는 `v-model` 바인딩의 경우, 생성된 prop 이름은 `arg + "수정자"`가 됩니다. 예를 들어

```vue-html
<MyComponent v-model:title.capitalize="myText">
```

해당 선언은 다음과 같아야 합니다:

<div class="composition-api">

```js
const props = defineProps(['title', 'titleModifiers'])
defineEmits(['update:title'])

console.log(props.titleModifiers) // { capitalize: true }
```

</div>
<div class="options-api">

```js
export default {
  props: ['title', 'titleModifiers'],
  emits: ['update:title'],
  created() {
    console.log(this.titleModifiers) // { capitalize: true }
  }
}
```
</div>
