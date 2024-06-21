## Game > GameAnvil > サーバー概念の説明 > Suspendable



## Suspendable

次の内容は、ファイバーベースのコードを作成する際の注意事項です。ユーザーは、ファイバー単位に気を使う必要はありませんが、以下の注意事項を必ず守ってください。

- **Suspend**は、ファイバーがコードの実行を他のファイバーに譲り、待機状態に入ることを意味します。これをファイバーブロッキングと表現できます。
- Suspendを使用するためには**throws SuspendExecution**という例外シグネチャを必ず追加する必要があります。これは、そのメソッドがファイバーを中断させる可能性のあるブロックコードが含まれていることをエンジンに伝える約束です。

```
void someSuspendableMethod() throws SuspendExecution {

// some fiber-blocking call can be here

}
```

- もし特殊な理由でthrows SuspendExecution例外シグネチャを指定できない場合は、**@Suspendable**アノテーション(annotation)を使用できます。それ以外の場合は、必ずthrows SuspendExecution例外シグネチャを優先的に使用してください。

```
@Suspendable
void someSuspendableMethod() {

// some blocking call can be here

}
```

- もちろん、このようなSuspendableメソッドを呼び出すメソッドもSuspendableになります。例えば、前述したsomeSuspendableMethod()を呼び出すsomeCaller()というメソッドがある場合、someCaller()も必ずSuspendableに宣言する必要があります。

```
void someCaller() throws SuspendExecution {

    someSuspendableMethod();

}
```

- SuspendExecutionは、エンジンで使用するQuasarライブラリがファイバーを処理するために使用する特殊な技法です。 ファイバーがまだJavaの標準ではないため、Quasarライブラリが使用する一種の迂回テクニックで、実際の例外ではありません。したがって、絶対にSuspendExecution例外をcatchしてはいけません。よく発生するエラーのため注意してください。

```
// !!間違った使用方法!!

void someCaller() {

    try {

        someSuspendableMethod();

    } catch (SuspendExecution e) {
        // 絶対にSuspendExecutionを明示的にcatchしてはいけません！
    }
}
```

- しかし、例外全体をcatchすることは問題ありません。この部分については自動で処理してくれます。

```
void someCaller() throws SuspendExeuction {

    try {

        someSuspendableMethod();

    } catch (Exception e) {
        // 問題なし
    }
}
```
