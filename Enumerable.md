---
tags: LinQ, LinQ基礎 , C#
---

# LINQ基礎 - Enumerable

### [Enumerable Class](https://docs.microsoft.com/zh-tw/dotnet/api/system.linq.enumerable?view=netframework-4.8)

> 提供一組 static (在 Visual Basic 中為 Shared) 方法，用於查詢實作 IEnumerable<T> 的物件

> 傳回值序列的查詢中所使用的方法並不會耗用目標資料，直到列舉查詢物件。 這稱為 「 延後執行 」。 傳回單一值的查詢中所使用的方法執行，而且會立即耗用目標資料

上面告訴我們三件事情
1. Enumerable 這個靜態類別就是對型別 IEnumerable<T> 進行擴充. 讓所有實作 IEnumerable<T> 的型別都可以使用 Enumerable 內的擴充方法.
2. 方法的回傳型別若不是基礎型別(e.g. IEnumerable<T> ) , 則代表執行此方法 僅是回傳一個物件 , 但尚未開始執行動作. 會直到使用 Foreach 才會開始. - 延後執行 
3. 方法的回傳型別若是基礎型別(e.g. double) , 則會馬上開始走訪.

### Enumerable 的內容

- > 參考91大的blog
![](https://i.imgur.com/TzQDTov.jpg)

由上面這張圖 , 我們可以知道其內部放置三種方法.
- 針對 IEnumerable<T> 的擴充方法
    - Where , Select , Max , Average 等等...
    - ![](https://i.imgur.com/Ccf7Izj.png)

- 針對 IEnumerable 的擴充方法
    - Cast<T> : 把 IEnumerable 中的成員都強制轉型成T , 若轉型失敗 , 會拋出例外.
    - OfType<T> : 把 IEnumerable 中 , 屬於 T 型別的成員都抽出來
    - 透過上述兩個方法處理後 , 其型別都變成 IEnumerable<T> 了 , 因此也可以使用 IEnumerable<T> 的擴充方法.
- Enumerable class 本身的靜態方法
    - Range : 傳入 start 以及 count 兩個變數後 , 回傳一組 IEnumerable<int> 的序列
    - Repeat : 產生一個重複 n 個 T 的序列 (IEnumerable<T>)
    - Empty : 產生一個空的 IEnumerable<T>
  
---

### 總結

- System.Linq 的精華幾乎都在 Enumerable 這個類別上, 因為其針對 IEnumerable 型別進行擴充. ( LinQ大部分的方法都在這個類別內)
- 可以用 foreach 關鍵字進行走訪的型別一定有實作 IEnumerable 或 IEnumerable<T> 這兩個介面. 換句話說 , 可以用 foreach 走訪的型別都可能可以使用 LinQ 的方法進行處理.
- LinQ 的方法( IEnumerable<T> 的擴充方法)就是透過泛型委派的參數（e.g. Func<T1, T2, …Tn, TResult>） , 來簡化繁複的 foreach 迴圈處理。


---

### Thank you! 

You can find me on

- [GitHub](https://github.com/s0920832252)
- [Facebook](https://www.facebook.com/fourtune.chen)

若有謬誤 , 煩請告知 , 新手發帖請多包涵

# :100: :muscle: :tada: :sheep: 
