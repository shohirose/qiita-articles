<!--
title:   OpenFOAMのコーディングスタイルおよびエディタのカスタマイズ（Emacs＆Vim）
tags:    Emacs,OpenFOAM,Vim,codingRule,foam-extend
id:      8f78e180be0cc7221327
private: false
-->
# OpenFOAMのコーディングスタイル

[The OpenFOAM Foundation: Coding Style Guide](http://openfoam.org/dev/coding-style-guide/)

# OpenFOAMのコーディングスタイルへエディタを対応させる

## XEmacs/Emacs

[OpenFOAM wiki: HowTo xemacsIndentation](https://openfoamwiki.net/index.php/HowTo_xemacsIndentation)

## Vim

### プラグイン

#### vim-OpenFOAM-syntax

Vim用のシンタックスプラグインは
[GitHub: effi/vim-OpenFoam-syntax (Ver 2.1と古い)](https://github.com/effi/vim-OpenFoam-syntax)
[Bitbucket: vimExtensionOpenFOAM (Ver 4.0)](https://bitbucket.org/shor-ty/vimextensionopenfoam/src)
で手に入ります。

GitHubの古いバージョンではSolarized color schemeを使用することを推奨しており、それは
[GitHub: altercation/vim-colors-solarized](https://github.com/altercation/vim-colors-solarized)
から手に入れることができます。しかしBitbucketにある最新版では、foam256 color schemeを使用しているためSolarized color schemeを導入する必要はありません。

vim用プラグイン作成過程でのやり取りはCFD Online Forum
[CFD Online: Home > Forums > OpenFOAM Pre-Processing > vim Addon Highlight for OpenFOAM](http://www.cfd-online.com/Forums/openfoam-pre-processing/99343-vim-addon-highlight-openfoam.html)
に残っています。

#### その他

- [dein.vim（プラグイン管理）](https://github.com/Shougo/dein.vim)
- [neocomplete（オートコンプリート機能）](https://github.com/Shougo/neocomplete.vim)
- [vim-indent-guides（インデント可視化）](https://github.com/nathanaelkane/vim-indent-guides)
- [neosnippet（コード簡略化）](https://github.com/Shougo/neosnippet.vim)
- [Align（テキスト整形）](https://github.com/vim-scripts/Align)
- [surround（括弧・クォート類）](https://github.com/vim-scripts/surround.vim)
- [unite.vim（統合環境）](https://github.com/Shougo/unite.vim)
- [vimfiler（エクスプローラー）](https://github.com/Shougo/vimfiler.vim)

[GitHub vim-scripts](https://github.com/vim-scripts?tab=repositories)を探せば便利そうなプラグインが色々見つかります。

### 基本設定

OpenFOAM用に`$HOME/.vimrc`に記述すると便利な基本設定は以下の通りです。

```vim:.vimrc
" 編集設定
set expandtab           " タブをスペースに変換（OpenFOAMはタブ使用不可のため）
set tabstop=4           " タブ幅=スペース4個分（OpenFOAMコーディングスタイルガイド参照）
set shiftwidth=4        " シフトオペレータ（>>）やautoindentで挿入される量
set softtabstop=0       " タブを押した時にスペース何個分カーソルが進むか（tabstopよりも優先順位が高い）。0に設定するとtabstopの値になる。
set shiftround          " '<'や'>'でインデントする際に'shiftwidth'の倍数に丸める
set infercase           " 補完時に大文字小文字を区別しない
set showmatch           " 対応する括弧などをハイライト表示する
set matchtime=3         " 対応括弧のハイライト表示を3秒にする（お好みで）
set smartcase           " 検索時に大文字小文字を区別しない
set incsearch           " インクリメンタル検索を有効にする
set autoindent          " 改行時に前行と同じインデントを保つ
set smartindent         " いくつかのC構文を認識し適切にインデントを増減させる
set cindent             "
set whichwrap=b,s,h,l,<,>,[,]  " カーソルを行をまたいで移動 << compatibleが有効になっている場合のみ
set mouse=a             " マウス入力を受け付ける
" コマンドラインモードでTABキーによるファイル名補完を有効にする
set wildmenu wildmode=list:longest,full

" 表示設定
set number              " 行番号の表示
set wrap                " 長いテキストの折り返し
set textwidth=80        " 80文字目で自動的に改行（OpenFOAMは80カラム以内の規定があるため）
set colorcolumn=81      " 81文字目にラインを入れる
set foldlevel=1         " 折りたたむインデントレベルの設定
set foldcolumn=1        " 左端に折りたたみの状況を表示する
set hlsearch            " 検索した文字をハイライト表示
set ruler               " カーソル位置をステータス行に表示
" set list                " 不可視文字の表示
set cmdheight=2         " メッセージ表示欄を2行確保
set laststatus=2        " ステータス行を常に表示
set scrolloff=8         " 上下8行の視界を確保
set sidescrolloff=16    " 左右スクロール時の視界を確保
set sidescroll=1        " 左右スクロールは一文字づつ行う
set wildmenu            " コマンド入力時、ファイル候補を表示
set showcmd             " 入力中のコマンドを表示する
set cusorline           " カーソル行にラインを表示

" 折り畳み設定
set foldmethod=syntax   " コードの折りたたみ方法
set fold column=2       " 左端の折り畳み状況を表示する列数
let g:c_gnu=1             " gcc固有のsyntax
let g:c_comment_string=1  " C言語のコメント用syntax
let g:c_curly_error=1     " C言語の括弧用syntax
```

ちなみに[こちら](http://stackoverflow.com/questions/2447109/showing-a-different-background-colour-in-vim-past-80-characters)によると、下記のようにすれば81カラム以降の色を全て変えることができます。（ただし少しvimが遅くなるのと引き換えです。）

```vim
let &colorcolumn=join(range(81,999),",")
```