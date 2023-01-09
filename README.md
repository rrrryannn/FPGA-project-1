# 此為 NCNU FPGA 期末專案 github readme 檔案範例:memo:

# FPGA-project-1
### Authors:109321067黃世昕 110321008張瑞恩 110321010張庭綸 

#### Input/Output unit:<br>
* 8x8LED矩陣，用來顯示鳥闖關遊戲畫面，分為按下reset的初始畫面、闖關畫面及死亡畫面 

  * reset:
  ![S__41558025](https://user-images.githubusercontent.com/122210192/211285088-90134631-90bc-49a6-a26b-0b82a0dbb1e2.jpg)
  * 闖關:
  ![S__41558026](https://user-images.githubusercontent.com/122210192/211285186-1f4da61f-4548-451b-b5dd-6f6be343b17d.jpg)
  * 死亡:
  ![S__41558022](https://user-images.githubusercontent.com/122210192/211285226-0f886add-5003-4f91-8c8e-2b5b11dbc966.jpg)


* 七段顯示器:用來計算時間。
 ![S__41558024](https://user-images.githubusercontent.com/122210192/211285562-53c13885-2944-4a5b-adac-f1e829a8475d.jpg)


* LED陣列:用來顯示是否獲得無敵星星效果
![S__41558030](https://user-images.githubusercontent.com/122210192/211287654-26ec006f-cce2-4502-a9ba-9170dd5df7f3.jpg)


#### 功能說明:<br>
遊戲的玩法為：操控綠色的小鳥飛行且避開藍色的管道；如果小鳥碰到了障礙物，遊戲就會結束。如果吃到紅色的無敵星星可以無視障礙物兩秒。

#### 程式模組說明:<br>
`seg_output` 7段顯示器

`seg_COMM`控制哪個七段顯示器做動

`DATA_R,DATA_G,DATA_B` LED矩陣的RGB

`LED_COMM` 控制LED矩陣哪一個column要做動

`Star_indication` 吃到無敵星星時的提示LED燈

`Bird_Jum`p 按下Bird就會jump(往上一格)

```
module FlappyBird(
                  output reg [6:0] seg_output,
		  output reg [3:0] seg_COMM,
		  output reg [7:0] DATA_R, DATA_G, DATA_B,
                  output reg [3:0] LED_COMM,
                  output reg Star_indication,
		  input  Bird_Jump, CLK, Reset);
```

#### 程式變數說明:<br>
`seg3,seg2,seg1,seg0` 四個七段顯示器要顯示的數字

`BCD_0, BCD_1, BCD_2, BCD_3` 計算分數用的(皆為mod 10)

`LED8x8` 在此array產生障礙物，`Bird` 在此array產生Bird，`star` 在此array產生無敵星星

`Barrier_Ceiling_random, Barrier_Ground_random, Star_random`作為產生障礙物及無敵星星的種子
`R_GND, R_CIL, R_STAR` 為障礙物及星星的實際產生位置

`Is_dead` 判斷是否死亡
`Star_generate` 判斷是否該產生星星
`Super_Star` 判斷現在是不是無敵的狀態

`a, b`為障礙物及星星所在的column
`i, j`作為for loop的index使用
`Star_Position` 表示現在無敵星星row位置
`Bird_Position` 表示現在Bird row位置
`Star_count` 是在吃到無敵星星後用來計算無敵時間的


*** 請說明各 I/O 變數接到哪個 FPGA I/O 裝置，例如: button, button2 -> 接到 4-bit SW
*** 請加強說明程式邏輯


