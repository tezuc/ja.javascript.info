
私たちは2つのハンドラを使う必要があります: `document.onkeydown` と `document.onkeyup` です。


Set `pressed` は現在押されているキーを保持する必要があります。

最初のハンドラでそこに追加し、一方で2つ目のハンドラではそこから削除します。すべての `keydown` で必要なキーを押しているかをチェックし、その場合に関数を実行します。