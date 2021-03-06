---
title:  連結とエンコード
description:  メッセージはバイト長に応じて複数の SMS として送信されるかどうかが決まります。
navigation_weight:  2

---


連結とエンコード
========

メッセージを SMS で受信/送信する場合、全体の長さに注意する必要があります。メッセージの長さに応じて単独または複数に分けて送信されるかどうかが決まるためです。SMS で使用されるエンコード方式 (`text`または`unicode`) に応じて単一メッセージに収まる最大文字数は変わります。

このドキュメントでは次のトピックを取り上げます。

* [連結](#concatenation)
* [エンコード](#encoding) 
  * [GSM 文字セット](#gsm-character-sets)
  * [Unicode](#unicode)
  * [エンコードの例](#encoding-examples)

* [最大文字数](#maximum-number-of-characters)
* [着信メッセージ](#inbound-messages)

連結
---

最大文字数を超えるメッセージを送信する場合、Nexmo は *連結 SMS* を送信します。連結 SMS は複数のパーツで構成され、各パーツは[ユーザーデータヘッダ (UDH)](https://en.wikipedia.org/wiki/User_Data_Header) のセグメント情報で連結されます。

このセグメント情報によって連結 SMS の構成パーツの数とメッセージ内の各構成パーツの位置が携帯電話に伝えられます。携帯電話が受信したすべての構成パーツは、受信者には単一テキストのように表示されます。

[複数のパートで構成される SMS](https://help.nexmo.com/hc/en-us/articles/204014833-How-is-a-Multipart-SMS-Constructed-) の詳細については、Nexmo ナレッジベースを参照してください。

エンコード
-----

Nexmo の SMS 用 API がサポートする主なエンコード方式は`text`と`unicode`の2つです。

リクエストで`type`パラメーターを設定して、エンコード方式を指定します。どのエンコード方式を使用するかはメッセージに含まれる文字に応じて決まります。

* メッセージに含まれるのが [GSM 標準文字と拡張文字セット](#gsm-character-sets)の文字のみである場合、`type`パラメーターを`text`に設定します。
* [GSM 文字セット以外](#unicode)の文字 (中国語、日本語、韓国語の文字など) がメッセージに含まれる場合は、`type`パラメーターを`unicode`に設定します。

### GSM 文字セット

Nexmo はすべての標準 GSM 文字および GSM 拡張テーブルの掲載文字をサポートしています。標準テーブルの掲載文字はエンコードする文字あたり 7 ビット長が必要です。

```text
! " # $ % ' ( ) * + , - . / : ; < = > ? @ _ ¡ £ ¥ § ¿ & ¤
0 1 2 3 4 5 6 7 8 9
A B C D E F G H I J K L M N O P Q R S T U V W X Y Z
a b c d e f g h i j k l m n o p q r s t u v w x y z
Ä Å Æ Ç É Ñ Ø ø Ü ß Ö à ä å æ è é ì ñ ò ö ù ü Δ Φ Γ Λ Ω Π Ψ Σ Θ Ξ
```

[拡張テーブル](https://en.wikipedia.org/wiki/GSM_03.38#GSM_7-bit_default_alphabet_and_extension_table_of_3GPP_TS_23.038_/_GSM_03.38)の掲載文字をエンコードするには、`ESC`文字のプレフィックスとそれに続く拡張テーブルの次の必須文字の 2 つの文字が必要です。

```text
| ^ € { } [ ] ~ \
```
### Unicode

アラビア語、中国語、韓国語、日本語、キリル文字を使用する言語などには GSM 標準文字と拡張文字に加え Unicode 文字が必要です。これらの文字には 16 ビットの USC-2 エンコードが必要です。

`type`パラメーターを`unicode`に設定すると、メッセージ内の **すべての** 文字は GSM 標準文字セットに含まれる場合でも UCS-2 を使ってエンコードされます。

### エンコードの例

|      メッセージ      |    タイプ    |            文字あたりバイト数             | 必要な合計バイト数 |  メッセージで使用される文字セット  |
|-----------------|-----------|----------------------------------|-----------|--------------------|
| `Bonjour monde` | `text`    | 1                                | 13        | GSM 標準             |
| `This ^ That`   | `text`    | 1 (`^` は拡張文字セットの掲載文字であるため 2) | 12        | GSM 標準文字と GSM 拡張文字 |
| `こんにちは世界`       | `unicode` | 2 (UCS-2)                  | 14        | Unicode            |

最大文字数
-----

SMS メッセージの最大バイト長は 140 であり、160 標準 GSM 7 ビット文字または 70 UCS-2 16 ビット文字と等しい長さです。このバイト長を超えるメッセージは複数のパートに分割されます。

> **注** : GSM 拡張テーブルの掲載文字はエンコードする文字あたり 2 バイト長が必要です。

`type`の値が`text`であるメッセージを送信している場合は、次の文字制限が適用されます。

| パート | 最大文字数 | 計算 |
| -- | -- | -- |
| 1 | 160 | UDH 無しで 160 文字を利用可能 |
| 2 | 304 | `(160 - 7) * 2 = 306` |
| 3 | 456 | `(160 - 7) * 3 = 459` |
| 4 | 608 | `(160 - 7) * 4 = 612` |

`type`が`unicode`のメッセージを送信している場合は、メッセージの各文字には 2 バイト長が必要です。

Nexmo は最大 3,200 文字の SMS をサポートしますが、すべての通信会社がサポートするとは限らないため、SMS の構成パーツは 6 以下に抑えることをお勧めします。

> 注: 連結 SMS では構成パートごとに料金がかかります。

以下のツールでメッセージテンプレートをテストしましょう。

```partial
source: app/views/tools/concatenation.html
```

着信メッセージ
-------

着信メッセージは SMS 仕様に準拠しています。仮想番号に送信された SMS の長さが単一 SMS に許容される[最大文字数](#maximum-number-of-characters)を超える場合、複数のパートから成る連結メッセージを受信します。

ただし、着信連結 SMS を受信できるのはメッセージの通信に利用される通信会社が連結メッセージをサポートする場合に限ります。通信会社が連結 SMS をサポートしない場合、ごく短時間に同じ番号から送信されたすべての SMS を連結されたものとして扱うのが現実的な対応です。詳細については、[着信 SMS の連結](https://help.nexmo.com/hc/en-us/articles/205704158)に関するナレッジベースの記事を参照してください。

着信 SMSの連結に関する追加情報については、[着信 SMS](inbound-sms) の基本概念ドキュメントを参照してください。

関連情報
====

* [単一 SMS の本文の長さについて](https://help.nexmo.com/hc/en-us/articles/204076866-How-long-is-a-single-SMS-body-)

