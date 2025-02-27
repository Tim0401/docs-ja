# リアクティビティー API: コア

:::info See also
よりリアクティビティー API を理解するために、ガイド内の次の章を読むことを推奨します:

- [リアクティビティーの基礎](/guide/essentials/reactivity-fundamentals.html) (API 環境設定が Composition API に設定されている場合)
- [Reactivity in Depth](/guide/extras/reactivity-in-depth.html)
  :::

## ref()

内部値を受け取り、リアクティブでミュータブルな ref オブジェクトを返します。またそれは、内部値を示した単一プロパティである `.value` を持っています。

- **型**

  ```ts
  function ref<T>(value: T): Ref<UnwrapRef<T>>

  interface Ref<T> {
    value: T
  }
  ```

- **詳細**

  ref オブジェクトはミュータブルです - すなわち、`.value` に新しい値を割り当てることができます。それはまたリアクティブです。つまり、`.value` へのあらゆる読み取り操作は追跡され、書き込み操作は関連する作用を引き起こします。

  ref の値としてオブジェクトが代入された場合、[reactive()](#reactive) でそのオブジェクトは深いリアクティブになります。これはオブジェクトがネストした ref を含む場合、それが深くアンラップされることも意味します。

  深い変換を避けるためには、代わりに [`shallowRef()`](./reactivity-advanced.html#shallowref) 使用します。

- **例**

  ```js
  const count = ref(0)
  console.log(count.value) // 0

  count.value++
  console.log(count.value) // 1
  ```

- **参照:**
  - [Guide - Reactive Variables with `ref()`](/guide/essentials/reactivity-fundamentals.html#reactive-variables-with-ref)
  - [Guide - Typing `ref()`](/guide/typescript/composition-api.html#typing-ref)

## computed()

ゲッター関数を受け取り、ゲッターからの戻り値に対して読み取り専用のリアクティブな [ref](#ref) オブジェクトを返します。また、`get` 関数と `set` 関数を持つオブジェクトを受け取り、書き込み可な ref オブジェクトを作成することができます。

- **型**

  ```ts
  // 読み取り専用
  function computed<T>(
    getter: () => T,
    // 下記の "Computed Debugging" リンクをご参照ください
    debuggerOptions?: DebuggerOptions
  ): Readonly<Ref<Readonly<T>>>

  // 書き込み可
  function computed<T>(
    options: {
      get: () => T
      set: (value: T) => void
    },
    debuggerOptions?: DebuggerOptions
  ): Ref<T>
  ```

- **例**

  読み取り専用の算出 ref の作成:

  ```js
  const count = ref(1)
  const plusOne = computed(() => count.value + 1)

  console.log(plusOne.value) // 2

  plusOne.value++ // エラー
  ```

  書き込み可の算出 ref の作成:

  ```js
  const count = ref(1)
  const plusOne = computed({
    get: () => count.value + 1,
    set: (val) => {
      count.value = val - 1
    }
  })

  plusOne.value = 1
  console.log(count.value) // 0
  ```

  デバッグ:

  ```js
  const plusOne = computed(() => count.value + 1, {
    onTrack(e) {
      debugger
    },
    onTrigger(e) {
      debugger
    }
  })
  ```

- **参照:**
  - [Guide - Computed Properties](/guide/essentials/computed.html)
  - [Guide - Computed Debugging](/guide/extras/reactivity-in-depth.html#computed-debugging)
  - [Guide - Typing `computed()`](/guide/typescript/composition-api.html#typing-computed)

## reactive()

オブジェクトのリアクティブなプロキシを返します。

- **型**

  ```ts
  function reactive<T extends object>(target: T): UnwrapNestedRefs<T>
  ```

- **詳細**

  リアクティブの変換は"深い"です: それは、全てのネストされたプロパティに対して影響を及ぼします。リアクティブオブジェクトは、リアクティビティーを維持しながら、[refs](#ref) であるすべてのプロパティを深くアンラップします。

  また、ref がリアクティブな配列の要素や、`Map` のようなネイティブなコレクションタイプとしてアクセスされた場合、ref のアンラップは行われないことにも注意が必要です。

  深い変換を避け、ルートレベルのリアクティビティーのみを保持するためには、代わりに [shallowReactive()](./reactivity-advanced.html#shallowreactive) 使用します。

  返されたオブジェクトとそのネストされたオブジェクトは [ES Proxy](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy) でラップされ、元のオブジェクトと**等しくなりません**。元のオブジェクトに依存することを避け、リアクティブなプロキシのみを使用することが推奨されます。

- **例**

  リアクティブなオブジェクトの作成:

  ```js
  const obj = reactive({ count: 0 })
  obj.count++
  ```

  Ref のアンラップ:

  ```ts
  const count = ref(1)
  const obj = reactive({ count })

  // ref はアンラップされます
  console.log(obj.count === count.value) // true

  // `obj.count` を更新します
  count.value++
  console.log(count.value) // 2
  console.log(obj.count) // 2

  // `count` ref も更新します
  obj.count++
  console.log(obj.count) // 3
  console.log(count.value) // 3
  ```

  配列やコレクションの要素としてアクセスする場合、ref は**アンラップされない**ことに注意してください:

  ```js
  const books = reactive([ref('Vue 3 Guide')])
  // ここでは .value が必要です
  console.log(books[0].value)

  const map = reactive(new Map([['count', ref(0)]]))
  // ここでは .value が必要です
  console.log(map.get('count').value)
  ```

  `reactive` プロパティに [ref](#ref) を割り当てると、その ref も自動的にアンラップされます:

  ```ts
  const count = ref(1)
  const obj = reactive({})

  obj.count = count

  console.log(obj.count) // 1
  console.log(obj.count === count.value) // true
  ```

- **参照:**
  - [Guide - Reactivity Fundamentals](/guide/essentials/reactivity-fundamentals.html)
  - [Guide - Typing `reactive()`](/guide/typescript/composition-api.html#typing-reactive)

## readonly()

オブジェクト(リアクティブ、もしくはプレーン)または [ref](#ref) を受け取り、元のオブジェクトへの読み取り専用なプロキシを返します。

- **型**

  ```ts
  function readonly<T extends object>(
    target: T
  ): DeepReadonly<UnwrapNestedRefs<T>>
  ```

- **詳細**

  読み取り専用プロキシは深く、アクセスされた、ネストされたプロパティも読み取り専用になります。また、`reactive()` と同様にアンラップすることができますが、アンラップされた値も読み取り専用になります。

  深い変換を避けるためには、代わりに [shallowReadonly()](./reactivity-advanced.html#shallowreadonly) を使用します。

- **例**

  ```js
  const original = reactive({ count: 0 })

  const copy = readonly(original)

  watchEffect(() => {
    // リアクティビティーの追跡に有効です
    console.log(copy.count)
  })

  // 変数 original を変更することは、変数 copy に依存するウォッチャーが起動します
  original.count++

  // 変数 copy を変更することは、失敗し警告が発生します
  copy.count++ // warning!
  ```

## watchEffect()

関数を即座に実行しながら、その依存関係をリアクティブに追跡し、依存関係が変更されるたびに再実行します。

- **型**

  ```ts
  function watchEffect(
    effect: (onCleanup: OnCleanup) => void,
    options?: WatchEffectOptions
  ): StopHandle

  type OnCleanup = (cleanupFn: () => void) => void

  interface WatchEffectOptions {
    flush?: 'pre' | 'post' | 'sync' // 初期値: 'pre'
    onTrack?: (event: DebuggerEvent) => void
    onTrigger?: (event: DebuggerEvent) => void
  }

  type StopHandle = () => void
  ```

- **詳細**

  第 1 引数には、実行するエフェクト関数を指定します。エフェクト関数は、クリーンアップコールバックを登録するために使用することができる関数を受け取ります。クリーンアップコールバックは、次にエフェクトが再実行される直前に呼び出され、保留中の非同期リクエストなどの無効な副作用をクリーンアップするために使用できます（以下の例を参照してください）。

  第 2 引数は省略可能なオプションオブジェクトで、エフェクトのフラッシュタイミングを調整したり、エフェクトの依存関係をデバッグするために使用することができます。

  デフォルトでは、ウォッチャーはコンポーネントレンダリングの直前に実行されます。`flush: 'post'` を設定すると、コンポーネントのレンダリングが終わるまでウォッチャーを遅延させることができます。詳しくは[コールバックが実行されるタイミング](/guide/essentials/watchers.html#コールバックが実行されるタイミング)を参照してください。まれに、キャッシュを無効にする目的などで、リアクティブな依存関係が変化したときにすぐにウォッチャーを起動させる必要がある場合があります。これは `flush: 'sync'` を使って実現することができます。ただし、この設定は、複数のプロパティが同時に更新される場合、パフォーマンスやデータの一貫性に問題が生じる可能性があるため、注意して使用する必要があります。

  戻り値は、エフェクトの再実行を停止するために呼び出されるハンドル関数です。

- **例**

  ```js
  const count = ref(0)

  watchEffect(() => console.log(count.value))
  // -> logs 0

  count.value++
  // -> logs 1
  ```

  副作用のクリーンアップ:

  ```js
  watchEffect(async (onCleanup) => {
    const { response, cancel } = doAsyncWork(id.value)
    // `cancel` は `id` が変更された場合に呼ばれます
    // 前の保留中のリクエストが完了していない場合は
    // キャンセルされます
    onCleanup(cancel)
    data.value = await response
  })
  ```

  ウォッチャーの停止:

  ```js
  const stop = watchEffect(() => {})

  // ウォッチャーが不要になった場合:
  stop()
  ```

  オプション:

  ```js
  watchEffect(() => {}, {
    flush: 'post',
    onTrack(e) {
      debugger
    },
    onTrigger(e) {
      debugger
    }
  })
  ```

- **参照**:
  - [Guide - Watchers](/guide/essentials/watchers.html#watcheffect)
  - [Guide - Watcher Debugging](/guide/extras/reactivity-in-depth.html#watcher-debugging)

## watchPostEffect()

`flush: 'post'` オプションをつけた [`watchEffect()`](#watcheffect) のエイリアス

## watchSyncEffect()

`flush: 'sync'` オプションをつけた [`watchEffect()`](#watcheffect) のエイリアス

## watch()

1 つ以上のリアクティブなデータソースを監視し、ソースが変更されたときにコールバック関数を呼び出します。

- **型**

  ```ts
  // 単一ソースの監視
  function watch<T>(
    source: WatchSource<T>,
    callback: WatchCallback<T>,
    options?: WatchOptions
  ): StopHandle

  // 複数ソースの監視
  function watch<T>(
    sources: WatchSource<T>[],
    callback: WatchCallback<T[]>,
    options?: WatchOptions
  ): StopHandle

  type WatchCallback<T> = (
    value: T,
    oldValue: T,
    onCleanup: (cleanupFn: () => void) => void
  ) => void

  type WatchSource<T> =
    | Ref<T> // ref
    | (() => T) // ゲッター
    | T extends object
    ? T
    : never // リアクティブなオブジェクト

  interface WatchOptions extends WatchEffectOptions {
    immediate?: boolean // 初期値: false
    deep?: boolean // 初期値: false
    flush?: 'pre' | 'post' | 'sync' // 初期値: 'pre'
    onTrack?: (event: DebuggerEvent) => void
    onTrigger?: (event: DebuggerEvent) => void
  }
  ```

  > 読みやすくするため、型は単純化されています。

- **Details**

  `watch()` は初期状態では遅延します - つまり、監視されたソースが変更された時にだけコールバックが呼び出されます。

  第 1 引数はウォッチャーの**ソース**です。ソースは以下のいずれかになります:

  - 値を返すゲッター関数
  - ref
  - リアクティブなオブジェクト
  - ... もしくは上記の配列

  第 2 引数はソースが変更した時に呼ばれるコールバックです。 コールバックは 3 つの引数を受け取ります: 新しい値、古い値、そして副作用のクリーンアップコールバックを登録するための関数です。クリーンアップコールバックは、次に作用が再実行される直前に呼び出され、保留中の非同期リクエストのような無効化された副作用をクリーンアップするために使用することができます。

  複数のソースを監視する場合、コールバックはソース配列に対応する新しい値 / 古い値を含む 2 つの配列を受け取ります。

  省略可能な第 3 引数は、以下のオプションをサポートするオプションオブジェクトです:

  - **`immediate`**: ウォッチャーが作成されたら、すぐにコールバックを起動します。最初の呼び出しでは、古い値は `undefined` になります。
  - **`deep`**: オブジェクトの場合、深い変更の際にコールバックが発生するように、ソースの深い探索を強制します。詳しくは [Deep Watchers](/guide/essentials/watchers.html#deep-watchers) をご参照ください。
  - **`flush`**: コールバックのフラッシュタイミングを調整します。詳しくは[コールバックが実行されるタイミング](/guide/essentials/watchers.html#コールバックが実行されるタイミング)や [`watchEffect()`](/api/reactivity-core.html#watcheffect) をご参照ください。
  - **`onTrack / onTrigger`**: ウォッチャーの依存関係をデバッグします。詳しくは [Watcher Debugging](/guide/extras/reactivity-in-depth.html#watcher-debugging) をご参照ください。

  [`watchEffect()`](#watcheffect) と比較すると、`watch()` は以下のことができます:

  - 副作用を遅延的に実行します
  - どのような状態でウォッチャーを再実行させるべきかについて、より具体的に説明します
  - ウォッチ状態の前の値と現在の値の両方にアクセスします

- **例**

  ゲッターを監視する:

  ```js
  const state = reactive({ count: 0 })
  watch(
    () => state.count,
    (count, prevCount) => {
      /* ... */
    }
  )
  ```

  ref を監視する:

  ```js
  const count = ref(0)
  watch(count, (count, prevCount) => {
    /* ... */
  })
  ```

  複数のソースを監視する場合、コールバックはソース配列に対応した新しい値 / 古い値を含む配列を受け取ります:

  ```js
  watch([fooRef, barRef], ([foo, bar], [prevFoo, prevBar]) => {
    /* ... */
  })
  ```

  ゲッターソースを使う場合、ウォッチャーはゲッターの戻り値が変更されたときだけ起動します。もし、深い変更があってもコールバックを起動させたい場合は、`{ deep: true }` で明示的にウォッチャーをディープモードに強制する必要があります。ディープモードでは、コールバックが深い変更によって引き起こされた場合、新しい値と古い値は同じオブジェクトになることに注意してください:

  ```js
  const state = reactive({ count: 0 })
  watch(
    () => state,
    (newValue, oldValue) => {
      // newValue === oldValue
    },
    { deep: true }
  )
  ```

  リアクティブオブジェクトを直接ウォッチする場合、ウォッチャーは自動的にディープモードになります:

  ```js
  const state = reactive({ count: 0 })
  watch(state, () => {
    /* 状態への深い変更のトリガー */
  })
  ```

  `watch()` は [`watchEffect()`](#watcheffect) と同じフラッシュタイミングとデバッグオプションを共有します:

  ```js
  watch(source, callback, {
    flush: 'post',
    onTrack(e) {
      debugger
    }
  })
  ```

- **参照**:

  - [Guide - Watchers](/guide/essentials/watchers.html)
  - [Guide - Watcher Debugging](/guide/extras/reactivity-in-depth.html#watcher-debugging)
