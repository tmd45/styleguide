# 目次

- [クラスの定義](#Definitions_of_classes)
- [メソッドの定義](#Definitions_of_methods)
- [プロパティの定義](#Property)
- [変数](#Variables)
- [型](#Types)
- [Enum / Options] (#Enum_and_Options)
- [命名] (#Naming)
- [構文] (#Misc)

# Objective-C コーディング規準

## はじめに

本文書は、CookpadにおけるObjective-C コードのスタイル規準を定めるものである。

Appleのスタイルを基本として採用する。理由はAppleのフレームワークを多用するため、Apple スタイルのヘッダファイルを見ることが多いためである。

* [Apple Coding Guidelines](https://developer.apple.com/library/ios/#documentation/Cocoa/Conceptual/CodingGuidelines/CodingGuidelines.html)

Cookpadにおける標準的なスタイルを定めている。また、Swiftへの移行を見越して、複数の選択肢があるときはSwiftに近いスタイルを採用する。なお、「正しいコードを書く」というのは前提なので、そのためのtipsなどは記載しない。

<a id="Definitions_of_classes"></a>
## クラスの定義

- **[MUST]** 公開する必要のないプロパティやメソッドの定義はクラス拡張を使用して実装ファイルに記述する。ヘッダファイルには公開メソッドとプロパティのみを記述すること

```objc
// Bad
// ViewController.h

@interface ViewController : UIViewController
@property (nonatomic, weak) IBOutlet UILabel *label;
@property (nonatomic, weak) IBOutlet UIButton *button;
@property (nonatomic, weak) IBOutlet UITextView *textView;
@end

// Good
// ViewController.m

@interface ViewController ()
@property (nonatomic, weak) IBOutlet UILabel *label;
@property (nonatomic, weak) IBOutlet UIButton *button;
@property (nonatomic, weak) IBOutlet UITextView *textView;
@end

@implementation ViewController
@end
```

- **[MUST]** プロトコルの適用と実装も公開の必要がなければ実装ファイルでクラス拡張により行う
  - もちろん公開の必要があればヘッダファイルに書いてよい

```objc
// Bad
// ViewController.h

@interface ViewController : UIViewController <UITableViewDataSource>
@end

// Good
// ViewController.m

@interface ViewController () <UITableViewDataSource>
@end

@implementation ViewController
@end
```

<a id="Definitions_of_methods"></a>
## メソッドの定義

- **[MUST]** プライベートメソッドには、 `_` プレフィックスを付けてはいけない

```objc
// Bad
- (void)_privateMethod1
{
  // ...
}

// Good
- (void)privateMethod1
{
  // ...
}
```

- **[MUST]** オーバーライドした時に、確実に親クラスのメソッドを呼んでほしいメソッドには `NS_REQUIRES_SUPER` をつけること

<a id="Property"></a>
## プロパティの定義

- **[MUST]** プロパティの属性として `nonatomic` をつけること
  - atomicによってスレッドセーフになるのは実質的に構造体のみと考えられるため
  - またatomicであってもそのクラス自体がスレッドセーフになるわけではない
- **[MUST]** `atomic` をつける妥当な理由がある場合は、それをコメントに書いたうえで使うこと
- **[MUST]** プロパティの属性として `weak` を使える場所では使うこと

```objc
// Bad
@property (nonatomic) id delegate;
@property (nonatomic) IBOutlet UILabel *label;

// Good
@property (nonatomic, weak) id<FooDelegate> delegate;
@property (nonatomic, weak) IBOutlet UILabel *label;
```

- **[MUST]** 代入が必要ないプロパティはreadonlyにすること
 - できるだけ副作用の少ないプログラムにするため

```objc
// Bad
@interface OriginalView
@property (nonatomic, weak) UILabel *label; // 例えば、このlabelのtextは変更したいがlabel自体は変更しないケース
@end

// Good
@interface OriginalView
@property (nonatomic, readonly) UILabel *label;
@end
```

- **[MUST]** `assign`, `strong` は冗長なので指定しないこと
- **[MUST]** `NSString`, `NSArray`, `NSDictionary`, `NSSet` のプロパティは `copy` を指定すること
  - 実体がimmutableなオブジェクトであればメモリ割り当てのコストは発生しないし、mutableなオブジェクトであれば安全のためにcopyすべきであるため
  - その他、必要に応じて `NSCopying` プロトコルに準拠するクラスには `copy` を指定すること
- **[MUST]** 原則としてインスタンス変数は宣言せず、必要なプロパティは `@property` として宣言すること
  - インスタンス変数名でアクセスするのはゲッター、セッター、イニシャライズメソッドでのみにすること

```objc
// Bad
@interface ViewController () <UITableViewDataSource>
{
    NSString *_recipeID;
}
@end

// Good
@interface ViewController () <UITableViewDataSource>
@property (nonatomic, copy) NSString *recipeID;
@end

```

- **[MUST]** プロパティへのアクセスはメソッド呼び出しではなくドット記法を使うこと

```objc
// Bad
[view setBackgroundColor:[UIColor whiteColor]];

// Good
view.backgroundColor = [UIColor whiteColor];
```

* **[MUST]** プロパティの宣言などでカラムの位置は調整しなくてよい
  * 修正時の差分を最小にするため

```objc
// Bad
@property (nonatomic)       NSInteger foo;
@property (nonatomic, copy) NSString  *bar;

// Good
@property (nonatomic) NSInteger foo;
@property (nonatomic, copy) NSString *bar;
```


<a id="Variables"></a>
## 変数

- **[MUST]** スコープがもっとも狭くなるように宣言すること
- **[MUST]** 変数の再利用をしてはいけない
  - 同じ型の変数であっても、用途が違う場合は都度用途に適した名前を付けて宣言すること
- **[MUST]** 定数としては `extern NSString *const FooBar` または `NS_ENUM` を使うこと
  - 定数としてマクロを使ってはならない
  - `k` Prefix は推奨しない。定数名は大文字始まりにすること
  - ソースコードファイルをまたいで使用したい定数を宣言する場合は `extern` 指定子を用い、クラス名を頭につけた名前にすること
  - ファイルスコープの定数を宣言する場合は `static` 指定子を用いること。クラス名やベンダープレフィックスを名前に含めない

<a id="Types"></a>
## 型

- **[MUST]** ポインタのアスタリスクは型ではなく変数につけること
- **[MUST]** 原則として生の `id` 型を使用してはいけない
  - CocoaTouchの仕様上必要な場合を除き、明示的に使う必要はないはずである
  - プロトコルつきの `id<Protocol>` はこの限りではない
  - 基底クラスが同一の様々なクラスとなりうる型を表現する際は `__kindof` キーワードをつけること
- **[MUST]** コレクション型にはジェネリクスを使い要素の型を指定すること
  - 例えば `NSArray<NSString *> *` などのように宣言する
  - ジェネリクスにできない場合は設計が間違っている可能性がある
- **[SHOULD]** `nullable` / `nonnull` が使える場所ではなるべく使うこと
  - [Nullability and Objective-C - Swift Blog - Apple Developer](https://developer.apple.com/swift/blog/?id=25)
- **[SHOULD]** 可能な場合は `NS_ASSUME_NONNULL_BEGIN` / `NS_ASSUME_NONNULL_END` マクロを使い、 `nonnull` は明示しないこと

```objc
// Bad
@interface SomeClass
@property (nonatomic, nullable, copy) NSString *nullableString;
@property (nonatomic, nonnull, copy) NSString *nonnullString;
@end

// Good
NS_ASSUME_NONNULL_BEGIN

@interface SomeClass
@property (nonatomic, nullable, copy) NSString *nullableString;
@property (nonatomic, copy) NSString *nonnullString;
@end

NS_ASSUME_NONNULL_END
```

<a id="Enum_and_Options"></a>
## Enum / Options

- **[MUST]** enumでなく`NS_ENUM`を使う

```objc
// Bad
typedef enum {
    TypeNone = 0,
    TypeDone = 1,
} Type;

// Good
typedef NS_ENUM(char, Type) {
    TypeNone = 0,
    TypeDone = 1,
};
```

後者だと`Type`型として定義できる数値を限定できる。例えば、誤って`TypeNG = 0xFF`とかを加えてしまったときにコンパイルエラーを出してくれる。

- **[MUST]** ビットフラグを定義する場合は、`NS_OPTIONS`を使う

<a id="Naming"></a>
## 命名

- **[MUST]** UIViewControllerを親に持つControllerはYYYAbcViewController、その他のControllerはYYYAbcControllerとする（YYYAbcを任意の名称で）。
- **[MUST]** その他UIKitフレームワークのUIXXXを継承する場合は、基本的にYYYAbcXXXとする。ただしUITableViewCellなど長いクラス名で末尾の単語のみ残せば意味が通るものは例外とする。

```objc
// Samples
UIView *originalView = [CKDOriginalView new];
UIButton *originalButton = [CKDOriginalButton buttonWithType:UIButtonTypeCustom];
UITextField *originalTextField = [CKDOriginalTextField new];

// 例外
UITableViewCell *originalCell = [[CKDOriginalCell alloc] initWithStyle:UITableViewCellStyleDefault reuseIdentifier:@"XXX"];
```

- **[MUST]** UIKit や各種ライブラリなどのプロジェクト外で定義されているクラスに対してカテゴリでメソッドを追加する場合は、メソッド名にベンダープレフィックスをつけること

```objc
// Bad
@interface UIImageView (CKDRecipeThumbnailUtil)
- (void)setRecipeThumbnailImageWithURL:(NSURL *)url;
@end

// Good
@interface UIImageView (CKDRecipeThumbnailUtil)
- (void)ckd_setRecipeThumbnailImageWithURL:(NSURL *)url;
@end
```


<a id="Misc"></a>
## 構文

- **[MUST]** Objective-C Literalsを使うこと

[Objective-C Literals](http://clang.llvm.org/docs/ObjectiveCLiterals.html)

```objc
// Bad
NSNumber *answer = [NSNumber numberWithInt:42];

NSArray *array = [NSArray arrayWithObjects:@"one", @"two", nil];

NSDictionary *dictionary = [NSDictionary dictionaryWithObjectsAndKeys:@"value", @"key", nil];

// Good
NSNumber *answer = @42;

NSArray *array = @[@"one", @"two"];

NSDictionary *dictionary = @{@"key" : @"value"};
```

- **[SHOULD]** インクリメント、デクリメントは for の再初期化式以外では使わないこと
  - Swift では廃止されるため、使用を避けること
  - CoreFoundation API などを使った低レイヤの処理を書く際に必要な場合はこの限りではない

- **[MUST]** `nil` の条件判定は `if (x == nil)` または `if (x != nil)` とする

```objc
// Bad
if (nil != x) { /* ... */ }
if (nil == x) { /* ... */ }
if (!x) { /* ... */ }
if (x) { /* ... */ }

// Good
if (x != nil) { /* ... */ }
if (x == nil) { /* ... */ }
```

- **[MUST]** `__block` 変数は原則として使わないこと
  - 必要なケースはあるが、ほとんどの場合使わずに済むように書き直せる
  - たとえば、`- enumerateObjectsUsingBlock:` は for-each文に書き換えられる
