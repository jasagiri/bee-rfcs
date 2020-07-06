+ Feature name: `configuration`
+ Start date: 2020-05-06
+ RFC PR: [iotaledger/bee-rfcs#31](https://github.com/iotaledger/bee-rfcs/pull/31)
+ Bee issue: [iotaledger/bee#71](https://github.com/iotaledger/bee/issues/71)

# Summary
<!--
This RFC proposes a configuration pattern for Bee binary and library crates.
-->

このRFCはBeeバイナリとライブラリクレートのための設定パターンを提案しています。

# Motivation
<!--
This RFC contains a set of recommendations regarding configuration management in order to ensure consistency across the
different Bee crates.
-->

このRFCには、別のBeeクレート間で一貫性を確保するための設定管理に関する推奨事項が含まれています。

# Detailed design

## Libraries

- [serde](https://github.com/serde-rs/serde): the go-to framework for **ser**ializing and **de**serializing Rust data
  structures efficiently and generically;
- [toml-rs](https://github.com/alexcrichton/toml-rs): a TOML encoding/decoding library for Rust with
  serialization/deserialization on top of `serde`;

## Recommendations
<!--
With the following recommendations, one can:
- read a configuration builder from a file;
- write a configuration to a file;
- manually override fields of a configuration builder;
- construct a configuration from a configuration builder;
-->

以下の推奨事項を使用すると、下記のようなことができます。
- ファイルから設定ビルダを読み込む
- 設定をファイルに書き込む。
- 設定ビルダのフィールドを手動で上書きする。
- 設定ビルダから設定を構築する。

### Configuration Builder
<!--
The [Builder](https://rust-lang.github.io/api-guidelines/type-safety.html#c-builder) pattern is very common in Rust when
it comes to constructing complex objects like configurations.

A configuration builder type should:
- have a name suffixed by `ConfigBuilder`;
- derive the following traits;
    - `Default` to easily implement the `new` method as convention;
    - `Deserialize`, from `serde`, to deserialize from a configuration file;
- provide a `new` method;
- have `Option` fields;
    - if there is a configuration file, the fields should have the same names as the keys;
- provide setters to set/override the fields;
- provide a `finish` method constructing the actual configuration object;
- have default values defined as `const` and set with `Option::unwrap_or`/`Option::unwrap_or_else`;

Here is a small example fitting all these requirements.
-->

[Builder](https://rust-lang.github.io/api-guidelines/type-safety.html#c-builder) パターンは、コンフィグレーションのような複雑なオブジェクトを構築するとき、Rustで非常によく見られるます。

コンフィグレーションビルダの型は
- `ConfigBuilder`を接尾語を持つ。
- 次のような特徴を持っている必要がある。
    - `Default`は`new`メソッドを簡単に実装できるようにする。
    - `serde`の`Deserialize`は、設定ファイルからデシリアライズされる。
- `new`メソッドを提供する。
- `Option`フィールドを持つ。
    - 設定ファイルがある場合、フィールドはキーと同じ名前の必要がある。
- フィールドを設定または上書きするセッターを提供する。
- 実際の設定オブジェクトを構築する、`finish`メソッドを提供する。
- デフォルトを`const`として定義し、`Option::unwrap_or`や`Option::unwrap_or_else`で設定する。

ここに、これをの要求をすべて満たす小さな例を示します。

`config.toml`
```toml
[snapshot]
meta_file_path  = "./data/snapshot/mainnet.snapshot.meta"
state_file_path = "./data/snapshot/mainnet.snapshot.state"
```

`config.rs`
```rust
const DEFAULT_META_FILE_PATH: &str = "./data/snapshot/mainnet.snapshot.meta";
const DEFAULT_STATE_FILE_PATH: &str = "./data/snapshot/mainnet.snapshot.state";

#[derive(Default, Deserialize)]
pub struct SnapshotConfigBuilder {
    meta_file_path: Option<String>,
    state_file_path: Option<String>,
}

impl SnapshotConfigBuilder {
    pub fn new() -> Self {
        Self::default()
    }

    pub fn meta_file_path(mut self, meta_file_path: String) -> Self {
        self.meta_file_path.replace(meta_file_path);
        self
    }

    pub fn state_file_path(mut self, state_file_path: String) -> Self {
        self.state_file_path.replace(state_file_path);
        self
    }

    pub fn finish(self) -> SnapshotConfig {
        SnapshotConfig {
            meta_file_path: self.meta_file_path.unwrap_or(DEFAULT_META_FILE_PATH.to_string()),
            state_file_path: self.state_file_path.unwrap_or(DEFAULT_STATE_FILE_PATH.to_string()),
        }
    }
}
```

### Configuration
<!--
A configuration type should:
- have a name suffixed by `Config`;
- derive the following traits;
  - `Clone`, since all configuration are most probably aggregated in a common configuration after reading from a file,
    this trait is needed to give components a unique ownership of their own configuration;
  - `Serialize` if the configuration is expected to be updated and saved;
- provide a `build` method that returns a new instance of the associated builder;
- have the same fields with the same names, without `Option`, as the builder;
- have no public fields;
- provide setters/updaters only on fields that are expected to be updatable;
- have getters or `pub(crate)` fields;

Here is a small example fitting all these requirements:
-->

設定の型は
- `Config`を接尾後に名前を持つべきです。
- 以下のような特徴を持っていなければなりません。
    - `Clone`：ファイルから読み込んだあとに、すべての設定が共通の設定に集約されることがほとんどだからです。
    この特徴はコンポーネントに独自の設定の所有権を与えるために必要です。
    - 設定が更新されて保存されることが期待される場合は、`Serialize`を指定します。
- 関連するビルダの新しいインスタンスを返す`build`メソッドを提供します。
- ビルダと同じ名前で同じフィールドを持ち、`Option`を使いません。
- パブリックフィールドを持ちません。
- 更新可能なフィールドのみにセッターやアップデーターを提供します。
- ゲッターまたは`pub(crate)`フィールドを持ちます。

これをの要件をすべて満たす小さな例を以下に示します。

`config.toml`
```toml
[snapshot]
meta_file_path  = "./data/snapshot/mainnet.snapshot.meta"
state_file_path = "./data/snapshot/mainnet.snapshot.state"
```

`config.rs`
```rust
#[derive(Clone)]
pub struct SnapshotConfig {
    meta_file_path: String,
    state_file_path: String,
}

impl SnapshotConfig {
    pub fn build() -> SnapshotConfigBuilder {
      SnapshotConfig::new()
    }

    pub fn meta_file_path(&self) -> &String {
        &self.meta_file_path
    }

    pub fn state_file_path(&self) -> &String {
        &self.state_file_path
    }
}

```

### Read a configuration builder from a file

```rust
let config_builder = match fs::read_to_string("config.toml") {
    Ok(toml) => match toml::from_str::<SnapshotConfigBuilder>(&toml) {
        Ok(config_builder) => config_builder,
        Err(e) => {
            // Handle error
        }
    },
    Err(e) => {
        // Handle error
    }
};

// Override fields if necessary e.g. with CLI arguments.

let config = config_builder.finish();
```

### Write a configuration to a file

```rust
match toml::to_string(&config) {
    Ok(toml) => match fs::File::create("config.toml") {
        Ok(mut file) => {
            if let Err(e) = file.write_all(toml.as_bytes()) {
                // Handle error
            }
        }
        Err(e) => {
            // Handle error
        }
    },
    Err(e) => {
        // Handle error
    }
}
```

### Sub-configuration
<!--
It is also very easy to create sub-configurations by nesting configuration builders and configurations.
-->

コンフィグレーションビルダとコンフィグレーションを入れ子にして、サブコンフィグレーションを作成するのも非常に簡単です。

`config.toml`
```toml
[snapshot]
[snapshot.local]
meta_file_path  = "./data/snapshot/local/mainnet.snapshot.meta"
state_file_path = "./data/snapshot/local/mainnet.snapshot.state"
[snapshot.global]
file_path       = "./data/snapshot/global/mainnet.txt"
```

`config.rs`
```rust
#[derive(Default, Deserialize)]
pub struct LocalSnapshotConfigBuilder {
    meta_file_path: Option<String>,
    state_file_path: Option<String>,
}

#[derive(Default, Deserialize)]
pub struct GlobalSnapshotConfigBuilder {
    file_path: Option<String>,
}

#[derive(Default, Deserialize)]
pub struct SnapshotConfigBuilder {
    local: LocalSnapshotConfigBuilder,
    global: GlobalSnapshotConfigBuilder,
}

impl SnapshotConfigBuilder {
    pub fn new() -> Self {
        Self::default()
    }

    // Setters

    pub fn finish(self) -> SnapshotConfig {
        SnapshotConfig {
            local: LocalSnapshotConfig {
                meta_file_path: self
                    .local
                    .meta_file_path
                    .unwrap_or(DEFAULT_LOCAL_SNAPSHOT_META_FILE_PATH.to_string()),
                state_file_path: self
                    .local
                    .state_file_path
                    .unwrap_or(DEFAULT_LOCAL_SNAPSHOT_STATE_FILE_PATH.to_string()),
            },
            global: GlobalSnapshotConfig {
                file_path: self
                    .global
                    .file_path
                    .unwrap_or(DEFAULT_GLOBAL_SNAPSHOT_FILE_PATH.to_string()),
            },
        }
    }
}
#[derive(Clone)]
pub struct LocalSnapshotConfig {
    meta_file_path: String,
    state_file_path: String,
}

#[derive(Clone)]
pub struct GlobalSnapshotConfig {
    file_path: String,
}

#[derive(Clone)]
pub struct SnapshotConfig {
    local: LocalSnapshotConfig,
    global: GlobalSnapshotConfig,
}

// Impl

```

# Drawbacks
<!--
No specific drawback with this approach.
-->

この方法では特に欠点はありません。

# Rationale and alternatives
<!--
Many configuration formats are usually considered:
-->

通常、多くのコンフィグレーション形式が考えられます。
- [TOML](https://github.com/toml-lang/toml)
- [JSON](https://www.json.org/json-en.html)
- [HOCON](https://github.com/lightbend/config/blob/master/HOCON.md)
- [YAML](https://yaml.org/)
- [XML](https://www.w3.org/XML/)
- [RON](https://github.com/ron-rs/ron)
<!--
This RFC and its expected implementations are choosing `TOML` as a first default configuration format because it is the
preferred option in the Rust ecosystem. However, it is not excluded by this RFC that other formats may be provided in
the future, `serde` making it very easy to support other formats. It is also important to note that TOML only provides
a limited amount of layers of nesting due to its non-recursive syntax, which may eventually become an issue.

`Serde` itself has been chosen because it is the standard for serialization/deserialization in Rust.
-->

このRFCと期待される実装では、Rustエコシステムで好まれているので、最初のデフォルトコンフィグレーションフォーマットとして`TOML`を選択しています。ただし、他のフォーマットが将来的に提供される可能性があることは、このRFCによって除外されているわけではなく、`serde`によって他のフォーマットのサポートが非常に簡単になります。また、TOMLは非再帰的な構文のため、入れ子になるレイヤー数が限られることに注意してください。これは最終的に問題になる可能性があります。

`Serde`自体は、Rustでのシリアライズやデシリアライズの標準であるために選択されました。

# Unresolved questions
<!--
In case of binary crates, e.g.`bee-node`, configuration with CLI arguments is not described in this RFC but everything
is already set up to support it seamlessly. The builder setters allow setting fields or overriding fields that may
have already pre-filled by the parsing of a configuration file. A CLI parser library like
[clap](https://github.com/clap-rs/clap) may be used on top of the builders;
-->

`bee-node`のようなバイナリクレートの場合、CLI引数を使用したコンフィグレーションはこのRFCには記載されていませんが、すべてシームレスにサポートするように設定されています。ビルダセッターでは、フィールドの設定やコンフィグレーションファイルの解析により、すでに事前に入力されています。[clap](https://github.com/clap-rs/clap)のようなCLIパーサーライブラリをビルダ上で使用できます。
