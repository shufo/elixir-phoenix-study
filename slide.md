# Learning Elixir/Phoenix

---

## Elixir とは

- Erlang VM上で動作する言語
- スケーラブルでメンテナンス性が高い
- 関数型言語
- 最新バージョンは1.2.6 (2016/06現在)
- 最初のリリースは2012年のためまだ若い言語
- 開発者はRailsのコア開発者でもあるJosé Valim
- Erlangのいいとこを取り入れつつmixなどのビルドシステムも装備

---

## Erlang とは

- 可用性の高いリアルタイムシステムを作れる
- スケーラブル
- 電話局や銀行、電子商取引などで使われてる
- インスタントメッセージなどにも使われている
- 並列、分散、耐障害性をランタイムレベルでサポート
- 最新バージョンは18
- 30年近くの歴史を持つ言語
- LINE, WhatsApp, Facebook, dwangoなどで採用実績あり

---

## Phoenix とは

- Elixir製のフレームワーク
- MVCパターンで書ける
- RailsやDjangoのようなフレームワークに影響を受けてる

![](https://qiita-image-store.s3.amazonaws.com/0/47983/04b87bc3-0fd2-72c9-c270-f64df3c0f7a2.png)

---

## Elixir 基礎知識

---

## 四則演算

```elixir
iex(1)> 1 + 1
2
iex(2)> 2 - 1
1
iex(3)> 2 * 2
4
iex(4)> 5 / 2
2.5
```

- div, rem

```elixir
iex(6)> div(5, 2)
2
iex(7)> rem(5, 2)
1
```

---

## nil, boolean

```elixir
iex(2)> nil
nil
iex(3)> true
true
iex(4)> false
false
```

---

true/falseの実体はAtom

```elixir
iex(12)> i true
Term
  true
Data type
  Atom
Reference modules
  Atom
iex(13)> i false
Term
  false
Data type
  Atom
Reference modules
  Atom
```

比較してみる

```elixir
iex(23)> true == :true
true
iex(24)> false == :false
true
```

---

## Atom

不変の値。RubyでいうところのSymbol。

```elixir
iex(19)> :ok
:ok
iex(20)> :error
:error
```

---

## Map

Mapの中にはどんな型も入る

```elixir
iex(26)> %{a: 1}
%{a: 1}
iex(27)> i %{a: 1}
Term
  %{a: 1}
Data type
  Map
Reference modules
  Map
```

`%{a: 1}` は `%{:a => 1}` のsyntax sugar

```elixir
iex(30)> %{a: 1} == %{:a => 1}
true
```

---

## List

Listの中にもどんな型も入る

```elixir
iex(28)> [1, 2]
[1, 2]
iex(29)> i [1, 2]
Term
  [1, 2]
Data type
  List
Reference modules
  List
```

---

## Tuple

Tupleの中にもどんな型も入る

```elixir
iex(5)> i {1, 2}
Term
  {1, 2}
Data type
  Tuple
Reference modules
  Tuple
```

よくある返り値

```elixir
{:ok, result}
{:error, reason}
```

---

## 文字列

ダブルクォーテーションだとバイナリストリング

```elixir
iex(33)> i "aaa"
Term
  "aaa"
Data type
  BitString
Byte size
  3
Description
  This is a string: a UTF-8 encoded binary. It's printed surrounded by
  "double quotes" because all UTF-8 encoded codepoints in it are printable.
Raw representation
  <<97, 97, 97>>
Reference modules
  String, :binary
```

---

シングルクォーテーションは文字列のリスト

```elixir
iex(34)> i 'aaa'
Term
  'aaa'
Data type
  List
Description
  This is a list of integers that is printed as a sequence of characters
  delimited by single quotes because all the integers in it represent valid
  ASCII characters. Conventionally, such lists of integers are referred to as
  "char lists" (more precisely, a char list is a list of Unicode codepoints,
  and ASCII is a subset of Unicode).
Raw representation
  [97, 97, 97]
Reference modules
  List
```

---

変数の文字列中での展開

```elixir
iex(1)> a = 123
123
iex(2)> "foo#{a}"
"foo123"
```

---

## パターンマッチング

`=` でパターンマッチング

```elixir
# a と 1 をパターンマッチング。 a に 1 がバインドされる。
iex(9)> a = 1
1

iex(10)> {:ok, res} = {:ok, 2}
{:ok, 2}
iex(11)> res
2
```

- ワイルドカード

```elixir
# _ には何でもマッチするが後から参照出来ないので捨ててもいい値の時使う
iex(19)> {:ok, _} = {:ok, "foobar"}
{:ok, "foobar"}
```

---

## パイプライン

pipe 演算子( `|>` )

```elixir
# 前の式の結果が次の式の第一引数に渡される
5 / 2
|> Float.ceil
|> round

```

```elixir
iex(29)> 1..100 |> Enum.sum
5050
```

---

## Phoenix 基礎知識

---

## コンポーネント(1)

### Endpoint

- すべてのリクエストを処理しRouterに渡す役割を持ってる
- リクエストに適用するためのplug(web開発でよくある処理をまとめたmodule。HTTPサーバ)の基本セットがある
- 指定されたRouterにリクエストを振り分ける

```elixir
plug Example.Router
```

---

### Router

- リクエストをパースし指定されたコントローラー、アクション、必要であればパラメータも渡す
- リソースに対してパスを生成するヘルパーが生成される(`*_path`)
- リクエストを渡すpipelineを定義出来る
- pipelineというのはplugをまとめて特定のRouteに適用させたりするためのグループ化の方法
- Controllers
- リクエストを処理するアクションを定義
- データストアとやりとりしたりそれをviewに渡したりリダイレクトしたりする

例)

```elixir
pipe_through :api
get "/dispatch", DispatcherController, :show
get "/healthcheck", HealthCheckController, :index
post "/auth/register", AuthController, :register
post "/oauth2/token", AuthController, :token
```

---

## コンポーネント(2)

### Views

- Templateをレンダリングする
- プレゼンテーション層として動作
- ヘルパーを定義してTemplateなどで使うことが出来る

```elixir
defmodule Example.Api.UserView do
  use Example.Web, :view
  use JaSerializer.PhoenixView
  def attributes(data, conn) do
    case conn.private.phoenix_action do
      _ -> data
    end
  end
end
```

---

### Templates

- いわゆるテンプレート。Viewを埋め込む。

テンプレートの拡張子は `.eex`。

```elixir
<%= for user <- @users do %>
<p>こんにちは <%= user %> さん</p>
<% end %>
```

---

## コンポーネント(3)

### Channels

- リアルタイムアプリケーションを作るためのsocketを管理してくれる
- Controllerと似ているが、Controllerはpersistentな双方向通信を扱える

```elixir
defmodule Example.RoomChannel do
  use Phoenix.Channel
  
  def join("rooms:" <> room_id, message, socket) do
    {:ok, socket}
  end

  def handle_in("like:send", %{"type" => type}, socket) do
    broadcast! socket, "like:receive", %{type: type}
    {:reply, :ok, socket}
  end
end

```


---

### PubSub

- Channelレイヤーの下にありクライアントにpublishしたtopicをsubscribeさせることが出来る
- サードパーティのPubSub実装を抽象(Redisとか)

---

## コードリーディング

---

## 以上
