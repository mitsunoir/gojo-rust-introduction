# 変数と型 - 呪力の容器

## 変数宣言 - 呪力を蓄える器

呪術師にとって呪力をコントロールするのは基本中の基本だろう？Rustでも同じだ。変数は呪力を蓄える器のようなものだ。

### 基本的な変数宣言

```rust
fn main() {
    let power = 1000;  // 不変な変数（デフォルト）
    println!(\"俺の基本呪力: {}\", power);

    // power = 2000;  // エラー！不変な変数は変更できない
}
```

!!! note "五条先生のメモ"
    Rustの変数はデフォルトで\*\*不変（immutable）\*\*だ。これは俺の無下限術式みたいに、安定した状態を保つためだ。
    変更したい場合は明示的に`mut`キーワードを使う。

### 可変な変数

```rust
fn main() {
    let mut power = 1000;  // 可変な変数
    println!(\"初期呪力: {}\", power);

    power = 2000;  // OK！可変だから変更できる
    println!(\"増強後の呪力: {}\", power);

    power += 1000;  // さらに増加
    println!(\"最終呪力: {}\", power);
}
```

### 変数のシャドーイング

```rust
fn main() {
    let technique = \"基本術式\";
    println!(\"現在の技: {}\", technique);

    let technique = \"上級術式\";  // 同じ名前で新しい変数を作成
    println!(\"進化した技: {}\", technique);

    let technique = technique.len();  // 型も変更可能
    println!(\"技名の文字数: {}\", technique);
}
```

!!! tip "シャドーイングの利点"
    シャドーイングを使えば、同じ名前で違う型の値を扱える。
    俺の術式も「蒼」→「赤」→「茈」と進化するように、変数も段階的に進化させられる。

## 型アノテーション - 術式の詳細指定

```rust
fn main() {
    // 明示的な型指定
    let power_level: i64 = 999_999_999;
    let technique_name: &str = \"無下限呪術\";
    let is_active: bool = true;

    // 型推論（コンパイラが推論）
    let enemy_count = 10;        // i32と推論される
    let damage_rate = 99.9;      // f64と推論される

    println!(\"呪力: {}, 技名: {}\", power_level, technique_name);
}
```

## スコープ - 術式の有効範囲

```rust
fn main() {
    let outer_power = 1000;  // 外側のスコープ

    {
        let inner_power = 500;  // 内側のスコープ
        println!(\"内側の呪力: {}\", inner_power);
        println!(\"外側の呪力: {}\", outer_power);  // 外側の変数も使える
    }  // inner_powerはここで解放される

    // println!(\"{}\", inner_power);  // エラー！スコープ外
    println!(\"外側の呪力: {}\", outer_power);  // まだ使える
}
```

## タプル - 複数の値をまとめる

```rust
fn main() {
    // タプルの作成
    let technique_info = (\"術式順転『蒼』\", 9999, true);

    // 分解して取得
    let (name, power, is_offensive) = technique_info;
    println!(\"技名: {}, 威力: {}, 攻撃技: {}\", name, power, is_offensive);

    // インデックスでアクセス
    println!(\"技名: {}\", technique_info.0);
    println!(\"威力: {}\", technique_info.1);

    // 空のタプル（ユニット型）
    let unit = ();  // 値を返さない関数の戻り値型
}
```

### タプルの実用例

```rust
fn get_technique_stats() -> (String, i32, bool) {
    (String::from(\"術式順転『蒼』\"), 9999, true)
}

fn main() {
    let (technique, power, is_offensive) = get_technique_stats();

    if is_offensive {
        println!(\"{} - 威力: {} (攻撃技)\", technique, power);
    } else {
        println!(\"{} - 威力: {} (防御技)\", technique, power);
    }
}
```

## 配列 - 同じ型の値の集合

```rust
fn main() {
    // 固定長配列
    let techniques = [\"蒼\", \"赤\", \"茈\"];
    let power_levels = [1000, 1500, 3000];

    // 型と長さを明示
    let enemy_hp: [i32; 5] = [100, 200, 300, 400, 500];

    // 同じ値で初期化
    let damage = [999; 10];  // 999が10個の配列

    // 要素へのアクセス
    println!(\"最初の技: {}\", techniques[0]);
    println!(\"最後の技: {}\", techniques[2]);

    // 配列の長さ
    println!(\"技の数: {}\", techniques.len());
}
```

### 配列の反復処理

```rust
fn main() {
    let techniques = [\"蒼\", \"赤\", \"茈\", \"紫\"];

    // インデックス付きループ
    for i in 0..techniques.len() {
        println!(\"技{}: {}\", i + 1, techniques[i]);
    }

    // 直接要素をループ
    for technique in techniques.iter() {
        println!(\"術式: {}\", technique);
    }

    // 列挙付きループ
    for (index, technique) in techniques.iter().enumerate() {
        println!(\"{}番目の技: {}\", index + 1, technique);
    }
}
```

## ベクター - 動的配列

```rust
fn main() {
    // 空のベクター作成
    let mut enemy_list: Vec<String> = Vec::new();

    // マクロで作成
    let mut power_levels = vec![1000, 1500, 2000];

    // 要素の追加
    enemy_list.push(String::from(\"特級呪霊\"));
    enemy_list.push(String::from(\"特級呪霊(偽夏油)\"));

    power_levels.push(9999);

    // 要素へのアクセス
    println!(\"最初の敵: {}\", enemy_list[0]);
    println!(\"最大呪力: {}\", power_levels[3]);

    // 安全なアクセス
    match enemy_list.get(1) {
        Some(enemy) => println!(\"2番目の敵: {}\", enemy),
        None => println!(\"敵がいない\"),
    }
}
```

### ベクターの操作

```rust
fn main() {
    let mut techniques = vec![\"蒼\", \"赤\"];

    // 要素の追加
    techniques.push(\"茈\");
    techniques.push(\"紫\");

    // 要素の削除
    let last_technique = techniques.pop();  // Option<T>を返す
    match last_technique {
        Some(tech) => println!(\"削除した技: {}\", tech),
        None => println!(\"技がない\"),
    }

    // 長さとキャパシティ
    println!(\"技の数: {}\", techniques.len());
    println!(\"容量: {}\", techniques.capacity());

    // 全要素の表示
    for technique in &techniques {
        println!(\"習得済み: {}\", technique);
    }
}
```

## 文字列の詳細 - 呪文の扱い

### String vs &str

```rust
fn main() {
    // 文字列リテラル（&str）
    let greeting = \"やあやあ\";  // 静的な文字列

    // String型（所有権あり）
    let mut technique_name = String::from(\"術式順転\");
    technique_name.push_str(\"『蒼』\");

    // 変換
    let static_str: &str = &technique_name;  // StringからStrへ
    let owned_string: String = greeting.to_string();  // &strからStringへ

    println!(\"挨拶: {}\", greeting);
    println!(\"技名: {}\", technique_name);
}
```

### 文字列の操作

```rust
fn main() {
    let mut spell = String::from(\"術式\");

    // 文字列の結合
    spell.push_str(\"順転\");
    spell.push('『');
    spell.push_str(\"蒼』\");

    // format!マクロ
    let power = 9999;
    let description = format!(\"{}の威力は{}\", spell, power);

    // 文字列の分割
    let full_name = \"五条悟\";
    let parts: Vec<&str> = full_name.split(\"条\").collect();

    println!(\"呪文: {}\", spell);
    println!(\"説明: {}\", description);
    println!(\"名前の分割: {:?}\", parts);
}
```

## 型変換とキャスト - 呪力の変換

```rust
fn main() {
    // 数値型間の変換
    let power_i32: i32 = 1000;
    let power_i64: i64 = power_i32 as i64;  // 安全な拡大変換
    let power_u32: u32 = power_i32 as u32;  // 注意が必要

    // 文字列から数値
    let power_str = \"9999\";
    let power_num: i32 = power_str.parse()
        .expect(\"数値変換失敗\");

    // 数値から文字列
    let level = 99;
    let level_str = level.to_string();

    // より安全な変換
    let safe_conversion: Result<i32, _> = \"1234\".parse();
    match safe_conversion {
        Ok(num) => println!(\"変換成功: {}\", num),
        Err(e) => println!(\"変換失敗: {}\", e),
    }

    println!(\"変換された呪力: {}\", power_num);
}
```

## 練習問題

<div class=\"exercise\">

### 問題1: 呪力計算機

3つの術式の威力を配列に保存し、合計威力を計算するプログラムを書け。

```rust
fn main() {
    // ここにコードを書く
}
```

</div>

<details>
<summary>解答を見る</summary>
<div class=\"solution\">

```rust
fn main() {
    let techniques = [1000, 1500, 3000];  // 蒼、赤、茈の威力
    let mut total_power = 0;

    for power in techniques.iter() {
        total_power += power;
    }

    println!(\"合計威力: {}\", total_power);

    // またはイテレータのsum()を使用
    let total: i32 = techniques.iter().sum();
    println!(\"合計威力(sum使用): {}\", total);
}
```

</div>
</details>

<div class=\"exercise\">

### 問題2: 敵リスト管理

ベクターを使って敵リストを作成し、敵を追加・削除する機能を実装せよ。

</div>

<details>
<summary>解答を見る</summary>
<div class=\"solution\">

```rust
fn main() {
    let mut enemies = Vec::new();

    // 敵を追加
    enemies.push(\"特級呪霊\");
    enemies.push(\"1級呪霊\");
    enemies.push(\"呪詛師\");

    println!(\"敵リスト:\");
    for (i, enemy) in enemies.iter().enumerate() {
        println!(\"{}. {}\", i + 1, enemy);
    }

    // 最後の敵を倒す
    if let Some(defeated) = enemies.pop() {
        println!(\"\n{}を倒した！\", defeated);
    }

    println!(\"\\n残りの敵: {} 体\", enemies.len());
}
```

</div>
</details>

## まとめ

変数と型の基本はマスターできたか？重要なポイントを復習しよう：

1. **不変性がデフォルト** - 安全性のため
1. **mut で可変性を明示** - 必要な時だけ
1. **シャドーイング** - 同名で新しい変数作成
1. **型推論** - コンパイラが賢く推論
1. **タプル** - 複数の値をまとめる
1. **配列 vs ベクター** - 固定長 vs 動的

次は制御構文について学ぼう。if文、ループ、match式...これらは術式の組み合わせを作るのに必須だ。

______________________________________________________________________

*「変数は呪力の器。大切に扱え」*
