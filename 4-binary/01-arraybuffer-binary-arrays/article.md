# ArrayBuffer, binary arrays

Web 開発では、ファイル(作成、更新、ダウンロード)を処理するときにバイナリデータに出くわします。その他の典型的なユースケースは画像処理です。

これらはすべて JavaScript で可能です。また、バイナリ操作も高性能です。

ですが、多くのクラスが存在するため少し混乱しやすいです。いくつか例を挙げます:
- `ArrayBuffer`, `Uint8Array`, `DataView`, `Blob`, `File`, etc.

JavaScript でのバイナリデータは、他の言語と比べて非標準的な方法で実装されています。しかし、一度整理できれば、すべてがとても簡単になります。

**基本となるバイナリオブジェクトは `ArrayBuffer` です。これは固定長の連続したメモリ領域への参照です。**

次のようにして生成できます:
```js run
let buffer = new ArrayBuffer(16); // 長さ 16 のバッファを作成
alert(buffer.byteLength); // 16
```

16バイトの連続したメモリ領域が割り当てられ、事前にゼロで埋められます。

```warn header="`ArrayBuffer` は何かの配列ではありません"
混乱の元は排除しましょう。`ArrayBuffer` は `Array` とは何の共通点もありません。:
- 長さは固定で、増減することはできません。
- メモリの中で、長さちょうどのスペースをとります。
- 個々のバイトにアクセスするには、`buffer[index]` ではなく、別の "view" オブジェクトが必要です。
```

`ArrayBuffer` はメモリ領域です。何が格納されているでしょう？それは分かりません。単に生のバイト列です。

**`ArrayBuffer` を操作するには、"ビュー(view)" オブジェクトを使用する必要があります。**

ビューオブジェクトは自身には何も格納しません。それは `ArrayBuffer` に格納されたバイトへの解釈を与える "メガネ" です。

例:

- **`Uint8Array`** -- `ArrayBuffer` にある各バイトを、 0 から 255 (1バイトは8ビット)までの値となる、別々の数として扱います。このような値は "8ビット符号なし整数(8-bit unsigned integer)" と呼ばれます。
- **`Uint16Array`** -- 各2バイトを、0 から 65535 までの値となる整数として扱います。これは "16ビット符号なし整数" と呼ばれます。
- **`Uint32Array`** -- 各4バイトを 0 から 4294967295 までの値となる整数として扱います。これは "32ビット符号なし整数" と呼ばれます。
- **`Float64Array`** -- 各8バイトを <code>5.0x10<sup>-324</sup></code> から <code>1.8x10<sup>308</sup></code> までの値となる浮動小数点として扱います。

したがって、16バイトの `ArrayBuffer` にあるバイナリデータは 16 個の "小さい数字", あるいは 8 個のより大きい数字(2バイトずつ), または 4 個のより大きな数字(4バイトずつ), 高精度の2個の浮動小数点値(8バイトずつとして解釈できます。

![](arraybuffer-views.png)

`ArrayBuffer` はコアとなるオブジェクト、すべてのもののルート、生のバイナリデータです。

ですが、そこに書き込んだり反復しようとする場合、基本的にほぼすべての操作に対して、ビューを使用する必要があります。e.g:

```js run
let buffer = new ArrayBuffer(16); // 長さ 16 のバッファを作成

*!*
let view = new Uint32Array(buffer); // 32 ビット整数列としてバッファを扱います

alert(Uint32Array.BYTES_PER_ELEMENT); // 4 バイト毎の整数
*/!*

alert(view.length); // 4, 多くの整数が格納されます
alert(view.byteLength); // 16, バイトサイズ

// 値の書き込み
view[0] = 123456;

// 値のイテレート
for(let num of view) {
  alert(num); // 123456, 次に 0, 0, 0 (全部で 4 つの値)
}

```

## TypedArray

これらすべてのビュー (`Uint8Array`, `Uint32Array`, etc) の共通の用語は、[TypedArray](https://tc39.github.io/ecma262/#sec-typedarray-objects) です。これらは同じメソッドとプロパティのセットを共有します。

これらは通常の配列によく似ています: インデックスがあり、反復可能(iterable)です。

型付き配列のコンストラクタ (`Int8Array` でも `Float64Array` でも構いません)は、引数の種類に応じて異なる振る舞いをします。

5 つの引数のパターンがあります:

```js
new TypedArray(buffer, [byteOffset], [length]);
new TypedArray(object);
new TypedArray(typedArray);
new TypedArray(length);
new TypedArray();
```

1. `ArrayBuffer` の引数が与えられると、それに対するビューが作られます。我々はすでにその構文を使いました。

    オプションで、`byteOffset` で開始位置(デフォルトは 0)、`length` で長さ(デフォルトではバッファの終わりまで) を指定することができ、その場合はビューは `buffer` の一部だけをカバーします。 

2. `Array` または配列ライクなオブジェクトが与えられた場合は、同じ長さの型付き配列を生成し、内容をコピーします。

    これを使って配列にデータを事前に埋め込むことができます:
    ```js run
    *!*
    let arr = new Uint8Array([0, 1, 2, 3]);
    */!*
    alert( arr.length ); // 4
    alert( arr[1] ); // 1
    ```
3. If another `TypedArray` is supplied, it does the same: creates a typed array of the same length and copies values. Values are converted to the new type in the process.
    ```js run
    let arr16 = new Uint16Array([1, 1000]);
    *!*
    let arr8 = new Uint8Array(arr16);
    */!*
    alert( arr8[0] ); // 1
    alert( arr8[1] ); // 232 (tried to copy 1000, but can't fit 1000 into 8 bits)
    ```

4. For a numeric argument `length` -- creates the typed array to contain that many elements. Its byte length will be `length` multiplied by the number of bytes in a single item `TypedArray.BYTES_PER_ELEMENT`:
    ```js run
    let arr = new Uint16Array(4); // create typed array for 4 integers
    alert( Uint16Array.BYTES_PER_ELEMENT ); // 2 bytes per integer
    alert( arr.byteLength ); // 8 (size in bytes)
    ```

5. Without arguments, creates an zero-length typed array.

We can create a `TypedArray` directly, without mentioning `ArrayBuffer`. But a view cannot exist without an underlying `ArrayBuffer`, so gets created automatically in all these cases except the first one (when provided).

To access the `ArrayBuffer`, there are properties:
- `arr.buffer` -- references the `ArrayBuffer`.
- `arr.byteLength` -- the length of the `ArrayBuffer`.

So, we can always move from one view to another:
```js
let arr8 = new Uint8Array([0, 1, 2, 3]);

// another view on the same data
let arr16 = new Uint16Array(arr8.buffer);
```


Here's the list of typed arrays:

- `Uint8Array`, `Uint16Array`, `Uint32Array` -- for integer numbers of 8, 16 and 32 bits.
  - `Uint8ClampedArray` -- for 8-bit integers, "clamps" them on assignment (see below).
- `Int8Array`, `Int16Array`, `Int32Array` -- for signed integer numbers (can be negative).
- `Float32Array`, `Float64Array` -- for signed floating-point numbers of 32 and 64 bits.

```warn header="No `int8` or similar single-valued types"
Please note, despite of the names like `Int8Array`, there's no single-value type like `int`, or `int8` in JavaScript.

That's logical, as `Int8Array` is not an array of these individual values, but rather a view on `ArrayBuffer`.
```

### Out-of-bounds behavior

What if we attempt to write an out-of-bounds value into a typed array? There will be no error. But extra bits are cut-off.

For instance, let's try to put 256 into `Uint8Array`. In binary form, 256 is `100000000` (9 bits), but `Uint8Array` only provides 8 bits per value, that makes the available range from 0 to 255.

For bigger numbers, only the rightmost (less significant) 8 bits are stored, and the rest is cut off:

![](8bit-integer-256.png)

So we'll get zero.

For 257, the binary form is `100000001` (9 bits), the rightmost 8 get stored, so we'll have `1` in the array:

![](8bit-integer-257.png)

In other words, the number modulo 2<sup>8</sup> is saved.

Here's the demo:

```js run
let uint8array = new Uint8Array(16);

let num = 256;
alert(num.toString(2)); // 100000000 (binary representation)

uint8array[0] = 256;
uint8array[1] = 257;

alert(uint8array[0]); // 0
alert(uint8array[1]); // 1
```

`Uint8ClampedArray` is special in this aspect, its behavior is different. It saves 255 for any number that is greater than 255, and 0 for any negative number. That behavior is useful for image processing.

## TypedArray methods

`TypedArray` has regular `Array` methods, with notable exceptions.

We can iterate, `map`, `slice`, `find`, `reduce` etc.

There are few things we can't do though:

- No `splice` -- we can't "delete" a value, because typed arrays are views on a buffer, and these are fixed, contiguous areas of memory. All we can do is to assign a zero.
- No `concat` method.

There are two additional methods:

- `arr.set(fromArr, [offset])` copies all elements from `fromArr` to the `arr`, starting at position `offset` (0 by default).
- `arr.subarray([begin, end])` creates a new view of the same type from `begin` to `end` (exclusive). That's similar to `slice` method (that's also supported), but doesn't copy anything -- just creates a new view, to operate on the given piece of data.

These methods allow us to copy typed arrays, mix them, create new arrays from existing ones, and so on.



## DataView

[DataView](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/DataView) is a special super-flexible "untyped" view over `ArrayBuffer`. It allows to access the data on any offset in any format.

- For typed arrays, the constructor dictates what the format is. The whole array is supposed to be uniform. The i-th number is `arr[i]`.
- With `DataView` we access the data with methods like `.getUint8(i)` or `.getUint16(i)`. We choose the format at method call time instead of the construction time.

The syntax:

```js
new DataView(buffer, [byteOffset], [byteLength])
```

- **`buffer`** -- the underlying `ArrayBuffer`. Unlike typed arrays, `DataView` doesn't create a buffer on its own. We need to have it ready.
- **`byteOffset`** -- the starting byte position of the view (by default 0).
- **`byteLength`** -- the byte length of the view (by default till the end of `buffer`).

For instance, here we extract numbers in different formats from the same buffer:

```js run
let buffer = new Uint8Array([255, 255, 255, 255]).buffer;

let dataView = new DataView(buffer);

// get 8-bit number at offset 0
alert( dataView.getUint8(0) ); // 255

// now get 16-bit number at offset 0, that's 2 bytes, both with max value
alert( dataView.getUint16(0) ); // 65535 (biggest 16-bit unsigned int)

// get 32-bit number at offset 0
alert( dataView.getUint32(0) ); // 4294967295 (biggest 32-bit unsigned int)

dataView.setUint32(0, 0); // set 4-byte number to zero
```

`DataView` is great when we store mixed-format data in the same buffer. E.g we store a sequence of pairs (16-bit integer, 32-bit float). Then `DataView` allows to access them easily.

## Summary

`ArrayBuffer` is the core object, a reference to the fixed-length contiguous memory area.

To do almost any operation on `ArrayBuffer`, we need a view.

- It can be a `TypedArray`:
    - `Uint8Array`, `Uint16Array`, `Uint32Array` -- for unsigned integers of 8, 16, and 32 bits.
    - `Uint8ClampedArray` -- for 8-bit integers, "clamps" them on assignment.
    - `Int8Array`, `Int16Array`, `Int32Array` -- for signed integer numbers (can be negative).
    - `Float32Array`, `Float64Array` -- for signed floating-point numbers of 32 and 64 bits.
- Or a `DataView` -- the view that uses methods to specify a format, e.g. `getUint8(offset)`.

In most cases we create and operate directly on typed arrays, leaving `ArrayBuffer` under cover, as a "common discriminator". We can access it as `.buffer` and make another view if needed.

There are also two additional terms:
- `ArrayBufferView` is an umbrella term for all these kinds of views.
- `BufferSource` is an umbrella term for `ArrayBuffer` or `ArrayBufferView`.

These are used in descriptions of methods that operate on binary data. `BufferSource` is one of the most common terms, as it means "any kind of binary data" -- an `ArrayBuffer` or a view over it. 


Here's a cheatsheet:

![](arraybuffer-view-buffersource.png)