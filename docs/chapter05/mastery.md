# 最強への道 - 全ての技術の統合

## 最強の境地 - 技術の完全融合

ついに到達したな、最強の境地へ。これまでの全ての技術を習得し、それらを完璧に統合した君なら、真の最強プログラマーとして立てるはずだ。

俺が最強である理由は、個々の術式が強いからではない。**全ての技術を完璧に理解し、それらを自在に組み合わせられるから**だ。君もそれを目指せ。

!!! note "五条先生の最終教示"
    最強とは、全ての技術を適材適所で使い分け、完璧に統合できる状態。
    基本構文、所有権、エラーハンドリング、ジェネリクス、トレイト、非同期、マクロ - これら全てが一つの作品の中で調和する。

## 統合プロジェクト - 呪術師管理システム

### アーキテクチャ設計

```rust
//! 呪術師管理システム - 完全統合版
//!
//! このシステムは以下の技術を統合します：
//! - 所有権システムによる安全なメモリ管理
//! - ジェネリクスとトレイトによる抽象化
//! - エラーハンドリングによる堅牢性
//! - 非同期処理による効率性
//! - マクロによるコード生成

use std::collections::HashMap;
use std::sync::Arc;
use std::time::{Duration, Instant};
use tokio::sync::{Mutex, mpsc, RwLock};
use serde::{Serialize, Deserialize};

// エラー型の定義（第3章の技術）
#[derive(Debug, Clone)]
pub enum SystemError {
    SorcererNotFound(String),
    InvalidPower(i32),
    DatabaseError(String),
    NetworkError(String),
    ValidationError(String),
    ConcurrencyError(String),
}

impl std::fmt::Display for SystemError {
    fn fmt(&self, f: &mut std::fmt::Formatter) -> std::fmt::Result {
        match self {
            SystemError::SorcererNotFound(name) => write!(f, "呪術師 '{}' が見つかりません", name),
            SystemError::InvalidPower(power) => write!(f, "無効な呪力値: {}", power),
            SystemError::DatabaseError(msg) => write!(f, "データベースエラー: {}", msg),
            SystemError::NetworkError(msg) => write!(f, "ネットワークエラー: {}", msg),
            SystemError::ValidationError(msg) => write!(f, "バリデーションエラー: {}", msg),
            SystemError::ConcurrencyError(msg) => write!(f, "並行処理エラー: {}", msg),
        }
    }
}

impl std::error::Error for SystemError {}

// 結果型のエイリアス
pub type Result<T> = std::result::Result<T, SystemError>;

// トレイト定義（第4章の技術）
pub trait Entity {
    type Id: Clone + PartialEq + std::fmt::Debug;

    fn id(&self) -> &Self::Id;
    fn validate(&self) -> Result<()>;
}

pub trait Combatant: Entity {
    fn power_level(&self) -> i32;
    fn grade(&self) -> &str;
    fn is_active(&self) -> bool;

    fn battle_effectiveness(&self) -> f64 {
        let base = self.power_level() as f64;
        let multiplier = match self.grade() {
            "特級" => 1.5,
            "1級" => 1.2,
            "2級" => 1.0,
            "3級" => 0.8,
            "4級" => 0.6,
            _ => 0.5,
        };
        base * multiplier
    }
}

pub trait Persistent<T> {
    async fn save(&self, entity: &T) -> Result<()>;
    async fn load(&self, id: &T::Id) -> Result<Option<T>> where T: Entity;
    async fn delete(&self, id: &T::Id) -> Result<bool>;
    async fn list_all(&self) -> Result<Vec<T>>;
}

// マクロによるコード生成（第5章の技術）
macro_rules! define_sorcerer_types {
    (
        $(
            $name:ident {
                $(
                    $field:ident: $type:ty
                ),*
            }
        )*
    ) => {
        $(
            #[derive(Debug, Clone, Serialize, Deserialize, PartialEq)]
            pub struct $name {
                $(
                    pub $field: $type,
                )*
            }

            impl Entity for $name {
                type Id = String;

                fn id(&self) -> &Self::Id {
                    &self.name
                }

                fn validate(&self) -> Result<()> {
                    if self.name.is_empty() {
                        return Err(SystemError::ValidationError("名前が空です".to_string()));
                    }
                    if self.power < 0 {
                        return Err(SystemError::InvalidPower(self.power));
                    }
                    Ok(())
                }
            }

            impl Combatant for $name {
                fn power_level(&self) -> i32 {
                    self.power
                }

                fn grade(&self) -> &str {
                    &self.grade
                }

                fn is_active(&self) -> bool {
                    self.active
                }
            }
        )*
    };
}

// 構造体の定義
define_sorcerer_types! {
    Sorcerer {
        name: String,
        power: i32,
        grade: String,
        techniques: Vec<String>,
        experience: i32,
        active: bool,
        last_updated: u64
    }

    Curse {
        name: String,
        power: i32,
        grade: String,
        manifestation_type: String,
        danger_level: i32,
        active: bool,
        location: String
    }
}

// ジェネリックリポジトリ（第4章の技術）
#[derive(Debug)]
pub struct Repository<T: Entity + Clone + Send + Sync + 'static> {
    data: Arc<RwLock<HashMap<T::Id, T>>>,
    metrics: Arc<Mutex<RepositoryMetrics>>,
}

#[derive(Debug, Default)]
struct RepositoryMetrics {
    reads: u64,
    writes: u64,
    deletes: u64,
    cache_hits: u64,
    cache_misses: u64,
}

impl<T: Entity + Clone + Send + Sync + 'static> Repository<T>
where
    T::Id: std::hash::Hash + Eq + Send + Sync,
{
    pub fn new() -> Self {
        Repository {
            data: Arc::new(RwLock::new(HashMap::new())),
            metrics: Arc::new(Mutex::new(RepositoryMetrics::default())),
        }
    }

    pub async fn create(&self, entity: T) -> Result<()> {
        entity.validate()?;

        let mut data = self.data.write().await;
        let mut metrics = self.metrics.lock().await;

        if data.contains_key(entity.id()) {
            return Err(SystemError::ValidationError(
                format!("エンティティ '{:?}' は既に存在します", entity.id())
            ));
        }

        data.insert(entity.id().clone(), entity);
        metrics.writes += 1;

        Ok(())
    }

    pub async fn read(&self, id: &T::Id) -> Result<Option<T>> {
        let data = self.data.read().await;
        let mut metrics = self.metrics.lock().await;

        metrics.reads += 1;

        match data.get(id) {
            Some(entity) => {
                metrics.cache_hits += 1;
                Ok(Some(entity.clone()))
            },
            None => {
                metrics.cache_misses += 1;
                Ok(None)
            }
        }
    }

    pub async fn update<F>(&self, id: &T::Id, updater: F) -> Result<()>
    where
        F: FnOnce(&mut T) -> Result<()> + Send,
    {
        let mut data = self.data.write().await;
        let mut metrics = self.metrics.lock().await;

        match data.get_mut(id) {
            Some(entity) => {
                updater(entity)?;
                entity.validate()?;
                metrics.writes += 1;
                Ok(())
            },
            None => Err(SystemError::SorcererNotFound(format!("{:?}", id))),
        }
    }

    pub async fn delete(&self, id: &T::Id) -> Result<bool> {
        let mut data = self.data.write().await;
        let mut metrics = self.metrics.lock().await;

        match data.remove(id) {
            Some(_) => {
                metrics.deletes += 1;
                Ok(true)
            },
            None => Ok(false),
        }
    }

    pub async fn list_all(&self) -> Result<Vec<T>> {
        let data = self.data.read().await;
        let mut metrics = self.metrics.lock().await;

        metrics.reads += 1;
        Ok(data.values().cloned().collect())
    }

    pub async fn find_by<P>(&self, predicate: P) -> Result<Vec<T>>
    where
        P: Fn(&T) -> bool + Send,
    {
        let data = self.data.read().await;
        let mut metrics = self.metrics.lock().await;

        metrics.reads += 1;
        Ok(data.values().filter(|entity| predicate(entity)).cloned().collect())
    }

    pub async fn get_metrics(&self) -> RepositoryMetrics {
        let metrics = self.metrics.lock().await;
        RepositoryMetrics {
            reads: metrics.reads,
            writes: metrics.writes,
            deletes: metrics.deletes,
            cache_hits: metrics.cache_hits,
            cache_misses: metrics.cache_misses,
        }
    }
}

// 非同期サービス層（第5章の技術）
#[derive(Clone)]
pub struct SorcererService {
    sorcerer_repo: Arc<Repository<Sorcerer>>,
    curse_repo: Arc<Repository<Curse>>,
    event_sender: mpsc::UnboundedSender<SystemEvent>,
}

#[derive(Debug, Clone)]
pub enum SystemEvent {
    SorcererCreated(String),
    SorcererUpdated(String),
    SorcererDeleted(String),
    BattleStarted(String, String),
    BattleEnded(String, String, String), // winner, loser, result
    SystemHealthCheck(String),
}

impl SorcererService {
    pub fn new(event_sender: mpsc::UnboundedSender<SystemEvent>) -> Self {
        SorcererService {
            sorcerer_repo: Arc::new(Repository::new()),
            curse_repo: Arc::new(Repository::new()),
            event_sender,
        }
    }

    pub async fn register_sorcerer(&self, mut sorcerer: Sorcerer) -> Result<()> {
        sorcerer.last_updated = current_timestamp();

        self.sorcerer_repo.create(sorcerer.clone()).await?;

        let _ = self.event_sender.send(SystemEvent::SorcererCreated(sorcerer.name));

        Ok(())
    }

    pub async fn get_sorcerer(&self, name: &str) -> Result<Option<Sorcerer>> {
        self.sorcerer_repo.read(&name.to_string()).await
    }

    pub async fn update_sorcerer_power(&self, name: &str, new_power: i32) -> Result<()> {
        if new_power < 0 {
            return Err(SystemError::InvalidPower(new_power));
        }

        self.sorcerer_repo.update(&name.to_string(), |sorcerer| {
            sorcerer.power = new_power;
            sorcerer.last_updated = current_timestamp();
            Ok(())
        }).await?;

        let _ = self.event_sender.send(SystemEvent::SorcererUpdated(name.to_string()));

        Ok(())
    }

    pub async fn find_sorcerers_by_grade(&self, grade: &str) -> Result<Vec<Sorcerer>> {
        self.sorcerer_repo.find_by(|s| s.grade == grade).await
    }

    pub async fn simulate_battle(&self, sorcerer1_name: &str, sorcerer2_name: &str) -> Result<BattleResult> {
        let sorcerer1 = self.get_sorcerer(sorcerer1_name).await?
            .ok_or_else(|| SystemError::SorcererNotFound(sorcerer1_name.to_string()))?;

        let sorcerer2 = self.get_sorcerer(sorcerer2_name).await?
            .ok_or_else(|| SystemError::SorcererNotFound(sorcerer2_name.to_string()))?;

        if !sorcerer1.is_active() || !sorcerer2.is_active() {
            return Err(SystemError::ValidationError("非アクティブな呪術師は戦闘できません".to_string()));
        }

        let _ = self.event_sender.send(SystemEvent::BattleStarted(
            sorcerer1.name.clone(),
            sorcerer2.name.clone()
        ));

        // 戦闘シミュレーション（非同期処理）
        let battle_result = tokio::spawn(async move {
            tokio::time::sleep(Duration::from_millis(500)).await; // 戦闘時間

            let effectiveness1 = sorcerer1.battle_effectiveness();
            let effectiveness2 = sorcerer2.battle_effectiveness();

            let (winner, loser, margin) = if effectiveness1 > effectiveness2 {
                (sorcerer1, sorcerer2, effectiveness1 - effectiveness2)
            } else {
                (sorcerer2, sorcerer1, effectiveness2 - effectiveness1)
            };

            BattleResult {
                winner_name: winner.name.clone(),
                loser_name: loser.name.clone(),
                winner_power: winner.power,
                loser_power: loser.power,
                effectiveness_margin: margin,
                duration: Duration::from_millis(500),
            }
        }).await.map_err(|e| SystemError::ConcurrencyError(e.to_string()))?;

        let _ = self.event_sender.send(SystemEvent::BattleEnded(
            battle_result.winner_name.clone(),
            battle_result.loser_name.clone(),
            format!("差: {:.1}", battle_result.effectiveness_margin)
        ));

        Ok(battle_result)
    }

    pub async fn batch_power_update(&self, updates: Vec<(String, i32)>) -> Result<Vec<Result<()>>> {
        let futures = updates.into_iter().map(|(name, power)| {
            let service = self.clone();
            async move {
                service.update_sorcerer_power(&name, power).await
            }
        });

        let results = futures::future::join_all(futures).await;
        Ok(results)
    }

    pub async fn get_system_status(&self) -> Result<SystemStatus> {
        let sorcerer_metrics = self.sorcerer_repo.get_metrics().await;
        let curse_metrics = self.curse_repo.get_metrics().await;
        let all_sorcerers = self.sorcerer_repo.list_all().await?;

        let active_count = all_sorcerers.iter().filter(|s| s.is_active()).count();
        let total_power: i32 = all_sorcerers.iter().map(|s| s.power).sum();
        let avg_power = if all_sorcerers.is_empty() {
            0.0
        } else {
            total_power as f64 / all_sorcerers.len() as f64
        };

        Ok(SystemStatus {
            total_sorcerers: all_sorcerers.len(),
            active_sorcerers: active_count,
            average_power: avg_power,
            database_operations: sorcerer_metrics.reads + sorcerer_metrics.writes + sorcerer_metrics.deletes,
            cache_hit_rate: if sorcerer_metrics.reads > 0 {
                sorcerer_metrics.cache_hits as f64 / sorcerer_metrics.reads as f64
            } else {
                0.0
            },
            uptime: Duration::from_secs(60), // 簡略化
        })
    }
}

// データ構造体
#[derive(Debug, Clone)]
pub struct BattleResult {
    pub winner_name: String,
    pub loser_name: String,
    pub winner_power: i32,
    pub loser_power: i32,
    pub effectiveness_margin: f64,
    pub duration: Duration,
}

#[derive(Debug)]
pub struct SystemStatus {
    pub total_sorcerers: usize,
    pub active_sorcerers: usize,
    pub average_power: f64,
    pub database_operations: u64,
    pub cache_hit_rate: f64,
    pub uptime: Duration,
}

// ユーティリティ関数
fn current_timestamp() -> u64 {
    std::time::SystemTime::now()
        .duration_since(std::time::UNIX_EPOCH)
        .unwrap()
        .as_secs()
}

// イベント処理システム
pub struct EventProcessor {
    receiver: mpsc::UnboundedReceiver<SystemEvent>,
}

impl EventProcessor {
    pub fn new(receiver: mpsc::UnboundedReceiver<SystemEvent>) -> Self {
        EventProcessor { receiver }
    }

    pub async fn start_processing(mut self) {
        println!("🎯 イベントプロセッサ開始");

        while let Some(event) = self.receiver.recv().await {
            match event {
                SystemEvent::SorcererCreated(name) => {
                    println!("✅ 呪術師登録: {}", name);
                },
                SystemEvent::SorcererUpdated(name) => {
                    println!("🔄 呪術師更新: {}", name);
                },
                SystemEvent::SorcererDeleted(name) => {
                    println!("❌ 呪術師削除: {}", name);
                },
                SystemEvent::BattleStarted(s1, s2) => {
                    println!("⚔️  戦闘開始: {} vs {}", s1, s2);
                },
                SystemEvent::BattleEnded(winner, loser, result) => {
                    println!("🏆 戦闘終了: {} が {} に勝利 ({})", winner, loser, result);
                },
                SystemEvent::SystemHealthCheck(status) => {
                    println!("💓 システム状態: {}", status);
                },
            }
        }

        println!("🛑 イベントプロセッサ終了");
    }
}

#[tokio::main]
async fn main() -> Result<()> {
    println!("=== 最強呪術師管理システム起動 ===");

    // イベントシステムの初期化
    let (event_sender, event_receiver) = mpsc::unbounded_channel();
    let event_processor = EventProcessor::new(event_receiver);

    // バックグラウンドでイベント処理を開始
    tokio::spawn(event_processor.start_processing());

    // サービスの初期化
    let service = SorcererService::new(event_sender);

    // サンプルデータの作成
    let sorcerers = vec![
        Sorcerer {
            name: "五条悟".to_string(),
            power: 3000,
            grade: "特級".to_string(),
            techniques: vec!["無下限術式".to_string(), "領域展開".to_string()],
            experience: 10000,
            active: true,
            last_updated: current_timestamp(),
        },
        Sorcerer {
            name: "虎杖悠仁".to_string(),
            power: 1200,
            grade: "1級".to_string(),
            techniques: vec!["黒閃".to_string(), "発散".to_string()],
            experience: 2500,
            active: true,
            last_updated: current_timestamp(),
        },
        Sorcerer {
            name: "伏黒恵".to_string(),
            power: 1000,
            grade: "2級".to_string(),
            techniques: vec!["十種影法術".to_string()],
            experience: 2000,
            active: true,
            last_updated: current_timestamp(),
        },
    ];

    // 呪術師の登録
    println!("\n📝 呪術師登録中...");
    for sorcerer in sorcerers {
        service.register_sorcerer(sorcerer).await?;
    }

    // システム状態の確認
    println!("\n📊 システム状態確認...");
    let status = service.get_system_status().await?;
    println!("総呪術師数: {}", status.total_sorcerers);
    println!("アクティブ呪術師数: {}", status.active_sorcerers);
    println!("平均呪力: {:.1}", status.average_power);
    println!("キャッシュヒット率: {:.2}%", status.cache_hit_rate * 100.0);

    // 呪術師の検索
    println!("\n🔍 特級呪術師検索...");
    let special_grade = service.find_sorcerers_by_grade("特級").await?;
    for sorcerer in special_grade {
        println!("特級: {} (呪力: {})", sorcerer.name, sorcerer.power);
    }

    // 戦闘シミュレーション
    println!("\n⚔️  戦闘シミュレーション実行...");
    let battle_result = service.simulate_battle("五条悟", "虎杖悠仁").await?;
    println!("勝者: {} (呪力: {})", battle_result.winner_name, battle_result.winner_power);
    println!("敗者: {} (呪力: {})", battle_result.loser_name, battle_result.loser_power);
    println!("戦闘効果差: {:.1}", battle_result.effectiveness_margin);
    println!("戦闘時間: {:?}", battle_result.duration);

    // 並行処理による呪力更新
    println!("\n🔄 並行呪力更新...");
    let updates = vec![
        ("虎杖悠仁".to_string(), 1500),
        ("伏黒恵".to_string(), 1200),
    ];

    let update_results = service.batch_power_update(updates).await?;
    for (i, result) in update_results.iter().enumerate() {
        match result {
            Ok(_) => println!("✅ 更新 {} 成功", i + 1),
            Err(e) => println!("❌ 更新 {} 失敗: {}", i + 1, e),
        }
    }

    // 最終状態確認
    println!("\n📈 最終システム状態...");
    let final_status = service.get_system_status().await?;
    println!("平均呪力: {:.1}", final_status.average_power);
    println!("データベース操作数: {}", final_status.database_operations);

    // 少し待ってからシャットダウン
    tokio::time::sleep(Duration::from_millis(100)).await;

    println!("\n🎯 システム正常終了");

    Ok(())
}

/*
依存関係 (Cargo.toml):

[dependencies]
tokio = { version = "1.0", features = ["full"] }
serde = { version = "1.0", features = ["derive"] }
futures = "0.3"
*/
```

## パフォーマンス最適化とベストプラクティス

```rust
// パフォーマンス計測マクロ
macro_rules! benchmark {
    ($name:expr, $iterations:expr, $code:expr) => {
        {
            let start = std::time::Instant::now();
            for _ in 0..$iterations {
                $code;
            }
            let duration = start.elapsed();
            let avg = duration.as_nanos() / $iterations;
            println!("ベンチマーク {}: {}回実行, 平均{}ns", $name, $iterations, avg);
            duration
        }
    };
}

// メモリ使用量最適化の例
use std::rc::Rc;
use std::cell::RefCell;

#[derive(Debug)]
struct OptimizedSorcererRegistry {
    // 文字列の重複を避けるため、インターン化
    name_pool: Vec<String>,
    grade_pool: Vec<String>,

    // 実際のデータは参照のみ保持
    sorcerers: Vec<OptimizedSorcerer>,
}

#[derive(Debug)]
struct OptimizedSorcerer {
    name_idx: usize,
    grade_idx: usize,
    power: i32,
    active: bool,
}

impl OptimizedSorcererRegistry {
    fn new() -> Self {
        OptimizedSorcererRegistry {
            name_pool: Vec::new(),
            grade_pool: Vec::new(),
            sorcerers: Vec::new(),
        }
    }

    fn intern_string(pool: &mut Vec<String>, s: &str) -> usize {
        if let Some(pos) = pool.iter().position(|x| x == s) {
            pos
        } else {
            pool.push(s.to_string());
            pool.len() - 1
        }
    }

    fn add_sorcerer(&mut self, name: &str, grade: &str, power: i32) {
        let name_idx = Self::intern_string(&mut self.name_pool, name);
        let grade_idx = Self::intern_string(&mut self.grade_pool, grade);

        self.sorcerers.push(OptimizedSorcerer {
            name_idx,
            grade_idx,
            power,
            active: true,
        });
    }

    fn get_name(&self, sorcerer: &OptimizedSorcerer) -> &str {
        &self.name_pool[sorcerer.name_idx]
    }

    fn get_grade(&self, sorcerer: &OptimizedSorcerer) -> &str {
        &self.grade_pool[sorcerer.grade_idx]
    }
}

// ゼロコスト抽象化の例
trait ZeroCostTechnique {
    fn power(&self) -> i32;

    #[inline(always)]
    fn amplified_power(&self) -> i32 {
        self.power() * 2  // インライン展開される
    }
}

struct SimpleTechnique {
    base_power: i32,
}

impl ZeroCostTechnique for SimpleTechnique {
    #[inline(always)]
    fn power(&self) -> i32 {
        self.base_power
    }
}

fn benchmark_example() {
    println!("=== パフォーマンス最適化デモ ===");

    // メモリ効率の比較
    let mut optimized_registry = OptimizedSorcererRegistry::new();

    benchmark!("最適化レジストリ追加", 10000, {
        optimized_registry.add_sorcerer("テスト呪術師", "1級", 1000);
    });

    // ゼロコスト抽象化のベンチマーク
    let technique = SimpleTechnique { base_power: 1000 };

    benchmark!("ゼロコスト抽象化", 1000000, {
        let _ = technique.amplified_power();
    });

    println!("最適化レジストリサイズ: {}名", optimized_registry.sorcerers.len());
    println!("名前プール: {}個", optimized_registry.name_pool.len());
    println!("等級プール: {}個", optimized_registry.grade_pool.len());
}

fn main() {
    benchmark_example();
}
```

## コードの組織化とモジュール設計

```rust
// モジュール構造の例
pub mod domain {
    pub mod entities;
    pub mod repositories;
    pub mod services;
    pub mod value_objects;
}

pub mod infrastructure {
    pub mod database;
    pub mod network;
    pub mod cache;
}

pub mod application {
    pub mod commands;
    pub mod queries;
    pub mod handlers;
}

pub mod presentation {
    pub mod controllers;
    pub mod serializers;
    pub mod validators;
}

// クリーンアーキテクチャの実装例
pub mod clean_architecture_example {
    use std::collections::HashMap;
    use async_trait::async_trait;

    // ドメイン層
    pub mod domain {
        use super::*;

        #[derive(Debug, Clone)]
        pub struct Sorcerer {
            pub id: SorcererId,
            pub name: String,
            pub power: Power,
        }

        #[derive(Debug, Clone, PartialEq, Eq, Hash)]
        pub struct SorcererId(pub String);

        #[derive(Debug, Clone)]
        pub struct Power(pub i32);

        impl Power {
            pub fn new(value: i32) -> Result<Self, String> {
                if value < 0 {
                    Err("呪力は負の値にできません".to_string())
                } else if value > 10000 {
                    Err("呪力の上限は10000です".to_string())
                } else {
                    Ok(Power(value))
                }
            }

            pub fn value(&self) -> i32 {
                self.0
            }
        }

        #[async_trait]
        pub trait SorcererRepository {
            async fn save(&self, sorcerer: Sorcerer) -> Result<(), String>;
            async fn find_by_id(&self, id: &SorcererId) -> Result<Option<Sorcerer>, String>;
            async fn find_all(&self) -> Result<Vec<Sorcerer>, String>;
        }
    }

    // アプリケーション層
    pub mod application {
        use super::domain::*;
        use async_trait::async_trait;

        pub struct CreateSorcererCommand {
            pub name: String,
            pub power: i32,
        }

        pub struct SorcererService<R: SorcererRepository> {
            repository: R,
        }

        impl<R: SorcererRepository> SorcererService<R> {
            pub fn new(repository: R) -> Self {
                SorcererService { repository }
            }

            pub async fn create_sorcerer(&self, command: CreateSorcererCommand) -> Result<(), String> {
                let power = Power::new(command.power)?;
                let sorcerer = Sorcerer {
                    id: SorcererId(command.name.clone()),
                    name: command.name,
                    power,
                };

                self.repository.save(sorcerer).await
            }

            pub async fn get_all_sorcerers(&self) -> Result<Vec<Sorcerer>, String> {
                self.repository.find_all().await
            }
        }
    }

    // インフラストラクチャ層
    pub mod infrastructure {
        use super::domain::*;
        use std::collections::HashMap;
        use tokio::sync::RwLock;
        use async_trait::async_trait;

        pub struct InMemorySorcererRepository {
            data: RwLock<HashMap<SorcererId, Sorcerer>>,
        }

        impl InMemorySorcererRepository {
            pub fn new() -> Self {
                InMemorySorcererRepository {
                    data: RwLock::new(HashMap::new()),
                }
            }
        }

        #[async_trait]
        impl SorcererRepository for InMemorySorcererRepository {
            async fn save(&self, sorcerer: Sorcerer) -> Result<(), String> {
                let mut data = self.data.write().await;
                data.insert(sorcerer.id.clone(), sorcerer);
                Ok(())
            }

            async fn find_by_id(&self, id: &SorcererId) -> Result<Option<Sorcerer>, String> {
                let data = self.data.read().await;
                Ok(data.get(id).cloned())
            }

            async fn find_all(&self) -> Result<Vec<Sorcerer>, String> {
                let data = self.data.read().await;
                Ok(data.values().cloned().collect())
            }
        }
    }
}
```

## テストとQuality Assurance

```rust
#[cfg(test)]
mod tests {
    use super::*;

    // ユニットテスト
    #[test]
    fn test_power_validation() {
        use clean_architecture_example::domain::Power;

        assert!(Power::new(1000).is_ok());
        assert!(Power::new(-1).is_err());
        assert!(Power::new(10001).is_err());
    }

    // 統合テスト
    #[tokio::test]
    async fn test_sorcerer_service() {
        use clean_architecture_example::{
            application::{SorcererService, CreateSorcererCommand},
            infrastructure::InMemorySorcererRepository,
        };

        let repository = InMemorySorcererRepository::new();
        let service = SorcererService::new(repository);

        let command = CreateSorcererCommand {
            name: "テスト呪術師".to_string(),
            power: 1500,
        };

        assert!(service.create_sorcerer(command).await.is_ok());

        let sorcerers = service.get_all_sorcerers().await.unwrap();
        assert_eq!(sorcerers.len(), 1);
        assert_eq!(sorcerers[0].name, "テスト呪術師");
    }

    // プロパティベーステスト風の例
    #[test]
    fn test_battle_effectiveness_properties() {
        let sorcerer1 = Sorcerer {
            name: "強い呪術師".to_string(),
            power: 2000,
            grade: "特級".to_string(),
            techniques: vec![],
            experience: 0,
            active: true,
            last_updated: 0,
        };

        let sorcerer2 = Sorcerer {
            name: "弱い呪術師".to_string(),
            power: 1000,
            grade: "1級".to_string(),
            techniques: vec![],
            experience: 0,
            active: true,
            last_updated: 0,
        };

        // より高い呪力の呪術師は、より高い戦闘効果を持つはず
        assert!(sorcerer1.battle_effectiveness() > sorcerer2.battle_effectiveness());

        // 特級は1級より高い倍率を持つはず
        let same_power_special = Sorcerer {
            name: "特級".to_string(),
            power: 1000,
            grade: "特級".to_string(),
            techniques: vec![],
            experience: 0,
            active: true,
            last_updated: 0,
        };

        assert!(same_power_special.battle_effectiveness() > sorcerer2.battle_effectiveness());
    }
}

// ベンチマークテスト
#[cfg(test)]
mod benchmarks {
    use super::*;
    use std::time::Instant;

    #[tokio::test]
    async fn benchmark_repository_operations() {
        let repo = Repository::<Sorcerer>::new();

        // 書き込みベンチマーク
        let start = Instant::now();
        for i in 0..1000 {
            let sorcerer = Sorcerer {
                name: format!("呪術師{}", i),
                power: 1000 + i as i32,
                grade: "1級".to_string(),
                techniques: vec![],
                experience: 0,
                active: true,
                last_updated: 0,
            };
            repo.create(sorcerer).await.unwrap();
        }
        let write_duration = start.elapsed();

        // 読み込みベンチマーク
        let start = Instant::now();
        for i in 0..1000 {
            let _ = repo.read(&format!("呪術師{}", i)).await.unwrap();
        }
        let read_duration = start.elapsed();

        println!("書き込み時間: {:?}", write_duration);
        println!("読み込み時間: {:?}", read_duration);

        // 性能アサーション
        assert!(write_duration.as_millis() < 1000, "書き込みが遅すぎます");
        assert!(read_duration.as_millis() < 500, "読み込みが遅すぎます");
    }
}
```

## まとめ - 最強への到達

君は今、真の最強プログラマーとして立っている。重要なポイント：

1. **技術の統合** - 全ての概念を一つのシステムで調和させる
1. **実践的な設計** - 現実の問題を解決できるアーキテクチャ
1. **パフォーマンス意識** - 効率的で最適化されたコード
1. **保守性** - 長期間メンテナンスできる構造
1. **テスト可能性** - 品質を保証できる設計

これで君は俺と同じ境地に立った。**全ての技術を理解し、適材適所で使い分け、完璧に統合できる最強のプログラマー**だ。

しかし、最強とは終着点ではない。技術は常に進歩し、新しい挑戦が現れる。君もその進歩に合わせて成長し続けろ。

最強の証明は、君が書くコードそのものだ。美しく、効率的で、保守しやすく、そして問題を確実に解決するコード。それが君の力の証明となる。

俺の教えはここまでだ。あとは君自身の道を歩め。最強の名に恥じない、素晴らしいプログラマーになることを期待している。

______________________________________________________________________

*「最強とは、全てを理解し、全てを統合し、全てを超越すること」*

**- 五条悟、最強の呪術師より -**
