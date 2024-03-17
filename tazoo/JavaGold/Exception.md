# Exception

## 例外と例外処理

- 実行時にプログラムがエラー(例外)が発生する = 例外がスローされる
  - スロー(throw)はJVMが行っている

- 実際実行時にエラーが発生したとき、突如異常終了することは防がないといけない
  - 例外処理というメカニズムへ誘導し、そこで対応を記述する

## 例外クラス   

- checked例外(例外処理が**必須**)
  - JVM以外が原因で発生する例外(ex:DBへアクセスできない)
  - **RuntimeExceptionクラス以外のExceptionのサブクラス**
- unchecked例外(例外処理は**任意**)
  - プログラムの例外処理では復旧できない例外(ex:実行時例外、メモリ不足)
  - **Errorクラスおよびそのサブクラス**
  - **RuntimeExceptionおよびそのサブクラス**

### 例外処理の必須基準
- **処理の結果どんな例外クラスがスローされる可能性があるか**

 <!-- ![alt text](origin-2.png) -->

- Throwable
  - Error <!--checked例外-->
  - Exception
    - RuntimeException <!--unchecked例外-->
      - ArithmeticException
      - NullPointerException
    - IOException
      - FileNotFoundException
      - SocketException
    - ...

## マルチキャッチ
- **catch(xxxException | xxxException e)**

- **注意 :** 
  - 継承関係にある例外は併記できない(ex:FileNotFoundException | IOException)

  - キャッチした参照変数は暗黙的にfinal 

```java
// マルチキャッチでなければ参照変数eはfinalでなくでもいい
try {
    int a = 10/0;
    FileReader rf = new FileReader("a.txt");
    rf.read();
} catch(ArithmeticException | FileNotFoundException e) {
    e = null; //コンパイルエラー
} catch(IOException e) {
    throw xxxException("入出力エラーです")
    //e = null; //コンパイルは通る
}
```

## throwsとthrow

### throws

- 例外が発生した場合、メソッドの呼び出し元に転送
- JVMがスローするイメージ(プログラム内で明示的にスローはしない。するのはthrowの方)

```java
// メソッド名 throws xxxException {}
```

- サンプルソース
```java
public static void main(String args) {
    try {
        methodA();
        methodB();
    } catch(ArrayStoreException | IOException e) {
        System.out.println(e);
    }
}

public void methodA() throws ArrayStoreException {
    //static void methodA() { //この書き方でもok
    throw new ArrayStoreException();
}

public void methodB() throws IOException {
    //static void methodB() { //コンパイルエラー
    throw new IOException();
}


//実行結果　java.lang.ArrayStoreException
```

- methodA()を実行したとき
  1. 明示的に```ArrayStoreException例外オブジェクト```をインスタンス化し、```throw```により例外をスロー
  
  2. ```throws ArrayStoreException```により呼び出し元に転送(ここではmainメソッド)
   
  3. 呼び出し元の```catchブロック```で受け取る


**注意**
- methodB()ではthrows ```IOExceptionが必須```
  - IOExceptionは```checked例外```であり、呼び出し元に処理を転送するには明示的に指定する必要がある 
  - ```コンパイル時のチェック: Checked例外は、メソッド内で処理されない場合、コンパイルエラーを引き起こします。そのため、メソッドを呼び出す側が例外処理を行うか、または同様にthrowsを使って上位の呼び出し元に例外を伝播させる必要があります。by GPT-3.5```

- unchecked例外は例外処理がなくても```勝手に呼び出し元に転送される```ので縛りはない。**実行時例外を扱うのでコンパイル時にはチェックされないから。**



### throw
- プログラム内で明示的にスローする場合に使用

### rethrow
- 受け取った例外に何かしらの処理の追加を行ったり別の例外クラスに変更して再度例外スローすること


**サンプルソース**
```java
import java.io.IOException;

public class RethrowExample {
    public static void main(String[] args) {
        try {
            // 例外が発生する可能性がある処理
            throwIOException();
        } catch (RuntimeException re) {
            // Rethrowした例外をここでキャッチ
            System.out.println("Caught rethrown exception: " + re.getMessage());
        }
    }

    public static void throwIOException() {
        try {
            // IOExceptionが発生する可能性がある処理
            throw new IOException("Sample IOException");
        } catch (IOException ioException) {
            // IOExceptionをRuntimeExceptionにラップして再スロー
            throw new RuntimeException("Rethrown IOException", ioException);
        }
    }
}
```

## オーバーライドの注意点

1. サブクラスのメソッドがスローする例外
   - スーパークラスのメソッドがスローする例外クラスと同じか、その例外クラスのサブクラス

```java
import java.io.IOException;

class Superclass {
    // スーパークラスのメソッドがIOExceptionをスローする例
    public void exampleMethod() throws IOException {
        // メソッドの本体
    }
}

class Subclass extends Superclass {
    // OKの例: スーパークラスの例外と同じ例外クラスをスローする
    @Override
    public void exampleMethod() throws IOException {
        // サブクラスのメソッドの本体
    }

    // NGの例: スーパークラスの例外よりも広範な例外クラスをスローしてしまう
    public void anotherMethod() throws Exception {
        // サブクラスのメソッドの本体
    }
}

public class OverrideExample {
    public static void main(String[] args) {
        // サブクラスのオブジェクトを作成
        Subclass subclass = new Subclass();

        try {
            // OKの例: サブクラスのメソッドを呼び出し
            subclass.exampleMethod();
        } catch (IOException e) {
            // 例外の処理
            System.out.println("Caught IOException: " + e.getMessage());
        }

        try {
            // NGの例: 別のメソッドでNGな例外をキャッチ
            subclass.anotherMethod();
        } catch (IOException e) {
            // 例外の処理
            System.out.println("Caught IOException: " + e.getMessage());
        } catch (Exception e) {
            // もしNGな例外がキャッチされた場合の処理
            System.out.println("Caught Exception: " + e.getMessage());
        }
    }
}
```

2. RuntimeExceptionおよびそのサブクラスの例外はスーパークラスのメソッドに関係なくスローできる
   - 実行時例外だからかな？

3. スーパークラスのメソッドにthrowsがあっても、サブクラス側でthrowsを記述しないことは可能
   - サブクラスでのthrows宣言が不要な理由は、スーパークラスと同一の例外クラスがスローされるため、既にスーパークラスでの宣言がカバーされているから
  
```java
class Superclass {
    // スーパークラスのメソッドがIOExceptionをスローする例
    public void exampleMethod() throws IOException {
        // メソッドの本体
    }
}

class Subclass extends Superclass {
    // スーパークラスのメソッドと同じ例外クラスをスローするため、throws宣言は不要
    @Override
    public void exampleMethod() {
        // サブクラスのメソッドの本体
    }
}
```

## try-with-resources
- <span style="color: pink; ">リソースを自動で開放する</span>

  
```java
try ((リソースの初期化1);
     (リソースの初期化2)) {
    // リソースを使用するコード
} catch (例外の型1 変数1) {
    // 例外1のハンドリング
} catch (例外の型2 変数2) {
    // 例外2のハンドリング
} finally {
    // クローズなどの最終処理（必要に応じて）
}
```

```java
import java.sql.*;

class MyResource implements AutoCloseable {
    private String msg;
    //オーバーロードしたコンストラクタ
    public MyResource(String) { this.msg = msg; }
    //オーバーロードしたclose()
    public void close() throws Exception {
        System.out.println("close() : " + msg);
    }
}

public class Main {
    public static void main(String[] args) {
        try ()
    }
}
```


## Assertion

**構文**
```
assert 戻り値booleanの式;
assert 戻り値booleanの式: メッセージ;
```

- boolean式
  - true
    - assert文以下の操作実行
  - false
    - **AssertionError発生**
      - メッセージあり
        - **デフォルトエラーとメッセージ表示**
      - メッセージなし
        - デフォルトメッセージ表示
  
### AssertionError
- <span style="color: pink; ">Errorクラスのサブクラス</span>
- プログラム自体にバグがある
  - **これが起きないようにプログラムを修正しないといけない**
  
**注意**
- デフォルトだと実行時にアサーション機能が無効になってしまう
- **理由(by Claude3)**

  - 1. **パフォーマンス上の理由**
   アサーションは実行時にチェックが行われるため、プログラムの実行速度に影響を与えます。プロダクション環境では、このようなチェックは不要になることがあるため、アサーションを無効にすることでパフォーマンスを向上させることができます。

  - 2. **デバッグと本番環境の分離**
   アサーションは主にデバッグ時に役立つ機能です。デバッグ後、本番環境ではアサーションによるチェックは不要になる場合があります。オンオフを切り替えることで、デバッグ時と本番環境で異なる動作をさせることができます。

  - 3. **セキュリティ上の理由**
   アサーションの失敗時には、プログラムの内部状態が表示されることがあります。本番環境ではこの情報を外部に漏らすことは望ましくないため、アサーションを無効にすることでセキュリティリスクを軽減できます。

  - 4. **柔軟性の確保**
   アサーションのオンオフ切り替え機能があることで、特定のクラスやメソッドに対してのみアサーションを有効にしたり、無効にしたりと、より細かな制御が可能になります。

- 対策として**-ea**コマンドで明示的に有効にする(逆は **-da**)

```java
java -ea Main

java -da:Foo -ea AssertSample //特定のassertを有効/無効にできる
```




## メモ

### 問3-8 catch(Foo e){ e = new Exception(); }がコンパイルエラーになる理由
- Exceptionクラス自体は抽象クラスではありませんが、直接Exceptionのインスタンスを生成することはできません。

- つまり、以下のようなコードはコンパイルエラーになります。

```java
Exception e = new Exception();  // コンパイルエラー
```
- その理由は、```Exceptionクラスのコンストラクタがprotectedで定義されているため```です。protectedコンストラクタは、同一パッケージまたはサブクラスからのみアクセス可能です。

- したがって、Exceptionのインスタンスを生成するには、RuntimeExceptionやその他のExceptionサブクラスのインスタンスを生成する必要があります。

```java
Exception e = new RuntimeException(); // OK
Exception e = new IOException();      // OK
```
この挙動は、Exceptionクラスが抽象クラスではないものの、直接インスタンス化を防ぐ目的で、このようなprotectedコンストラクタの設計になっているためです。


