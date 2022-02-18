# 專題-機台連網與中控顯示整合
**※透過C#撰寫的機台程式碼與python撰寫的繪圖程式碼屬於實驗室財產無法開源**

該專題與另外三組合作，
 * 我負責將機台控制沖頭的電壓數據透過MQTT publish給樹莓派架的Broker，並在本地端用matplotlib呈現數據。
 * 第一組負責在沒憑證的情況下攔截封包並偽造封包給Broker，在有憑證的情況下證明無法解讀封包。
 * 第二組負責架設Broker並透過python接收數據以及INSERT到Mysql。
 * 第三組負責將數據透過Power BI進行遠端呈現。
![image](https://user-images.githubusercontent.com/88305396/152937965-6471c28a-95a7-4bce-9e5d-22051040e11f.png)

# 數據傳輸
該機台與[沖壓裁切機收料系統](https://github.com/Elmo6000/project_2)同機台，此機台的沖頭透過電壓的高低進行沖壓控制，當電壓值越大沖頭會越高，反之則越低。
  
為了確認是否達到沖壓的上下極限，有一個function負責不停地擷取電壓值，所以我在該function將擷取到的電壓值及當前時間分別放入兩個陣列，
然後在當次沖壓後進行寫檔及publish並將兩個陣列清空。
  
在MQTT傳輸的部分，我使用的是NuGet內的[M2Mqtt](https://www.nuget.org/packages/M2Mqtt/)套件，
由於只能publish byte[]，所以我將1000筆由 ',' 和 '-' 作為分割符號所組成的字串encode成byte[]，然後publish給Broker。
  
在Broker的部分，我先將收到的內容decode，然後透過`split('-')`分成多筆資料並將每筆資料包成tuple放入陣列，再透過`executemany(sql, data)`
一次INSERT到Mysql。

# 近端數據顯示
在近端數據顯示的部分我是直接對先前寫的檔案做讀檔，我寫了一個function會針對指定參數(第幾次沖壓)去做繪圖。  

首先會將讀取的列分割成行，然後比對是否為指定的參數，若是就會將電壓及時間放到對應的陣列，該指定參數的第一筆時間會特別特別紀錄，
在該參數的電壓與時間都放入陣列後，為了讓每次沖壓的時間起始都為0(繪製多筆沖壓時能夠將線型疊在一起)，會將時間陣列的每筆時間減掉第一筆，
最後就是利用matplotlib將兩個陣列呈現。  

另外也可篩掉不想要的數據，如下圖僅呈現小於2的電壓。
![圖片1](https://user-images.githubusercontent.com/88305396/153044901-eaa9560a-3163-47fa-bfbd-ec89ab0da6ae.png)

# 展示影片
首先會先確認資料表是空白的，我針對選取的三片物件執行沖壓，然後開了一個連到樹莓派(Broker)的遠端視窗，我們可以看到當完成一次沖壓Broker會將資料INSERT給Mysql，然後我會開啟spyder繪圖並存檔於桌面，最後回去查看INSERT後的資料庫。

https://user-images.githubusercontent.com/88305396/152934630-a8bcb0dd-3ba9-4c17-8785-edb21f599877.mp4
