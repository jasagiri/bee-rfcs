+ Feature name: `network`
+ Start date: 2020-04-29
+ RFC PR: [iotaledger/bee-rfcs#33](https://github.com/iotaledger/bee-rfcs/pull/33)
+ Bee issue: [iotaledger/bee#76](https://github.com/iotaledger/bee/issues/76)

# Summary
<!-->
This RFC proposes a networking crate (`bee-network`) to be added to the **Bee** framework that provides means to exchange byte messages with connected endpoints.
-->
このRFCでは、接続されたエンドポイントとバイトメッセージを交換する手段を提供する**Bee**フレームワークに追加するネットワーキングクレート(`bee-network`)を提案します。

# Motivation
<!--
Bee nodes need a way to share messages with each other and other compatible node implementations as part of the IOTA gossip protocol or they won't be able to form a network and synchronize with each other.
-->
BeeノードはIOTAゴシッププロトコルの一部として、他の互換性のあるノード実装とメッセージを共有する方法が必要です。

# Detailed design
<!--
The functionalities of this crate are relatively low-level from a node perspective in the sense, that they are independent from any specifics defined by the IOTA protocol. Consequently, it has no dependencies on other crates of the framework, and can easily be reused in different contexts.

The aim of this crate is to make it simple and straightforward for developers to build layers on top of it (e.g. a protocol layer) by abstracting away all the underlying networking logic. It is therefore a lot easier for them to focus on the modelling of other important aspects of a node software, like:

* `Peer`s: other nodes in the network,
* `Message`es: the information exchanged between peers, and its serialization/deserialization.

Given some identifier `epid`, sending a message to its corresponding endpoint becomes a single line of asynchronous code:
-->

このクレートの機能は、IOTAプロトコルによって定義された仕様から独立しているという意味で。ノードの観点からは比較的低レベルなものです。その結果、フレームワークの他のクレートに依存することなく、異なるコンテキストで簡単に再利用することができます。

このクレートの目的は、基礎となるネットワークロジックをすべて抽象化することで、開発者がその上のレイヤ（プロトコルレイア等）を構築することです。そのため、開発者はノードソフトウェアの他の重要な側面のモデリングに集中しやすくなります。

* `Peer`s：ネットワーク上の他のノード
* `Message`es：ピア間で交換される情報とそのシリアライズまたはデシリアライズ。

ある識別子`epid`が与えられると、それに対応するエンドポイントへのメッセージ送信は１行の非同期コードになります。

```rust
network.send(SendMessage { epid, bytes: "hello".as_bytes() }).await?;
```
<!--
The purpose of this crate is to provide the following functionalities:
* maintain a list of endpoints,
* establish and maintain connections with endpoints,
* allow to send, multicast or broadcast byte encoded messages of variable size to any of the endpoints,
* allow to receive byte encoded messages of variable size from any of the endpoints,
* reject connections from unknown, i.e. not whitelisted, endpoints,
* manage all connections and message transfers asynchronously,
* provide an extensible, yet convenient-to-use, and documented API.

The key design decisions are now being discussed in the following sub-sections.
-->

このクレートの目的は、以下の機能を提供することです。
* エンドポイントのリストを保持する。
* エンドポイントとの接続を確立し、維持する。
* 可変サイズのバイトエンコードされたメッセージを任意のデン度ポイントに送信、マルチキャスト、またはブロードキャストする。
* 任意のエンドポイントから可変サイズのバイトエンコードされたメッセージを受信する。
* 未知の、つまり許可リストに登録されていないエンドポイントからの接続を拒否する。。
* すべての接続とメッセージ転送を非同期で管理する。
* 拡張性がありながらも使いやすく、文書化されたAPIを提供する。

設計上の重要な決定事項は、以下のサブセクションで議論されています。

## async/await
<!--
This crate is by nature very much dependent on events happening outside of the control of the program, e.g. listening for incoming connections from peers, waiting for packets on a specific socket, etc. Hence, - under the hood - this crate makes heavy use of Rust's concurrency abstractions. Luckily, with the stabilization of the `async/await` syntax, writing asynchronous code has become almost as easy to read, write, and maintain as synchronous code. In an experimental implementation of this crate the [`async_std`](https://github.com/async-rs/async-std) library was used, which comes with asynchronous drop-in replacements for their synchronous equivalents found in Rust's standard library `std`. Additionally, asynchronous mpsc channels were taken from the [`futures`](https://github.com/rust-lang/futures-rs) crate.
-->
このクレートは本来、ピアからの着信接続を待ち受ける特定のパケットを待つなどプログラムの制御外で発生するイベントに大きく依存しています。したがって、内部では、このクレートはRustの同時実行抽象化を多用します。幸いなことに、`async/await`構文の安定化により、非同期のコードの記述は、同期コードと同じくらいに簡単に読み取ったり書き込んだりと維持できるようになりました。このクレートの実験的な実装では[`async_std`](https://github.com/async-rs/async-std)ライブラリが使用されました。これにはRustの標準ライブラリ`std`にある同期同等の非同期ドロップイン置換が付属しています。さらに、非同期mpscチャネルは[`futures`](https://github.com/rust-lang/futures-rs)クレートから取得されました。

## Message passing
<!--
This crate favors message passing via channels over globally shared state and locks. Instead of keeping the list of endpoints in a globally accessible hashmap this crate separates and moves such state into workers that run asynchronously, and are listening for commands and events to act upon, and also notify listeners by sending events.
-->
このクレートは、グローバルに共有された状態やロックよりも、チャネルを介したメッセージの受け渡しを優先します。エンドポイントのリストをグローバルにアクセス可能なハッシュマップに保持する代わりに、このクレートはそのような状態を分離して、非同期に実行され、動作するコマンドとイベントを待ち受けているワーカーに移動し、イベントを送信してリスナーに通知します。

## Initialization
<!--
In order for this crate to be used, it has to be initialized first by providing a `NetworkConfig`:
-->
このクレートを使用するには、最初に`NetworkConfig`を指定して初期化する必要があります。

```rust
struct NetworkConfig {
    // The server or binding TCP port that remote endpoints can connect and send to.
    binding_port: u16,
    // The binding address that remote endpoints can connect and send to.
    binding_addr: IpAddr,
    // The interval between two connection attempts in case of connection failures.
    reconnect_interval: u64,
    // The maximum message length in terms of bytes.
    max_message_length: usize

    ...
}
```
<!--
This has the advantage of being easily extensible and keeping the signature of the static `init` function small:
-->
これには、簡単に拡張でき、静的な`init`関数の署名を小さく保つという利点があります。

```rust
fn init(config: NetworkConfig) -> (Network, Events, Shutdown) { ... }
```
<!--
This function returns a tuple struct that allows the consumer of this crate to send commands (`Network`), listen to events (`Events`), and gracefully shutdown all running asynchronous tasks that were spawned within this system (`Shutdown`).
-->

この関数は、このクレートの利用者が（`Network`）コマンドを送信したり、（`Event`）イベントを待ち受けたり、このシステム内で生成されたすべての実行中の非同期タスクを正常に（`Shutdown`）できるタプル構造体を返します。
<!-->
See also  [RFC 31](https://github.com/iotaledger/bee-rfcs/blob/master/text/0031-configuration.md), which describes a configuration pattern for Bee binary and library crates.
-->
[RFC 31](https://github.com/iotaledger/bee-rfcs/blob/master/text/0031-configuration.md)も参照してください。これは、Beeバイナリとライブラリクレートの構成パターンについて説明しています。

## `Port`, `Address`, `Protocol`, and `Url`
<!--
In order to send a message to an endpoint, a node needs to know the endpoint's address and the communication protocol it uses. This crate provides the following abstractions to deal with that:
-->
エンドポイントにメッセージを送信するには、ノードはエンドポイントのアドレスと使用する通信プロトコルを知っている必要があります。このクレートは、それに対処するために次の抽象化を提供します。

<!--
* `Port`: a 0-cost wrapper around a `u16`, which is introduced for type safety and better readability:
-->
* `Port`: 型安全とよみやすさのために導入された、`u16`の0コストのラッパー。

    ```rust
    struct Port(u16);
    ```

    For convenience, `Port` dereferences to `u16`.

<!--
* `Address`: a 0-cost wrapper around a `SocketAddr` which provides an adjusted, but overall simpler API than its inner type:
-->
* `Address`: 内部の型より全体的にシンプルに調整されたAPI`SocketAddr`を提供します。

    ```rust
    #[derive(Clone, Copy, Debug, Eq, Hash, PartialEq)]
    struct Address(SocketAddr);
    ```

    An `Address` can be constructed from Ipv4 and Ipv6 addresses and a port:

    ```rust
    // Infallible construction of an `Address` from Ipv4.
    fn from_v4_addr_and_port(address: Ipv4Addr, port: Port) -> Self { ... }

    // Infallible construction of an `Address` from Ipv6.
    fn from_v6_addr_and_port(address: Ipv6Addr, port: Port) -> Self { ... }

    // Fallible construction of an `Address` from `&str`.
    async fn from_addr_str(address: &str) -> Result<Self> { ... }
    ```

    Note, that the last function is `async`. That is because of possible delays when performing domain name resolution.

<!--
* `Protocol`: an `enum`eration of supported communication protocols:
-->
* `Protocol`: サポートされる通信プロトコルの列挙`enum`。

    ```rust
    #[non_exhaustive]
    enum Protocol {
        Tcp,
    }
    ```

    Note, that future updates may support different protocols, which is why this enum is declared as `non_exhaustive`, see `Unresolved Questions` section.

<!--
* `Url`: an `enum`eration that can always be constructed from an `Address` and a `Protocol` (infallible), or from a `&str`, which can fail if parsing or domain name resolution fails. If successfull however, it resolves to an Ipv4 or Ipv6 address stored in a variant of the `enum` depending on the url scheme part. Note, that this crate therefore expects the `Url` string to always provide a scheme (e.g. `tcp://`) and a port (e.g. `15600`) when specifying an endpoint's address.
-->
* `Url`: `Address`と`Protocol`（間違いのない）、あるいは`&str`から常に構築された`enum`列挙体、解析やドメイン名の解決に失敗した場合に、失敗する可能性があります。ただし、成功した場合は、URLスキームの部分に応じて、`enum`のバリアントに格納されている
Ipv4またはIpv6アドレスに解決されます。このため、このクレートでは、エンドポイントのアドレスを指定するときに、`Url`文字列が常にスキーマ（例：`tcp://`）とポート（例：`15600`）を指定する必要があることに注意してください。

    ```rust
    #[non_exhaustive]
    enum Url {
        // Represents the address of an endpoint communicating over TCP.
        Tcp(Address),
    }
    ```

    Note, that future updates may support different protocols, which is why this enum is declared as `non_exhaustive`, see `Unresolved Questions` section.

    ```rust
    // Infallible construction of a `Url` from an `Address` and a `Protocol`.
    fn new(addr: Address, proto: Protocol) -> Self { ... }

    // Fallible construction of a `Url` from `&str`.
    async fn from_url_str(url: &str) -> Result<Self> { ... }
    ```

    Note, that the second function is `async`. That is because of possible delays when performing domain name resolution.



## `Endpoint` and `EndpointId`
<!--
To model the remote part of a connection, this crate introduces the `Endpoint` type:
-->
接続のリモート部分をモデル化するために、このクレートは`Endopoint`型を導入します。

```rust
struct Endpoint {
    // The id of the endpoint.
    id: EndpointId,

    // The address of the endpoint.
    address: Address,

    // The protocol used to communicate with that endpoint.
    protocol: Protocol,
}
```
<!--
Note: In a peer-to-peer (p2p) network peers usually have two endpoints initially as they are actively trying to connect, but also need to accept connections from their peers. This crate is agnostic about how to handle duplicate connections as this is usually resolved by a handshaking protocol defined at a higher layer.
-->
注意：ピアツーピア（p2p）ネットワークでは、ピアは積極的に接続しようとしているが、ピアからの接続を受け入れる必要があるため、通常、最初の２つはエンドポイントを持っています。このクレートは、重複接続をどのように処理するかについては不可知論的です。

<!--
To uniquely identify an `Endpoint`, this crate proposes the `EndpointId` type, which can be implemented as a wrapper around an `Address`.
-->
`Endpoint`を一意に識別するために、このクレートでは`EndpointId`型を提案します。

```rust
#[derive(Clone, Copy, Eq, PartialEq, Hash)]
struct EndpointId(Address);
```
<!--
Note, that this `EndpointId` should be `Clone` and `Copy`, and requires to implement `Eq`, `PartialEq`, and `Hash`, because it is used as a hashmap key in several instances.
-->
注意すること、`EndpointId`は`Clone`と`Copy`で有るべきで、いくつかのインスタンスでハッシュマップキーとして利用されているため、`Eq`、`PartialEq`、`Hash`を実装すること。

## `Command`
<!--
`Command`s are messages that are supposed to be issued by higher level business logic like a protocol layer. They are implemented as an `enum`, which in Rust is one way of expressing a polymorphic type:
-->
`Command`sはプロトコル層のような高レベルのビジネスロジックから発行されることになっているメッセージです。これらは`enum`として実装されており、Rustでは多相型を表現する一つの方法となっています。

```rust
enum Command {
    // Adds an endpoint to the list.
    AddEndpoint { url: Url, ... },

    // Removes an endpoint from the list.
    RemoveEndpoint { epid: EndpointId, ... },

    // Attempts to connect to an endpoint.
    Connect { epid: EndpointId, ... },

    // Disconnects from an endpoint.
    Disconnect { epid: EndpointId, ... },

    // Sends a message to a particular endpoint.
    SendMessage { epid: EndpointId, bytes: Vec<u8>, ... },

    // Sends a message to multiple endpoints.
    MulticastMessage { epids: Vec<EndpointId>, bytes: Vec<u8>, ...},

    // Sends a message to all endpoints in its list.
    BroadcastMessage { bytes: Vec<u8>, ... },
}
```
<!--
This `enum` makes the things the consumer can do with the crate very explicit and descriptive, and also easily extensible.
-->
この`enum`は、消費者がクレートでできることを明示的かつ記述的にし、簡単に拡張できるようにします。

### `Event`
<!--
Similarly to `Command`s, the different kinds of `Event`s in the system are implemented as an `enum` allowing various types of concrete events being sent over event channels:
-->
`Command`sと同様に、システム内の様々な種類の`Event`sは、イベントチャネルを介して様々な型の具体的なイベントを送信できるように、`enum`として実装されています。

```rust
#[non_exhaustive]
enum Event {
    // An endpoint was added to the list.
    EndpointAdded { epid: EndpointId, ... },

    // An endpoint was removed from the list.
    EndpointRemoved { epid: EndpointId, ... },

    // A new connection is being set up.
    NewConnection { ep: Endpoint, ... },

    // A broken connection.
    LostConnection { epid: EndpointId },

    // An endpoint was successfully connected.
    EndpointConnected { epid: EndpointId, address: Address, ... },

    // An endpoint was disconnected.
    EndpointDisconnected { epid: EndpointId, ... },

    // A message was sent.
    MessageSent { epid: EndpointId, num_bytes: usize },

    // A message was received.
    MessageReceived { epid: EndpointId, bytes: Vec<u8> },

    // A connection attempt should occur.
    TryConnect { epid: EndpointId, ... },
}

```
<!--
In contrast to `Command`s though, `Event`s are messages created by the system, and those that are considered relevant are published for the consumer to execute custom logic. It is attributed as `non_exhaustive` to accommodate for possible additions in the future.
-->
しかし、`Command`sとは対象的に、`Event`sはシステムが生成したメッセージであり、関連性があると考えられるものはカスタムロジックを実行するために公開されます。将来的に追加される可能性を考慮して、`non_exhaustive`としています。

## Workers
<!--
There are two essential asynchronous workers in the system that are running for the application's whole lifetime. That is the `EndpointWorker` and the `TcpWorker`.
-->
システムには、アプリケーションの全ライフタイムに渡って動作する２つの本質的な非同期ワーカーがあります。それが`EndpointWorker`と`TcpWorker`です。

### EndpointWorker
<!--
This worker manages the list of `Endpoint`s and processes the `Command`s issued by the consumer of this crate, and publishes `Event`s to be consumed by the user.
-->
このワーカーは、`Endpoint`sのリストを管理し、このクレートの消費者が発行する`Command`sを処理し、ユーザーが消費する`Event`sを発行します。

### TcpWorker
<!--
This worker is responsible for accepting incoming TCP connection requests from other endpoints. Once a connection is established two additional asynchronous tasks are spawned that respectively handle incoming and outgoing messages.
-->
このワーカーは、他のエンドポイントからの着信TCP接続要求を受け付ける責任があります。接続が確立されると、更に２つの非同期タスクが生成され、それぞれ受信メッセージと送信メッセージを処理します。

<!--
Note: connection attempts from unknown IP addresses (i.e. not part of the static whitelist) will be rejected.
-->
注意：未知のIPアドレス（つまり静的許可リストに含まれていない）からの接続は拒否されます。

## Shutdown
<!-->
`Shutdown` is a `struct` that facilitates a graceful shutdown. It stores the sender halfs of `oneshot` channels (`ShutdownNotifier`) and the task handles (`WorkerTask`).
-->
`Shutdown`は優雅なシャットダウンを容易にするための`構造体`です。これは`oneshot`チャネルの送信者の半分（`ShutdownNotifier`）とタスクハンドル（`WorkerTask`）を格納します。

```rust
struct Shutdown {
    notifiers: Vec<ShutdownNotifier>,
    tasks: Vec<WorkerTask>,
}
```
<!--
Apart from providing the necessary API to register channels and tasks, executing the shutdown happens by calling:
-->
チャネルやタスクを登録するために必要なAPIを提供することとは別に、シャットダウンの実行を呼び出されます。

```rust

async fn execute(self) -> Result<()> { ... }

```
<!--
This method will then try to send a shutdown signal to all registered workers and wait for those tasks to complete.
-->
このメソッドは、登録されているすべてのワーカーにシャットダウン信号を送信しようとして、それらのタスクが完了するのを待ち受けます。

# Drawbacks
<!--
* This design might not be the most performant solution as it requires each byte message to be wrapped into a `SendMessage` command, that is then sent through a channel to the `EndpointWorker`. Proper benchmarking is necessary to determine if this design can be a performance bottleneck;
-->
* このデザインは、各バイトメッセージを`SendMessage`コマンドにラップして、チャネルを介して`EndpointWorker`に送信する必要があるため、最もパフォーマンスの高いソリューションとは言えないかもしれません。このデザインがパフォーマンスのボトルネックになるかどうかを判断するには、適切なベンチマークが必要です。

<!--
* Currently UDP is not a requirement, so the proposal focuses on TCP only. Nonetheless, the crate is designed in way that allows for simple addition of UDP support;
-->
* 現在、UDPは必須ではないので、TCPのみに焦点を当てています。しかしUDPサポートを簡単に追加できるように設計されています。

# Rationale and alternatives
<!--
* The reason for not choosing an already available crate, is that there aren't many crates yet, that are implemented in the proposed way, and are also widely used in the Rust ecosystem. Additionally we reduce dependencies, and have the option to tailor the crate exactly to our needs even on short notice;
-->
* 既に利用可能なクレートを選択しない理由は、提案された方法で実装されており、Rustエコシステムでも広く利用されているクレートがまだ多くないからです。さらに、依存性を減らし、急な通知でもニーズに合わせてクレートをカスタマイズすることができます。

<!--
* There are many ways to implement such a crate, many of which are probably more performant than the proposed one. However, the main focus of this proposal is ease-of-use and extensibility. It leaves performance optimization to later versions and/or RFCs, when more data has been collected through stress tests and benchmarks;
-->
* このようなクレートを実装する方法はたくさんありますが、その多くはおそらく提案されているものよりも性能が高いと思われます。しかし、この提案の主な焦点は、使いやすさと拡張性です。性能の最適化は、ストレステストやベンチマークでより多くのデータが収集された後のバージョンやRFCに任せます。

<!--
* In particular, the `Command` abstraction could be removed entirely, and replaced by an additional static function for each command. This might be a more efficient thing to do, but it is still unclear how much there is to be gained by doing this, and if this would be necessary taking into account the various other CPU expensive things like hashing, signature verification, etc, that a node needs to do;
-->
* 特に、`Command`の抽象化は完全に取り除き、各コマンドのための追加の静的関数に置き換えることができます。これはより効率的なことかもしれませんが、これを行うことでどれだけ利益があるのか、またノードが行う必要があるハッシュや署名検証など、他の様々なCPUコストのかかることを考慮するとこれが必要かどうかはまだ不明です。

<!--
* Since mpsc channels are still unstable in `async_std` at the time of this writing, this RFC suggests using those from the `futures` crate. Once this has been stabilized, a swith to those channels might improve performance, since to the authors knowledge those are based on the more efficient `crossbeam` implementation;
-->
* この記事を書いている時点では、`async_std`のmpscチャネルはまだ不安定なので、このRFCでは`futures`クレートのものを使うことを提案しています。これが安定してきたら、これらのチャネルに移行することでパフォーマンスが向上するかもしれません。

<!--
* To implement this RFC, any async library or mix can be used. Other options to consider are [`tokio`](https://github.com/tokio-rs/tokio), and for asynchronous channels [`flume`](https://github.com/zesterer/flume). Using those libraries, might result in a much more efficient implementation;
-->
* このRFCを実装するためには、非同期ライブラリやミックスを使用できます。その他のオプションとしては、[`tokio`](https://github.com/tokio-rs/tokio)や非同期チャネル用には [`flume`](https://github.com/zesterer/flume)があります。これらのライブラリを使用することで、より効率的な実装になるかもしれません。

<!--
* `EndpointId` may be something different than a wrapper around an `Address`, but instead be derived from some sort of cryptographic hash;
-->
* `EndpointId`は`Address`のラッパーとは別物で、その代わりに暗号化ハッシュのようなものから派生したものである可能性があります。

# Unresolved questions
<!--
* The design has been tested in a prototype, and it seems to work reliably, so there are no unresolved questions in terms of the validity of this proposal;
-->
* この設計は試作機でテストしており、確実に動作しているようであり、本提案の妥当性については未解決の問題はありません。

<!--
* It is still unclear - due to the lack of benchmark results - how efficient this design is; however, due to its asynchronous design it should be able to make use from multicore systems;
-->
* ベンチマーク結果が出ていないため、どの程度の効率性があるのかは不明ですが、非同期設定のため、マルチコアシステムでも利用できるはずです。

<!--
* Handling of endpoints with dynamic IPs;
-->
* 動的IPを持つエンドポイントの処理。
