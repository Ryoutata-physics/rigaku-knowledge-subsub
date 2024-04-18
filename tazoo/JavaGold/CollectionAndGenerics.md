## ジェネリクス

### ジェネリクスとは？

- コレクション内部の型をあらかじめ指定できる機能
  - 従来だと意図しない型の変数が混入したり、キャストが意図しないで実行されてしまう
  - そのためあらかじめ型を指定することでより安全なコードを提供する

```java
//Stringで型指定されたlistを定義し、インスタンスを代入
//ダイヤモンド演算子<>内の型が一致しているかチェックしている
ArrayList<String> list = new ArrayList<String>();
//JavaSE7からは右辺の表記は簡略化可能
ArrayList<String> list = new ArrayList<>();
//コンパイルエラー
ArrayList<Object> list = new ArrayList<String>();
//コンパイルエラー
ArrayList<String> list = new ArrayList<Object>();
```

## ジェネリクスを用いた独自クラスの定義

- 型パラメータ：仮の型を示す汎用的な型
  - T：Type
  - K：Key
  - V：Value
  - E：Element

- 型パラメータで扱えるデータ型は```参照型```のみ
- staticメンバには使用できない