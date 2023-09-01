---
title: Sample
paginate: true
marp: true
theme: argent
---

<!-- _class: titlepage -->

# :dmarc-shiro: とはいうけれど話はそう単純ではなくてですね…
## IETF 117 dmarc WG recap
## Ryo Kajiwara, 2023/09/05

----

# 誰？

- 梶原 龍
  - やせいのプログラマ
  - 2021/5に "Email, Messaging, and Self-Sovereign Identity" というトークを研究会でさせていただきました（QRコードはそのスライド）
    - **「SMTPをやめろ」の人**
  - ISOC-JPのOfficerをしています(2020-)

![bg right:40% contain](github-20210528.png)

----

# スライド(のソース)は以下のURLから見れます

`https://github.com/sylph01/20230905-wide-email-bof`


![bg right:40% contain](github-20230905.png)


----

![bg 90%](Screenshot_20230902_060258.png)

<!-- email関係治安悪くね？ -->

----

# agenda @ IETF 117 (1)

- `p=reject` の扱いについて
  - メーリングリストと相性が悪い。どうする？
    - no change
    - ARCをstandards trackにする
    - "domains MUST NOT publish restrictive DMARC policies due to interoperability issues"
    - `p=reject` のnormative requirementsを除外し"Interoperability Considerations"セクションに動かす

----

# agenda @ IETF 117 (2)

- DMARCからSPFを除外する提案について
  - （これは **DMARC内においてSPFを判断に利用するかどうかについてのみ**）
  - どうする？
    - no change
    - DMARC considerationからSPFを除外
    - DMARC record tagを追加し、SPF/DKIM/両方を利用することを指定

----

# quick recap: DMARC (RFC 7489)

- ドメインオーナーはDNSレコードでポリシーを公開する
- メール受信者は `From` アドレスを SPF, DKIM, DMARC ポリシーと照合する
  - SPF/DKIM validation domainが `From` のドメインとalignしていることを確認する
- 受信者はそれをもとにメールの取り扱いを決定できる
- 受信者はドメインオーナーになりすましメールに対して通知できる

<!--
Identifier AlignmentについてはRFC 7489 section 3.1
RFC5322 Fromヘッダが用いられるのはこれがrequired headerであるから。

Envelope FromとHeader Fromの一致取るのってどこに記述あるんだっけ？
-->

----

# ところでDMARCは何を解決するの？

DMARC（をはじめとするドメイン認証技術）は **spoofing problemを解決する** のであって **spam problemを解決するのではない。**

これは常時HTTPS化がphishingを解決するのではないのと似ている。phishing siteはHTTPS化してくるし、spammerはSPF/DKIM/DMARCすべてをPASSするspamを送ってくる。

"DMARC solves **direct domain abuse**, not spam"

----

# `p=reject` って何

https://datatracker.ietf.org/doc/html/rfc7489#section-6.3

メール受信者に対して要求するポリシーの一つ。

- `none`: 特に受信者に何も要求しない。
- `quarantine`: DMARC FAILした場合に「怪しい」とフラグを立てることを要求する。spamフォルダ行きなど。
- `reject`: DMARC FAILした場合**SMTPトランザクション中にメールをrejectする。**

----

# DMARC FAILしてるのなんてRFC違反でspam確定なんだからrejectしてよくない？

（その表現が正しいかどうかはさておいて…）

2014年にYahoo、ついでAOLがDMARC policyを `p=reject` にしたところメーリングリストが多数不通になる事案が発生した。

なおこれはDMARCが **まだInternet-Draftであった頃！**

DMARC WGはこのあとに結成されている。WGの歴史的に抱えるトラウマの一つになっているように見える。

<!--
日本語Googleで'dmarc yahoo p=reject'で検索するとYahoo! Japanの最近の取り組みしか出てこない、なので英語版Wikipediaにある事実の記述を確認した。
https://en.wikipedia.org/wiki/DMARC
-->

----

# 「『`p=reject` でinteroperability problem引いたことない』は『私はタバコを50年吸ってるけどまだ死んでないぞ』」と同じ

----

# DMARCにおけるmailing list problem

----

# ARC

----

# SPFをDMARCから除外する？なんで？

- SPF (as per RFC 7208) は `MAIL FROM` を検証するのに対して
- DKIM (as per RFC 6376) はDKIM signerが秘密鍵を持っているドメインのどれかについてsignする、としか書かれていない
  - 5.1 Determine Whether the Email Should Be Signed and by Whom
- これらがalignしないことがありうる
- メーリングリストとかだと SPF FAIL の DKIM PASS はよくある

----

# 個人的には(1)

- `p=reject` についてno changesはナシ
  - せめてinteroperability considerationsに何か書いておかないと、メーリングリストで事故を起こすことはそんなに自明ではない
  - ちゃんとRFC読んでない誰かに唆されて `p=reject` にして問題起こす、が起こりうる
    - セキュリティチェックシートの指摘で `p=reject` じゃないのなんでですか、みたいな…

----

# 個人的には(2)

- DKIMのほうがsenderについてより強い保証してるのでDKIM PASSを信用すべきと思う。メーリングリストについても送信者の真正性の検証にはDKIMで十分
  - 一方でメーリングリストではDKIM replay attack…ｳｯ頭が…
  - [DKIM Replayについては2023/3にProblem StatementのI-Dが出ている](https://www.ietf.org/id/draft-crocker-dkim-replay-00.html)
- メーリングリスト + DMARCを本格的にやろうと思ったらARCがもっと実装されてる必要があるけど実際にGmailくらいしかやってないよね？
- ~~みんなまともに実装できてないんだしSMTPやめない？~~

----

# 便利リンク集

- [SPF (RFC 7208)](https://datatracker.ietf.org/doc/html/rfc720)
- [DKIM (RFC 6376)](https://datatracker.ietf.org/doc/html/rfc6376)
- [DMARC (RFC 7489)](https://datatracker.ietf.org/doc/html/rfc7489)
- [ARC (RFC 8617)](https://datatracker.ietf.org/doc/html/rfc8617)
- [Experimental Domain-Based Message Authentication, Reporting, and Conformance (DMARC) Extension for Public Suffix Domains (RFC 9091)](https://datatracker.ietf.org/doc/html/rfc9091)