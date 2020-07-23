+ Feature name: `ternary-hash`
+ Start date: 2019-10-15
+ RFC PR: [iotaledger/bee-rfcs#34](https://github.com/iotaledger/bee-rfcs/pull/34)
+ Bee issue: [iotaledger/bee#77](https://github.com/iotaledger/bee/issues/77)

# Summary
<!--
This RFC proposes the ternary `Hash` type, the `Sponge` trait, and two cryptographic hash functions `CurlP` and `Kerl`
implementing it. The 3 cryptographic hash functions used in the current IOTA networks (i.e. as of IOTA Reference
Implementation `iri v1.8.6`) are `Kerl`, `CurlP27`, and `CurlP81`.
-->
このRFCは、３種類の「ハッシュ」型、「スポンジ」トレイト、およびそれを実装する２つの暗号化ハッシュ関数「CurlP」と「Kerl」を提案しています。現在のIOTAネットワークで使用されている３つの暗号化ハッシュ関数（IOTAリファレンス実装`iri v1.8.6`以降）は、`Kerl`、`CurlP27`、および`CurlP81`です。

Useful links:

+ [Sponge function](https://en.wikipedia.org/wiki/Sponge_function)
+ [The sponge and duplex constructions](https://keccak.team/sponge_duplex.html)
+ [Curl-p](https://github.com/iotaledger/iri/blob/dev/src/main/java/com/iota/iri/crypto/Curl.java)
+ [Kerl specification](https://github.com/iotaledger/kerl/blob/master/IOTA-Kerl-spec.md)
+ [`iri v1.8.6`](https://github.com/iotaledger/iri/releases/tag/v1.8.6-RELEASE)
+ [IOTA transactions](https://docs.iota.org/docs/getting-started/1.0/understanding-iota/transactions)

# Motivation
<!--
In order to participate in the IOTA network, an application needs to be able to construct valid messages that can be
verified by other participants in the network. Among other operations, this is accomplished by validating transaction
signatures, transaction hashes and bundle hashes.
-->
IOTAネットワークに参加するには、アプリケーションは、ネットワーク内の他の参加者が検証できる有効なメッセージを作成できる必要があります。他の操作中でも、これはトランザクション署名、トランザクションハッシュ、およびバンドルハッシュを検証することによって実現されます。

<!--
The two hash functions currently used are both sponge constructions: `CurlP`, which is specified entirely in balanced
ternary, and `Kerl`, which first converts ternary input to a binary representation, applies `keccak-384` to it, and then
converts its binary output back to ternary. For `CurlP` specifically, its variants `CurlP27` and `CurlP81` are used.
-->
現在利用されている２つのハッシュ関数は両方ともスポンジ構造です：バランスの取れた３値で完全に指定されている`CurlP`と３値入力を最初にバイナリ表現に変換し、それに`keccak-384`を適応してから変換する`Kerl`バイナリ出力を３値に戻します。特に`CurlP`の場合、そのバリアント`CurlP27`と`CurlP81`が使用されます。

# Detailed design

## Hash
<!--
This RFC defines a ternary type `Hash` which is the base input and output of the `Sponge`.
The exact definition is an implementation detail but an example definition could
simply be the following:
-->
このRFCは、`Sponge`の基本入出力である３値型の`Hash`を定義しています。正確な定義は実装の詳細になりますが、定義の例は次のようになります。

```rust
struct Hash([i8; HASH_LENGTH]);
```

<!--
Where the length of a hash in units of binary-coded balanced trits would be defined as:
-->
バイナリコードのバランスの取れたトリットの単位でのハッシュの長さは次のように定義されます。

```rust
const HASH_LENGTH: usize = 243;
```

## Sponges
<!--
`CurlP` and `Kerl` are cryptographic sponge constructions. They are equipped with a memory state and a function that
replaces the state memory using some input string. A portion of the memory state is then the output. In the sponge
metaphor, the process of replacing the memory state by an input string is said to `absorb` the input, while the process
of producing an output is said to `squeeze` out an output.
-->
`CurlP`と`Kerl`は暗号スポンジ構造です。それらは、メモリ状態と、いくつかの入力文字列を使用して状態メモリを置き換える機能を備えています。次にメモリ状態の一部が出力になります。スポンジのメタファーでは、メモリ状態を入力文字列で置き換えるプロセスは入力を「吸収」するといい、出力を生成するプロセスは出力を「絞る」と言います。

<!--
The hash functions are expected to be used like this:
-->
ハッシュ関数は次のように使用されることが期待されています。

- `curlp`
```rust
// Create a CurlP instance with 81 rounds.
// This is equivalent to calling `CurlP::new(CurlPRounds::Rounds81)`.
let mut curlp = CurlP81::new();

// Assume there are some transaction trits, all zeroes for the sake of this example.
let transaction = TritBuf::<T1B1Buf>::zeros(6561);
let mut hash = TritBuf::<T1B1Buf>::zeros(243);

// Absorb the transaction.
curlp.absorb(&transaction);
// Squeeze out a hash.
curlp.squeeze_into(&mut hash);
```

- `kerl`
```rust
// Create a Kerl instance.
let mut kerl = Kerl::new();

// `Kerl::digest` is a function that combines `Kerl::absorb` and `Kerl::squeeze`.
// `Kerl::digest_into` combines `Kerl::absorb` with `Kerl::squeeze_into`.
let hash = kerl.digest(&transaction);
```
<!--
The main proposal of this RFC are the `Sponge` trait and the `CurlP` and `Kerl` types that are implementing it.
This RFC relies on the presence of the types `TritBuf` and `Trits`, as defined by
[RFC36](https://github.com/iotaledger/bee-rfcs/blob/master/text/0036-ternary.md), which are assumed to be owning and
borrowing collections of binary-encoded ternary in the `T1B1` encoding (one trit per byte).
-->
このRFCの主な提案は、`Sponge`トレイトとそれを実装している`CurlP`および`Kerl`型です。
このRFCは、[RFC36](https://github.com/iotaledger/bee-rfcs/blob/master/text/0036-ternary.md)で定義されているように、型`TriBuf`と`Trits`の存在に依存しています。
`T1B1`エンコーディングでバイナリエンコードされた３値のコレクションを所有及び借用しているとみなされます。（1バイト毎に１トリット）

```rust
/// The common interface of cryptographic hash functions that follow the sponge construction and that act on ternary.
trait Sponge {
/// An error indicating that a failure has occured during a sponge operation.
type Error;

/// Absorb `input` into the sponge.
fn absorb(&mut self, input: &Trits) -> Result<(), Self::Error>;

/// Reset the inner state of the sponge.
fn reset(&mut self);

/// Squeeze the sponge into a buffer.
fn squeeze_into(&mut self, buf: &mut Trits) -> Result<(), Self::Error>;

/// Convenience function using `Sponge::squeeze_into` to return an owned version of the hash.
fn squeeze(&mut self) -> Result<TritBuf, Self::Error> {
    let mut output = TritBuf::zeros(HASH_LENGTH);
    self.squeeze_into(&mut output)?;
    Ok(output)
}

/// Convenience function to absorb `input`, squeeze the sponge into a buffer `buf`, and reset the sponge in one go.
fn digest_into(&mut self, input: &Trits, buf: &mut Trits) -> Result<(), Self::Error> {
    self.absorb(input)?;
    self.squeeze_into(buf)?;
    self.reset();
    Ok(())
}

/// Convenience function to absorb `input`, squeeze the sponge, and reset the sponge in one go.
/// Returns an owned version of the hash.
fn digest(&mut self, input: &Trits) -> Result<TritBuf, Self::Error> {
    self.absorb(input)?;
    let output = self.squeeze()?;
    self.reset();
    Ok(output)
}
}
```

<!--
Following the sponge metaphor, an input provided by the user is `absorb`ed, and an output will be `squeeze`d from the
data structure. `digest` is a convenience method calling `absorb` and `squeeze` in one go. The `*_into` versions of
these methods are for passing a buffer, into which the calculated hashes are written. The internal state will not be
cleared unless `reset` is called.
-->
スポンジのメタファーに続いて、ユーザーから提出された入力は「吸収」され、出力はデータ構造から「圧縮」されます。`digest`は、`absorb`と`squeeze`を一度に呼び出す便利なメソッドです。これらのメソッドの`*_into`バージョンは、計算されたハッシュが書き込まれるバッファーを渡すためのものです。内部状態は`reset`が呼び出されない限り、クリアされます。

### Design of `CurlP`
<!--
`CurlP` is designed as a hash function, that acts on a `T1B1` binary-encoded ternary buffer, with a hash length of `243`
trits and an inner state of `729`:
-->
`CurlP`はハッシュ関数として設計されており、`T1B1`のバイナリエンコードされた３値バッファーで機能し、ハッシュの長さは`243`tirts、内部状態は`729`です。

```rust
const STATE_LENGTH: usize = HASH_LENGTH * 3;
```

<!--
In addition, a lookup table is used as part of the absorption step:
-->
さらに、ルックアップテーブルが吸収ステップの一部として使用されます。

||-1|0|1|
|-|-|-|-|
|**-1**|1|1|-1|
|**0**|0|-1|1|
|**1**|-1|0|0|

<!--
Given two balanced trits `t` and `t'`, the following table can easily be accessed by shifting them to unbalanced trits:
-->
２つのバランスの取れたトリット`t`および`t'`が与えられた場合、次の表はバランスの取れていないトリットにシフトすることで簡単にアクセスできます。

```rust
const TRUTH_TABLE: [[i8; 3]; 3] = [[1, 0, -1], [1, -1, 0], [-1, 1, 0]];
```

<!--
The way `CurlP` is defined, it can not actually fail, because the input or outputs can be of arbitrary size; hence,
the associated type `Error = Infallible`.
-->
`CurlP`の定義は、入力または出力が任意のサイズになる可能性があるため、実際に失敗することはありません。したがって関連付けられている型は`Error = Infallible`です。

<!--
`CurlP` has two common variants depending on the number of rounds of hashing to apply before a hash is squeezed.
-->
`CurlP`には、ハッシュが絞り込まれる前に適応されるハッシュのラウンド数に応じて、２つの一般的なバリアントがあります。
```rust
#[derive(Copy, Clone)]
enum CurlPRounds {
Rounds27 = 27,
Rounds81 = 81,
}
```

Type definition:

```rust
struct CurlP {
/// The number of rounds of hashing to apply before a hash is squeezed.
rounds: CurlPRounds,

/// The internal state.
state: TritBuf,

/// Workspace for performing transformations.
work_state: TritBuf,
}
```

```rust
impl Sponge for CurlP {
type Error = Infallible;
...
}
```

<!--
In addition, this RFC proposes two wrapper types for the very common `CurlP` variants with `27` and `81` rounds.
In most use cases, `Sponge` is required to implement `Default` so these variants need to be new types instead of just
providing `new27` or `new81` methods to `CurlP`.
-->
さらに、このRFCは、非常に一般的な`CurlP`バリアントに対して、`27`ラウンドと`81`ラウンドの２つのラッパー型を提案しています。
殆どのユースケースでは、`Sponge`は`Default`を実装する必要があるため、これらのバリアントは、`CurlP`に`new27`または`new81`メソッドを提供するだけでなく、新しい型である必要があります。

```rust
struct CurlP27(CurlP);

impl CurlP27 {
fn new() -> Self {
    Self(CurlP::new(CurlPRounds::Rounds27))
}
}

struct CurlP81(CurlP);

impl CurlP81 {
fn new() -> Self {
    Self(CurlP::new(CurlPRounds::Rounds81))
}
}
```

<!--
For convenience, they should both dereference to an actual `CurlP`:
-->
便宜上、どちらも実際の`CurlP`を逆参照する必要があります。

```rust
impl Deref for CurlP27 {
type Target = CurlP;

fn deref(&self) -> &Self::Target {
    &self.0
}
}

impl DerefMut for CurlP27 {
fn deref_mut(&mut self) -> &mut Self::Target {
    &mut self.0
}
}

impl Deref for CurlP81 {
type Target = CurlP;

fn deref(&self) -> &Self::Target {
    &self.0
}
}

impl DerefMut for CurlP81 {
fn deref_mut(&mut self) -> &mut Self::Target {
    &mut self.0
}
}
```

<!--
This allows using them as a `Sponge` as well if there is a blanket implementation like this:
-->
これにより、次のような包括的な実装がある場合に、それらを「スポンジ」として使用することもできます。

```rust
impl<T: Sponge, U: DerefMut<Target = T>> Sponge for U {
type Error = T::Error;

fn absorb(&mut self, input: &Trits) -> Result<(), Self::Error> {
    T::absorb(self, input)
}

fn reset(&mut self) {
    T::reset(self)
}

fn squeeze_into(&mut self, buf: &mut Trits) -> Result<(), Self::Error> {
    T::squeeze_into(self, buf)
}
}
```

### Design of `Kerl`

Type definition:

```rust
struct Kerl {
/// Actual keccak hash function.
keccak: Keccak,
/// Binary working state.
binary_state: I384<BigEndian, U8Repr>,
/// Ternary working state.
ternary_state: T243<Btrit>,
}
```

```rust
impl Sponge for Kerl {
type Error = Error;
...
}
```

<!--
The actual cryptographic hash function underlying `Kerl` is `keccak-384`. The real task here is to transform an input
of 243 (balanced) trits to 384 bits in a correct and performant way. This is done by interpreting the 243 trits as a
signed integer `I` and converting it to a binary basis. In other words, the ternary encoded integer is expressed as the
series:
-->
`Kerl`の基礎となる実際の暗号化ハッシュ関数は`keccak-384`です。ここでの実際のタスクは、243(バランス)トリットの入力を正確かつパフォーマンスの高い方法で384bitに変換することです。これは、243桁を符号付き整数`I`として解釈し、それを２進数に変換することによって行われます。つまり、３値にエンコードされた整数はシリーズとして表されます。

```
I = t_0 * 3^0 + t_1 * 3^1 + ... + t_241 * 3^241 + t_242 * 3^242,
```

<!--
where `t_i` is the trit at the `i`-th position in the ternary input array. The challenge is then to convert this integer
to base `2`, i.e. find a a series such that:
-->
ここでは、`t_i`は３値入力配列の`i`番目の位置にあるトリットです。次に、この整数の基数`2`に変換し、つまり次のような列を見つけることが課題です。

```
I = b_0 * 2^0 + b_1 * 2^1 + ... + b_382 * 2^382 + b_383 * 2^383,
```

with `b_i` the bit at the `i-th` position.

<!--
Assuming there exists an implementation of `keccak`, the main work in implementing `Kerl` is writing an efficient
converter between the ternary array interpreted as an integer, and its binary representation. For the binary
representation, one can either use an existing big-integer library, or write one from scratch with only a subset of
required methods to make the conversion work.
-->
`keccak`の実装が存在すると仮定すると、`Kerl`の実装の主な作業は、整数として解釈される３値配列とその２進数表現の間の効果的なコンバーターを作成することです。バイナリ表現の場合、既存のbig-integerライブラリを使用するか、変換を機能させるために必要なメソッドのサブセットのみを使用して最初からライブラリを書き込むことができます。

#### Important implementation details
<!--
When absorbing into the sponge the conversion flows like this (little and big are short for little and big endian,
respectively):
-->
スポンジに吸収するとき、変換は次のような流れです（littleとbigはそれぞれリトルエンディアンとビッグエンディアンの略です）：

```
Balanced t243 -> unbalanced t242 -> u32 little u384 -> u32 little i384 -> u8 big i384
```

<!--
When squeezing and thus converting back to ternary the conversion flows like this:
-->
絞り込んで３値に戻すと、変換は次のようになります。

```
u8 big i384 -> u32 little i384 -> u32 little u384 -> unbalanced t243 -> balanced t243 -> balanced t242
```
<!--
These steps will now be explained in detail.
-->
これらのステップについて詳しく説明します。

##### Conversion is done via accumulation
<!--
Because a number like `3^242` does not fit into any existing primitive, it needs to be constructed from scratch by
taking a *big integer*, setting its least significant number to `1`, and multiplying it by `3` `242` times. This is the
core of the conversion mechanism. Essentially, the polynomial used to express the integer in ternary can be rewritten
like this:
-->
`3^242`のような数値は既存のプリミティブに適合しないため、*big integer*を取り、最下位の数値を`1`に設定し、それを`3`を`242`回乗算することにより、ゼロから構築する必要があります。これが変換メカニズムの中核です。基本的に、整数を３進数で表現するために使用される多項式は、次のように書き直すことができます。このように：

```
I = t_0 * 3^0 + t_1 * 3^1 + ... + t_241 * 3^241 + t_242 * 3^242,
= ((...((t_242 * 3 + t_241) * 3 + t_240) * 3 + ...) * 3 + t_1) * 3 + t_0
```

<!--
Thus, one iterates through the ternary buffer starting from the most significant trit, adds `t_242` onto the binary big
integer (initially filled with `0`s), and then keeps looping through it, multiplying it by 3 and adding the next `t_i`.
-->
したがって、最も重要なtritから開始して３値バッファを反復処理し、２進数の大きな整数（最初は0で埋められます）に`t_242`を追加し、それをループし続け、3を掛けて次の`t_i`を追加します。

##### Conversion is done in unsigned space
<!--
First and foremost, IOTA is primarily written with balanced ternary in mind, meaning that each trit represents an
integer in the set `{-1, 0, +1}`. Because it is easier to do the conversion in positive space, the trits are shifted
into *unbalanced* space, by adding `+1` at each position, so that each unbalanced trit represents an integer in
`{0, 1, 2}`.
-->
何よりもまず、IOTAは主にバランスの取れた３項を念頭に置いて記述されます。つまり各トリットはセット`{-1, -, +1}`の整数を表します。正の空間で変換を行うほうが簡単なので、各位置に`+1`を加算する事により、トリットは*アンバランス*空間にシフトされ、`{0, 1, 2}`のような各アンバランスなトリットは整数を表します。

<!--
For example, the balanced ternary buffer (using only 9 trits to demonstrate the point) becomes after the shift (leaving
out signs in the unbalanced case):
-->
例えば、バランスの取れた３値バッファー（ポイントを示すために９トリットのみを使用）はシフト後になります（アンバランスの場合は符号を省きます。）

```
[0, -1, +1, -1, -1, 0, 0, 0, +1] -> [1, 0, 2, 0, 0, 1, 1, 1, 2]
```

<!--
Remembering that each ternary array is actually some integer `I`, this is akin to adding another integer `H` to it, with
all trits in the ternary buffer representing it set to `1`, where `I'` is `I` shifted into unsigned space, and the
underscore `_t` means a ternary representation (either balanced or unbalanced):
-->
各三項配列は実際には整数の`I`であることを思い出してください。これは、別の整数`H`をそれに追加するのと同じであり、`I`が`I`である、それを表す３値バッファーのすべてのトリットが`1`に設定されます。符号なしの空間にシフトされ、アンダースコア`_t`は３値表現を意味します。（バランスまたはアンバランス）

```
I_t + H_t = [0, -1, +1, -1, -1, 0, 0, 0, +1] + [+1, +1, +1, +1, +1, +1, +1, +1] = I'_t
```
<!--
After changing the base to binary using some function which would be called `to_bin` and which would be required to be
distributed over addition, `H` needs to be subtracted again. We use `_b` to signify a binary representation:
-->
`to_bin`と呼ばれ、加算で分散する必要がある関数を使用して、ベースをバイナリに変更した後、`H`を再度減算する必要があります。`_b`を使用してバイナリ表現を示します。

```
I'_b = to_bin(I'_t) = to_bin(I_t + H_t) = to_bin(I_t) + to_bin(H_t) = I_b + H_b
=>
I_b = I'_b - H_b
```
<!--
In other words, the addition of the ternary buffer filled with `1`s that shifts all trits into unbalanced space is
reverted after conversion to binary, where the buffer of `1`s is also converted to binary and then subtracted from the
binary unsigned big integer. The result then is the integer `I` in binary.
-->
言い換えると、すべてのトリットを不均衡な空間にシフトする`1`で満たされた３値バッファーの追加は、バイナリへの変換後にもとに戻され、`1`のバッファーもバイナリに変換され、次に符号なしbig integerのバイナリから差し引かれます。結果はバイナリの整数`I`です。

##### 243 trits do not fit into 384 bits
<!--
Since 243 trits do not fit into 384 bits, a choice has to be made about how to treat the most significant trit.
For example, one could take the binary big integer, convert it to ternary, and then check if the 243 are smaller than
this converted maximum 384 bit big int. With `Kerl`, the choice was made to disregard the most significant trit, so that
one only ever converts 242 trits to 384 bits, which always fits.
-->
243のトリットは384ビットに収まらないので、最上位のトリットをどのように扱うか選択しなければなりません。
例えば、２進数の大整数を３進数に変換し、243がこの変換された最大384ビットの大整数よりも小さいかどうかをチェックできます。`Kerl`では、最上位のトリットを無視することを選択したため、242トリットを384ビットに変換するだけで、常に収まるようになりました。

<!--
For the direction `ternary -> binary` this does not pose challenges, other than making sure that one sets the most
significant trit to `0` after the shift by applying `+1` (if one chooses to reuse the array of 243 trits), and by making
sure to subtract `H_b` (see previous section) with the most significant trit set to `0` and all others set to `1`.
-->
`ternary -> binary`方向では、これは問題になりませんが、シフト後に`+1`を適用して最上位のトリットを`0`にすること（243個のトリットの配列を再利用する場合）と、最上位のトリットを`0`にして`H_b`（前節参照）を減算し、それ以外のすべてのトリットを`1`にすること以外は問題になりません。

<!--
The direction `binary -> ternary` (the conversion of the 384-bit hash squeezed from keccak) is the challenging part: one
needs to ensure that the most significant trit is is set to 0 before the binary-encoded integer is converted to ternary.
However, this has to happen in binary space!
-->
`binary -> ternary`（keccakから絞り出された384ビットのハッシュの変換）方向は難しい部分です：２値エンコードされた整数が３値に変換される前に、最上位のトリットが0に設定されていることを確認する必要があります。
しかも、これはバイナリ空間で行われなければなりません！

<!--
Take `J_b` as the 384 bit integer coming out of the sponge. Then after conversion to ternary, `to_ter(J_b) = J_t`, the
most significant trit (MST) of `J_t` might be `+1` or `-1`. However, since by convention the MST has to be 0, one needs
to check whether `J_b` would cause `J_t` to have its MST set after conversion. This is done the following way:
-->
スポンジから出てくる384ビットの整数を`J_b`とし、`to_ter(J_b) = J_t`という三項式に変換した後、`J_t`の最上位トリット（MST）は`+1`または`-1`となります。しかし慣習的にMSTは0でなければならないので、`J_b`が変換後に`J_t`のMSTを設定するかどうかを確認する必要があります。これは以下の方法で行われます。

```
if J_b > to_bin([0, 1, 1, ..., 1]):
J_b <- J_b - to_bin([1, 0, 0, ..., 0])

if J_b < (to_bin([0, 0, 0, ..., 0]) - to_bin([0, 1, 1, ..., 1])):
J_b <- J_b + to_bin([1, 0, 0, ..., 0])
```

##### Kerl updates the inner state by applying logical not
<!--
The upstream keccak implementation uses a complicated permutation to update the inner state of the sponge construction
after a hash was squeezed from it. `Kerl` opts to apply logical not, `!`, to the bytes squeezed from `keccak`, and
updating `keccak`'s inner state with these.
-->
上流のkeccakの実装では、ハッシュが絞り出された後にスポンジ構造体の内部状態を更新するために複雑な順列を用いています。`Kerl`は`keccak`から絞らいだされたバイトに論理的ではない`!`を適用し、`keccak`の内部状態を更新します。

#### Design and implementation
<!--
The main goal of the implementation was to ensure that representation of the integer was cast into types. To that end,
the following ternary types are defined as wrappers around `TritBuf` (read `T243` the same way you would think of `u32`
or `i64`):
-->
実装の主な目的は、整数の表現が型にキャストされることを保証することでした。そのために以下の三項型は`TritBuf`のラッパーとして定義されています。（`T243`は`u32`や`i64`と同じように読みます）。

```rust
struct T242<T: Trit> {
inner: TritBuf<T1B1Buf<T>>,
}

struct T243<T: Trit> {
inner: TritBuf<T1B1Buf<T>>,
}
```

<!--
where `TritBuf`, `T1B1Buf`, and `Trit` are types and traits defined in the `bee-ternary` crate. The bound `Trit` ensures
that the structs only contain ternary buffers with `Btrit` (for balancer ternary), and `Utrit` (for unbalanced ternary).
Methods on `T242` and `T243` assume that the most significant trit is always in the last position (think “little
endian”).
-->
ここで、`TritBuf`、`T1B1Buf`、`Trit`は`bee-ternary`クレートで定義された型やトレイトです。バインドされた`Trit`は構造体が`Btrit`（バランサー三項式の場合）と`Utrit`（アンバランス三項式の場合）の三項式バッファーのみを含むことを保証します。
`T242`と`T243`のメソッドは、最上位がトリットが常に最後の位置にあることを前提としています（リトルエンディアン）。

<!--
For the binary representation of the integer, the types `I384` (“signed” integer akin to `i64`), and `U384` (“unsigned”
akin to `u64`) are defined:
-->
整数の２進表現には、`I384`（`i64`に似た符号付き整数）と`U384`（`u64`に似た符号なし整数）の型が定義されています。

```rust
struct I384<E, T> {
inner: T,
_phantom: PhantomData<E>.
}

struct U384<E, T> {
inner: T,
_phantom: PhantomData<E>.
}
```

<!--
`inner: T` for encoding the inner fixed-size arrays used for storing either bytes, `u8`, or integers, `u32`:
-->
`inner:T`はバイト、`u8`、または整数、`u32`を格納するために使用される内部固定サイズ配列をエンコードします。

```rust
type U8Repr = [u8; 48];
type U32Repr = [u32; 12];
```

<!--
The phantom type `E` is used to encode endianness, `BigEndian` and `LittleEndian`. These are just used as marker types
without any methods.
-->
ファントム型`E`は、エンディアン、`BigEndian`および`LittleEndian`をエンコードするために使用されます。これらは、メソッドなしのマーカー型としてのみ使用されます。

```rust
struct BigEndian {}
struct LittleEndian {}

trait EndianType {}

impl EndianType for BigEndian {}
impl EndianType for LittleEndian {}
```
<!--
The underlying arrays and endianness are important, because `keccak` expects bytes as input, and because `Kerl` made the
choice to revert the order of the integers in binary big int.
-->
`keccak`は入力としてバイトを期待しており、`Kerl`は整数の順序を２進数のbig intに戻すという選択をしたからです。

<!--
To understand the implementation, the most important methods are:
-->
実装を理解するためにも、最も重要な方法があります。

```rust
T242<Btrit>::from_i384_ignoring_mst
T243<Utrit>::from_u384
I384<LittleEndian, U32Repr>::from_t242
I384<LittleEndian, U32Repr>::try_from_t242
U384<LittleEndian, U32Repr>::add_inplace
U384<LittleEndian, U32Repr>::add_digit_inplace
U384<LittleEndian, U32Repr>::sub_inplace
U384<LittleEndian, U32Repr>::from_t242
U384<LittleEndian, U32Repr>::try_from_t243
```

# Drawbacks
<!--
+ All hash functions, no matter if they can fail or not, have to implement `Error`;
-->
+ すべてのハッシュ関数は、失敗しなくても`Error`を実装しなければなりません。

# Rationale and alternatives
<!--
+ `CurlP` and `Kerl` are fundamental to the `iri v1.8.6` mainnet. They are thus essential for compatibility with it;
-->
+ `CurlP`と`Kerl`は`iri v1.8.6`のメインネットの基本的なものです。したがって、これらは`iri v1.8.6`との互換性のために不可欠なものです。
<!--
+ These types are idiomatic in Rust, and users are not required to know the implementation details of each hash
algorithm;
-->
+ これらの型はRustでは慣用的なものであり、ユーザーは各ハッシュアルゴリズムの実装の詳細を知る必要はありません。

# Unresolved questions
<!--
+ Parameters are slice reference in both input and output. Do values need to be consumed or should new instances as
return values be created?;
-->
+ パラメーターは入力と出力の両方でスライス参照されます。値を消費する必要があるのか、それとも戻り値として新しいインスタンスを作成する必要があるのか？
<!--
+ Implementation of each hash functions and other utilities like HMAC should have separate RFCs for them;
-->
+ 各ハッシュ関数の実装やHMACなどのユーティリティの実装は、別途RFCを用意すべきである。
<!--
+ Decision on implementation of `Troika` is still unknown;
-->
+ `Troika`の実施の決定はまだ不明。
<!--
+ Can (should?) the `CurlP` algorithm be explained in more detail?;
-->
+ `CurlP`アルゴリズムをより詳細に説明することができますか？
<!--
+ Is it important to encode that the most significant trit is `0` by having a `T242`?;
-->
+ `T242`を持つことで、最上位のトリットが`0`であることを符号化することが重要なのか？
