# Bee RFCs

[![Bee RFC Book](https://github.com/iotaledger/bee-rfcs/workflows/Bee%20RFC%20Book/badge.svg)](https://iotaledger.github.io/bee-rfcs/)
<!--
> This process is modelled after the approach taken by the Rust programming language, see [Rust RFC repository] for more
information. Also see [maidsafe's RFC process] for another project into the crypto space. Our approach is taken and
adapted from these.
-->
> このプロセスは、Rustプログラミング言語で採用されたアプローチに従ってモデル化されています。[Rust RFC repository]を参照して情報を得てください。暗号空間への別のプロジェクトについては、[maidsafe's RFC process]も参照してください。私達のアプローチが取られ、これらから適応されています。  

<!--
To make more substantial changes to Bee, we ask for these to go through a more organized design process --- a *RFC*
(request for comments) process. The goal is to organize work between the different developers affiliated with the IOTA
Foundation, and the wider open source community. We want to vet the ideas early on, get and give feedback, and only then
start the implementation once the biggest questions are taken care of.
-->
Beeへのより実質的な変更を行うために、私達は、より組織化された設計プロセス（*RFC* (request for comment)）を通過するためにお願いしています。IOTA財団に所属する様々な開発者と、より広いオープンソースコミュニティの間で作業を整理することを目的とします。私達は、早い段階でアイデアを吟味し、フィードバックを得て、最大の疑問が解決してから実装を開始したいと考えています。

## What is *substantial* and when to follow this process
<!--
You need to follow this process if you intend to make *substantial* changes to Bee and any of its constituting
subcrates.
-->
あなたはBeeとそれを構成しているサブクレートのいずれかに*実質的な*変更を行う場合は、このプロセスに従う必要があります。
<!--
+ Anything that constitutes a breaking change as understood in the context of *semantic versioning*.
+ Any semantic or syntactic change to the existing algorithms that is not a bug fix.
+ Any proposed additional functionality and feature.
+ Anything that reduces interoperability (e.g. changes to public interfaces, the wire protocol or data serialisation.)
-->
+ *セマンティックバージョンニング*の文脈で理解されているように、破壊的な変更を構成するもの。
+ 既存のアルゴリズムへの意味的または構文的な変更で、バグフィックスではないもの。
+ 提案された追加機能や特徴
+ 相互運用性を低下させるもの。（例：パブリックインターフェース、ワイヤプロトコル、データのシリアライゼーションへの変更）

<!--
Some changes do not require an RFC:
-->
RFCを必要としない変更もあります。

<!--
+ Rephrasing, reorganizing, refactoring, or otherwise "changing shape does not change meaning".
+ Additions that strictly improve objective, numerical quality criteria (warning removal, speedup, better platform
  coverage, more parallelism, trap more errors, etc.)
+ *Internal* additions, i.e. additions that are invisible to users of the public API, and which probably will be only
noticed by developers of Bee.
-->
+ 再現性、再編成、リファクタリングなど「形を変えても意味が変わらない」もの
+ 客観的で定量的な品質基準を厳密に改善する追加（警告の除去、高速化、より良いプラットフォームカバレッジ、より多くの並列性、より多くのエラーをトラップするなど）
+ *内部*の追加、つまり公開APIのユーザーには見えない追加で、おそらくはBee開発者が気づいたもの。

<!--
If you submit a pull request to implement a new feature without going through the RFC process, it may be closed with a
polite request to submit an RFC first.
-->
RFCプロセスを経ずに新機能を実装するためのプルリクエストを提出した場合、最初にRFCを提出するように丁寧にお願いされます。

## The workflow of the RFC process
<!--
> With both Rust and maidsafe being significantly larger projects, not all the steps below might be relevant to Bee's
RFC process (for example, at the moment there are no dedicated subteams). However, to instill good open source
governance we attempt to follow this process already now.
-->
> Rustとmaidsafeの療法がかなり大きなプロジェクトであるため、以下のすべてのステップがBeeのRFCプロセスに関連しているとは限りません（例えば、今のところ専用のサブチームはありません）。しかし良いオープンソースガバナンスを浸透させるために、私たちは既にこのプロセスに従うつもりです。

<!--
In short, to get a major feature added to Bee, one must first get the RFC merged into the RFC repository as a markdown
file. At that point the RFC is "active" and may be implemented with the goal of eventual inclusion into Bee.
-->
要するに、Beeに追加された主要な機能を取得するには、まずRFCをMarkdownファイルとしてRFCレポジトリにマージする必要があります。その時点でRFCは「アクティブ」であり、最終的にBeeに含めることを目標に実装される可能性があります。

<!--
+ Fork the RFC repository
-->
+ RFCリポジトリをフォークする。
<!--
+ Copy `0000-template.md` to `text/0000-my-feature.md` (where "my-feature" is descriptive; don't assign
  an RFC number yet; extra documents such as graphics or diagrams go into the new folder).
-->
+ `0000-template.md`を`text/0000-my-feature.md`にコピーします（`0000-my-feature`は説明のための例です。まだRFC番号を割り当てないでください。グラフィックスや図表などの追加のドキュメントは新しいフォルダに移動します。）
<!--
+ Fill in the RFC. Put care into the details: RFCs that do not present convincing motivation, demonstrate lack of
  understanding of the design's impact, or are disingenuous about the drawbacks or alternatives tend to be
  poorly-received.
-->
+ RFCを書きます。詳細に注意します。説得力のある動機を示さず、設計の影響への理解の欠如を示し、欠点や代替案について不誠実なRFCは受け入れられない傾向にあります。
<!--
+ Submit a pull request. As a pull request the RFC will receive design feedback from the larger community, and the
  author should be prepared to revise it in response.
-->
+ プリリクエストを送信します。プルリクエストとして、RFCはより大きなコミュニティから設計のフィードバックを受け取り、著者はそれに応じて改定する準備をする必要があります。
<!--
+ Each pull request will be labeled with the most relevant sub-team, which will lead to its being triaged by that team
  in a future meeting and assigned to a member of the subteam.
-->
+ 各プルリクエストには、最も関連性の高いサブチームのラベルが付けられます。これにより、将来の会議でそのチームによってトリアージされ、サブチームのメンバーに割り当てられます。
<!--
+ Build consensus and integrate feedback. RFCs that have broad support are much more likely to make progress than those
  that don't receive any comments. Feel free to reach out to the RFC assignee in particular to get help identifying
  stakeholders and obstacles.
-->
+ コンセンサスを構築し、フィードバックを統合します。幅広いサポートがあるRFCはコメントを受け取らないものより遥かに進捗する可能性が高くなります。特にRFCを譲り受けた人に連絡して、利害関係者と障害を特定するのを手伝ってください。
<!--
+ The sub-team will discuss the RFC pull request, as much as possible in the comment thread of the pull request itself.
  Offline discussion will be summarized on the pull request comment thread.
-->
+ サブチームは、プルリクエスト自体のコメントスレッドで、できる限りRFCプルリクエストについて話し合います。オフラインディスカッションは、プルリクエストのコメントスレッドにまとめられます。
<!--
+ RFCs rarely go through this process unchanged, especially as alternatives and drawbacks are shown. You can make edits,
  big and small, to the RFC to clarify or change the design, but make changes as new commits to the pull request, and
  leave a comment on the pull request explaining your changes. Specifically, do not squash or rebase commits after they
  are visible on the pull request.
-->
+ 特に代替案と欠点が示されるため、RFCがこのプロセスを変更することは殆どありません。RFCを大小を問わず編集して、設計を明確化または変更でき、プルリクエストへの新しいコミットを行いプルリクエストに変更を説明するコメントを残すことができます。具体的には、プルリクエストでコミットが表示され後にコミットをsquashしたりrebaseしたりしないでください。
<!--
+ At some point, a member of the subteam will propose a "motion for final comment period" (FCP), along with a
  disposition for the RFC (merge, close, or postpone).
    + This step is taken when enough of the tradeoffs have been discussed that the subteam is in a position to make a
      decision. That does not require consensus amongst all participants in the RFC thread (which is usually
      impossible). However, the argument supporting the disposition on the RFC needs to have already been clearly
      articulated, and there should not be a strong consensus against that position outside of the subteam.
      Subteam members use their best judgment in taking this step, and the FCP itself ensures there is ample time and
      notification for stakeholders to push back if it is made prematurely.
    + For RFCs with lengthy discussion, the motion to FCP is usually preceded by a summary comment trying to lay out the
      current state of the discussion and major tradeoffs/points of disagreement.
    + Before actually entering FCP, all members of the subteam must sign off; this is often the point at which many
      subteam members first review the RFC in full depth.
-->
+ ある時点で、サブチームのメンバーは、RFCの処置（merge、close、または延期）とともに「最終コメント期間の申し立て」（FCP）を提案します。
    + このステップは、サブチームが決定を下せる立場にあるという十分なトレードオフが議論されたときに行われます。これはRFCスレッドのすべての参加者間の合意を必要としません（通常は不可能です）。ただしRFCでの処置を支持する議論は既に明確に示されている必要があり、サブチーム外のその位置に対して強いコンセンサスがあるべきではありません。
     サブチームメンバーはこのステップを実行する際に最善の判断を行います。FCP自体は、時期尚早になされた場合に関係者がプッシュバックする十分な時間を持って通知されることを保証します。
    + 長い議論のあるRFCの場合、FCPへの動議は通常、議論の現在の状態と主要なトレードオフや意見の相違点を説明しようとする要約コメントが先行します。
    + 実際にFCPに入る前に、サブチームのすべてのメンバーがサインオフする必要があります。多くの場合、これは多くのサブチームメンバーが最初にRFCを完全に精査する時点です。
<!--
+ The FCP lasts ten calendar days, so that it is open for at least 5 business days. It is also advertised widely, e.g.
  on discord or in a blog post. This way all stakeholders have a chance to lodge any final objections before a decision
  is reached.
-->
+ RFCは１０暦日続くため、discordまたはブログの投稿によって、最低５営業日は公開され広く宣伝されています。このようにして、すべての利害関係者は、決定が下される前に最終的な意義を申し立てる機会があります。
<!--
+ In most cases, the FCP period is quiet, and the RFC is either merged or closed. However, sometimes substantial new
  arguments or ideas are raised, the FCP is canceled, and the RFC goes back into development mode.
-->
+ 殆どの場合、FCP期間は静かで、RFCはmergeまたはcloseされます。ただし、実質的な新しい議論やアイデアが提起され、FCPがキャンセルされ、RFCが開発モードに戻ることもあります。

## The RFC life-cycle
<!--
Once an RFC becomes active then authors may implement it and submit the feature as a pull request to the repo. Being
"active" is not a rubber stamp and in particular still does not mean the feature will ultimately be merged. It does mean
that in principle all the major stakeholders have agreed to the feature and are amenable to merging it.
-->
RFCがアクティブになると、作成者はそれを実装し、機能をプルリクエストとしてリポジトリに送信できます。「アクティブ」はスタンプではなく、特に機能が最終的にマージされることを意味しません。それは原則として、すべての主要な利害関係者鉾の機能に同意しており、その機能をマージすることができることを意味します。

<!--
Furthermore, the fact that a given RFC has been accepted and is "active" implies nothing about what priority is assigned
to its implementation, nor does it imply anything about whether a developer has been assigned the task of implementing
the feature. While it is not necessary that the author of the RFC also write the implementation, it is by far the most
effective way to see an RFC through to completion. Authors should not expect that other project developers will take on
responsibility for implementing their accepted feature.
-->
さらに、特定のRFCが受け入れられ、「アクティブ」であるという事実は、どの優先順位が割り当てられているかについては何も意味しません。その機能を実装するように開発者が実装のタスクを割り当てられているかどうかについても意味しません。RFCの作成者も実装を記述する必要はありませんがこれは最もRFCを最後まで確認する効果的な方法です。作成者は、受け入れられた機能を他のプロジェクト開発者が引き継ぐことを期待するべきではありません。

<!--
Modifications to active RFCs can be done in follow up PRs. We strive to write each RFC in a manner that it will reflect
the final design of the feature, however, the nature of the process means that we cannot expect every merged RFC to
actually reflect what the end result will be at the time of the next major release. We therefore try to keep each RFC
document somewhat in sync with the network feature as planned, tracking such changes via followup pull requests to the
document.
-->
アクティブなRFCの変更は、フォローアップPRで行うことができます。機能の最終的な設計を反映するように各RFCを作成するよう努めていますが、プロセスの性質上、マージされたすべてのRFCが次の時点でのメジャーリリースの最終結果を実際に反映するとは期待できません。したがって、私たちは各RFCドキュメントを計画通りにネットワーク機能といくらか同期させて、フォローアッププルリクエストを介して変更を追跡します。

<!--
An RFC that makes it through the entire process to implementation is considered "implemented" and is moved to the
"implemented" folder. An RFC that fails after becoming active is "rejected" and moves to the "rejected" folder.
-->
プロセス全体から実装に至るまでのRFCは、「実装済み」とみなされ、「implemented」フォルダに移動されます。アクティブになった後に失敗したRFCは「拒否」され、「rejected」フォルダに移動します。

## Reviewing RFCs
<!--
While the RFC pull request is up, the sub-team may schedule meetings with the author and/or relevant stakeholders to
discuss the issues in greater detail, and in some cases the topic may be discussed at a sub-team meeting. In either case
a summary from the meeting will be posted back to the RFC pull request.
-->
RFCプルリクエストがアップしている間、サブチームは作成者や関連する利害関係者とのミーティングをスケジュールして、問題をより詳細に話し合う場合があります。場合によっては、サブチームミーティングでトピックを話し合うこともあります。どちらの場合も、会議の要約がRFCプルリクエストに投稿されます。

<!--
A sub-team makes final decisions about RFCs after the benefits and drawbacks are well understood. These decisions can be
made at any time, but the sub-team will regularly issue decisions. When a decision is made, the RFC pull request will
either be merged or closed. In either case, if the reasoning is not clear from the discussion in thread, the sub-team
will add a comment describing the rationale for the decision.
-->
サブチームは、利点と欠点がよく理解されたあとで、RFCに関する最終決定を行います、これらの決定はいつでも行うことができますが、サブチームは定期的に決定を行います。決定がくだされると、RFCプルリクエストはmergeまたはcloseされます。いずれの場合も、スレッドでの議論から理由が明確でない場合、サブチームは決定の理由を説明するコメントを追加します。

## Implementing an RFC
<!--
Some accepted RFCs represent vital features that need to be implemented right away. Other accepted RFCs can represent
features that can wait until some arbitrary developer feels like doing the work. Every accepted RFC has an associated
issue tracking its implementation in the affected repositories. Therefore, the associated issue can be assigned a
priority via the triage process that the team uses for all issues in the appropriate repositories.
-->
いくつかの受け入れられたRFCは、すぐに実装する必要がある重要な機能を表しています。他の受け入れられたRFCは、任意の開発者が作業したいと感じるまで待つことができる機能を表せます。承認されたすべてのRFCには、影響を受けるリポジトリでの実装を追跡する関連する問題があります。したがって、関連する問題には、チームが適切なリポジトリ内のすべての問題に使用するトリアージプロセスを介して優先順位を割り当てることができます。

<!--
The author of an RFC is not obligated to implement it. Of course, the RFC author (like any other developer) is welcome
to post an implementation for review after the RFC has been accepted.
-->
RFCの作成者は、それを実装する義務はありません。もちろん、RFCの作成者（他の開発者と同様）は、RFCが承認された後、レビューのために実装を投稿できます。

<!--
If you are interested in working on the implementation for an "active" RFC, but cannot determine if someone else is
already working on it, feel free to ask (e.g. by leaving a comment on the associated issue).
-->
「アクティブ」なRFCの実装に取り組みたいが、他の誰かが担当しているかどうかわからない場合、すでに取り組んでいるので、気軽に質問してください（例えば、関連する問題にコメントを残すなど）。

## RFC postponement
<!--
Some RFC pull requests are tagged with the "postponed" label when they are closed (as part of the rejection process).
An RFC closed with "postponed" is marked as such because we want neither to think about evaluating the proposal nor
about implementing the described feature until some time in the future, and we believe that we can afford to wait until
then to do so. Historically, "postponed" was used to postpone features until after 1.0. Postponed pull requests may be
re-opened when the time is right. We don't have any formal process for that, you should ask members of the relevant
sub-team.
-->
一部のRFCプルリクエストは、（拒否プロセスの一部として）closeされるときに「postponed」ラベルでタグ付けされます。
「postponed」で閉じられたRFCは、延期にマークされています、提案を評価することも、記述された機能を将来的に実装することも考えたくないため、それまで待つことができると考えています。歴史的には、「postponed」は1.0以降までは機能を延期するために使用されていました。延期されたプルリクエストは適切なタイミングで再開される場合があります。そのため正式なプロセスはありません。関連するサブチームのメンバーに尋ねてください。

<!--
Usually an RFC pull request marked as "postponed" has already passed an informal first round of evaluation, namely the
round of "do we think we would ever possibly consider making this change, as outlined in the RFC pull request, or some
semi-obvious variation of it." (When the answer to the latter question is "no", then the appropriate response is to
close the RFC, not postpone it.)
-->
通常、「postponed」としてマークされたRFCプルリクエストは、非公式の第一ラウンドの評価、つまり「RFCプルリクエストで、この変更を検討する可能性があるか、またはある程度明確なバリエーションであるか概説されます。」（後者の質問への回答が「いいえ」の場合、適切な応答はRFCを延期するのではなく、closeすることになります）。

## Help! This is all too informal
<!--
The process is intended to be as lightweight as reasonable for the present circumstances. As usual, we are trying to let
the process be driven by consensus and community norms, not impose more structure than necessary.
-->
このプロセスは、現在の状況に適した程度に軽量であることを目的としています。いつものように、私たちはプロセスをコンセンサスとコミュニティの規範によって推進させ、必要以上の構造を課さないようにしています。

# Contributions, license, copyright

This project is licensed under Apache License, Version 2.0, ([LICENSE-APACHE] or
http://www.apache.org/licenses/LICENSE-2.0). Any contribution intentionally submitted for inclusion in the work by you,
as defined in the Apache-2.0 license, shall be licensed as above, without any additional terms or conditions.

[maidsafe's RFC process]: https://github.com/maidsafe/rfcs
[LICENSE-APACHE]: https://github.com/iotaledger/bee-rfcs/blob/master/LICENSE-APACHE
[Rust RFC repository]: https://github.com/rust-lang/rfcs
