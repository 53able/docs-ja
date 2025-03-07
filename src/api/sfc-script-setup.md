# \<script setup> {#script-setup}

`<script setup>` は単一ファイルコンポーネント（SFC）内で Composition API を使用するコンパイル時のシンタックスシュガー（糖衣構文）です。SFC と Composition API の両方を使うならば、おすすめの構文です。これは通常の `<script>` 構文よりも、多くの利点があります:

- ボイラープレートが少なくて、より簡潔なコード
- 純粋な TypeScript を使ってプロパティと発行されたイベントを宣言する機能
- 実行時のパフォーマンスの向上（テンプレートは中間プロキシなしに同じスコープ内のレンダー関数にコンパイルされます）
- IDE で型推論のパフォーマンス向上（言語サーバーがコードから型を抽出する作業が減ります）

## 基本の構文 {#basic-syntax}

この構文を導入するには、`setup` 属性を `<script>` ブロックに追加します:

```vue
<script setup>
console.log('hello script setup')
</script>
```

内部のコードは、コンポーネントの `setup()` 関数の内容としてコンパイルされます。これはつまり、通常の `<script>` とは違って、コンポーネントが最初にインポートされたときに一度だけ実行されるのではなく、`<script setup>` 内のコードは **コンポーネントのインスタンスが作成されるたびに実行される** ということです。

### トップレベルのバインディングはテンプレートに公開 {#top-level-bindings-are-exposed-to-template}

`<script setup>` を使用する場合、`<script setup>` 内で宣言されたトップレベルのバインディング（変数、関数宣言、インポートを含む）は、テンプレートで直接使用できます:

```vue
<script setup>
// 変数
const msg = 'Hello!'

// 関数
function log() {
  console.log(msg)
}
</script>

<template>
  <button @click="log">{{ msg }}</button>
</template>
```

インポートも同じように公開されます。これはつまり、インポートされたヘルパー関数を `methods` オプションで公開することなくテンプレート内の式で直接使用できます:

```vue
<script setup>
import { capitalize } from './helpers'
</script>

<template>
  <div>{{ capitalize('hello') }}</div>
</template>
```

## リアクティビティー {#reactivity}

リアクティブな状態は [リアクティビティー API](./reactivity-core) を使って明示的に作成する必要があります。`setup()` 関数から返された値と同じように、テンプレート内で参照されるときに ref は自動的にアンラップされます:

```vue
<script setup>
import { ref } from 'vue'

const count = ref(0)
</script>

<template>
  <button @click="count++">{{ count }}</button>
</template>
```

## コンポーネントの使用 {#using-components}

`<script setup>` のスコープ内の値は、カスタムコンポーネントのタグ名としても直接使用できます:

```vue
<script setup>
import MyComponent from './MyComponent.vue'
</script>

<template>
  <MyComponent />
</template>
```

`MyComponent` を変数として参照していると考えてください。JSX を使ったことがあれば、このメンタルモデルは似ています。ケバブケースの `<my-component>` も同じようにテンプレートで動作します。しかし、一貫性を保つために、パスカルケースのコンポーネントタグを強く推奨します。これはネイティブのカスタム要素と区別するのにも役立ちます。

### 動的コンポーネント {#dynamic-components}

コンポーネントは、文字列キーで登録されるのではなく変数として参照されるため、`<script setup>` 内で動的コンポーネントを使う場合は、動的な `:is` バインディングを使う必要があります:

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

三項演算子で変数としてコンポーネントをどのように使うことができるかに注意してください。

### 再帰的コンポーネント {#recursive-components}

SFC はそのファイル名を介して、暗黙的に自身を参照できます。例えば、`FooBar.vue` というファイル名は、そのテンプレート内で `<FooBar/>` として自身を参照できます。

これはインポートされたコンポーネントよりも優先度が低いことに注意してください。コンポーネントの推論された名前と競合する名前付きインポートがある場合、インポートでエイリアスを作成できます:

```js
import { FooBar as FooBarChild } from './components'
```

### 名前空間付きコンポーネント {#namespaced-components}

`<Foo.Bar>` のようにドット付きのコンポーネントタグを使って、オブジェクトプロパティの下にネストしたコンポーネントを参照できます。これは単一のファイルから複数のコンポーネントをインポートするときに便利です:

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

## カスタムディレクティブの使用 {#using-custom-directives}

グローバルに登録されたカスタムディレクティブは通常通りに動作します。`<script setup>` ではローカルのカスタムディレクティブは明示的に登録する必要はありませんが、`vNameOfDirective` という命名規則に従う必要があります:

```vue
<script setup>
const vMyDirective = {
  beforeMount: (el) => {
    // 要素を使って何かする
  }
}
</script>
<template>
  <h1 v-my-directive>This is a Heading</h1>
</template>
```

他の場所からディレクティブをインポートする場合、必須の命名規則に合うようにリネームすることができます:

```vue
<script setup>
import { myDirective as vMyDirective } from './MyDirective.js'
</script>
```

## defineProps() & defineEmits() {#defineprops-defineemits}

完全な型推論のサポートつきで `props` と `emits` のようなオプションを宣言するために、`defineProps` と `defineEmits` の API を使用できます。これらは `<script setup>` の中で自動的に利用できるようになっています:

```vue
<script setup>
const props = defineProps({
  foo: String
})

const emit = defineEmits(['change', 'delete'])
// セットアップのコード
</script>
```

- `defineProps` と `defineEmits` は、`<script setup>` 内でのみ使用可能な**コンパイラーマクロ**です。インポートする必要はなく、`<script setup>` が処理されるときにコンパイルされます。

- `defineProps` は `props` オプションと同じ値を受け取り、`defineEmits` は `emits` オプションと同じ値を受け取ります。

- `defineProps` と `defineEmits` は、渡されたオプションに基づいて、適切な型の推論を行います。

- `defineProps` および `defineEmits` に渡されたオプションは、setup のスコープからモジュールのスコープに引き上げられます。そのため、オプションは setup のスコープで宣言されたローカル変数を参照できません。参照するとコンパイルエラーになります。しかし、インポートされたバインディングはモジュールのスコープに入っているので、参照できます。

### 型のみの props/emit 宣言<sup class="vt-badge ts" /> {#type-only-props-emit-declarations}

`defineProps` や `defineEmits` にリテラル型の引数を渡すことで、純粋な型の構文を使って props や emits を宣言することもできます:

```ts
const props = defineProps<{
  foo: string
  bar?: number
}>()

const emit = defineEmits<{
  (e: 'change', id: number): void
  (e: 'update', value: string): void
}>()

// 3.3+: より簡潔な代替構文
const emit = defineEmits<{
  change: [id: number] // 名前付きタプル構文
  update: [value: string]
}>()
```

- `defineProps` または `defineEmits` は、実行時の宣言か型宣言のどちらかしか使用できません。両方を同時に使用すると、コンパイルエラーになります。

- 型宣言を使用する場合は、同等の実行時宣言が静的解析から自動的に生成されるため、二重の宣言を行う必要がなくなり、実行時の正しい動作が保証されます。

  - 開発モードでは、コンパイラーは型から対応する実行時バリデーションを推測しようとします。上記の例では `foo: string` という型からは `foo: String` が推測されます。もし型がインポートされた型への参照である場合、コンパイラーは外部ファイルの情報を持っていないので、推測される結果は `foo: null`（`any` 型と同じ）になります。

  - プロダクションモードでは、バンドルサイズを小さくするために、コンパイラーが配列形式の宣言を生成します（上記の props は `['foo', 'bar']` にコンパイルされます）。

- バージョン 3.2 以下では、`defineProps()` の型引数はローカルの型への参照か、型リテラルのどちらかに制限されていました。

  この制限は 3.3 で解決されました。Vue の最新バージョンは型引数の位置でインポートされた複雑な型の限定されたセットを参照することをサポートしています。しかし、型からランタイムへの変換は依然として AST ベースであるため、実際の型解析を必要とするいくつかの複雑な型、例えば条件型など、はサポートされていません。条件型は単一のプロパティの型には使用できますが、props オブジェクト全体の型には使用できません。

### 型宣言を使用時のデフォルトのプロパティ値 {#default-props-values-when-using-type-declaration}

型のみの `defineProps` 宣言の欠点は、プロパティのデフォルト値を提供する方法がないことです。この問題を解決するために、`withDefaults` コンパイラーマクロも用意されています:

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

これは、同等な実行時のプロパティの `default` オプションにコンパイルされます。さらに、`withDefaults` ヘルパーは、デフォルト値の型チェックを行います。また、返される `props` の型が、デフォルト値が宣言されているプロパティに対して、省略可能フラグが削除されていることを保証します。

## defineExpose() {#defineexpose}

`<script setup>` を使用したコンポーネントは、**デフォルトで閉じられています**。つまり、テンプレート参照や `$parent` チェーンを介して取得されるコンポーネントのパブリックインスタンスは、`<script setup>` 内で宣言されたバインディングを公開**しません**。

`<script setup>` コンポーネントのプロパティを明示的に公開するには、`defineExpose` コンパイラーマクロを使用します:

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

親がテンプレート参照を介してこのコンポーネントのインスタンスを取得すると、取得されたインスタンスは `{ a: number, b: number }` という形状になります（ref は通常のインスタンスと同様、自動的にアンラップされます）。

## defineOptions() {#defineoptions}

このマクロは、`<script>` ブロックを別途使用することなく、`<script setup>` 内で直接コンポーネントオプションを宣言するために使用できます:

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

- 3.3 以上でのみサポートされています。
- これはマクロです。オプションはモジュールスコープに巻き上げられ、`<script setup>` 内のリテラル定数でないローカル変数にはアクセスできません。

## defineSlots() <sup class="vt-badge ts"/> {#defineslots}

このマクロは、スロット名とプロパティの型チェックのために IDE に型ヒントを提供するために使用することができます。

`defineSlots()` は型パラメーターのみを受け取り、実行時引数はありません。型パラメーターは、プロパティキーがスロット名で、値の型がスロット関数である型リテラルでなければなりません。関数の最初の引数はスロットが受け取ることを期待するプロパティで、その型はテンプレート内のスロットのプロパティに使用されることになります。戻り値の型は現在無視されており、`any` を指定することができますが、将来的にはスロットの内容チェックのために活用するかもしれません。

また、`slots` オブジェクトを返します。これはセットアップコンテキストで公開されている `slots` オブジェクト、または `useSlots()` が返す `slots` オブジェクトと同じものです。

```vue
<script setup lang="ts">
const slots = defineSlots<{
  default(props: { msg: string }): any
}>()
</script>
```

- 3.3 以上でのみサポートされています。

## `useSlots()` & `useAttrs()` {#useslots-useattrs}

`<script setup>` 内で `slots` や `attrs` を使用することは比較的少ないはずです。なぜなら、テンプレート内で `$slots` と `$attrs` として直接アクセスできるからです。万が一、必要になった場合には、それぞれ `useSlots` と `useAttrs` ヘルパーを使用してください:

```vue
<script setup>
import { useSlots, useAttrs } from 'vue'

const slots = useSlots()
const attrs = useAttrs()
</script>
```

`useSlots` と `useAttrs` は、`setupContext.slots` と `setupContext.attrs` と同等のものを返す実際のランタイム関数です。これらは通常の Composition API の関数内でも使用できます。

## 通常の `<script>` との併用 {#usage-alongside-normal-script}

`<script setup>` は、通常の `<script>` と一緒に使うことができます。次のことが必要な場合は、通常の `<script>` が必要になることがあります:

- `inheritAttrs` や、プラグインで有効になるカスタムオプションなど、`<script setup>` では表現できないオプションを宣言する
- 名前付きのエクスポートを宣言する
- 副作用を実行したり、一度しか実行してはいけないオブジェクトを作成する

```vue
<script>
// 通常の <script>、モジュールのスコープで実行される（1 回だけ）
runSideEffectOnce()

// 追加のオプションを宣言
export default {
  inheritAttrs: false,
  customOptions: {}
}
</script>

<script setup>
// setup() のスコープで実行される(インスタンスごとに)
</script>
```

同じコンポーネント内で `<script setup>` と `<script>` を組み合わせることは、上記のシナリオに限定してサポートします。具体的には:

- `props` や `emits` のような `<script setup>` で定義できるオプションは、`<script>` セクションで定義**しない**でください。
- `<script setup>` 内で作成された変数は、コンポーネントインスタンスのプロパティとして追加されないので、Options API からはアクセスできません。このように API を混在させることは、強くお勧めしません。

もし、サポートされていないシナリオに遭遇した場合は、`<script setup>` の代わりに、明示的な [`setup()`](/api/composition-api-setup) 関数に切り替えることを検討する必要があります。

## トップレベルの `await` {#top-level-await}

`<script setup>` の中ではトップレベルの `await` を使うことができます。その結果、コードは `async setup()` としてコンパイルされます:

```vue
<script setup>
const post = await fetch(`/api/post/1`).then((r) => r.json())
</script>
```

さらに、await の対象の式は、`await` 後の現在のコンポーネントのインスタンスのコンテキストを保持する形式で自動的にコンパイルされます。

:::warning Note
`async setup()` は、現在まだ実験的な機能である `Suspense` と組み合わせて使用する必要があります。将来のリリースで完成させてドキュメント化する予定ですが、もし今興味があるのであれば、その[テスト](https://github.com/vuejs/core/blob/main/packages/runtime-core/__tests__/components/Suspense.spec.ts)を参照することで、どのように動作するかを確認できます。
:::

### ジェネリクス <sup class="vt-badge ts" /> {#generics}

`<script>` タグの `generic` 属性を使ってジェネリック型パラメーターを宣言できます:

```vue
<script setup lang="ts" generic="T">
defineProps<{
  items: T[]
  selected: T
}>()
</script>
```

`generic` の値は、TypeScript の `<...>` 間のパラメーターリストと全く同じ働きをします。例えば、複数のパラメーター、`extends` 制約、デフォルトの型、インポートされた型を参照できます:

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

## 制限 {#restrictions}

モジュールの実行セマンティクスの違いにより、`<script setup>` 内のコードは、SFC のコンテキストに依存しています。外部の `.js` や `.ts` ファイルに移動すると、開発者とツールの両方に混乱を招く可能性があります。そのため、**`<script setup>`** は、`src` 属性と一緒に使うことはできません。
