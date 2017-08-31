---
layout: post
title:  "Racket 起手式"
date:   2017-08-18 21:56:00 +0800
categories: Racket
---

## 前言

在我所身處的環境中，這裡的工程師大多聽過 Python、Ruby、PHP、Java、C#、JavaScript 諸如此類的程式語言，這些語言是這時代的顯學，是進入資訊領域的叩門磚。若有些人，從事的是比較靠近系統層面相關的工作，或開發嵌入式的軟體，則會對像 C、C++、Assembly 之類的語言相當熟悉。

然而我今天要談的，是一個相對冷門，但很有趣的語言。在開始介紹之前，我們先來想想一件事：

_程式語言如何被執行？_

例如，一個最簡單的 Java 例子：

```java
int a = 3 + 5;
```

或用 Python 表示：

```python
a = 3 + 5
```

Java 是一個靜態型別語言，所以我們在使用任何一個 identify 時，都必須明確指定它的型別。而 Python 是個動態型別語言，因此 identify 的型別，是從它的陳述中被推斷出來的。然後程式透過 = (等號)，指明賦值的動作。賦什麼值呢？則要看後頭的 3 + 5 這個敘述；而程式讀到 + (加號) 時，它會知道這一段要進行的是什麼操作 (operation)，這個操作接受什麼樣的參數 (parameter)，是兩個數字。

換句話說，程式語言看這一段敘述，大概像這樣子：

![ast.png]({{ site.baseurl }}/assets/imgs/racket-basic-01.png)

然後，它從最深處， 3 + 5 這一段開始解讀，再解讀 a = ? 這樣的句構。但在 Java 中，型別宣告會先於等號，好使程式預先知道接下來要接到的是什麼內容。

這樣的句子，我們在現代程式語言中已經習以為常。現在，我們要開始做些變化。首先，將每一個環節用小括號包起來：

```
(a = (3 + 5))
```

然後，這是關鍵，把剛剛所說的操作 (operator) 移到小括號的第一個位子：

```
(= a (+ 3 5))
```

因為 = (等號) 在早期的語言裡，大多是拿來進行內容判別用的，因此我們換另一個正規的操作，define：

```
(define a (+ 3 5))
```

這樣的語言結構，是不是跟剛剛那張圖很像？

這張圖，是一個簡單的樹狀結構，它有個正式名詞，叫做抽象語法樹 (Abstract Syntax Tree, AST)，而這不是我們要解釋的，因為我並不是程式語言專家。這個改寫過後的程式，便是一種名為 Scheme 的語言。

Scheme 是一種很特別的語言，原本是由麻省理工學院的教授根據一種更早期的語言 Lisp 改良而來，便是所謂的 MIT Scheme。它拿掉了 Lisp 一些令人困惑的特性，並用在 CS 課程的教學上。而後來 Matthias Felleisen 的 PLT 團隊按著 Scheme 的語法規範，再另外設計了自己的 Scheme，故稱為 PLT Scheme，而 PLT Scheme 於 2010 年改名為 Racket，就是我們今天要介紹的語言。

雖然 Scheme 是由 Lisp 而來，而 Racket 是由 Scheme 而來，它們有許多共通之處，也有些許差異之處，但本文只會針對 Racket 做說明。

## 工具

Racket 有附一個很好的開發環境 — **Dr. Racket**，當你安裝完整語言工具時，就會附加在其中。程式的安裝，便不做贅述。

待安裝完成後，第一次啟動 **Dr. Racket** 時，應該會需要選擇開發語言，請選 **The Racket Language** 就好。這時，你會看到 IDE 呈現兩個上下分割的畫面：

![racket view]({{ site.baseurl }}/assets/imgs/racket-basic-02.png)

上面的是程式碼編輯視窗，下方則是即時互動的 REPL。如果你想測試某些簡單的程式碼片段，可以在下方輸入。但我們關注點會在上方的編輯區內。

在程式碼編輯區內，最上面一行便是剛剛選擇的開發語言，預設是 racket，前面加上 #lang，不是註解，而是一種特殊的宣告，它沒有什麼副作用，我們只需要先把它放在那。

## 資料與它們的操作

Racket 雖然有物件導向的概念可以使用，可是它並非所謂純物件導向語言。在開發的過程中，資料或物件的操作是透過具體的函式宣告進行的，而非像現代高階語言這樣，可以融入在語法的敘述當中。例如在 Java 裡頭可以這麼寫；

```java
String str = "a" + "b";
```

在 Racket 則必須這樣：

```scheme
(define str (string-append "a" "b"))
```

字串相接這操作，在大多數語言裡都有很便利的做法可以完成，但在 Racket 中，必須呼叫字串相關函式進行處理。因此，初學 Racket 的時候，花一點時間熟悉它的幾種資料類型，以及操作各種資料的相關函式很重要。相關文件可從這篇看起：[Built-In Datatypes](https://docs.racket-lang.org/guide/datatypes.html)。

我們這次要處理的，不是字串類的問題，而是數字類的。在 Racket 裡頭，數字可以有許多種表示，除了「無上限」的整數以及不精確的 IEEEE 754 浮點數操作之外，Racket 還支援複數、分數(有理數)等數值的表示與操作。

但因這是起頭，我們先別管它那些次要的東西，我們必須先談語法。每一個程式語言最重要的特性，通常表現於它的語法設計與隱喻中。

## 問題

我們要來實作一個很簡單的東西—費氏數列。費氏數列的例子，也是 Scheme 著名教材 SICP 的基本題型之一。不多介紹，直接講定義：

給定：

```python
f(0) = 0
f(1) = 1
f(n) = f(n - 1) + f(n - 2)
```

就這麼簡單，但它很有意思。

## 熟悉的語言解

我們先用遞迴解法，使用 Java 來說明：

```java
int fibonacci(int n) {
    if ( n == 0 ) {
        return 0;
    } else if ( n == 1 ) {
        return 1;
    } else {
        return fibonacci( n - 1 ) + fibonacci( n - 2 );
    }
}
```

當然，這是一個很沒效率的解法，你若傳 10 或 50 進去還行，若傳 100，大概可以去樓下的咖啡店買一杯再上來。我們會討論什麼是比較有效率的解法。

## lambda

在現代高階語言裡，談到 lambda，好像是一種神秘的武器，它可以跳過現行語言的限制，靈活地定義好用的功能。其實它是一種很古老的函式概念，有人稱為匿名函式，但在 Racket 裡面，lambda 就等於函式。重複一次，「lambda 就等於函式」。

例如：

```scheme
((lambda (x) (* x x)) 10)
```

這是指，我定義一個匿名函式，其操作為 x 的平方，然後隨即傳入一個整數給它。我們可以結合之前的 define 宣告，給一個完整的程式碼：

```scheme
(define square
	(lambda (x) (* x x)))
(square 10)
```

在 Racket 裡頭，程式碼是用小括號包起來的，因此需要妥善地排版，以分辨程式碼區塊之間的關係。我們現在給予剛剛那個沒寫名字的 lambda 一個正式名字 square，然後就可以用這個名字呼叫它。這樣寫，像不像 JavaScript 裡的這個寫法呢？

```javascript
var square = function(x){
	return x * x;
};

square(10);
```

而函式宣告在 Racket，可以更進一步整合為：

```scheme
(define (square x)
	(* x x))
```

類似在 JavaScript 裡這樣宣告：

```javascript
function square(x){
	return x * x;
}
```

而以上兩種宣告方式，在 Racket 裡是相等的意思。這有助於我們接下來要示範的，費氏數列 Racket 的 lambda 解法：

```scheme
#lang racket

(define fib
  (lambda (n)
  	(cond
      ((= n 0) 0)
      ((= n 1) 1)
      (else (+ (fib (- n 1)) (fib (- n 2)))))))
```

如果不寫 lambda 也可以：

```scheme
#lang racket

(define (fib n)
  (cond
    ((= n 0) 0)
    ((= n 1) 1)
    (else (+ (fib (- n 1)) (fib (- n 2))))))
```

其實看起來像是把參數放到了函式名稱宣告後面，然後少了一層括號，
雖然我個人比較喜歡 lambda 式的宣告，但在寫的時候，還是大多會用第二種寫法。

## 條件判斷

Racket 的條件判斷很特別，我們在 Java 或 Python 裡會使用 if ... else if ... else 的結構來寫，而在 Racket 裡面，提供了兩種基本的條件判斷機制：cond (condition)、if。

cond 的用法如範例所述，除了 else 的敘述之外，其他皆為條件的列舉，類似 Java 之類語言的 switch-case，只是它的條件設定比 switch-case 更靈活，可以是另一個函式呼叫，只要回傳 boolean 值就好。在 Racket 或甚至整個 Scheme 裡面，所有非空值的回傳值，都可以當作 true (Racket 寫作 #t)，而否定的 false (#f) 則不能這樣代換。嚴格來說，這是很 tricky 的特性 (C 語言也是如此)，我個人倒是比較喜歡 Java 的作法：true / false 不可由任何非 boolean 值的內容取代。

而 if 敘述，並不用寫出 else，它的結構是這樣的：

```scheme
(if (condition)
  (expression when true)
  (expression when false))
```

雖然 if 省掉了 else 的字眼，可是 if 一定要寫反面情況的敘述。如果有人說，我就只有 true 的 expression，false 的時候程式不幹事，好吧，那就寫個 (void) 吧。

## 函式回傳

我們再回頭看看前頭的程式，嗯？沒有 return？是的，在 Racket 裡，計算的結果就是回傳值。因此當我們看到這段：

```scheme
(cond
  ((= n 0) 0)
  ((= n 1) 1))
```

就只要很直接地理解：

```
((=  n   0)     0)
  當 n 為 0，回傳 0
((=  n   1)     1)
  當 n 為 1，回傳 1
```

## 遞迴呼叫

在現代高階語言中，遞迴是一個很基本的功能，雖然從演算法角度來看，遞迴可能會造成相當沒有效率的執行結果，但是它卻是一個簡化問題的利器。遞迴的結構，通常會設定一個結束點，並設定一個呼叫自己的操作，並傳入限縮後的資料。例如在這個費氏數列的問題裡：

```scheme
(cond
  ((= n 0) 0)
  ((= n 1) 1))
```

即是設定停止點，如果有一天，你發現你的遞迴程式停不了，就得先檢查停止點是否正確。然後最後呼叫自己：

```scheme
(else (+ (fib (- n 1)) (fib (- n 2))))
```

## 更有效率的演算

我們先跳開費氏數列的問題，來想想階乘好了。用遞迴能不能寫階乘？可以：

```scheme
(define (factor n)
  (cond
    ((= n 0) 1)
    (else (* n (factor (- n 1))))))
```

那，有人會想，階乘不是只要用個迴圈跑一下就行了。為何要跑遞迴？遞迴的問題在哪裡呢？你有試過用費氏數列遞迴解跑 100 嗎？

假設我們傳 5 進去好了：

**(fib 5)** 解開來後會變成這樣：

```scheme
(+ (fib 4) (fib 3))
```

```scheme
(+ (+ (fib 3) (fib 2)) (+ (fib 2) (fib 1)))
```

```scheme
(+ (+ (+ (fib 2) (fib 1)) (+ (fib 1) (fib 0))) (+ (+ (fib 1) (fib 0)) 1))
```

```scheme
(+ (+ (+ (+ (fib 1) (fib 0)) (fib 1)) (+ (fib 1) (fib 0))) (+ (+ (fib 1) (fib 0)) 1))
```

這樣拆解看起來很複雜，但化作圖形是這樣的：

![fib tree]({{ site.baseurl }}/assets/imgs/racket-basic-03.png)

從這張圖可以觀察，簡單地呼叫 fib 5，便會經過 4 層的樹狀結構，並且，程式會重複多次地呼叫同一個值，如 (fib 3)、(fib 2)。雖然遞迴看起來很簡單，其實它執行下來非常複雜，導致我們若用這個費氏數列的解法，只要超過一個數字，它的效率就會超過我們等待的耐性。

## 尾遞迴與迭代