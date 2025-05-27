### オブジェクト指向におけるプロパティとは？
- オブジェクト（インスタンス）が持つ「状態（データ）」を表すものです。
~~~
public class Person
{
    // プロパティ（状態）
    public string Name { get; set; }
    public int Age { get; set; }

    // メソッド（振る舞い）
    public void Greet()
    {
        Console.WriteLine($"こんにちは、私は{Name}、{Age}歳です。");
    }
}

~~~

#### 疑問点
- Q：フィールドを直接参照することは防げますが、結局setterを呼び出せば値を書き換えられるので意味ないのでは？

- A：setterは「無制限に書き換えさせるため」ではなく、「制限付きで安全に変更させるため」に存在します。
  - フィールドに直接アクセスされると：どんな値でも、ルール無視で入れられる（防げない）
  - setter 経由にすれば：バリデーション（制限）が可能

- **setterがあるから防げるケース** 
~~~
person.Age = -10; // setterがなければこの値が入ってしまう
~~~

- **フィールドを直接公開していたら…** 
~~~
public int Age; // これだと無条件で -10 が入る
~~~

- **setterがある場合…** 
~~~
public int Age {
    set {
        if (value < 0) throw new ArgumentException("年齢は0以上でなければならない");
        age = value;
    }
}
~~~

- Q：バリデーションが不要な変数にもgetter/setterやプロパティを使う意味は？

- A：たとえ今は単なる代入でも、将来変更に耐えられるようにプロパティ（getter/setter）を使うべきです。これは「将来の設計変更に強くする（＝カプセル化）」という、オブジェクト指向の基本的な考え方です。

- **フィールドを直接公開していた場合（bad）**
~~~
// --- プレイヤークラス ---
public class Player
{
    // フィールドを直接公開してしまっている
    public int Score;
}

// --- ゲームの他の場所 ---
public class Game
{
    private Player player = new Player();

    public void AddScore()
    {
        // 直接フィールドを変更している
        player.Score += 10;

        // あとから仕様追加「スコア音を鳴らす」
        PlayScoreUpdateSound(); // ←手動で追加する必要あり
    }

    public void BonusScore()
    {
        player.Score += 50;

        // あとから仕様追加「スコア音を鳴らす」
        PlayScoreUpdateSound(); // ←また手動で追加しなきゃいけない
    }

    private void PlayScoreUpdateSound()
    {
        Console.WriteLine("ピロン♪（スコア音）");
    }
}

~~~

- **プロパティで実装していればどうなる？（good）**
~~~
// --- プレイヤークラス（安全な設計） ---
public class Player
{
    private int score;

    public int Score
    {
        get => score;
        set
        {
            score = value;
            PlayScoreUpdateSound(); // スコア更新時に音を鳴らす
        }
    }

    private void PlayScoreUpdateSound()
    {
        Console.WriteLine("ピロン♪（スコア音）");
    }
}

// --- ゲームの他の場所 ---
public class Game
{
    private Player player = new Player();

    public void AddScore()
    {
        player.Score = 10; // set が呼ばれ音も鳴る
    }

    public void BonusScore()
    {
        player.Score += 5; // get → 加算 → set が呼ばれ音も鳴る
    }
}
~~~

#### getter/setterを実装するときに考慮する基本原則

- **「何でもかんでも」getter/setterにしない** 
~~~
// データ構造に成り下がっていて「オブジェクトのふるまい」が失われている
public int X { get; set; }
public int Y { get; set; }
~~~

- **「意図を持った命名」と「責任の所在の明確化」** 
  - Age → 書き換えOK？バリデーションは？
  - IsActive → どういう状態を見て活性とする？計算で出せる？
  - Status → ただのラベル？ビジネスルールがある？
  
- ** 設計のベストプラクティス集 **

| 観点                   | ベストプラクティス                                               | 解説                                                  |
| ---------------------- | ---------------------------------------------------------------- | ----------------------------------------------------- |
| **カプセル化**         | フィールドは原則 `private` にして、必要に応じて `get/set` を公開 | 内部状態を外部に漏らさない                            |
| **片方向アクセス**     | 書き込み不可にしたい場合は `get` のみを公開                      | `public int Age { get; }`                             |
| **副作用に注意**       | `get` は原則副作用を持たない                                     | `get` の中でDBアクセスなどはNG                        |
| **バリデーション**     | `set` にチェックを入れる                                         | 例： `if (value < 0) throw ...`                       |
| **命名ルール**         | 状態なら「Is〜」、値なら「Get〜」は避ける（プロパティなら）      | 例： `IsAlive` はOK、`GetAge()` より `Age` の方が自然 |
| **意味のある型を使う** | たとえば「年齢」なら `int` より `Age` 型にする                   | ドメイン駆動設計っぽい話になります                    |

- ** setterは“制御ポイント”と捉える ** 
  - 「setter＝自由に書き込める入口」ではなく「制御の門番として設計する」のが良いです。
 ~~~
private int hp;
public int HP
{
    get => hp;
    set
    {
        hp = Math.Max(0, value); // HPは0未満にならない
        if (hp == 0) Die();      // HPが0なら死亡処理
    }
}
 ~~~

 - ** getterも“計算プロパティ”として活用できる ** 
  - 「setter＝自由に書き込める入口」ではなく「制御の門番として設計する」のが良いです。
 ~~~
public bool IsAlive => HP > 0;
public int TotalScore => baseScore + bonus;
 ~~~


 - ** getter/setterを使わず、メソッドにすべきケース ** 
  
| パターン                                           | 理由                            |
| -------------------------------------------------- | ------------------------------- |
| 値を変更する際に副作用が大きい（例：DB更新、通知） | setterより`UpdateX()`の方が明確 |
| 値の取得が重い（API/DBアクセスなど）               | `GetStatus()`などの方が良い     |
| パラメータ付きで何かする                           | `ApplyDiscount(rate)`など       |

#### 🎓 まとめ：getter/setter設計の心得
- ✅ 単なる「変数アクセス」と思わず、「制御ポイント」と考える
- ✅ カプセル化と拡張性のために設計する
- ✅ 状態を守る/公開する責任を明確にする
- ✅ 呼び出し側が何も知らずとも使いやすい設計を意識する