---
layout: post
title: "ドキュメントメンテナンス大作戦"
date: 2014-07-21 15:27:14 +0900
comments: true
categories: [poem,documentation]
---

![Swagger](https://helloreverb.com/img/xswagger-hero.png.pagespeed.ic.RM2hi3MU7Z.png "ドキュメント自動生成ツールの例。APIを試してみることもできる")

同僚の[まさたん (@ma3tk)](https://twitter.com/ma3tk) の[ドキュメントを常に残し続けることが大切な3つの理由 - anti-good.](http://ma3tk.hateblo.jp/entry/2014/07/21/013940)というブログを読んで、「ドキュメント書くのはいいけどメンテナンスが大変だからその方策も書いておいたほうがいいのでは」と思ったので、自分で書いてみた。


## Abstract

まず結論からだが、「変わりやすいものは管理しやすい場所に置く」というのが大原則になる。

一口にドキュメントといってもいろいろあるので管理方法ごとに3種類に分けられる。


<table class="table compact hover order-column">
<thead>
    <tr>
        <th>#</th>
        <th>ドキュメント</th>
        <th>可変性</th>
        <th>メンテナンス方法</th>
    </tr>
</thead>
<tbody>
    <tr>
        <td>1</td>
        <td>仕様</td>
        <td>◎</td>
        <td>ツールで自動化 + 人の手で補足</td>
    </tr>
    <tr>
        <td>2</td>
        <td>コードの意図</td>
        <td>◎</td>
        <td>ソースコード中のコメントに書く</td>
    </tr>
    <tr>
        <td>3</td>
        <td>設計思想</td>
        <td>×</td>
        <td>README, ブログ, ドキュメントシステム, ...etc</td>
    </tr>
</tbody>
</table>


以下、それぞれのメンテナンス方法を掘り下げていく。


## 1. 仕様のドキュメントは自動化する

まず1つ目の仕様を書きまとめたドキュメントの場合はできるだけツールを使って自動生成して、人間がそれに補足する体制がいいと思う。

APIの場合は冒頭にキャプチャを掲げたSwaggerなど既存のツールがそろっている。

* テストケースから自動生成できるツール http://blog.inouetakuya.info/entry/2013/10/20/132928
* ソースコード中のコメントから自動生成するツール https://helloreverb.com/developers/swagger

APIでなく、アプリケーションの機能としての画面単位などでの仕様をドキュメントにまとめたい場合はE2Eのテストと連携させるのがよさそうだ。

この種のドキュメントを欲しがるのがプロダクトマネジャの人だと勝手に推測するんだけど、彼彼女らに [Turnip のテスト](http://magazine.rubyist.net/?0042-FromCucumberToTurnip#l44)などを読めるようになってもらうという手がある。

例えばテストはこんな感じになっているが、非プログラマでも「書けなくても読むことはできる」ようになれそうという気がしないだろうか。

```ruby
# encoding: utf-8
# language: ja
機能: ポータル画面からログイン
　シナリオ: トップページにアクセスしてログインする
　　前提 hoge サイトにアクセスする
　　もしトップページを表示する
　　ならば 画面にようこそと表示されていること
　　かつ id と password を入力する
　　かつ サインインボタンをクリックする
　　ならば 画面にユーザ名 testuser が表示されること
```

```ruby
# encoding: utf-8

step 'hoge サイトにアクセスする' do
  Capybara.app_host = "http://www.hoge.jp/"
end

step 'トップページを表示する' do
  visit '/'
end

step '画面にようこそと表示されていること' do
#  page.should have_content('ようこそ') # should はもう古い
  expect(page).to have_content('ようこそ')
end

step 'id と password を入力する' do
  fill_in 'session_login', :with => 'testuser'
  fill_in 'session_password', :with => 'password'
end

step 'サインインボタンをクリックする'
  click_button 'サインイン'
end

step "画面にユーザ名 :user が表示されること" do |user|
  expect(page).to have_content(user)
end
```


## 2. コードの意図の説明はソースコードのコメントに書く

次にコードを書いた人に質問しなければ分からないようなことを減らすためのドキュメントだが、これはソースコード中のコメントに書けばいい。いわゆる「コメントはコードの意図を説明するべき」というやつ。

例えば、どうして中東地域のユーザーの場合だけこのチェックメソッドが呼び出されているのかとか、このキャッシュが3600秒になっている理由はなぜなのかとか。

実際の現場では先述の仕様のドキュメントがしっかりと作られていない場合もあると思うけど、そういう場合であればビジネスロジックレベルのことまでコメントに書いてしまっていいと思う。

※ソースコードのコメントについては『[リーダブルコード](http://www.amazon.co.jp/gp/product/4873115655?&tag=m0b55-22)』や『[Code Complete](http://www.amazon.co.jp/gp/product/B00JEYPPOE?tag=m0b55-22)』にも詳しく書いてあったはず



## 3. 設計思想を説明するドキュメント

設計思想というと大げさだが、巨大ライブラリから1回きりの作業手順書まで対象はいろいろ。共通の特徴としては「1度書かれるとあまり変更されないもの」。

どういう要件があってこのライブラリは作られたのか、歴史的経緯や問題解決に至る経緯、その他ポエムなどもこの範疇に当てはまると思う。

これはREADMEに書いてもいいし、ブログに書くのもいいし、Wikiでもいいし、社内ドキュメントシステムでもいい。

メンテナブルなドキュメントにするポイントは、変更が多くなるであろう細かな仕様やコードに関する話は別途自動生成のドキュメントやソースコード中のコメントに委譲してしまうこと。





