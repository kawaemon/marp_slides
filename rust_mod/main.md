---
marp: true
theme: default
class: invert
---

# Rust のモジュールシステムを理解したい

### かわえもん

---

# 自己紹介

`format!("{:#?}", Option::<SelfIntro>::None);`

---

# 背景

みなさん Rust のモジュールシステムの理解に苦しんでそう

解説すっか！

---

# ルール

-   基本 1 `.rs` ファイルにつき自動的に 1 モジュール生成されます。
-   モジュールが存在することは`mod hogefuga;` のように宣言します。
-   但し`src/main.rs`と`src/lib.rs`は特別扱いされます。
    crate の root モジュールとして扱われます。

---

# 例

```rs
// src/main.rs
mod pencil;

fn main() {
    pencil::write();
}
```

```rs
// src/pencil.rs
pub fn write() {
    println!("Hello!");
}
```

###### todo: replace with image

---

# 躓きポイント

モジュールのための `.rs` の配置の仕方が複数存在する

---

# パターン 1

子の子モジュールが存在しないときによく使います。
(`pencil` 以下にモジュールを作らない場合)

```rs
// src/main.rs
mod pencil;

fn main() {
    pencil::write();
}

```

```rs
// src/pencil.rs
pub fn write() {
    println!("Hello!");
}
```

---

# パターン 2

子の子モジュールが存在するときによく使います。
Rust 2015 Edition 時点で仕様に存在します。

```rs
// src/main.rs
mod pencil;

fn main() {
    pencil::write();
}
```

```rs
// src/pencil/mod.rs
pub fn write() {
    println!("Hello!"):
}
```

---

# パターン 2 補足

子の子モジュールの例

```rust
// src/main.rs
mod pencil;
fn main() { pencil::write(); }
```

```rust
// src/pencil/mod.rs
mod lead;
pub fn write() { println!("{}", lead::MESSAGE) }
```

```rust
// src/pencil/lead.rs
pub const MESSAGE: &str = "Hello!";
```

###### todo: ファイル構造を書く

---

# パターン 3

子の子モジュールが存在するパターン
Rust 2018 Edition から使えます。

```rust
// src/main.rs
mod pencil;
fn main() { pencil::write(); }
```

```rust
// src/pencil.rs
mod lead;
pub fn write() { println!("{}", lead::MESSAGE) }
```

```rust
// src/pencil/lead.rs
pub const MESSAGE: &str = "Hello!";
```

---

# パターン 2 と 3 で何が違うのか

`src/pencil/mod.rs` を `src/pencil.rs` に移動することができます。

`pencil`モジュールの子モジュールは`src/pencil/*.rs`に置きます。

別にパターン 2 が古いからと言って非推奨と言うわけではありません。

筆者が今まで見てきたプロジェクトでは、パターン 2 の方が圧倒的に多い印象です。

<style>
    .treeimage {
        display: flex;
        flex-direction: row;
        justify-content: space-evenly;
    }

    .treeimage > figure {
        margin: 0;
        margin-top: 10px;
        width: 300px;
    }

    .treeimage > figure > figcaption {
        text-align: center;
    }
</style>

<div class="treeimage">
    <figure>
        <img src="https://imgur.com/j5zrE0E.png" />
        <figcaption>パターン2</figcaption>
    </figure>
    <figure>
        <img src="https://imgur.com/QOOCEX2.png" />
        <figcaption>パターン3</figcaption>
    </figure>
</div>

---

# なんで`mod pencil;`が必要なのか

使う機能だけコンパイルして時間削減

```rs
#[cfg(feature = "pencil")]
mod pencil;

#[cfg(feature = "pen")]
mod pen;
```

OS 専用の機能のコンパイル

```rs
#[cfg(target_os = "windows")]
mod windows_pen;
#[cfg(target_os = "windows")]
pub use windows_pen as pen;

#[cfg(target_os = "linux")]
mod linux_pen;
#[cfg(target_os = "linux")]
pub use linux_pen as pen;
```

---

# たまに使うテクニック

テストの時によく使います。

新しいモジュールを新しいファイルを作らずその場で定義します。

```rust
// src/pen.rs

pub fn hoge(i: i32) -> i32 {
    i + 1
}

// テストするときだけこのモジュールが存在する
#[cfg(test)]
mod test {
    use super::*;
    #[test]
    fn test_hoge() {
        assert_eq!(hoge(3), 4);
    }
}
```

---

# おわり

Rust みんな書いてくれ！！！！！
