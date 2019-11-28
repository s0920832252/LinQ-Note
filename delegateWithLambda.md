---
tags: LinQ, LinQ基礎 , C#
---

# LINQ基礎 - 委派與Lambda

### 前言
為了要對於每一筆資料做特定的處理 , LinQ 方法常會在走訪資料集合的時候 , 透過執行委派來得到期望的結果. 而為求方便與簡潔 , LinQ 常使用 Lambda 來指定委派. 所以 Lambda 對於 LinQ 來說 , 非常重要. 

### [委派](https://docs.microsoft.com/zh-tw/dotnet/csharp/programming-guide/delegates/)
委派的概念類似於 C++ 的函式指標 , 也許可以想成是儲存方法的變數(?)

使用委派的過程如下
* 建立委派一個 , 步驟如下
    1. 定義委派的結構 -> E.g. delegate void ShowMoneyType(string s, int x)
    2. 宣告一個委派變數 -> E.g. ShowMoneyType someThing
    3. 定義一個滿足上述委派結構的方法 -> E.g. void DisplayCash(string str , int value)
        - 輸入參數個數以及型態和回傳值都必須與該委派結構相同 , 否則無法存入委派.
    5. 指派給委派一個實體 -> E.g. someThing=DisplayCash
* 呼叫委派實體 (可使用 invoke , 進行呼叫)

由上面的步驟 , 或許可將委派想成有三個層面
- 宣告端 : 定義委派結構
    - 宣告委派和定義方法很類似. 它包含一個回傳類型和任意數量的參數. 所以此階段最主要工作就是要明確定義回傳型別和參數型別
- 邏輯端 : 設定執行步驟細節(定義滿足委派結構的方法)
    - 可透過具體函式或是匿名函式來設定.
- 呼叫端 : 負責連接宣告端和邏輯端 , 並呼叫委派
    - 此端會需要建立一個委派實體 , 以便呼叫委派  

以下為**指派**實作細節(邏輯端)給委派的三種方式.
1. 具名函式 : 有函式名的函式 lol
2. 匿名函式
    * 匿名方法
    * Lambda運算式

### C#1.0 具名函式

將已經宣告的方法()指派給委派.

##### 範例
```C#
static class Program
{
     // 邏輯端 - 實際要做的事情細節
     public static void DisplayCash(string name , int value)
     {
         Console.WriteLine($"{name} have {value}");
     }
     
     static void Main(string[] args)
     {
          // 呼叫端 - 傳入參數以實體化委派(做了someThing=DisplayCash這個動作)
          // 然後執行doSomeThing() 
          new City().DoSomeThing(DisplayCash);
          Console.ReadLine();
     }
}

public class City
{
     private readonly int _money = 150;
     private readonly string _name = "City";

     // 宣告端 - 宣告委派
     public delegate void ShowMoneyType(string s , int x);

     public void DoSomeThing(ShowMoneyType someThing) => someThing(_name , _money);
}
```

---

### C#2.0 匿名方法 delegate 關鍵字

格式 : delegate (arguments) { statements }
- delegate: 匿名方法的保留字
- arguments: 輸入參數 
    - 輸入參數可有多個. 使用逗號隔開. 但需要**定義參數型別**.
- statements: 此函式執行的程式碼片段

在 C# 2.0 中引入匿名方法後 , 就不必一定要使用某一個執行個體方法或靜態方法 , 來指定委派變數. **可直接透過匿名方法來定義委派要執行的內容**.

##### 範例
```C#
static class Program
{
     static void Main(string[] args)
     {
          // 使用匿名方法即時定義執行細節.
          new City().DoSomeThing(delegate (string name, int money) 
          { 
               Console.WriteLine($"{name} have {money}");
          });
          Console.ReadLine();
     }
}

public class City
{
     private readonly int _money = 150;
     private readonly string _name = "City";

     public delegate void ShowMoneyType(string s, int x);

     public void DoSomeThing(ShowMoneyType someThing) => someThing(_name, _money);
}
```

---

### C#3.0 Lambda

格式 : (arguments) => expression | { statements }
- arguments : 輸入參數
    - 只有一個參數時可不加括號 , 但複數個參數必須加上括號 , 並且參數之間要用逗號隔開.
         - ```C# 
           (x) => x * 15 //合法
            x  => x * 15 //合法
           (x,y) => x * y //合法 
           ```
          
    - **可以不用明確指定型別** , 但明確指定型別時一定要加上括號
        - ```C#
          (int x) => x * 15 //合法
          ```
    - 用空括號()來表示沒有輸入參數.
        - ```C#
          () => 1 * 15 //合法
          ```

- expression: 運算式
    - 不使用大括號{} , 則僅能使用單行程式碼作為運算式.
    - 運算式後**不需要加分號 ;**
    - 若函數需要回傳 , expression 的運算結果代表回傳值.
        - ```C#
          (int x) => x * 15 //合法
          ```
- { statements }: 程式碼區塊
    - statement為此函數執行的程式碼敘述.
    - 程式碼區塊可以有複數行程式碼 , 另外每行**最後要加分號 ;**
    - 若函數需要回傳 , 使用 return 關鍵字.
        - ```C#
          (int x) => {
                       var value = x * 15;
                       return value;
                     } //合法
          ```

#### 比較 Lambda 與匿名方法的差異
1. **不用透過 delegate 關鍵字**來建立匿名函式.
2. 除非會影響可讀性 , 否則**不需要定義參數的型別** .
3. 括弧可以省略 .

參考此[資料](https://docs.microsoft.com/zh-tw/dotnet/csharp/programming-guide/statements-expressions-operators/anonymous-functions) , 微軟也建議我們開始使用 Lambda 取代匿名方法
> 我們建議使用 Lambda 運算式，因為它們提供更簡潔且更具表達性的方式來撰寫內嵌程式碼.

##### 範例
```C#
static class Program
{
     static void Main(string[] args)
     {
          // 使用Lambda即時定義執行細節.
          new City().DoSomeThing((name, money) => Console.WriteLine($"{name} have {money}"));
          Console.ReadLine();
     }
}

public class City
{
     private readonly int _money = 150;
     private readonly string _name = "City";

     public delegate void ShowMoneyType(string s, int x);

     public void DoSomeThing(ShowMoneyType someThing) => someThing(_name, _money);
}
```

---

### 結論
想要建立一個符合委派結構的實體 , 從一開始 C#1.0 只能透過指定一個具名函式 , 到 C#2.0 可以使用 delegate 關鍵字以建立匿名方法來定義匿名函式 , 到 C#3.0 可以使用**更簡易**的 Lambda 語法來定義匿名函式
透過 Lambda , 我們不必再額外定義一個方法去作為具名函式了 , 而是可以再需要呼叫委派時 , 馬上透過 Lambda 語法 , **即時**的實體化委派並呼叫.

ps : 網路上一些文章習慣稱呼這樣透過使用匿名函式實體化的委派為**匿名委派** , 不過我找了微軟文件很久 , 並沒有發現這樣的稱呼呀 :crying_cat_face: 

以下簡易地實作 LinQ 的 Where 方法來作為此篇的結論
```C#
public delegate bool CityPredicate<TSource>(TSource item);
static IEnumerable<TSource> MyWhere<TSource>(this IEnumerable<TSource> source, CityPredicate<TSource> predicate)
{
     foreach (var item in source)
     {
          if (predicate(item))
          {
               yield return item;
          }
     }
}

static void Main(string[] args)
{
     List<int> vs = new List<int>() { 5, 4, 8, 7 };
     foreach (var item in vs.MyWhere(number => number >= 5))
     {
          Console.WriteLine(item);
     }
     Console.ReadKey();
}
```

##### 輸出結果
5    
8    
7    

---

### 補充

#### [MulticastDelegate](https://docs.microsoft.com/zh-tw/dotnet/api/system.multicastdelegate?redirectedfrom=MSDN&view=netframework-4.8)

> 表示多重傳送的委派 (Delegate)；也就是說，委派可以在它的引動過程清單中包含一個以上的項目。

由上述參考可知 , 委派有一個清單儲存多個方法實體 , 並且再呼叫委派時 , 依序呼叫這些方法.

##### 範例

```C#
public delegate void Print();
public static Print print = null;

static void Main(string[] args)
{
    print += () => Console.WriteLine("Test1");
    print += () => Console.WriteLine("Test2");
    print += () => Console.WriteLine("Test3");

    print();            

    Console.ReadKey();
}
```



輸出結果

- ![](https://i.imgur.com/xWJm54q.png)



#### GetInvocationList 
若是委派具有回傳值 , 並需要個別取得其每一個方法的結果 , 可使用 GetInvocationList ()

###### 範例

```C#
public delegate int Math(int num);
public static Math math = null;

static void Main(string[] args)
{
    math += (num) => num + 1;
    math += (num) => num - 1;
    math += (num) => num * 1;

    foreach (Math deleglateItem in math.GetInvocationList())
    {
        Console.WriteLine(deleglateItem.Invoke(10));
    }

    Console.ReadKey();
}
```

##### 輸出
11    
9    
10    



---

### Thank you! 

You can find me on

- [GitHub](https://github.com/s0920832252)
- [Facebook](https://www.facebook.com/fourtune.chen)

若有謬誤 , 煩請告知 , 新手發帖請多包涵

# :100: :muscle: :tada: :sheep: 
