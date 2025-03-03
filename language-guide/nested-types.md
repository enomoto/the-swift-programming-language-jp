# ネスト型\(Nested Types\)

最終更新日: 2023/1/15  
原文: https://docs.swift.org/swift-book/LanguageGuide/NestedTypes.html

別の型のスコープ内に型を定義する。

列挙型は、特定のクラスまたは構造体の機能をサポートするために作成されることがよくあります。同様に、より複雑な型の中で使用するためにユーティリティクラスや構造体を定義すると便利な場合があります。これを実現するために、Swift ではネスト型を定義でき、列挙型、クラス、および構造体を、それらがサポートする型の定義内でネストして定義できます。

型を別の型内にネストするには、サポートする型の外側の中括弧\(`{}`\)内にその定義を記述します。型は必要なだけネストすることができます。

## ネスト型の挙動\(Nested Types in Action\)

下記の例では、`BlackjackCard` という構造体を定義しています。これは、ブラックジャックゲームで使用されるトランプをモデル化しています。`BlackjackCard` 構造体には、`Suit` と `Rank` と呼ばれる 2 つのネスト列挙型が含まれています。

ブラックジャックでは、`Ace` は `1` または `11` です。これは、`Rank` 列挙型内にネストされている `Values` と呼ばれる構造体によって表されます:

```swift
struct BlackjackCard {

    // ネストした Suit 列挙型
    enum Suit: Character {
        case spades = "♠", hearts = "♡", diamonds = "♢", clubs = "♣"
    }

    // ネストした Rank 列挙型
    enum Rank: Int {
        case two = 2, three, four, five, six, seven, eight, nine, ten
        case jack, queen, king, ace
        struct Values {
            let first: Int, second: Int?
        }
        var values: Values {
            switch self {
            case .ace:
                return Values(first: 1, second: 11)
            case .jack, .queen, .king:
                return Values(first: 10, second: nil)
            default:
                return Values(first: self.rawValue, second: nil)
            }
        }
    }

    // BlackjackCard のプロパティとメソッド
    let rank: Rank, suit: Suit
    var description: String {
        var output = "柄は \(suit.rawValue),"
        output += " 数字は \(rank.values.first)"
        if let second = rank.values.second {
            output += " と \(second)"
        }
        return output
    }
}
```

`Suit` の列挙型は、4 つの一般的なトランプの柄と、それらのシンボルを表す `Character` 値を示します。

`Rank` 列挙型は、13 のトランプの数値を、それらの額面通りの値を表す `Int` 値で示します。\(この `Int` 値は、ジャック、クイーン、キング、およびエースのカードには使用されません\)

前述のように、`Rank` 列挙型は、`Values` と呼ばれる独自のさらにネスト構造体を定義しています。この構造体は、ほとんどのカードには値が 1 つだけですが、エースには 2 つの値があることをカプセル化しています。`Values` 構造体は、これを表す 2 つのプロパティを定義します:

* `first` は `Int` 型
* `second` は Int? 型または"オプショナルの Int"

`Rank` は、`Values` 構造体のインスタンスを返す `values` と呼ばれる計算プロパティも定義します。この計算プロパティはカードのランクを考慮し、ランクに基づいて適切な値で新しい `Values` インスタンスを初期化します。`jack`、`queen`、`king`、`ace` には特別な値を使用します。数値カードの場合、ランクの `Int` 値を使用します。

`BlackjackCard` 構造体には、`rank` と `suite` の 2 つのプロパティがあります。また、`description` と呼ばれる計算プロパティも定義します。このプロパティは、`rank` と `suit` に格納されている値を使用して、カードの名前と値を説明します。`description` プロパティは、オプショナルバインディングを使用して、表示する 2 番目の値があるかどうかを確認し、ある場合は、その 2 番目の値を追加の説明に挿入します。

`BlackjackCard` はカスタムイニシャライザを持たない構造体のため、[Memberwise Initializers for Structure Types\(構造体のメンバワイズイニシャライザ\)](../language-guide/initialization.md#initialization-memberwise-initializers-for-structure-types)で説明されているように、暗黙的なメンバワイズイニシャライザがあります。このイニシャライザを使用して、`theAceOfSpades` という新しい定数を初期化できます:

```swift
let theAceOfSpades = BlackjackCard(rank: .ace, suit: .spades)
print("theAceOfSpades: \(theAceOfSpades.description)")
// theAceOfSpades: 柄は ♠, 数字は 1 と 11
```

`Rank` と `Suit` は `BlackjackCard` 内にネストされていますが、それらの型は推論できるため、このインスタンスの初期化では、ケース名\(`.ace` および `.spades`\) のみで列挙ケースを参照できます。上記の例では、`description` プロパティは、スペードのエースが `1` または `11` だと正しく示しています。

## ネスト型への参照\(Referring to Nested Types\)

定義の外部でネスト型を使用するには、その名前の前にネストされている型の名前を付けます:

```swift
let heartsSymbol = BlackjackCard.Suit.hearts.rawValue
// heartsSymbol は "♡"
```

上記の例では、それらが定義されているコンテキストからネスト型を推論できるため、`Suit`、`Rank`、および `Values` の名前を意図的に短くすることができます。

