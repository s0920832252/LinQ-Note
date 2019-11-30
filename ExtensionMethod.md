---
tags: LinQ, LinQ基礎 , C#
---

# LINQ基礎 - Extension Method(擴充方法)

### [擴充方法](https://docs.microsoft.com/zh-tw/dotnet/csharp/programming-guide/classes-and-structs/extension-methods)

> Extension Method 在 .NET 3.5後開始提供

> .NET 3.5擴充方法可讓您在**現有類型中「加入」方法，而不需要建立新的衍生類型、重新編譯，或是修改原始類型**。擴充方法是一種特殊的靜態方法，但是會將它們當成擴充類型上的執行個體方法來呼叫。

> 擴充方法會定義為**靜態方法**，但**透過執行個體方法語法呼叫**。 擴充方法的**第一個參數會指定方法作業所在的類型**，而且**參數前面會加上 this 修飾詞**。 您**必須使用 using 指示詞將命名空間明確匯入至原始程式碼**，擴充方法才會進入範圍中。

> 擴充方法是定義在非巢狀且非泛型的靜態類別

由上面的引用 , 我們可知
1.  不需修改原來型別或重新編譯 , 就可以新增方法到現有型別上 , 並且可以使用.
2.  擴充方法必須是靜態類別中的靜態方法
3.  擴充方法的第一個參數代表此方法所要依附的型別
4.  擴充方法的第一個參數必須要在參數前面加上this 修飾詞.
5.  擴充方法不可以定義在巢狀型別或泛型型別中。

### 宣告&呼叫方式
1. 宣告自己的 namespace -> 對照 : ConsoleApp1
2. 宣告自己的 Static class -> 對照 : CityDateTimeExtension
3. 宣告自己的 static function -> 對照 : NextWeekDay
4. 第一個參數前要加 this , 參數型別代表要依附的型別 -> 對照 : DateTime
5. function 回傳型別 , 代表此擴充方法回傳的型別. (與一般函數同) -> 對照 : DateTime

    以下新增一個擴充方法 NextWeekDay (回傳下一個星期的 DateTime ), 將其依附在 DateTime 類別. (原本 DateTime 類別沒有這個方法)
    ```
    namespace ConsoleApp1
    {
        public static class CityDateTimeExtension
        {
            public static DateTime NextWeekDay(this DateTime from, DayOfWeek end)
            {
                int addDays = (int)end - (int)from.DayOfWeek;
                return from.AddDays(addDays <= 0 ? addDays + 7 : addDays);
            }
        }
    }
    ```
    以下是測試程式碼 - 印出下一個禮拜四的日期時間.
    在 **namespace ConsoleApp1 範圍內 , DateTime 可以呼叫NextWeekDay 這個方法.**
    ```
    namespace ConsoleApp1
    {
        class Program
        {
            static void Main(string[] args)
            {
                Console.WriteLine(DateTime.Now.NextWeekDay(DayOfWeek.Thursday));
                Console.ReadKey();
            } 
        }
    }
    ```
---

### 呼叫方式種類

1. 靜態方法呼叫 : 擴充方法本來就是靜態方法 , 因此可以透過其型別的名稱去呼叫.
2. 擴充方法呼叫(物件) : 透過其依附型別的物件去呼叫
   
兩種呼叫方式 , 結果是相同的. 以下分別顯示兩種方法的呼叫方式.

##### 圖示說明
* namespace 為 ConsoleApp1
* 自定義一個 City 類別
* CityExtension 類別為靜態類別
* CityExtension 類別被定義在 ConsoleApp1 內
* 擴充方法 MaskName 依附在自定義類別 City 上
    ```C#
    public class City
    {
         public string Name = "CityIsGoodPerson";
    }

    public static class CityExtension
    {
         public static int _plainTextLen = 6;
         public static string _asterisk = "*";
         public static string MaskName(this City city)
         {
              var orinialName = city.Name;
              var result = orinialName;
              if (orinialName.Length > _plainTextLen)
              {
                   var plainText = orinialName.Substring(0, _plainTextLen);
                   var asteriskCount = orinialName.Length - _plainTextLen;
                   var encodingText = Enumerable.Repeat(_asterisk, asteriskCount).Aggregate((l, r) => l + r);
                   result = plainText + encodingText;
              }
              return result;
         }
    }
    ```

* 以下是呼叫程式碼
    ```C#
    static void Main(string[] args)
    {
         var cityMan = new City();
         Console.WriteLine("透過物件呼叫擴充方法結果 : " + cityMan.MaskName());
         Console.WriteLine("靜態呼叫擴充方法結果 : " + CityExtension.MaskName(cityMan));
         Console.ReadKey();
    }
    ```
* 輸出結果    
![](https://i.imgur.com/tJ6hPOl.png)

上述那個例子有兩點值得探討. 
1. 擴充方法是屬於該靜態類別的 , 因此若是權限允許的話 , 自然也能存取靜態成員(實務上需要避免) 
    ```C#
    static void Main(string[] args)
    {    // 測試程式碼
         Console.WriteLine($"_plainTextLen = {CityExtension._plainTextLen}");
         CityExtension._plainTextLen = 10;
         Console.WriteLine($"_plainTextLen = {CityExtension._plainTextLen}");
         Console.ReadKey();
    }
    ```
     輸出結果如下圖.
    ![](https://i.imgur.com/AKfX1Vd.png)
2. 即使是自定義類別 City , 也可以依附. 只要此擴充方法的靜態類別認識這個類別即可.

---

#### 結論
- 擴充方法的存在能夠讓程序員在不修改原本既有型別(包含介面)的前提下 , 就可以將新的方法內容附加到該型別之上.
- **擴充方法的使用範圍在其被定義的 namespace 中** , 因此只要 using 該 namespace , 也可以使用該擴充方法.
- 擴充方法無法修改不能內部資訊 , 所以不會破壞原有型別的封裝性. 固可以針對型別來設計方法 , 將方法封裝 , 讓原本的型別在使用上可以更加便利.
- **擴充方法不宜濫用** , 原因是會讓類別內的方法不一定被定義在該類別之中 , 此現象可能會造成**維護上的困難**.
- **擴充方法針對的是型別 !** , 而型別與物件導向的繼承有關 , 因此若是擴充方法依附的型別被其他子型別繼承或實現 , 則子型別也同樣可以使用該擴充方法.
- LinQ to Objects 的方法 , 其大部分都寫在 System.Linq(namespace) 裡面的Enumerable(static class) 中 , 且這些方法中大部分都依附在 IEnumerable<T> 這個**泛型介面**上. 也就是說只要 using System.Linq 這個命名空間 , 且繼承或實現 IEnumerable<T>這個介面的型別 , 都可以使用 Linq 的擴充方法. 例如 List<T> , IList<T> , Dictionary<TKey,TValue> 等等常見的泛型資料集合 , 都有實作 IEnumerable<T>. 另外有一些非泛型的資料集合 , 則通常都有實作 IEnumerable 這個**介面**(也就是可以使用 foreach )

---
### 補充
問題 : 假設現在某類別有一個方法A , 後來新增擴充方法A依附在該類別.
兩個方法的名稱 , 傳入參數型別 , 數量各方面都相同. 這樣是否會產生衝突或是編譯問題!?

回答 : 不會產生問題(編譯會過) , 但擴充方法不會被執行. 執行期間 , 系統會優先執行類別本身的方法.

---

### Thank you! 

You can find me on

- [GitHub](https://github.com/s0920832252)
- [Facebook](https://www.facebook.com/fourtune.chen)

若有謬誤 , 煩請告知 , 新手發帖請多包涵

# :100: :muscle: :tada: :sheep: 
