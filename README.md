# 鳥名データベースの構築について

[詳しくはこちらをご覧ください](https://www.torir.net/1.1.9-bird_list.html)
## 目的

- <span class="logo">toriR</span>で用いられる鳥名のマスターデータベース(MDB_species)の具体的構築方法と運用方法について検討した．
- <span class="logo">toriR</span>では生物の生態の解明を第一の目的としているので、特に、鳥類とコウモリのデータベースを構築する．
- 鳥類のデータベースとして、日本鳥学会の第７版バードリスト(以下、v7)、第8版(v8)とIOC(12.1c)のリストがあるが、差異を検討する．

## まとめ

- 世界のデータベースとするので学名を基本とする．そのため、IOCを基本とするが日本語データベースは鳥学会の目録(v8)を優先した．
- データベースは２種類あり、
  1. 各言語のデータベースをMDB_species_xxとする．日本語はjp、英語はenなど．日本語のみ日本産コウモリを載せる
  2. voice登録用のデータベースはDB_birdlistであり、MDB_species_xxから自動生成される．

## データベース名：MDB_species

- 日本語のデータベースはv8を英語版のデータベースはIOC12.1を採用した．ただし、v8はパブコメ段階であり、最終発行前である[更新日:2022-05025]
- 和名、学名は変更されうること、亜種を区別したいことが考えられるのでユーザ要望で追加する．リファレンスとバージョンの明記は必要．
- 日本産のコウモリは[コウモリの会](http://www.bscj.net/bunken/index.htm)にあるが世界哺乳類標準和名[日本哺乳類学会](https://www.mammalogy.jp/list/)の学名と必ずしも一致しないので、日本産のコウモリをコウモリの会のリストから抽出し、これに対応する学名を日本哺乳類学会から抽出した．

## データベース名：DB_birdlist

- annotationのキーリスト用でvoice登録用のデータベース．
- MDB_species_xxから自動生成される．
- キーの学名はシノニムを許す．これはvoice登録後に学名変更があった場合に対応できるようにするためである．
- voice登録（正確にはannotationのキーリスト）の学名は版の新しい方を反映する．

---

## 以下資料：

## 付録1. 国内と世界のバードリストの差異の検討

### 目的

- 国内は日本鳥学会のリストがあり、世界的にはIOCのリストがある．これら２つの整合性を検証する．
- また備忘録的にその処理方法を記す．

### 方法

- 各データベースをダウンロードして、学名と和名、英名の一致している指定内を把握する．

### 前提

- 種名の基礎となるデータベースの構築を記します．基本的にテキスト処理をLinux(MacOSX)のターミナル上で行う．シェルはbash、コマンドは[tukubai](https://pt.usp-lab.com/html/man/)を使う．

```bash
$ bash --version
GNU bash, バージョン 5.1.16(1)-release (x86_64-apple-darwin21.1.0)
```

- 前段階の処理

```bash
mkdir 1-日本鳥類目録改訂第7版 2-IOC 3-birdlist
```

### 結果

#### 国内の標準和名（日本鳥学会のリスト）

1. 日本鳥学会の「日本鳥類目録」改訂第7版`掲載鳥類リスト(Excelファイル）`は[こちら](http://ornithology.jp/katsudo/Publications/checklist7_contents/JPBirdList_7ed_ver1.xls) をダウンロードする．

```bash
$ cd ./1-日本鳥類目録改訂第7版
$ curl http://ornithology.jp/katsudo/Publications/checklist7_contents/JPBirdList_7ed_ver1.xls -o JPBirdList_7ed_ver1.xls	
```

2. XLSファイル(`JPBirdList_7ed_ver1.xls`)をCSVファイルに変換
   - Numbers/Macでxlsを開いてCSVにエクスポートすると、`PartA自然分布種-表1.csv`と`PartB外来種-表1.csv`ができる．
3. 自然分布種の種のみ取り出す．学名（英名）も一単語にするためスペースを`_`に変える．

```bash
$ cat PartA自然分布種-表1.csv | awk -F, '$2 == "種"{print $4"_"$5, $8}' | sort -u > JPv7

```

4. この変換でうまくいかない種をエディタで編集する．６３３種の登録がある

```bash
$ vi JPv7
~~~
Puffinus_bryani オガサワラヒメミズナギドリ
~~~
$ wc JPv7 
  633  1266 24142 JPv7
```

5. 外来種を取り出し４３種の登録がある．

```bash
$ cat PartB外来種-表1.csv |
awk -F, '$2 =="種"{print $4"_"$5, $8}' |
 # 学名でソートし重なりをとる
sort -u  >| alian
	# カワラバト（ドバト）はカワラバトとした．
$ vi allian
---
Columba_livia カワラバト 
---

$ wc alian 
  43   86 1635 alian
```

6. 国内の自然分布にも外来種にも登録がある種:９種

```bash
$ join0 key=1 JPv7 alian 
Ciconia_boyciana コウノトリ
Cygnus_olor コブハクチョウ
Himantopus_himantopus セイタカシギ
Nipponia_nippon トキ
Phasianus_colchicus キジ
Pica_pica カササギ
Streptopelia_decaocto シラコバト
Syrmaticus_soemmerringii ヤマドリ
Zosterops_japonicus メジロ
```

7. 国内鳥類リストを作り667種の登録がある．

```bash
$ cat JPv7 alian |
sort -u >| JPv7_all
$ wc JPv7_all 
  667  1334 25490 JPv7_all
```

#### 国内の標準和名（バードリサーチのリスト）

8. バードリサーチのリストと比較する

```bash
$ cd ./3-birdlist
$ curl https://www.bird-research.jp/1_shiryo/BR_listmake201703.xls -o BR_listmake201703.xls
```

9. CSVに変換

```bash
$ ls
BR_listmake201703.xls  第6版-表1.csv  第7版-表1.csv  使い方-表1.csv  環境省RL-表1.csv  完成見本1-表1.csv  完成見本2-表1.csv  完成見本3-表1.csv	リスト作成-表1.csv  種の保存法2017-表1.csv  文化財保護法-表1.csv
```

10. `第7版-表1.csv`を学名、種名の順に直す．なお、Numbersの吐き出すCSVの行末コードがWindowsの(CR/LF)になっているのでいったんUNIXのLFに変換してから処理している．667種ある

```bash
	#　行末コードの変換
nkf -Lu  第7版-表1.csv |
  # スペースを_に変換して一単語に繋げる
sed -e 's| |_|g' |
  # カラム２のIDが数字のみ出力する
awk -F, '$2 ~ /^[0-9]+$/{print  $5, $1'}  |
  # 学名でソートし重なりをとる
sort -u >| BR
$ wc BR
  667  1334 25490 BR
```

11. 鳥学会から得たものと同じか確認．出力がないので同じであった．

```bash
diff 3-birdlist/BR 1-日本鳥類目録改訂第7版/JPv7_all 
```

##### ここまでのまとめ

- 国内種：667種
- 鳥学会（JP_birdlist_v7）とバードリサーチから入手したリスト（第7版-表1.csv）の変換結果は、外来種を含めて重なりを取ることで完全に一致した．



#### 世界（IOCバードリスト）

1. IOCの最新の多言語のバードリスト`Multilingual Version (v12.1c, Excel file XLSX, 7.8Mb)`をダウンロードする．

```bash
cd ./2-IOC
curl https://worldbirdnames.org/Multiling%20IOC%2012.1c.xlsx -o IOCv12-1.xls
```

2. XLSファイル(`IOCv12-1.xls`)をCSVファイルに変換
   - Numbers/Macでxlsを開いてCSVにエクスポート`List-表1.csv`と`Sources-表1.csv`ができる．

```bash
open IOCv12-1.xls
```

3. 学名、英名、和名を抜き出す．全登録数11088種．

```bash
 wc List-表1.csv 
  11088  243562 6554330 List-表1.csv
  
cat List-表1.csv |
  # スペースを"_"に変えておく（一単語にする）
sed -e 's| |_|g' |
  # 学名、英名、和名として抜き出す．ヘッダを取っておく．先頭が数字でなければヘッダである．
awk -F, '/^[0-9]/{print $4,$5,$17}' |
  # 和名がなければ"-"とする
awk '{if(NF == 2){print $1, $2,"-"}else{print $1, $2,$3}}' |
  # 学名でソートしておく
sort > IOC

wc IOC
 11088  33264 746519 IOC
 
  # ヘッダを消したので一行少ないはずであるがファイル末に改行がない問題だと思われる．awkで見てみると確かに一行多くあり正常に動いていることがわかった．
 cat List-表1.csv |awk '{i++}END{print i}'
11089
```

##### IOC12.1cのまとめ

- 学名がついているもの:11088種

```bash
wc IOC
 11088  33264 746519 IOC
```

- 和名がついているもの:10799種

```bash
$ cat IOC  | awk '$3 != "-"' | wc
  10799   32397  734274
```

- 和名がついていないもの：289種

```bash
$ cat IOC  | awk '$3 == "-"' | wc
    289     867   12245
```



##### ここまでのディレクトリ構成

```
$ tree
.
├── 1-日本鳥類目録改訂第7版
│   ├── JPBirdList_7ed_ver1.xls
│   ├── JPv7
│   ├── JPv7_all
│   ├── PartA自然分布種-表1.csv
│   ├── PartB外来種-表1.csv
│   └── alian
├── 2-IOC
│   ├── IOC
│   ├── IOCv12-1.xls
│   ├── List-表1.csv
│   └── Sources-表1.csv
├── 3-birdlist
│   ├── BR
│   ├── BR_listmake201703.xls
│   ├── 第6版-表1.csv
│   ├── 第7版-表1.csv
│   ├── 使い方-表1.csv
│   ├── 環境省RL-表1.csv
│   ├── 完成見本1-表1.csv
│   ├── 完成見本2-表1.csv
│   ├── 完成見本3-表1.csv
│   ├── リスト作成-表1.csv
│   ├── 種の保存法2017-表1.csv
│   └── 文化財保護法-表1.csv

```



### 結果）IOCの和名とJPv7の標準和名との差

- v7とIOCで学名が同じもの：570種

```bash
$ join0 +ng key=1 2-IOC/IOC  1-日本鳥類目録改訂第7版/JPv7_all 2>| b | wc
    570    1140   21686
```

- v7の学名がIOCにないもの：９７種

```bash
$ cat b| wc
     97     194    3804
```

- 学名が同じでv7とIOC12.1と同じ和名の種：546（外来種を含む）、これらは英名もないことになる．

```bash
  # カラム１同士を比較しJPv7_allの行にIOCの行を加える
join1 key=1 1-日本鳥類目録改訂第7版/JPv7_all 2-IOC/IOC | 
awk '$2 == $4' |
wc
    546    2184   40877
```

- 学名が同じでv7とIOC12.1と異なる和名の種：24（外来種を含む）

```bash
$ join1 key=1 1-日本鳥類目録改訂第7版/JPv7_all 2-IOC/IOC | awk '$2 != $4' | wc
     24      96    1909
```

- これらはIOCとv7を統合すると同じ学名から２つの和名が出てくることを意味する．その内訳を以下に示す．

- #### 表　学名が同じでv7とIOC12.1と異なる和名

|学名|IOC12.1|英名|JPv7|
|-|---|---|-|
|Acridotheres_javanicus|ジャワハッカ|Javan_Myna|モリハッカ|
|Anser_fabalis|ニシヒシクイ|Taiga_Bean_Goose|ヒシクイ|
|Bubulcus_ibis|ニシアマサギ|Western_Cattle_Egret|アマサギ|
|Buteo_buteo|ヨーロッパノスリ|Common_Buzzard|ノスリ|
|Calandrella_brachydactyla|ニシヒメコウテンシ|Greater_Short-toed_Lark|ヒメコウテンシ|
|Caprimulgus_indicus|ジャングルヨタカ|Jungle_Nightjar|ヨタカ|
|Columba_livia|カワラバト|Rock_Dove|カワラバト(ドバト)|
|Lanius_excubitor|ヨーロッパオオモズ|Great_Grey_Shrike|オオモズ|
|Larus_cachinnans|カスピセグロカモメ|Caspian_Gull|キアシセグロカモメ|
|Melanitta_fusca|ヨーロッパビロードキンクロ|Velvet_Scoter|ビロードキンクロ|
|Motacilla_flava|ニシツメナガセキレイ|Western_Yellow_Wagtail|ツメナガセキレイ|
|Ninox_scutulata|フーアアオバズク|Brown_Hawk-Owl|アオバズク|
|Otus_lempiji|スンダオオコノハズク|Sunda_Scops_Owl|オオコノハズク|
|Phylloscopus_plumbeitarsus|フタオビムシクイ|Two-barred_Warbler|ヤナギムシクイ|
|Pica_pica|ユーラシアカササギ|Eurasian_Magpie|カササギ|
|Pterodroma_phaeopygia|ガラパゴスシロハラミズナギドリ|Galapagos_Petrel|ハワイシロハラミズナギドリ|
|Rallus_aquaticus|ヨーロッパクイナ|Water_Rail|クイナ|
|Remiz_pendulinus|ニシツリスガラ|Eurasian_Penduline_Tit|ツリスガラ|
|Riparia_paludicola|アフリカショウドウツバメ|Brown-throated_Martin|タイワンショウドウツバメ|
|Saxicola_torquatus|ニシノビタキ|African_Stonechat|ノビタキ|
|Turdus_merula|ニシクロウタドリ|Common_Blackbird|クロウタドリ|
|Turdus_naumanni|ハチジョウツグミ|Naumann's_Thrush|ツグミ|
|Turdus_ruficollis|ノドアカツグミ|Red-throated_Thrush|ノドグロツグミ|
|Zoothera_dauma|ミナミトラツグミ|Scaly_Thrush|トラツグミ|

- IOC和名とv7の和名の検証

```bash
$ grep モリハッカ 2-IOC/IOC 1-日本鳥類目録改訂第7版/JPv7_all 
2-IOC/IOC:Acridotheres_fuscus Jungle_Myna モリハッカ
1-日本鳥類目録改訂第7版/JPv7_all:Acridotheres_javanicus モリハッカ
```

IOCとv7を統合するとこの２４種は学名と和名が一致しないことがわかった．すなわち、IOCとv7をマージすると和名はユニークでなく、２つの学名が出てくることになる．

このため、この部分は見解が分かれている所なので、今日(2022-05-13)現在の国内産は鳥学会のv7に従うこととした．

## 結論

- 利便性向上のため海外種を採録したかったが断念する．国外種は採録しない．
- 国内種は鳥学会v7の標準和名と学名を採用する．
- 和名登録のない種は採用しない．
- 亜種や哺乳類など追加したい場合はユーザに任せる．
- voiceには和名(local name)と学名(science name)が一致したものを載せる．
- 国内産のリストが改定された場合はその時判断する．
- 国際化対応のため、必要に応じて各国対応版をIOCから生成できるようにしておく．すなわちLocal NameとScience Nameを用意する．



## 付録２ MDB_species_jpの生成方法

#### v7にコウモリの追加

- 抽出の方法
  1. 日本産のコウモリの和名を[コウモリの会](http://www.bscj.net/bunken/index.htm)から抽出する(以下、bat)
  2. 標準和名と学名を[日本哺乳類学会](https://www.mammalogy.jp/list/)の世界リスト（以下、ww_mam）からマッチするものを抽出する

```bash
mkdir 4-bat; cd 4-bat
curl http://www.bscj.net/bunken/index.htm -o index.html
cat index.html |nkf -w  > index.utf8
cat  index.utf8 |awk '/<li><a .*\>/,/<\/a>/' |awk -F[　\>\<] '{print  $5}' |sort >| bat
vim bat （学名追加、同名和名を整理）
（vim内でソートする方法．:%!sortとコマンドを打てばいい．その後セーブして終了．）
```

- エディタでbatに学名を追加した

- ２つ和名の登録があるものは世界哺乳類のリストに倣った．すなわち

  - アブラコウモリ（イエコウモリ）→アブラコウモリ
  - キタクビワコウモリ（ヒメホリカワコウモリ）→キタクビワコウモリ
  - コユビナガコウモリ（リュウキュウユビナガコウモリ）→リュウキュウユビナガコウモリ
  - オキナワオオコウモリは絶滅種なので記載されていない

  とした．

```bash
mkdir 5-ww_mam; cd 5-ww_mam
list_20211223.xlsxをダウンロードしてCSVに変換
ls
凡例-表1.csv  世界哺乳類標準和名リスト2021年度版-表1.csv
cat 世界哺乳類標準和名リスト2021年度版-表1.csv |awk -F, '/^[a-zA-Z]/{print $16, $13"_"$15}' | sort > ww_mam
```

- batとww_mamの学名の差は次の通り

```
cd ../4-bat
cjoin1 key=1 ../5-ww_mam/ww_mam  bat | awk '$2 != $3'
```

|和名|ww_mam|bat|
|-|-|-|
|ヒナコウモリ|Vespertilio_sinensis|Vespertilio_superans|
|チチブコウモリ|Barbastella_darjelingensis|Barbastella_leucomelas|
|テングコウモリ|Murina_hilgendorfi|Miniopterus_leucogaster|
|ノレンコウモリ|Myotis_bombinus|Myotis_nattereri|
|アブラコウモリ|Pipistrellus_abramus|Pipistrellu_abramus|
|コテングコウモリ|Murina_ussuriensis|Miniopterus_ussuriensis|
|オオアブラコウモリ|Hypsugo_savii|Pipistrellu_savii|
|モリアブラコウモリ|Pipistrellus_endoi|Pipistrellu_endoi|
|クチバテングコウモリ|Murina_tenebrosa|Miniopterus_tenebrosa|
|ドーベントンコウモリ|Myotis_petax|Myotis_daubentonii|
|オキナワオオコウモリ|Pteropus_loochoensis|Pteropus_loochoensis　|
|リュウキュウテングコウモリ|Murina_ryukyuana|Miniopterus_ryukyuana|
|オキナワコキクガシラコウモリ|Rhinolophus_pusillus|Rhinolophus_pumilus|

- コウモリの標準和名と学名(bat_ww_mam)

```bash
cjoin2 key=1 ../5-ww_mam/ww_mam  bat | self 1 3 > bat_ww_mam
```



### データベースのマージ

- 鳥類とコウモリをマージする

```bash
	# 鳥類に付加データを追加
	# コウモリに付加データを追加
	# 並びを学名、和名とする
 	# 登録者名と参照文献とEPOCHを足しておく
self 2 1 4-bat/bat_ww_mam | sort -u | awk '{print $1, $2, "jp woodie","日本哺乳類学会世界哺乳類標準和名リスト","v2021",1652687194}' >| bat_w_reference
cat 1-日本鳥類目録改訂第7版/JPv7_all | sort -u | awk '{print $1, $2, "jp woodie","日本鳥学会日本鳥類目録", "v7",1652687194}' >| JPv7_all_w_referencec
cat bat_w_reference JPv7_all_w_referencec | sort -u | awk '{print ++i, $0}' >| MDB_species_jp 
cp MDB_species_jp /var/www/cgi/html/template/SPECIES/
```

## 付録３　MDB_species_enの生成方法

- IOCから学名と英語を取り出す．

```bash
cd 2-IOC/IOCv12-1
cat List-表1.csv | sed -e 's| |_|g' | awk -F, '/^[0-9]/{print $4,$5}' | sort -u > IOC_en
cat IOC_en | awk '{print ++i, $1, $2, "en woodie IOC v12.1c 1652687194"}'  >| MDB_species_en
cp MDB_species_en /var/www/cgi/html/template/SPECIES/

```

## 付録４　MDB_species_jpにv8を反映

- 第８版（パブリックコメント）をダウンロード(`pbc1_list_20210218.xlsx`)
- XLSX→CSV
- 種名変更は１０３種に上った．

```
 cat 目録A-表1.csv  |
 awk -v FPAT='([^,]+)|(\"[^\"]+\")' '$1 =="種名" && $3 == "→" {gsub(" ","_");gsub("\"","");gsub("　", " ");print $2,$3,$5}' | awk '{print $4,$5}'  | sort -u >| JPv8
  
 cjoin2 +* key=3 JPv8 ../MDB_species_jp |
 awk '{if($4 == "*"){print $2, $3, $5, $6, $7,$8,$9}else{print $4,$3,$5,$6,$7,"v8",1653386212}}'|
 sort -u |
 awk '{print ++i, $0}'  >| MDB_species_jp_v8 
 
  cp /var/www/cgi/html/template/SPECIES/MDB_species_jp
```

- エディティングによる 日本鳥学会日本鳥類目録　v8 変更・追加種

```
cp /var/www/cgi/html/template/SPECIES/MDB_species_jp MDB_species_jp_edit
vim MDB_species_jp_edit
~~~~~~
Emberiza spodocephala	シベリアアオジ
セグロミズナギドリ	→	Puffinus bannermani	オガサワラミズナギドリ
Sittiparus olivaceus オリイガラ
ハシブトオオヨシキリ	→	Arundinax aedon	ハシブトヨシキリ
Zoothera dauma	ミナミトラツグミ
Emberiza spodocephala	シベリアアオジ （亜種か種かはっきりしなかったので載せた）
Phasianus colchicus Linnaeus　キジ		タイリクキジ
Ianthocincla cineracea ヒゲガビチョウ
Pterorhinus perspicillatus カオグロガビチョウ
Pterorhinus sannio カオジロガビチョウ
~~~~~~

cp MDB_species_jp_edit /var/www/cgi/html/template/SPECIES/MDB_species_jp
```



以上
