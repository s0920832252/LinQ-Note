---
tags: LinQ , LinQ基礎 , C#
---

# LINQ基礎 - 隱含型別var vs 匿名型別 vs 動態型別 dynamic

### [var 簡介](https://docs.microsoft.com/zh-tw/dotnet/csharp/language-reference/keywords/var)

> 從 Visual C# 3.0 開始，在方法範圍內宣告的變數可以使用隱含的「類型」var。 隱含型別區域變數如同您自己宣告的類型一樣是強型別

> var 關鍵字會指示編譯器從初始化陳述式右側的運算式推斷變數的型別。 推斷的型別可能是內建型別、匿名型別、使用者定義型別，或 .NET Framework 類別程式庫中定義的型別。
> 

#### 由上面的簡介引用 , 我們可以知道
1. var 在**編譯時期**就已經決定其真實型別 , 其真實型別依據等號右邊的賦值型別決定.
- ```C#
  static void Main(string[] args)
  {
       var result = 1;
       Console.WriteLine(result.GetType());
       Console.ReadKey();
  }
  ```
  執行結果 : ![](https://i.imgur.com/zSgKRER.png)

  - 因為在編譯時期決定真實型別 , 因此當使用 var 宣告變數型別後 , 要馬上初始化其變數. 否則編譯器會因為無法推斷其型別而無法編譯.
    
  - 因為在編譯時期決定真實型別 , 所以當用 var 宣告變數後 , intellisense 可以支援.
    
2. 宣告 var 的變數以及你自己宣告型別的變數並**無差別** , 差別僅在於 var 是用編譯器來幫你推斷真實型別.

3. var 宣告的變數是一種**區域變數** , 因此只能使用在
    - 區域變數（方法範圍內）
    - 迴圈控制變數（For、Foreach）
    - Using 陳述式中，用來建立資源名稱的型別
    - 方法參數以及方法回傳值是無法宣告成 var 的喔. :cry: 

#### 使用 var 的原因
- 開發人員宣告變數型別可以更輕鬆
    - 若是使用強型別方式宣告變數 , 當日後需要更改變數型態時 , 則必須要手動更改宣告變數型態. 但是使用 var 宣告變數型態則沒有這個困擾 , 開發者可以更專注在功能的開發.
    - 因為編譯器會替我們決定變數型態 , 所以開發者可以更專注在等號右側的賦值正確性上.
- 有些變數型別無法由開發者自行宣告 , 像是**匿名型別**

---

### [匿名型別的簡介](https://docs.microsoft.com/zh-tw/dotnet/csharp/programming-guide/classes-and-structs/anonymous-types)

> 匿名型別提供便利的方式將**唯讀屬性**集封裝至單一物件，而**不必先明確定義型別**。**型別名稱由編譯器產生**，而且在**來源程式碼層級中無法使用**。 每個**屬性型別由編譯器推斷**。
您可以**使用 new 運算子搭配物件初始設定式來建立匿名型別**。

#### 由上面的引用可知
1. 型別名稱是由編譯器產生 , 因此使用者無法自行宣告.
2. 建立匿名型態變數後 , 其內容值無法變更(唯讀屬性).
3. 依據下圖建立一匿名類別變數,並印出此變數的型態的程式.
    ```C#
    static void Main(string[] args)
    {
         var annoymous = new { Name = "小王", Score = 5 };
         var annoymous2 = new { Name = "小王2", Score = 10 };
         var annoymous3 = new { Name = "小王3", Score = 6.5 };
         var annoymous4 = new { Name2 = "小王4", Score2 = 6.5 };
         Console.WriteLine(annoymous.GetType());
         Console.WriteLine(annoymous2.GetType());
         Console.WriteLine(annoymous3.GetType());
         Console.WriteLine(annoymous4.GetType());
         Console.ReadKey();
    }
    ```    
    執行結果 : ![](https://i.imgur.com/GAktiaE.png)




    - 從上圖可以看到 , 變數型態是<>f__AnonymousType0\`2 以及 <>f__AnonymousType1\`2 這樣的東西. 如 MSDN 上所說 , 這是一個臨時產生的變數名稱 , 所以當離開方法區塊 , 這個變數的生命週期就結束了(失去使用上的意義).
    - 基於效率考量 , 編譯器會重複使用具有**相同參數個數與參數名稱**的匿名型別 , 而不會每碰到一個匿名型別的宣告就產生一個新的泛型類別. 所以 annoymous 和 annoymous2 以及annoymous3 的實際型別都一樣.
    
#### 使用時機
臨時需要一個物件來儲存一些簡單資料 , 但又不想為了這個簡單的需求另外定義一個類別或是使用弱型別的物件. 像是 ArrayList , DataTable , DataRow 等等…  , 此時可考慮使用 C# 的匿名型別 , 開發者只要用很簡單的 new 就能創造匿名型別物件. 
例如 : 
1. 想要組一個 JSON 字串的時候.
2. 希望使用某 LinQ 函數的回傳值繼續做下一個運算的時候. 此回傳值可先用匿名物件儲存 , 以作為下個函數的參數.

---

### 與 LinQ 的關聯

隱含型別與匿名型別最大的差異就是它們本來就是完全不同的東西 , 不過這兩樣東西在使用 LinQ 的時候 , 可能會使用到.

有些 LINQ 方法會回傳的型別是由傳入的委派方法決定的 , 而委派方法可以回傳匿名型別 , 因此只要委派方法的回傳內容一改 , 前面宣告的型別可能又要改 , 那還不如直接用 var 還比較省事 , 另外若是碰到回傳的是 IEnumerable<T> 時（T可能是匿名型別）則不得不用 var 來宣告.

- 必須使用 var 來宣告的情況.
```C#
var Students = new[] { 
                        new { Name = "小王", Score = 26 }, 
                        new { Name = "小小", Score = 78 }
                     };
                     
foreach (var studnet in Students)
{
     Console.WriteLine($"學生 {studnet.Name} 考了 {studnet.Score}");
}
```

#### 網路上的看法以及我希望培養的習慣
- 對於使用 var 會造成可讀性以及維護上的困難. 此引用大神91的看法.
  > 這沒有絕對的答案，單純看每個人的習慣來決定。有些人是除了非得用var的地方，才用var，否則不用var。我自己則是習慣另一種：除了不可以用var的地方，否則都用var。我的看法如下：
  > - 針對維護與偵錯上的情況來說，基本上有Visual Studio輔助，型別根本不是問題，當把滑鼠移到var或變數上，都會出現其型別名稱，除非維護與偵錯不透過Visual Studio，那我想說的是，這是維護偵錯的方式錯誤，而不是var的問題。
  > - 針對開發上，宣告一個變數，是否要先決定其型別？我的認為是不必的，就像變數名稱不需要把型別定義上去（例如：EmployeeArray這是個糟糕的命名）。重點是變數名稱所表達的意義，把關注點關心在抽象的行為與邏輯上。既然編譯器可以幫我們決定，不會造成開發上的問題，那麼只要關心"等號"右邊的值就夠了。

- 我自己認為使用 var 的時機點是除了必須使用 var 的情況外 , 能不能僅透過看等號"右邊的值"就一目了然來做區別. 不行的話 , 就不建議使用. 例如...
  1. 等號右邊的值是匿名型別. (必須使用 var)
  2. 等號右邊的值不是一個函數的回傳值.
      - 因為僅看函數名稱 , 不一定能夠僅透過看就知道該函數的回傳值. 你可能必須切換到該函數的 code 或是移動滑鼠到該變數上才有辦法知道其回傳值型別.
  3. 等號右邊的值是 LinQ 函數的回傳值.
---

### [補充比較 - C# 4.0 的dynamic型別](https://docs.microsoft.com/zh-tw/dotnet/csharp/language-reference/keywords/dynamic)

> dynamic 型別可讓它發生所在的作業略過編譯時期型別檢查。 這些作業會改在執行階段解析。

> dynamic 類型的行為與 object 類型類似。 不過，不會解決包含 dynamic 類型之運算式的作業，或編譯器不會對其進行類型檢查。 編譯器會將作業資訊封裝在一起，而且稍後在執行階段會使用這項資訊來評估作業。 在此程序期間，會將 dynamic 類型的變數編譯為 object 類型的變數。

由引用可知 , dynamic 在編譯時期的型態為 object , 其真實型態會在執行時期才決定. 因此下圖的程式編譯會過 , 但會在執行期間出錯.

![](https://i.imgur.com/P2TMGfB.png)

dynamic 通常使用於真實型態是動態決定或是未知的時候.
例如方法參數傳入一 object 型態的變數(真實型態可能是開發者自訂類別). 因此在宣告變數來接這個 object 變數的時候 , 無法寫死單一個類別. 然後你知道該類別的某個變數或是方法名稱 , 希望使用那些方法的時候. 當然此情況也可以使用 Reflection 解決

---


### Thank you! 

You can find me on

- [GitHub](https://github.com/s0920832252)
- [Facebook](https://www.facebook.com/fourtune.chen)

若有謬誤 , 煩請告知 , 新手發帖請多包涵

# :100: :muscle: :tada: :sheep: 
