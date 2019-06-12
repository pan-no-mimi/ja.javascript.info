# 引用符で囲まれた文字列を見つける

ダブルクォート `subject:"..."` 内の文字列を見つける正規表現を作成してください。

重要なことは、文字列は JavaScript の文字列がするのと同じ方法でエスケープをサポートする必要があるということです。例えば、引用符は `subject:\"` として挿入でき、改行は `subject:\n` 、スラッシュ自身は `subject:\\` です。

```js
let str = "Just like \"here\".";
```

エスケープされた引用符 `subject:\"` は文字列を終了しないことが重要です。

そのため、途中でエスケープされた引用符は無視して、ある引用符から他の引用符に目を向ける必要があります。

これがこのタスクの肝心な部分です。そうでなければ答えは明らかです。

マッチさせる文字列の例です:
```js
.. *!*"test me"*/!* ..  
.. *!*"Say \"Hello\"!"*/!* ... (内側にエスケープされた引用符がある)
.. *!*"\\"*/!* ..  (内側にダブルスラッシュがある)
.. *!*"\\ \""*/!* ..  (内側にダブルスラッシュとエスケープされた引用符がある)
```

JavaScript では、次のように、そのまま文字列として渡すにはスラッシュを2重にする必要があります:

```js run
let str = ' .. "test me" .. "Say \\"Hello\\"!" .. "\\\\ \\"" .. ';

// メモリ内部の文字列
alert(str); //  .. "test me" .. "Say \"Hello\"!" .. "\\ \"" ..
```