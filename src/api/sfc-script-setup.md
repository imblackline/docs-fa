# \<script setup> {#script-setup}

`<script setup>` یک سینتکس برای زیبا کردن (syntactic sugar) در زمان کامپایل است  برای استفاده از Composition API در کامپوننت های تک فایلی (SFC) است. اگر از SFC ها و Composition API استفاده می کنید، این سینتکس توصیه شده است. چندین مزیت نسبت به سینتکس معمولی `<script>` دارد:

- کد مختصر تر با boilerplate کمتر
- امکان تعریف props و ایونت‌های emit شده با استفاده از TypeScript خالص
- عملکرد بهتر در زمان اجرا (تمپلیت در یک تابع رندر در همان اسکوپ، بدون پروکسی میانی کامپایل می شود)
- عملکرد بهتر IDE در type-inference (کار کمتر برای سرور زبان برای استخراج تایپ ها از کد)

## سینتکس پایه {#basic-syntax}

برای شروع سینتکس، ویژگی `setup` را به بلوک `<script>` اضافه کنید:

```vue
<script setup>
console.log('hello script setup')
</script>
```
کد داخل به عنوان محتوای تابع `setup()` کامپوننت کامپایل می‌شود. این بدان معناست که برخلاف `<script>` معمولی، که تنها یک بار در هنگام ایمپورت شدن کامپوننت اجرا می‌شود، کد داخل `<script setup>` هر بار که نمونه‌ای از کامپوننت ایجاد می‌شود، اجرا می‌شود.

### اتصالات سطح بالا داخل تمپلیت در دسترس قرار می گیرند {#top-level-bindings-are-exposed-to-template}

هنگام استفاده از `<script setup>`، هر پیوند سطح بالا (شامل متغیرها، توابع، و ایمپورت ها) اعلام شده در `<script setup>` مستقیماً در تمپلیت قابل استفاده است:

```vue
<script setup>
// variable
const msg = 'Hello!'

// functions
function log() {
  console.log(msg)
}
</script>

<template>
  <button @click="log">{{ msg }}</button>
</template>
```
ایمپورت ها نیز به همین شکل در دسترس قرار می گیرند. این بدان معنی است که می توانید مستقیماً از یک تابع کمکی وارد شده در عبارات تمپلیت استفاده کنید بدون اینکه مجبور باشید آن را از طریق گزینه `methods` در دسترس قرار دهید:

```vue
<script setup>
import { capitalize } from './helpers'
</script>

<template>
  <div>{{ capitalize('hello') }}</div>
</template>
```

## Reactivity {#reactivity}

حالت reactive باید به صراحت با استفاده از [Reactivity API](./reactivity-core) ها ایجاد شود. مشابه مقادیر بازگردانده شده از یک تابع `setup()`، زمانی که در تمپلیت ها ارجاع داده می‌شود، ref‌ها به‌طور خودکار باز می‌شوند:

```vue
<script setup>
import { ref } from 'vue'

const count = ref(0)
</script>

<template>
  <button @click="count++">{{ count }}</button>
</template>
```

## استفاده از کامپوننت ها {#using-components}

مقادیر موجود در اسکوپ `<script setup>` همچنین می‌توانند مستقیماً به عنوان نام تگ کامپوننت های سفارشی استفاده شوند:

```vue
<script setup>
import MyComponent from './MyComponent.vue'
</script>

<template>
  <MyComponent />
</template>
```
`MyComponent` را به عنوان یک متغیر در نظر بگیرید. اگر از JSX استفاده کرده اید، مدل ذهنی در اینجا مشابه است. معادل  kebab-case تگ `<my-component>` نیز در تمپلیت کار می‌کند - با این حال، برچسب‌های کامپوننت PascalCase برای یکپارچگی به شدت توصیه می‌شوند. همچنین به تمایز از عناصر سفارشی بومی کمک می کند.

### کامپوننت های دینامیک {#dynamic-components}

از آنجایی که کامپوننت‌ها به‌جای ثبت‌شدن در ذیل کلیدهای رشته‌ای، به‌عنوان متغیر ارجاع می‌شوند، هنگام استفاده از کامپوننت های دینامیک در `<script setup>` باید از اتصال دینامیک &lrm;`:is` استفاده کنیم:

```vue
<script setup>
import Foo from './Foo.vue'
import Bar from './Bar.vue'
</script>

<template>
  <component :is="Foo" />
  <component :is="someCondition ? Foo : Bar" />
</template>
```
توجه داشته باشید که چگونه می توان از کامپوننت ها به عنوان متغیر در یک عبارت سه تایی (ternary expression) استفاده کرد.

### کامپوننت های بازگشتی {#recursive-components}

یک SFC می تواند به طور ضمنی از طریق نام فایل خود، به خودش اشاره کند. به عنوان مثال فایلی به نام `FooBar.vue` می‌تواند در تمپلیت خود به `<FooBar/>` اشاره کند.

توجه داشته باشید که اولویت کمتری نسبت به کامپوننت های ایمپورت شده دارد. اگر یک import با نام مشخص دارید که با نام استنباط شده مؤلفه تعارض دارد، می توانید با نام مستعار import کنید:

```js
import { FooBar as FooBarChild } from './components'
```

### کامپوننت های Namespaced {#namespaced-components}

می‌توانید از تگ‌های کامپوننت با نقطه‌هایی مانند `<Foo.Bar>` برای اشاره به کامپوننت‌های تودرتو در زیر ویژگی‌های شی استفاده کنید. این زمانی مفید است که چندین کامپوننت را از یک فایل وارد کنید:

```vue
<script setup>
import * as Form from './form-components'
</script>

<template>
  <Form.Input>
    <Form.Label>label</Form.Label>
  </Form.Input>
</template>
```

## Using Custom Directives {#using-custom-directives}

Globally registered custom directives just work as normal. Local custom directives don't need to be explicitly registered with `<script setup>`, but they must follow the naming scheme `vNameOfDirective`:

```vue
<script setup>
const vMyDirective = {
  beforeMount: (el) => {
    // do something with the element
  }
}
</script>
<template>
  <h1 v-my-directive>This is a Heading</h1>
</template>
```

If you're importing a directive from elsewhere, it can be renamed to fit the required naming scheme:

```vue
<script setup>
import { myDirective as vMyDirective } from './MyDirective.js'
</script>
```

## defineProps() & defineEmits() {#defineprops-defineemits}

To declare options like `props` and `emits` with full type inference support, we can use the `defineProps` and `defineEmits` APIs, which are automatically available inside `<script setup>`:

```vue
<script setup>
const props = defineProps({
  foo: String
})

const emit = defineEmits(['change', 'delete'])
// setup code
</script>
```

- `defineProps` and `defineEmits` are **compiler macros** only usable inside `<script setup>`. They do not need to be imported, and are compiled away when `<script setup>` is processed.

- `defineProps` accepts the same value as the `props` option, while `defineEmits` accepts the same value as the `emits` option.

- `defineProps` and `defineEmits` provide proper type inference based on the options passed.

- The options passed to `defineProps` and `defineEmits` will be hoisted out of setup into module scope. Therefore, the options cannot reference local variables declared in setup scope. Doing so will result in a compile error. However, it _can_ reference imported bindings since they are in the module scope as well.

### Type-only props/emit declarations<sup class="vt-badge ts" /> {#type-only-props-emit-declarations}

Props and emits can also be declared using pure-type syntax by passing a literal type argument to `defineProps` or `defineEmits`:

```ts
const props = defineProps<{
  foo: string
  bar?: number
}>()

const emit = defineEmits<{
  (e: 'change', id: number): void
  (e: 'update', value: string): void
}>()

// 3.3+: alternative, more succinct syntax
const emit = defineEmits<{
  change: [id: number] // named tuple syntax
  update: [value: string]
}>()
```

- `defineProps` or `defineEmits` can only use either runtime declaration OR type declaration. Using both at the same time will result in a compile error.

- When using type declaration, the equivalent runtime declaration is automatically generated from static analysis to remove the need for double declaration and still ensure correct runtime behavior.

  - In dev mode, the compiler will try to infer corresponding runtime validation from the types. For example here `foo: String` is inferred from the `foo: string` type. If the type is a reference to an imported type, the inferred result will be `foo: null` (equal to `any` type) since the compiler does not have information of external files.

  - In prod mode, the compiler will generate the array format declaration to reduce bundle size (the props here will be compiled into `['foo', 'bar']`)

- In version 3.2 and below, the generic type parameter for `defineProps()` were limited to a type literal or a reference to a local interface.

  This limitation has been resolved in 3.3. The latest version of Vue supports referencing imported and a limited set of complex types in the type parameter position. However, because the type to runtime conversion is still AST-based, some complex types that require actual type analysis, e.g. conditional types, are not supported. You can use conditional types for the type of a single prop, but not the entire props object.

### Default props values when using type declaration {#default-props-values-when-using-type-declaration}

One drawback of the type-only `defineProps` declaration is that it doesn't have a way to provide default values for the props. To resolve this problem, a `withDefaults` compiler macro is also provided:

```ts
export interface Props {
  msg?: string
  labels?: string[]
}

const props = withDefaults(defineProps<Props>(), {
  msg: 'hello',
  labels: () => ['one', 'two']
})
```

This will be compiled to equivalent runtime props `default` options. In addition, the `withDefaults` helper provides type checks for the default values, and ensures the returned `props` type has the optional flags removed for properties that do have default values declared.

## defineModel() <sup class="vt-badge" data-text="3.4+" /> {#definemodel}

This macro can be used to declare a two-way binding prop that can be consumed via `v-model` from the parent component. Example usage is also discussed in the [Component `v-model`](/guide/components/v-model) guide.

Under the hood, this macro declares a model prop and a corresponding value update event. If the first argument is a literal string, it will be used as the prop name; Otherwise the prop name will default to `"modelValue"`. In both cases, you can also pass an additional object which can include the prop's options and the model ref's value transform options.

```js
// declares "modelValue" prop, consumed by parent via v-model
const model = defineModel()
// OR: declares "modelValue" prop with options
const model = defineModel({ type: String })

// emits "update:modelValue" when mutated
model.value = 'hello'

// declares "count" prop, consumed by parent via v-model:count
const count = defineModel('count')
// OR: declares "count" prop with options
const count = defineModel('count', { type: Number, default: 0 })

function inc() {
  // emits "update:count" when mutated
  count.value++
}
```

### Modifiers and Transformers {#modifiers-and-transformers}

To access modifiers used with the `v-model` directive, we can destructure the return value of `defineModel()` like this:

```js
const [modelValue, modelModifiers] = defineModel()

// corresponds to v-model.trim
if (modelModifiers.trim) {
  // ...
}
```

When a modifier is present, we likely need to transform the value when reading or syncing it back to the parent. We can achieve this by using the `get` and `set` transformer options:

```js
const [modelValue, modelModifiers] = defineModel({
  // get() omitted as it is not needed here
  set(value) {
    // if the .trim modifier is used, return trimmed value
    if (modelModifiers.trim) {
      return value.trim()
    }
    // otherwise, return the value as-is
    return value
  }
})
```

### Usage with TypeScript <sup class="vt-badge ts" /> {#usage-with-typescript}

Like `defineProps` and `defineEmits`, `defineModel` can also receive type arguments to specify the types of the model value and the modifiers:

```ts
const modelValue = defineModel<string>()
//    ^? Ref<string | undefined>

// default model with options, required removes possible undefined values
const modelValue = defineModel<string>({ required: true })
//    ^? Ref<string>

const [modelValue, modifiers] = defineModel<string, 'trim' | 'uppercase'>()
//                 ^? Record<'trim' | 'uppercase', true | undefined>
```

## defineExpose() {#defineexpose}

Components using `<script setup>` are **closed by default** - i.e. the public instance of the component, which is retrieved via template refs or `$parent` chains, will **not** expose any of the bindings declared inside `<script setup>`.

To explicitly expose properties in a `<script setup>` component, use the `defineExpose` compiler macro:

```vue
<script setup>
import { ref } from 'vue'

const a = 1
const b = ref(2)

defineExpose({
  a,
  b
})
</script>
```

When a parent gets an instance of this component via template refs, the retrieved instance will be of the shape `{ a: number, b: number }` (refs are automatically unwrapped just like on normal instances).

## defineOptions() <sup class="vt-badge" data-text="3.3+" /> {#defineoptions}

This macro can be used to declare component options directly inside `<script setup>` without having to use a separate `<script>` block:

```vue
<script setup>
defineOptions({
  inheritAttrs: false,
  customOptions: {
    /* ... */
  }
})
</script>
```

- Only supported in 3.3+.
- This is a macro. The options will be hoisted to module scope and cannot access local variables in `<script setup>` that are not literal constants.

## defineSlots()<sup class="vt-badge ts"/> {#defineslots}

This macro can be used to provide type hints to IDEs for slot name and props type checking.

`defineSlots()` only accepts a type parameter and no runtime arguments. The type parameter should be a type literal where the property key is the slot name, and the value type is the slot function. The first argument of the function is the props the slot expects to receive, and its type will be used for slot props in the template. The return type is currently ignored and can be `any`, but we may leverage it for slot content checking in the future.

It also returns the `slots` object, which is equivalent to the `slots` object exposed on the setup context or returned by `useSlots()`.

```vue
<script setup lang="ts">
const slots = defineSlots<{
  default(props: { msg: string }): any
}>()
</script>
```

- Only supported in 3.3+.

## `useSlots()` & `useAttrs()` {#useslots-useattrs}

Usage of `slots` and `attrs` inside `<script setup>` should be relatively rare, since you can access them directly as `$slots` and `$attrs` in the template. In the rare case where you do need them, use the `useSlots` and `useAttrs` helpers respectively:

```vue
<script setup>
import { useSlots, useAttrs } from 'vue'

const slots = useSlots()
const attrs = useAttrs()
</script>
```

`useSlots` and `useAttrs` are actual runtime functions that return the equivalent of `setupContext.slots` and `setupContext.attrs`. They can be used in normal composition API functions as well.

## Usage alongside normal `<script>` {#usage-alongside-normal-script}

`<script setup>` can be used alongside normal `<script>`. A normal `<script>` may be needed in cases where we need to:

- Declare options that cannot be expressed in `<script setup>`, for example `inheritAttrs` or custom options enabled via plugins (Can be replaced by [`defineOptions`](/api/sfc-script-setup#defineoptions) in 3.3+).
- Declaring named exports.
- Run side effects or create objects that should only execute once.

```vue
<script>
// normal <script>, executed in module scope (only once)
runSideEffectOnce()

// declare additional options
export default {
  inheritAttrs: false,
  customOptions: {}
}
</script>

<script setup>
// executed in setup() scope (for each instance)
</script>
```

Support for combining `<script setup>` and `<script>` in the same component is limited to the scenarios described above. Specifically:

- Do **NOT** use a separate `<script>` section for options that can already be defined using `<script setup>`, such as `props` and `emits`.
- Variables created inside `<script setup>` are not added as properties to the component instance, making them inaccessible from the Options API. Mixing APIs in this way is strongly discouraged.

If you find yourself in one of the scenarios that is not supported then you should consider switching to an explicit [`setup()`](/api/composition-api-setup) function, instead of using `<script setup>`.

## Top-level `await` {#top-level-await}

Top-level `await` can be used inside `<script setup>`. The resulting code will be compiled as `async setup()`:

```vue
<script setup>
const post = await fetch(`/api/post/1`).then((r) => r.json())
</script>
```

In addition, the awaited expression will be automatically compiled in a format that preserves the current component instance context after the `await`.

:::warning Note
`async setup()` must be used in combination with `Suspense`, which is currently still an experimental feature. We plan to finalize and document it in a future release - but if you are curious now, you can refer to its [tests](https://github.com/vuejs/core/blob/main/packages/runtime-core/__tests__/components/Suspense.spec.ts) to see how it works.
:::

## Generics <sup class="vt-badge ts" /> {#generics}

Generic type parameters can be declared using the `generic` attribute on the `<script>` tag:

```vue
<script setup lang="ts" generic="T">
defineProps<{
  items: T[]
  selected: T
}>()
</script>
```

The value of `generic` works exactly the same as the parameter list between `<...>` in TypeScript. For example, you can use multiple parameters, `extends` constraints, default types, and reference imported types:

```vue
<script
  setup
  lang="ts"
  generic="T extends string | number, U extends Item"
>
import type { Item } from './types'
defineProps<{
  id: T
  list: U[]
}>()
</script>
```

## Restrictions {#restrictions}

- Due to the difference in module execution semantics, code inside `<script setup>` relies on the context of an SFC. When moved into external `.js` or `.ts` files, it may lead to confusion for both developers and tools. Therefore, **`<script setup>`** cannot be used with the `src` attribute.
- `<script setup>` does not support In-DOM Root Component Template.([Related Discussion](https://github.com/vuejs/core/issues/8391))
