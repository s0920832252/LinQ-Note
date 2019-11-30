---
tags: LinQ, LinQ基礎 , C#
---

# LinQ基礎 - 泛型委派與Func、Action、Predicate

### 前言
前一篇提到我們可以透過匿名函式去即時地實體化委派 , 而不用特別撰寫一個方法去指派. 但委派仍然必須要預先宣告. 若是型別有三種 , 則必須針對此宣告三個名稱不同的委派.
```C#
public delegate int CityDelegate(int arg, int arg2);
public delegate double CityDelegate2(double arg, double arg2);
public delegate string CityDelegate3(string arg, string arg2);
```
但事實上這三個委派可能僅是在傳入以及回傳型別不同. 此時 , 若是使用泛型在委派上. 則可以將其簡化.
```C#
public delegate T CityDelegate<T>(T arg, T arg2);
```

由此重新思考一下 , 委派與委派之間的差異其實也只有三點
1. 回傳的資料型別
2. 委派方法參數型別
3. 委派方法參數的數量

若是使用泛型後 , 則只剩下委派方法參數的數量這一點還需要開發者定義.

### [泛型](https://docs.microsoft.com/zh-tw/dotnet/csharp/programming-guide/generics/)

> 泛型是在 .Net 2.0 新增的功能。泛型將型別參數的概念引進 .NET Framework 中，使得類別和方法在設計時，可以先行擱置一或多個類型的規格，直到用戶端程式碼對類別或方法進行宣告或具現化時再行處理。

> 泛型在參考型別的運作方式稍有不同。第一次搭配任何參考型別建構泛型型別時，執行階段會建立特定的泛型型別，以物件參考取代 MSIL 中的參數。然後建構類型每次搭配參考型別具現化為其參數時，不論類型為何，執行階段都會重複使用先前建立的特定版本泛型型別。 

由上面引用可知 , 泛型會在**編譯時** , 將 T 定義為一個變數(?) , 直到**執行時期**第一次遇到才會代入為某個型別 , 但之後的使用 , 則會**重複使用而不用再次代入**. 也就是說使用泛型會先暫緩型別的規格定義，到了實體化時再定義其型別規格.

#### 泛型的優點
1. 類型安全
    - 泛型會將類型安全的負擔轉移給編譯器. 並不需要撰寫程式碼以測試是否為正確的資料類型 , 因為它在編譯時期會強制執行. 降低了類型轉換的需求以及執行階段錯誤的可能性.
2. 泛型委派讓類型安全回呼不需要建立多個委派類別
    - 像是前言中的例子 , 原本需要宣告複數個委派 , 但最後只需要一個.
3. 效能較佳 
    - 泛型集合類型在儲存和管理實值類型上 , 通常有較好的表現 , 因為不需要 box 實值類型.
    - 雖然我們也可以透過宣告型別為 object 來達到跟泛型類似的效果. 但其效能會比較差.
      ```C#
      public delegate object CityDelegate(object arg, object arg2);
      ```

#### 泛型委派
使用泛型的委派

##### 參考
[C++ 樣板和 C# 泛型之間的差異](https://docs.microsoft.com/zh-tw/dotnet/csharp/programming-guide/generics/differences-between-cpp-templates-and-csharp-generics)
[執行階段中的泛型](https://docs.microsoft.com/zh-tw/dotnet/csharp/programming-guide/generics/generics-in-the-run-time)
[.NET 的泛型](https://docs.microsoft.com/zh-tw/dotnet/standard/generics/index)
[泛型委派](https://docs.microsoft.com/zh-tw/dotnet/csharp/programming-guide/generics/generic-delegates)

---

### Func、Action、Predicate

雖然我們可以用泛型委派去解決不同型別的問題 , 但是因為還有**委派方法參數的數量**這一點尚未解決 , 所以我們仍然必須要宣告委派已明確它的結構. 但微軟已經預先替我們宣告好那些我們可能必須自行宣告的泛型委派了. 像是下面這樣:
```C#
public delegate TResult func<TResult>();
public delegate TResult func<T1, TResult>(T1 arg1);
public delegate TResult func<T1, T2, TResult>(T1 arg1, T2 arg2);
public delegate TResult func<T1, T2, T3, TResult>(T1 arg1, T2 arg2, T3 arg3);
.
.
```

#### [Func](https://docs.microsoft.com/zh-tw/dotnet/api/system.func-1?view=netframework-4.8)

Func 是一個具有回傳值的泛型委派. 其型式為 Func<T , T1 , … , Tn , TResult>.    
Func<T , T1 , … , Tn , TResult> 代表一個委派有 n+1 個輸入參數 , 其型別分別為 T ~ Tn , 且有一個型別為 TResult 的回傳值 ,

下圖是微軟預先定義的 Func , 總共有十七個. 也就是說 Func 的輸入參數最多可以接受十六個輸入參數. 最少零個.


![](https://i.imgur.com/5v7AYfi.png)

##### 範例
```C#
Func<int, string> IntToString = (int i) => $"string is {i}";
string ans = IntToString(5);
Console.WriteLine(ans);
```

結果會印出 **string is 5**

#### [Action](https://docs.microsoft.com/zh-tw/dotnet/api/system.action-1?view=netframework-4.8)
Action 是一個回傳 void 的委派 , 不一定會使用泛型. 其型式為 Action<T , T1 , … , Tn > .    
Action<T , T1 , … , Tn > 代表一個委派有 n+1 個輸入參數 , 其型別分別為 T ~ Tn , 沒有任何的回傳值.

與 Func 相同 , Action 的輸入參數最多可以有十六個 , 最少零個

##### 範例
```C#
Action<int> ShowString = (int i) => Console.WriteLine(i);
ShowString(5);
```
結果會印出 **5**

#### [Predicate](https://docs.microsoft.com/zh-tw/dotnet/api/system.predicate-1?view=netframework-4.8)
Predicate 是一個回傳 bool 值的委派. 其型式為 Predicate<T> .
其通常用來判斷傳入的**某個**實體是否符合某種規則.

```C#
Predicate<int> BigThanFive = (int i) => i > 5;
const int num = 7;
Console.WriteLine($"數字 {num} 是否大於 5 : {BigThanFive(num)}");
```
結果會印出 **數字 7 是否大於 5 : true**

---

### 總結

在 LinQ 中的 Enumerable 這個靜態類別 , 幾乎都是針對 IEnumerable<TSource> 來設計擴充方法. 其型式可能如下 : 
```C#
public static IEnumerable<TSource> MyWhere<TSource>(this IEnumerable<TSource> source, Func<TSource, bool> predicate)
{
     foreach (var item in source)
     {
          if (predicate(item))
          {
               yield return item;
          }
     }
}
```

然後在方法內走訪 IEnumerable<TSource> (不論是透過 Foreach 還是 GetEnumerator() ) , 在走訪的過程中 , 每一個 TSource item 會作為委派的參數傳入 , 並得到委派的回傳值 . 依照此回傳值做出相對應的處理 .
回傳結果則看處理是否會使用 yield return.

---

### Thank you! 

You can find me on

- [GitHub](https://github.com/s0920832252)
- [Facebook](https://www.facebook.com/fourtune.chen)

若有謬誤 , 煩請告知 , 新手發帖請多包涵

# :100: :muscle: :tada: :sheep: 
