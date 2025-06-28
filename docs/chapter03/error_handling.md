# 高度なエラーハンドリング - 反転術式の奥義

## エラーハンドリングの進化 - 複雑な術式の制御

基本的なOption型とResult型をマスターしたが、実際のプロジェクトではより複雑なエラーハンドリングが必要になる。複数のエラー型を統合し、カスタムエラーを作成し、エラーの連鎖を美しく処理する。これが反転術式の奥義だ。

!!! note "五条先生の解説"
    大規模なシステムでは、様々な種類のエラーが発生する。ファイルI/O、ネットワーク、パース、ビジネスロジック...
    これらを統一的に扱うための高度なテクニックを習得しよう。

## カスタムエラー型の設計

### 基本的なカスタムエラー

```rust
use std::fmt;

#[derive(Debug)]
enum SorceryError {
    InsufficientPower { required: i32, current: i32 },
    TechniqueNotFound(String),
    InvalidTarget(String),
    SystemError(String),
}

impl fmt::Display for SorceryError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        match self {
            SorceryError::InsufficientPower { required, current } => {
                write!(f, \"呪力不足: {}必要, {}しかありません\", required, current)
            },
            SorceryError::TechniqueNotFound(name) => {
                write!(f, \"術式'{}'が見つかりません\", name)
            },
            SorceryError::InvalidTarget(reason) => {
                write!(f, \"無効な対象: {}\", reason)
            },
            SorceryError::SystemError(msg) => {
                write!(f, \"システムエラー: {}\", msg)
            },
        }
    }
}

impl std::error::Error for SorceryError {}

// カスタムエラーを使った関数
fn cast_advanced_technique(
    power: i32,
    technique: &str,
    target: &str
) -> Result<String, SorceryError> {

    // 呪力チェック
    let required_power = match technique {
        \"青\" => 500,
        \"赤\" => 800,
        \"茈\" => 1500,
        \"紫\" => 3000,
        _ => return Err(SorceryError::TechniqueNotFound(technique.to_string())),
    };

    if power < required_power {
        return Err(SorceryError::InsufficientPower {
            required: required_power,
            current: power
        });
    }

    // 対象チェック
    if target.is_empty() {
        return Err(SorceryError::InvalidTarget(\"対象が指定されていません\".to_string()));
    }

    Ok(format!(\"{} を {} に向けて発動！\", technique, target))
}

fn main() {
    let test_cases = [
        (2000, \"青\", \"呪霊A\"),
        (300, \"赤\", \"呪霊B\"),
        (1000, \"未知の術式\", \"呪霊C\"),
        (2000, \"茈\", \"\"),
    ];

    for (power, technique, target) in test_cases.iter() {
        match cast_advanced_technique(*power, technique, target) {
            Ok(result) => println!(\"✓ {}\", result),
            Err(error) => println!(\"✗ {}\", error),
        }
    }
}
```

### エラーの階層化

```rust
#[derive(Debug)]
enum DatabaseError {
    ConnectionFailed,
    QueryFailed(String),
    DataCorrupted,
}

#[derive(Debug)]
enum NetworkError {
    Timeout,
    ConnectionRefused,
    InvalidResponse(String),
}

#[derive(Debug)]
enum ApplicationError {
    Database(DatabaseError),
    Network(NetworkError),
    Sorcery(SorceryError),
    ValidationError(String),
}

impl fmt::Display for DatabaseError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        match self {
            DatabaseError::ConnectionFailed => write!(f, \"データベース接続失敗\"),
            DatabaseError::QueryFailed(query) => write!(f, \"クエリ失敗: {}\", query),
            DatabaseError::DataCorrupted => write!(f, \"データが破損しています\"),
        }
    }
}

impl fmt::Display for NetworkError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        match self {
            NetworkError::Timeout => write!(f, \"ネットワークタイムアウト\"),
            NetworkError::ConnectionRefused => write!(f, \"接続が拒否されました\"),
            NetworkError::InvalidResponse(msg) => write!(f, \"無効なレスポンス: {}\", msg),
        }
    }
}

impl fmt::Display for ApplicationError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        match self {
            ApplicationError::Database(err) => write!(f, \"DB: {}\", err),
            ApplicationError::Network(err) => write!(f, \"Network: {}\", err),
            ApplicationError::Sorcery(err) => write!(f, \"Sorcery: {}\", err),
            ApplicationError::ValidationError(msg) => write!(f, \"Validation: {}\", msg),
        }
    }
}

impl std::error::Error for DatabaseError {}
impl std::error::Error for NetworkError {}
impl std::error::Error for ApplicationError {}

// エラー型の変換
impl From<DatabaseError> for ApplicationError {
    fn from(err: DatabaseError) -> Self {
        ApplicationError::Database(err)
    }
}

impl From<NetworkError> for ApplicationError {
    fn from(err: NetworkError) -> Self {
        ApplicationError::Network(err)
    }
}

impl From<SorceryError> for ApplicationError {
    fn from(err: SorceryError) -> Self {
        ApplicationError::Sorcery(err)
    }
}
```

## 外部クレートのエラーとの統合

```rust
use std::fs;
use std::io;
use std::num::ParseIntError;

#[derive(Debug)]
enum ConfigError {
    IoError(io::Error),
    ParseError(ParseIntError),
    ValidationError(String),
    MissingField(String),
}

impl fmt::Display for ConfigError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        match self {
            ConfigError::IoError(err) => write!(f, \"I/O エラー: {}\", err),
            ConfigError::ParseError(err) => write!(f, \"パースエラー: {}\", err),
            ConfigError::ValidationError(msg) => write!(f, \"バリデーションエラー: {}\", msg),
            ConfigError::MissingField(field) => write!(f, \"必須フィールド '{}'がありません\", field),
        }
    }
}

impl std::error::Error for ConfigError {}

// 外部クレートのエラーからの変換
impl From<io::Error> for ConfigError {
    fn from(err: io::Error) -> Self {
        ConfigError::IoError(err)
    }
}

impl From<ParseIntError> for ConfigError {
    fn from(err: ParseIntError) -> Self {
        ConfigError::ParseError(err)
    }
}

#[derive(Debug)]
struct SorcererConfig {
    name: String,
    power: i32,
    grade: String,
}

impl SorcererConfig {
    fn load_from_file(filename: &str) -> Result<Self, ConfigError> {
        let content = fs::read_to_string(filename)?;  // ?でio::Errorを自動変換

        let lines: Vec<&str> = content.lines().collect();

        if lines.len() < 3 {
            return Err(ConfigError::ValidationError(
                \"設定ファイルの行数が不足しています\".to_string()
            ));
        }

        let name = lines[0].trim();
        if name.is_empty() {
            return Err(ConfigError::MissingField(\"name\".to_string()));
        }

        let power: i32 = lines[1].trim().parse()?;  // ?でParseIntErrorを自動変換

        if power < 0 {
            return Err(ConfigError::ValidationError(
                \"呪力は0以上である必要があります\".to_string()
            ));
        }

        let grade = lines[2].trim();
        if grade.is_empty() {
            return Err(ConfigError::MissingField(\"grade\".to_string()));
        }

        Ok(SorcererConfig {
            name: name.to_string(),
            power,
            grade: grade.to_string(),
        })
    }

    fn validate(&self) -> Result<(), ConfigError> {
        if self.name.len() < 2 {
            return Err(ConfigError::ValidationError(
                \"名前は2文字以上である必要があります\".to_string()
            ));
        }

        let valid_grades = [\"特級\", \"1級\", \"2級\", \"3級\", \"4級\"];
        if !valid_grades.contains(&self.grade.as_str()) {
            return Err(ConfigError::ValidationError(
                format!(\"無効な等級: {}\", self.grade)
            ));
        }

        Ok(())
    }
}

fn load_and_validate_config(filename: &str) -> Result<SorcererConfig, ConfigError> {
    let config = SorcererConfig::load_from_file(filename)?;
    config.validate()?;
    Ok(config)
}

fn main() {
    // 設定ファイルの作成（テスト用）
    let test_config = \"五条悟\\n3000\\n特級\";
    if let Err(e) = fs::write(\"test_config.txt\", test_config) {
        println!(\"テストファイル作成エラー: {}\", e);
        return;
    }

    // 正常ケース
    match load_and_validate_config(\"test_config.txt\") {
        Ok(config) => println!(\"✓ 設定読み込み成功: {:?}\", config),
        Err(error) => println!(\"✗ 設定読み込み失敗: {}\", error),
    }

    // エラーケース（存在しないファイル）
    match load_and_validate_config(\"nonexistent.txt\") {
        Ok(config) => println!(\"✓ 設定読み込み成功: {:?}\", config),
        Err(error) => println!(\"✗ 設定読み込み失敗: {}\", error),
    }

    // 無効な設定ファイル
    let invalid_config = \"\\ninvalid_power\\n無効等級\";
    if let Err(e) = fs::write(\"invalid_config.txt\", invalid_config) {
        println!(\"テストファイル作成エラー: {}\", e);
        return;
    }

    match load_and_validate_config(\"invalid_config.txt\") {
        Ok(config) => println!(\"✓ 設定読み込み成功: {:?}\", config),
        Err(error) => println!(\"✗ 設定読み込み失敗: {}\", error),
    }
}
```

## anyhowとthiserrorクレート

実際のプロジェクトでは、`anyhow`と`thiserror`クレートがよく使われる：

```rust
// Cargo.tomlに追加:
// [dependencies]
// anyhow = \"1.0\"
// thiserror = \"1.0\"

use thiserror::Error;
use anyhow::{Context, Result as AnyhowResult};

#[derive(Error, Debug)]
enum AdvancedSorceryError {
    #[error(\"呪力不足: {required}必要, {current}しかありません\")]
    InsufficientPower { required: i32, current: i32 },

    #[error(\"術式'{name}'が見つかりません\")]
    TechniqueNotFound { name: String },

    #[error(\"IO エラー\")]
    Io(#[from] std::io::Error),

    #[error(\"パースエラー\")]
    Parse(#[from] std::num::ParseIntError),

    #[error(\"設定エラー: {message}\")]
    Config { message: String },
}

// anyhowを使った簡単なエラーハンドリング
fn complex_operation() -> AnyhowResult<String> {
    let content = std::fs::read_to_string(\"config.txt\")
        .context(\"設定ファイルの読み込みに失敗しました\")?;

    let power: i32 = content.trim().parse()
        .context(\"呪力の値をパースできませんでした\")?;

    if power < 1000 {
        anyhow::bail!(\"呪力が低すぎます: {}\", power);
    }

    Ok(format!(\"呪力{}で初期化完了\", power))
}

fn main() {
    match complex_operation() {
        Ok(result) => println!(\"✓ {}\", result),
        Err(error) => {
            println!(\"✗ エラー: {}\", error);

            // エラーチェーンの表示
            for cause in error.chain() {
                println!(\"  原因: {}\", cause);
            }
        }
    }
}
```

## 実践例 - 呪術師管理システム

```rust
use std::collections::HashMap;
use std::fs;
use std::io;

#[derive(Debug)]
enum SystemError {
    FileError(io::Error),
    ParseError(String),
    ValidationError(String),
    NotFound(String),
    PermissionDenied(String),
    DatabaseCorrupted,
}

impl fmt::Display for SystemError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        match self {
            SystemError::FileError(err) => write!(f, \"ファイルエラー: {}\", err),
            SystemError::ParseError(msg) => write!(f, \"パースエラー: {}\", msg),
            SystemError::ValidationError(msg) => write!(f, \"バリデーションエラー: {}\", msg),
            SystemError::NotFound(item) => write!(f, \"{}が見つかりません\", item),
            SystemError::PermissionDenied(action) => write!(f, \"{}の権限がありません\", action),
            SystemError::DatabaseCorrupted => write!(f, \"データベースが破損しています\"),
        }
    }
}

impl std::error::Error for SystemError {}

impl From<io::Error> for SystemError {
    fn from(err: io::Error) -> Self {
        SystemError::FileError(err)
    }
}

#[derive(Debug, Clone)]
struct Sorcerer {
    id: u32,
    name: String,
    power: i32,
    grade: String,
    techniques: Vec<String>,
    active: bool,
}

impl Sorcerer {
    fn new(id: u32, name: String, power: i32, grade: String) -> Result<Self, SystemError> {
        if name.is_empty() {
            return Err(SystemError::ValidationError(\"名前が空です\".to_string()));
        }

        if power < 0 {
            return Err(SystemError::ValidationError(\"呪力は0以上である必要があります\".to_string()));
        }

        let valid_grades = [\"特級\", \"1級\", \"2級\", \"3級\", \"4級\"];
        if !valid_grades.contains(&grade.as_str()) {
            return Err(SystemError::ValidationError(format!(\"無効な等級: {}\", grade)));
        }

        Ok(Sorcerer {
            id,
            name,
            power,
            grade,
            techniques: Vec::new(),
            active: true,
        })
    }

    fn add_technique(&mut self, technique: String) -> Result<(), SystemError> {
        if technique.is_empty() {
            return Err(SystemError::ValidationError(\"術式名が空です\".to_string()));
        }

        if self.techniques.contains(&technique) {
            return Err(SystemError::ValidationError(
                format!(\"術式'{}'は既に習得済みです\", technique)
            ));
        }

        self.techniques.push(technique);
        Ok(())
    }

    fn to_string(&self) -> String {
        format!(\"{}|{}|{}|{}|{}|{}\",
                self.id, self.name, self.power, self.grade,
                self.techniques.join(\",\"), self.active)
    }

    fn from_string(line: &str) -> Result<Self, SystemError> {
        let parts: Vec<&str> = line.split('|').collect();

        if parts.len() != 6 {
            return Err(SystemError::ParseError(
                format!(\"無効なフォーマット: {}\", line)
            ));
        }

        let id: u32 = parts[0].parse()
            .map_err(|_| SystemError::ParseError(\"IDのパースに失敗\".to_string()))?;

        let power: i32 = parts[2].parse()
            .map_err(|_| SystemError::ParseError(\"呪力のパースに失敗\".to_string()))?;

        let active: bool = parts[5].parse()
            .map_err(|_| SystemError::ParseError(\"アクティブ状態のパースに失敗\".to_string()))?;

        let techniques = if parts[4].is_empty() {
            Vec::new()
        } else {
            parts[4].split(',').map(|s| s.to_string()).collect()
        };

        Ok(Sorcerer {
            id,
            name: parts[1].to_string(),
            power,
            grade: parts[3].to_string(),
            techniques,
            active,
        })
    }
}

struct SorcererManager {
    sorcerers: HashMap<u32, Sorcerer>,
    next_id: u32,
    filename: String,
}

impl SorcererManager {
    fn new(filename: &str) -> Self {
        SorcererManager {
            sorcerers: HashMap::new(),
            next_id: 1,
            filename: filename.to_string(),
        }
    }

    fn load_from_file(&mut self) -> Result<(), SystemError> {
        match fs::read_to_string(&self.filename) {
            Ok(content) => {
                for line in content.lines() {
                    if line.trim().is_empty() {
                        continue;
                    }

                    let sorcerer = Sorcerer::from_string(line)?;
                    self.next_id = self.next_id.max(sorcerer.id + 1);
                    self.sorcerers.insert(sorcerer.id, sorcerer);
                }
                Ok(())
            },
            Err(err) if err.kind() == io::ErrorKind::NotFound => {
                // ファイルが存在しない場合は空の状態から開始
                Ok(())
            },
            Err(err) => Err(SystemError::from(err)),
        }
    }

    fn save_to_file(&self) -> Result<(), SystemError> {
        let mut content = String::new();

        for sorcerer in self.sorcerers.values() {
            content.push_str(&sorcerer.to_string());
            content.push('\\n');
        }

        fs::write(&self.filename, content)?;
        Ok(())
    }

    fn add_sorcerer(&mut self, name: String, power: i32, grade: String)
        -> Result<u32, SystemError> {
        let id = self.next_id;
        let sorcerer = Sorcerer::new(id, name, power, grade)?;

        self.sorcerers.insert(id, sorcerer);
        self.next_id += 1;

        self.save_to_file()?;
        Ok(id)
    }

    fn get_sorcerer(&self, id: u32) -> Result<&Sorcerer, SystemError> {
        self.sorcerers.get(&id)
            .ok_or_else(|| SystemError::NotFound(format!(\"呪術師ID: {}\", id)))
    }

    fn get_sorcerer_mut(&mut self, id: u32) -> Result<&mut Sorcerer, SystemError> {
        self.sorcerers.get_mut(&id)
            .ok_or_else(|| SystemError::NotFound(format!(\"呪術師ID: {}\", id)))
    }

    fn add_technique_to_sorcerer(&mut self, id: u32, technique: String)
        -> Result<(), SystemError> {
        let sorcerer = self.get_sorcerer_mut(id)?;
        sorcerer.add_technique(technique)?;
        self.save_to_file()?;
        Ok(())
    }

    fn deactivate_sorcerer(&mut self, id: u32) -> Result<(), SystemError> {
        let sorcerer = self.get_sorcerer_mut(id)?;

        if !sorcerer.active {
            return Err(SystemError::ValidationError(
                \"呪術師は既に非アクティブです\".to_string()
            ));
        }

        sorcerer.active = false;
        self.save_to_file()?;
        Ok(())
    }

    fn search_by_name(&self, name: &str) -> Vec<&Sorcerer> {
        self.sorcerers.values()
            .filter(|s| s.name.contains(name) && s.active)
            .collect()
    }

    fn get_by_grade(&self, grade: &str) -> Vec<&Sorcerer> {
        self.sorcerers.values()
            .filter(|s| s.grade == grade && s.active)
            .collect()
    }

    fn backup_database(&self, backup_filename: &str) -> Result<(), SystemError> {
        let content = fs::read_to_string(&self.filename)?;
        fs::write(backup_filename, content)?;
        Ok(())
    }

    fn restore_from_backup(&mut self, backup_filename: &str) -> Result<(), SystemError> {
        let content = fs::read_to_string(backup_filename)?;
        fs::write(&self.filename, content)?;

        // メモリ上のデータをリロード
        self.sorcerers.clear();
        self.next_id = 1;
        self.load_from_file()?;

        Ok(())
    }

    fn get_statistics(&self) -> HashMap<String, i32> {
        let mut stats = HashMap::new();

        let active_count = self.sorcerers.values()
            .filter(|s| s.active)
            .count() as i32;

        stats.insert(\"total_sorcerers\".to_string(), self.sorcerers.len() as i32);
        stats.insert(\"active_sorcerers\".to_string(), active_count);

        for grade in [\"特級\", \"1級\", \"2級\", \"3級\", \"4級\"].iter() {
            let count = self.get_by_grade(grade).len() as i32;
            stats.insert(format!(\"{}_count\", grade), count);
        }

        if active_count > 0 {
            let total_power: i32 = self.sorcerers.values()
                .filter(|s| s.active)
                .map(|s| s.power)
                .sum();
            stats.insert(\"average_power\".to_string(), total_power / active_count);
        }

        stats
    }
}

fn run_sorcerer_management_system() -> Result<(), SystemError> {
    let mut manager = SorcererManager::new(\"sorcerers.db\");

    // データベースの読み込み
    manager.load_from_file()
        .or_else(|err| {
            println!(\"警告: データベース読み込み失敗: {}\", err);
            println!(\"新しいデータベースを作成します\");
            Ok(())
        })?;

    println!(\"=== 呪術師管理システム ===\");

    // テストデータの追加
    let gojo_id = manager.add_sorcerer(
        \"五条悟\".to_string(),
        3000,
        \"特級\".to_string()
    )?;

    let yuji_id = manager.add_sorcerer(
        \"虎杖悠仁\".to_string(),
        1200,
        \"1級\".to_string()
    )?;

    let megumi_id = manager.add_sorcerer(
        \"伏黒恵\".to_string(),
        1000,
        \"2級\".to_string()
    )?;

    // 術式の追加
    manager.add_technique_to_sorcerer(gojo_id, \"無下限呪術\".to_string())?;
    manager.add_technique_to_sorcerer(gojo_id, \"術式順転『青』\".to_string())?;
    manager.add_technique_to_sorcerer(yuji_id, \"黒閃\".to_string())?;
    manager.add_technique_to_sorcerer(megumi_id, \"十種影法術\".to_string())?;

    // データの検索と表示
    println!(\"\\n=== 呪術師一覧 ===\");
    for id in [gojo_id, yuji_id, megumi_id].iter() {
        let sorcerer = manager.get_sorcerer(*id)?;
        println!(\"ID {}: {} - 呪力:{} 等級:{} 術式数:{}\",
                 sorcerer.id, sorcerer.name, sorcerer.power,
                 sorcerer.grade, sorcerer.techniques.len());
    }

    // 統計情報
    println!(\"\\n=== 統計情報 ===\");
    let stats = manager.get_statistics();
    for (key, value) in stats {
        println!(\"{}: {}\", key, value);
    }

    // バックアップの作成
    manager.backup_database(\"sorcerers_backup.db\")?;
    println!(\"\\nバックアップを作成しました\");

    Ok(())
}

fn main() {
    match run_sorcerer_management_system() {
        Ok(_) => println!(\"\\n✓ システム終了\"),
        Err(error) => {
            println!(\"\\n✗ システムエラー: {}\", error);
            std::process::exit(1);
        }
    }
}
```

## まとめ

高度なエラーハンドリングの習得は完了だ！重要なポイント：

1. **カスタムエラー型** - ドメイン固有のエラー表現
1. **エラーの階層化** - 複数のエラー型の統合
1. **外部エラーとの統合** - From traitによる自動変換
1. **エラーチェーン** - 根本原因の追跡
1. **実用的なパターン** - anyhow/thiserrorの活用

これで反転術式の奥義をマスターした。エラーという負の要素を、型安全で表現力豊かな力に変換できるようになったな。

次は第4章で領域展開編、ジェネリクスとトレイトの世界に進もう。より抽象的で強力な概念を習得する時だ。

______________________________________________________________________

*「エラーハンドリングを極めれば、どんな失敗も成長の糧となる」*
