# ダイアログの最適解を探る〜WEBアプリ編〜

　HTML,CSS,JSでアプリ作成するときダイアログが欲しい。でも満足いくダイアログを作るのが大変。

<!-- more -->

# ダイアログの必要性

　要約のためにダイアログが必要です。メイン画面には要約したシンプルな情報だけを表示し、ダイアログでその詳細を表示・編集します。これにより見通しのよいアプリになります。

　もしダイアログを使わず、詳細もすべてメイン画面に含めたら、概要を理解するのに時間がかかってしまうでしょう。画面スクロールに手間がかかります。今、何について話しているのか、わかりにくくなるでしょう。

　アプリが一定の規模を超えたら、ダイアログによる要約は是非とも欲しい機能です。

# 実装方法

　ダイアログの実装方法は次の３通りあります。

* [標準API](#api)
* [`<dialog>`](#dialog-element)
* [micromodal.js](#micromodal-js)

[`<dialog>`]:https://developer.mozilla.org/ja/docs/Web/HTML/Element/dialog
[micromodal.js]:https://micromodal.vercel.app/

<a id="api"></a>
# [§](#api) 標準API

　JavaScriptの標準APIでダイアログ表示します。

* [`alert()`][]
* [`confirm()`][]
* [`prompt()`][]

[`alert()`]:https://developer.mozilla.org/ja/docs/Web/API/Window/alert
[`confirm()`]:https://developer.mozilla.org/ja/docs/Web/API/Window/confirm
[`prompt()`]:https://developer.mozilla.org/ja/docs/Web/API/Window/prompt

```javascript
alert('アラートを表示します。OK押下で閉じます。');
```

```javascript
confirm('OKならtrue、キャンセルならfalseを返します。');
```

```javascript
prompt('入力したテキストを返します。', '初期値');
```

　非常にシンプルなダイアログです。残念ながらデザイン設計できません。ブラウザ毎にデザインが異なります。タイトル、ボタン位置、ラベルの変更ができません。コンボボックスなど他のUIを含めることもできません。あまりにシンプルすぎてダサいし使いにくいです。

* タイトルが指定できない（`{domain}の内容`になる）
* ボタン位置を変更できない（デフォルトでフォーカスを`キャンセル`に当てたいけど不可）
* ラベルの変更ができない（`OK`→`YES`, `キャンセル`→`NO`にしたいけど不可）
* 余計なUIが含まれる（`☐ このページでこれ以上ダイアログボックスを生成しない`）

<a id="dialog-element"></a>
# [§](#dialog-element) `<dialog>`

　[<dialog>][]はHTML要素なのでCSSでデザインを変更できます。異なるブラウザでも同じ見た目にできます。また、[<select>][]など任意のUI要素を含めることも可能です。

　ただ実装状況が微妙です。特にFirefoxが98以降でのみ使えます。割と最近に実装されたので、使えない環境の人もまだいるでしょう。

[<select>]:https://developer.mozilla.org/ja/docs/Web/HTML/Element/select

　さらに使ってみたら分かりますが、ダイアログとしての機能を自分で実装する必要があります。閉じるボタンや、ダイアログの外側をクリックしたときに閉じる機能など、最初から欲しい機能がありません。

　以下コードはダイアログを開く／閉じるボタンの機能を実装したものです。冗長ですね。

```html
<script>
window.addEventListener('DOMContentLoaded', async(event) => {
    for (const dialog of document.querySelectorAll(`dialog`)) {
        const closeButton = dialog.querySelector(`header button[data-close]`)
        if (!closeButton) { continue }
        closeButton.addEventListener('click', async(event) => { dialog.close() })
    }
    for (const button of document.querySelectorAll(`button[data-open]`)) {
        const id = button.getAttribute('data-open')
        button.addEventListener('click', async(event) => { document.querySelector(`#${id}`).showModal() })
    }
})
</script>

<button data-open="dialog-1">ダイアログを開く</button>
<dialog id="dialog-1">
  <header><h1 style="display:inline;">ダイアログの表題</h1><button data-close>✖</button></header>
  <main><p>ダイアログの本文</p></main>
</dialog>
```

　ここからさらに、外側をクリックしたら閉じる機能や、CSSでデザイン設計せねばなりません。

<a id="micromodal-js"></a>
# [§](#dialog-element) micromodal.js

　[micromodal.js][]はサードパーティ製ライブラリです。ダイアログとしての機能を最低限だけ実装します。

```html
<style>
.modal {display:none;}
.modal.is-open {display:block;}
</style>
<script src="https://unpkg.com/micromodal/dist/micromodal.min.js"></script>
<script>
window.addEventListener('DOMContentLoaded', async(event) => {
    MicroModal.init();
})
</script>

<button data-micromodal-trigger="modal-1" role="button">ダイアログを開く</button>

<div id="modal-1" aria-hidden="true">
  <div tabindex="-1" data-micromodal-close>
    <div role="dialog" aria-modal="true" aria-labelledby="modal-1-title" >
      <header>
        <h2 id="modal-1-title">ダイアログの見出し</h2>
        <button aria-label="Close modal" data-micromodal-close></button>
      </header>
      <div id="modal-1-content">
        <p>ダイアログの本文。</p>
      </div>
    </div>
  </div>
</div>
```

　[`<dialog>`][]を使わず[`<div>`][]で実装します。なのでFirefox 98以前でも動作します。

[`<div>`]:https://developer.mozilla.org/ja/docs/Web/HTML/Element/div

　以下のようなAPIになります。

```javascript
MicroModal.show('modal-id');
MicroModal.close('modal-id');
```
```javascript
MicroModal.init({
  onShow: modal => console.info(`${modal.id} is shown`),
  onClose: modal => console.info(`${modal.id} is hidden`),
  openTrigger: 'data-custom-open',
  closeTrigger: 'data-custom-close',
  openClass: 'is-open',
  disableScroll: true,
  disableFocus: false,
  awaitOpenAnimation: false,
  awaitCloseAnimation: false,
  debugMode: true
});
```

　特に`disableScroll`は設定したほうが良いでしょう。ダイアログを開いた時にマウスホイールでスクロールしても、背景の画面はスクロールされなくなります。普通はそれを期待するはずです。

```javascript
MicroModal.init({disableScroll:true});
```

　厄介なのはダイアログの開く／閉じる機能をCSSで表現している所です。

```css
.modal {display:none;}
.modal.is-open {display:block;}
```

　アニメーション用のオプションもありますが、CSSで定義していないと動作しません。

　公式ドキュメントの末尾に[CSS例][]へのリンクがありました。

[CSS例]:https://gist.github.com/ghosh/4f94cf497d7090359a5c9f81caf60699

　レスポンシブでない点が気になります。`max-width: 500px;`の所。他にもフォントサイズやパディングなど調整が必要そうです。アニメーションが実装されている所は嬉しいですね。

　結構大変そうですが、現状ベストなダイアログ実装方法です。

# まとめ

　ダイアログの実装方法は次の３通りあります。

* [標準API](#api)
* [`<dialog>`](#dialog-element)
* [micromodal.js](#micromodal-js)

　[micromodal.js](#micromodal-js)の方法が最も美しく高機能で楽に実装できるでしょう。

　（他にも候補はありますがフレームワークに依存しており手が出しにくいです）

