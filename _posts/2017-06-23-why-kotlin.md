---
layout: post
title:  "你為何要用 Kotlin"
date:   2017-06-23 06:45:00 +0800
categories: Kotlin
---

# 你為何要用 Kotlin?

嚴格來說，沒有為什麼。這或許是一個既生 J 何生 K 的問題，但實際上來說，舉凡是執行在 JVM 平台上的語言能做到的，用 Java 都能做到。這個推定，是 Java 這領域的基本共識。如同 Bjarne Stroustrup 在他的 The C++ Programming Language 裡所提到的，C++ 所作的事情，同樣能用 C 語言來完成，但問題在於使用 C 語言將花費更大的功夫才能完成。C 與 C++ 顯然是兩個極度不同的開發典範：結構化與物件導向。然而 Java 與 Kotlin 呢？我若在 JVM 平台上想要選擇一個與 Java 截然不同典範的語言，我不是應該去寫 Scala 或 Cloujure 嗎？為何要用 Kotlin 呢？我們即將探討這個問題。

## 收斂大哉問

如果你被人問到，為什麼要做這個工作？為什麼人要工作？這樣的大哉問，可能你會選擇不回答，或回答些高大上的想法，但面對範圍很大的問題，我們有兩種典型的處理方式：收斂與反詰。我們現在收斂一下上面的問題。或許可以這麼問：

# Kotlin 與 Java 有何不同？

## 重新開始

這個問題，或許你可以從 Kotlin 網站的列舉中看見。不過，我們用程式來實作，了解兩種語言編程思想的差異。

## 正規比較

根據 [Comparison to Java Programming Language](https://kotlinlang.org/docs/reference/comparison-to-java.html) 一文，我們可以列舉如下：

## Kotlin 與 Java 具體的差異

### Null references are controlled by the type system

這項直翻起來，就是說 Kotlin 使用型別系統來避免掉以往在 Java 裡常發生的 Null reference 問題。（NullpointerException）

### No raw types

Raw type 在 Java 裡頭指稱的是未經界定的泛型使用方式。例如：```Collection coll = new ArrayList();``` 這樣的一句，在 Java 1.4 之前完全沒問題，但在 Java 1.5 之後，已經讓 Joshua Bloch 的 Effective Java 給禁了。在 Kotlin 裡頭，無法使用 Raw type 這樣的機制。

### Arrays in Kotlin are invariant

試想，在 Java 裡頭，你有沒有寫過這樣的東西：

```java
public void caller(){
    Object[] results = callee("abcdefg.hijklmn.opqrs");
}

public String[] callee(String text){
    return text.split("\\.");
}
```

在 Java 裡頭這是合法的，但你若就著相同的概念在 Kotlin 裡這樣寫，這是非法的。說起來，Kotlin 裡的 Array<> 比較像是 Java 的 ArrayList<> 的操作感。

### Kotlin has proper function types, as opposed to Java's SAM-conversions

當 Java 出現了 Lambda 之後，大家可能很習慣寫這種語法：

```javva
public static void main(String[] args) throws InterruptedException {
    Thread t = new Thread(()->{
        System.out.println("Hi, I'm in a thread.");
    });

    t.run();
    t.join();
}
```

原本的 Thread 是傳一個 Runnable 介面進去，而 Runnable 介面也很簡單，就是一個 ```void run()``` 的執行。因此這有個簡稱：SAM (Single Abstract Method)。但 Kotlin 的型別系統裡，就內建了 function type (函數型別)，使得在開發 Kotlin 時，不用透過語法蜜糖來包裝原有的 OOP 型別系統，並可以直接將函數當作第一級物件來操作：

```kotlin
fun run(func: () -> Unit) {
    func()

    // same as func()
    func.invoke()
}

fun sayHi() {
    println("Hi, this is in Kotlin.")
}

fun main(args: Array<String>) {
    run(::sayHi)
}
```

### Use-site variance without wildcards

這是 Kotlin & Java 差異中，最難理解的一項。但如果你寫過 C#，你會很快地弄懂 Kotlin 的泛型型別系統的概念。而在此，我們不解釋整個泛型的機制。簡單舉例來說，我們參考 Effective Java 第二版，第五章第 28 條的範例，設計以下 Java 容器類別：

```java
public class Stack<E>{
    public E pop(){
        // pop the last element.
    }

    public void push(E element){
        // push the element to last.
    }

    public boolean isEmpty(){
        // return true if this stack is empty, vice elsa.
    }

    public void pushAll(Collection<? extends E> elements){
        for(E e : elemenst){
            push(e);
        }
    }

    public void popAll(Collection<? super E> container){
        while(!isEmpty()){
            container.add(pop());
        }
    }
}
```

其中，```pushAll``` 與 ```popAll``` 兩個函式在叫用時，用了萬用字元代表其參數化泛型類型的概念，在 Kotlin 裡面，你可以這麼寫：

```kotlin
class KtStack<T> {
    val stack = mutableListOf<T>()

    fun put(item: T) {
        stack.add(item)
    }

    fun pop(): T {
        return stack.removeAt(stack.lastIndex)
    }

    fun isEmpty(): Boolean{
        return stack.isEmpty()
    }

    fun plantAll(items: Collection<out T>) {
        stack.addAll(items)
    }

    fun popAll(container: MutableCollection<in T>) {
        container.addAll(stack)
    }
}
```

在使用端 (Use-site) 透過 ```Collection<out T>``` 來表示 ```Collection<? extends T>``` 的向上轉型機制（或稱「協變」），以及 ```MutableCollection<in T>``` 來表示 ```Collection<? super T>``` 的向下轉型機制（或稱「逆變」）。以省卻使用萬用字元來表示參數化泛型機制的歧義與模糊之處。

### Kotlin does not have checked exceptions

這個特性有好有壞，有贊成的人也有批評的人。但只要你對 Java 有些認識，這個特性幾乎不用解釋。無論如何，我們還是透過幾個簡單的程式來說明：

```java
public class IOTest {
    public static void main(String[] args){
        File file = new File(args[0]);

        try {
            FileInputStream fis = new FileInputStream(file);
        }catch (FileNotFoundException ex){
            ex.printStackTrace();
        }
    }
}
```

但在 Kotlin 裡，只須這樣寫：

```kotlin
fun main(args: Array<String>){
    val file: File = File(args[0])

    val fis: FileInputStream = FileInputStream(file)
}
```

省掉了 try-catch 的結構，好像少了幾行。嗯，說老實話，我並不喜歡這樣的做法。但我猜應該不只一個語言，跟隨 C# 的腳步，拿掉 Checked Exception。

# 下一問

即使從文件說明了 Kotlin 與 Java 之間的不同，仍然沒辦法解決我們一開始問的，為什麼要用 Kotlin 呢？我對這個語言尚且不熟，且還在試行過程中。它不像 Scheme、Scala、Haskell 是一種從本質上與 Java 不同的語言。也不是像 Python、Ruby 那樣刻意簡化、純化過某些語言特性與設計思想的語言。我在學這個語言時，最大的問題應該是這麼說：它到底解決了 Java 的什麼問題？引進了什麼來解決這個問題，又這些語言特性是否帶來新的問題呢？