---
layout: post.ja
title: 索引語辞書におけるキャッシュの採用
---
## 索引語辞書におけるキャッシュの採用

索引語の辞書をたくさん引くのは索引を構築するタイミングですが，そのとき必要になるのは単純な参照と追加くらいです．おまけに，最頻出の索引語をほんの少しキャッシュに入れておくだけで，ほとんどのクエリはカバーできます．そうであるならば，「高頻度語を検出するための工夫は必要になるけど，十分なリターンが期待できるのはないか？」というお話です．

written by s-yata.

### 索引構築を速くするために

全文検索用の索引を構築する上で，それなりのウェイトを占める処理の一つが索引語辞書の操作です．具体的には，入力文書から切り出された索引語を
ID
に変換する処理です．入力索引語が索引語辞書に登録されていなければ，新たに追加した後，割り当てた
ID を返します．登録されていれば，以前の追加によって割り当てられている ID
を返します．

索引語辞書の操作を速くするのに重要なことを考えてみます．

1.  まず考えつくのは，索引語辞書のデータ構造を高速なものにすることです．ただし，時間効率の高いデータ構造は空間効率の面で劣るというのが一般的です．検索対象が大規模になれば索引語辞書も大規模になってしまうので，空間効率を度外視することはできません．
2.  次に考慮すべき事項として，参照と追加の比重があります．全文検索システムでは文字
    N-gram
    や形態素を索引語とするのが一般的であり，入力文書から切り出される索引語の出現頻度は偏るので，どうしても参照の比重が大きくなります．つまり，データ構造の選定においては，追加時間より参照時間を重視すべきということです．
3.  また，文書を検索するときの都合により，Common prefix search や
    Predictive search を効率的に実現できることが理想です．
4.  さらに，検索を止めることなく文書を追加するには，ロックフリーな参照を実現する必要があります．

以上のことから，ダブル配列（※）を索引語辞書のデータ構造として採用することを検討しました．しかし，索引語の出現頻度を実際に計数してみると，空間効率・時間効率ともに優れた索引語辞書の構成が見えてきました．結論を述べると，ダブル配列やハッシュ表をキャッシュとして採用するという案になります．計測によって得られた出現頻度の偏りが大きかったため，頻出索引語のみを時間効率の高いデータ構造に格納してキャッシュとし，本体には空間効率の高いデータ構造を用いることに思い至ったというわけです．

※
ダブル配列でロックフリーな参照を実現する方法については，別の記事であらためて解説しようと思います．

### 索引語の出現頻度がどのくらい偏るか

索引語辞書にキャッシュを採用するという案の前提となっているのは，索引語の出現頻度が大きく偏ることです．そして，キャッシュの有効性を認めるに至った実験の結果が以下の表になります．頻出索引語があらかじめ分かっているものと仮定して，何件のキーをキャッシュに格納すればキャッシュヒット率が一定の値に到達するのかを調査しました．

入力文書として用いたのは Wikipedia の記事と Twitter
のつぶやきであり，索引語として試したのは文字 2-gram と MeCab
による分かち書きの結果です．

||_8=. キー数（%）|
||*4=. Wikipedia（日本語 + 英語）|*4=. Twitter（日本語）|
|*=. キャッシュ|*2=. 文字 Bigram + 英単語|*2=. 形態素|*2=. 文字
Bigram + 英単語|_2=. 形態素|
|*=. ヒット率|*=. キー数|*=. |*=. キー数|*=.|*=. キー数|_=. |*=.
キー数|*=.|
|_&gt;. 10%|&gt;. 4|&gt;. 0.000030|&gt;. 3|&gt;. 0.000022|&gt;.
14|&gt;. 0.000121|&gt;. 5|&gt;. 0.000038|
|_&gt;. 20%|&gt;. 10|&gt;. 0.000075|&gt;. 8|&gt;. 0.000058|&gt;.
51|&gt;. 0.000443|&gt;. 12|&gt;. 0.000090|
|_&gt;. 30%|&gt;. 32|&gt;. 0.000241|&gt;. 19|&gt;. 0.000137|&gt;.
138|&gt;. 0.001198|&gt;. 23|&gt;. 0.000173|
|_&gt;. 40%|&gt;. 125|&gt;. 0.000942|&gt;. 49|&gt;. 0.000353|&gt;.
319|&gt;. 0.002768|&gt;. 44|&gt;. 0.000331|
|_&gt;. 50%|&gt;. 441|&gt;. 0.003323|&gt;. 148|&gt;. 0.001067|&gt;.
729|&gt;. 0.006327|&gt;. 108|&gt;. 0.000811|
|_&gt;. 60%|&gt;. 1,350|&gt;. 0.010173|&gt;. 478|&gt;. 0.003447|&gt;.
1,635|&gt;. 0.014189|&gt;. 292|&gt;. 0.002193|
|_&gt;. 70%|&gt;. 3,861|&gt;. 0.029095|&gt;. 1,549|&gt;. 0.011169|&gt;.
3,834|&gt;. 0.033273|&gt;. 921|&gt;. 0.006918|
|_&gt;. 80%|&gt;. 11,412|&gt;. 0.085996|&gt;. 5,163|&gt;.
0.037228|&gt;. 10,030|&gt;. 0.087044|&gt;. 3,285|&gt;. 0.024677|
|_&gt;. 90%|&gt;. 45,219|&gt;. 0.340752|&gt;. 22,867|&gt;.
0.164882|&gt;. 36,247|&gt;. 0.314565|&gt;. 15,888|&gt;. 0.119349|
|_&gt;. 91%|&gt;. 54,215|&gt;. 0.408542|&gt;. 27,767|&gt;.
0.200213|&gt;. 43,018|&gt;. 0.373326|&gt;. 19,481|&gt;. 0.146340|
|_&gt;. 92%|&gt;. 66,059|&gt;. 0.497793|&gt;. 34,321|&gt;.
0.247471|&gt;. 51,827|&gt;. 0.449774|&gt;. 24,284|&gt;. 0.182420|
|_&gt;. 93%|&gt;. 82,213|&gt;. 0.619523|&gt;. 43,428|&gt;.
0.313137|&gt;. 63,663|&gt;. 0.552492|&gt;. 30,946|&gt;. 0.232464|
|_&gt;. 94%|&gt;. 105,287|&gt;. 0.793399|&gt;. 56,646|&gt;.
0.408445|&gt;. 80,116|&gt;. 0.695277|&gt;. 40,619|&gt;. 0.305127|
|_&gt;. 95%|&gt;. 140,105|&gt;. 1.055773|&gt;. 77,469|&gt;.
0.558589|&gt;. 104,166|&gt;. 0.903992|&gt;. 55,395|&gt;. 0.416123|
|_&gt;. 96%|&gt;. 196,848|&gt;. 1.483365|&gt;. 113,521|&gt;.
0.818541|&gt;. 141,727|&gt;. 1.229961|&gt;. 79,754|&gt;. 0.599106|
|_&gt;. 97%|&gt;. 300,097|&gt;. 2.261407|&gt;. 185,761|&gt;.
1.339427|&gt;. 206,542|&gt;. 1.792450|&gt;. 125,518|&gt;. 0.942882|
|_&gt;. 98%|&gt;. 527,496|&gt;. 3.974991|&gt;. 369,337|&gt;.
2.663098|&gt;. 339,842|&gt;. 2.949277|&gt;. 234,680|&gt;. 1.762899|
|_&gt;. 99%|&gt;. 1,294,298|&gt;. 9.753293|&gt;. 1,145,987|&gt;.
8.263120|&gt;. 750,373|&gt;. 6.512021|&gt;. 731,192|&gt;. 5.492659|

<!-- |_>. 100%|>. 13,270,369|>. 100.000000|>. 13,868,696|>. 100.000000|>. 11,522,890|>. 100.000000|>. 13,312,167|>. 100.000000| -->
実験結果を見ると，キャッシュヒット率が 90%
を超えるくらいに調整したとき，キャッシュに含まれる索引語の割合は全体の
1%
にも満たないことが分かります．つまり，空間効率の低いデータ構造をキャッシュとして採用したところで，索引語辞書のサイズにはほとんど影響しません．一方で，時間効率の高いデータ構造を採用すれば，索引構築にかかる時間を大幅に短縮できます．

たとえば，キャッシュヒット率を 90% に調整すると，キャッシュが索引語 1
つあたりに必要とするサイズが本体のそれと比べて 5 倍でも，全体の 5%
にも満たないということです．また，キャッシュの参照時間が本体の 1/5
になると仮定すれば，キャッシュミスしたときはキャッシュと本体の両方を参照することになるものの，1/5
x 90% + 6/5 * 10% = 30% にまで平均参照時間を短縮できることになります．

### 索引語辞書の構成はどうなるか

これまでの内容から，索引語辞書の本体とキャッシュに求められる特徴をまとめてみました．端的に述べると，本体は空間効率重視，キャッシュは時間効率重視ということになります．ただし，本体については
Common prefix search と Predictive search
をサポートすることが求められます．

  ------------------ ------------------------------------------- --------------------------
                     _=. 本体                                   _=. キャッシュ
  _&lt;. 参照時間   多少遅くても大丈夫                          できる限り高速な方が良い
  _&lt;. 追加時間   高速な方が良い                              多少遅くても大丈夫
  _&lt;. 空間効率   高い方が良い                                多少低くても大丈夫
  _&lt;. 拡張検索   Common prefix search と Predictive search   
  _&lt;. 補足事項   頻出索引語の検出                            静的な構築でも大丈夫
  ------------------ ------------------------------------------- --------------------------

入力文書が出揃うまで頻出索引語を正確に求めることはできないため，途中経過から以降の頻出索引語を予測する必要があります．シンプルな実装は，出現頻度が閾値を超えた索引語をキャッシュに追加する，あるいは一定の条件を満たしたときにキャッシュを再構築するというものです．正確な予測はできなくても，それなりの効果を期待できるでしょう．

今少し具体的な，つまり groonga 的な構成を示すとすれば，Bitwise
なパトリシアトライである grn_pat
を本体として，ダブル配列をキャッシュに用いるという構成になります．後は，頻出索引語を検出するために，出現頻度を格納するための領域が必要になるでしょう．

さらなる効率化を目指すのであれば，空間効率重視の静的なデータ構造を導入し，低頻度の索引語を本体から移行するべきかもしれません．運用方法と併せて検討することが肝要です．
