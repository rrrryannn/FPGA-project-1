/*
seg_output 7段顯示器
seg_COMM 控制哪個七段顯示器做動
DATA_R,DATA_G,DATA_B LED矩陣的RGB
LED_COMM 控制LED矩陣哪一個column要做動
Star_indication 吃到無敵星星時的提示LED燈
Bird_Jump 按下Bird就會jump(往上一格)

seg3,seg2,seg1,seg0 四個七段顯示器要顯示的數字
BCD_0, BCD_1, BCD_2, BCD_3 計算分數用的(皆為mod 10)
LED8x8 在此array產生障礙物
Bird 在此array產生Bird
star 在此array產生無敵星星

Barrier_Ceiling_random, Barrier_Ground_random, Star_random作為產生障礙物及無敵星星的種子
R_GND, R_CIL, R_STAR 為障礙物及星星的實際產生位置
Is_dead 判斷是否死亡
Star_generate 判斷是否該產生星星
Super_Star 判斷現在是不是無敵的狀態

a, b為障礙物及星星所在的column
i, j作為for loop的index使用
Star_Position 表示現在無敵星星row位置
Bird_Position 表示現在Bird row位置
Star_count 是在吃到無敵星星後用來計算無敵時間的
*/

module FlappyBird(output reg [6:0] seg_output,
						output reg [3:0] seg_COMM,
						output reg [7:0] DATA_R, DATA_G, DATA_B,
                  output reg [3:0] LED_COMM,
						output reg Star_indication,
						input				  Bird_Jump, CLK, Reset);
						
	reg [6:0] seg3, seg2, seg1, seg0;
	reg [3:0] BCD_0, BCD_1, BCD_2, BCD_3;
	reg [7:0] LED8x8 [7:0], Bird [7:0], star[7:0];
	reg [2:0] Barrier_Ground_random, Barrier_Ceiling_random, R_GND, R_CIL, Star_random, R_STAR;
	reg Is_dead = 1'b0, Star_generate = 1'b0, Super_Star = 1'b0;
	integer a, b, i, j, Star_Position, Bird_Position, Star_count;
	
	seven_seg S0(A0, B0, C0, D0, E0, F0, G0, BCD_0[3], BCD_0[2], BCD_0[1], BCD_0[0]);
	seven_seg S1(A1, B1, C1, D1, E1, F1, G1, BCD_1[3], BCD_1[2], BCD_1[1], BCD_1[0]);
	seven_seg S2(A2, B2, C2, D2, E2, F2, G2, BCD_2[3], BCD_2[2], BCD_2[1], BCD_2[0]);
	seven_seg S3(A3, B3, C3, D3, E3, F3, G3, BCD_3[3], BCD_3[2], BCD_3[1], BCD_3[0]);
	
	divfreq(CLK_1Hz, 25000000, CLK);
	divfreq(CLK_3Hz, 8333333, CLK);
	divfreq(CLK_4Hz, 6250000, CLK);
	divfreq(CLK_8Hz, 3125000, CLK);
	divfreq(CLK_100Hz, 250000, CLK);
	divfreq(CLK_1kHz, 25000, CLK);
	
	initial begin
		//segment initialization
		BCD_0 = 4'b0000;
		BCD_1 = 4'b0000;
		BCD_2 = 4'b0000;
		BCD_3 = 4'b0000;
		seg_COMM = 4'b1111;
		seg_display_count = 0;
		
		//LED matrix initialization
		LED_COMM = 4'b1000;
		DATA_R = 8'b11111111;
		DATA_B = 8'b11111111;
		DATA_G = 8'b11111111;
		
		LED8x8[0] = 8'b11111111;
		LED8x8[1] = 8'b11111111;
		LED8x8[2] = 8'b11111111;
		LED8x8[3] = 8'b11111111;
		LED8x8[4] = 8'b11111111;
		LED8x8[5] = 8'b11111111;
		LED8x8[6] = 8'b11111111;
		LED8x8[7] = 8'b11111111;
		
		star[0] = 8'b11111111;
		star[1] = 8'b11111111;
		star[2] = 8'b11111111;
		star[3] = 8'b11111111;
		star[4] = 8'b11111111;
		star[5] = 8'b11111111;
		star[6] = 8'b11111111;
		star[7] = 8'b11111111;
			
		Bird[0] = 8'b11101111;
		Bird[1] = 8'b11101111;
		Bird[2] = 8'b11111111;
		Bird[3] = 8'b11111111;
		Bird[4] = 8'b11111111;
		Bird[5] = 8'b11111111;
		Bird[6] = 8'b11111111;
		Bird[7] = 8'b11111111;
			
		Star_count = 0; //吃到無敵星星後的時間counter
		Bird_Position = 4; //預設鳥在LED matrix上的第4個raw
			
		//障礙物所在Column
		a = 7;
			
		//無敵星星所在Column
		b = 7;
			
		//無敵星星所在raw
		Star_Position = -1;
			
		//障礙物生成種子
		//由下往上的障礙物
		Barrier_Ground_random = (5*Barrier_Ground_random + 3)%16;
		R_GND = 3+ Barrier_Ground_random % 5; // 3~7
			
		//由上往下的障礙物
		Barrier_Ceiling_random = (5*Barrier_Ceiling_random + 1)%16;
		R_CIL = Barrier_Ceiling_random % 3; //0~2
			
		if(R_GND == 3) R_CIL = 0;
		else if(R_GND == 4) R_CIL = R_CIL % 2;
			
		//無敵星星生成種子
		Star_random = (5*Star_random + 2)%16;
		R_STAR = Star_random % 8;
	end
	
	//產生障礙物
	always @(posedge CLK_4Hz) begin
		if(Reset == 1'b1) begin
			//Is_dead = 1'b0;
			LED8x8[0] = 8'b11111111;
			LED8x8[1] = 8'b11111111;
			LED8x8[2] = 8'b11111111;
			LED8x8[3] = 8'b11111111;
			LED8x8[4] = 8'b11111111;
			LED8x8[5] = 8'b11111111;
			LED8x8[6] = 8'b11111111;
			LED8x8[7] = 8'b11111111;
				
			star[0] = 8'b11111111;
			star[1] = 8'b11111111;
			star[2] = 8'b11111111;
			star[3] = 8'b11111111;
			star[4] = 8'b11111111;
			star[5] = 8'b11111111;
			star[6] = 8'b11111111;
			star[7] = 8'b11111111;
			
			a = 7;
			b = 7;
			Star_generate = 1'b0;
			
			Barrier_Ground_random = (5*Barrier_Ground_random + 3)%16;
			R_GND = 3+ Barrier_Ground_random % 5; // 0~5
			
			Barrier_Ceiling_random = (5*Barrier_Ceiling_random + 1)%16;
			R_CIL = Barrier_Ceiling_random % 3;
			if(R_GND == 3) R_CIL = 0;
			else if(R_GND == 4) R_CIL = R_CIL % 2;
			
			Star_random = (5*Star_random + 2)%16;
			R_STAR = Star_random % 8;	
		end
		
		if(Is_dead == 1'b0) begin //若非死亡
			if(a == 7)	begin //障礙物在LED8x8的 111 的column時
				//產生一行的障礙物(由上往下)(範圍在raw 0 到 row 3)
				for(j = 0; j < 4; j = j + 1) begin
					if(j <= R_CIL)
						LED8x8[a][j] = 1'b0;
				end
				//產生一行的障礙物(由下往上)(範圍在raw 3 到 row 8)
				for(i = 3; i < 8; i = i + 1) begin
					if(i >= R_GND)
						LED8x8[a][i] = 1'b0;
				end
				a = a-1;
			end
			else if (a >= 0 && a < 7)	begin
				//將此column的障礙物移到下一個column並刪除此column的障礙物
				for(j = 0; j < 4; j = j + 1) begin
					if(j <= R_CIL) begin
						LED8x8[a+1][j] = 1'b1;
						LED8x8[a][j] = 1'b0;
					end
				end
					
				for(i = 3; i < 8; i = i + 1) begin
					if(i >= R_GND) begin
						LED8x8[a+1][i] = 1'b1;
						LED8x8[a][i] = 1'b0;
					end
				end
					//在障礙物在column 4時，產生一個無敵星星
				if(a == 4) Star_generate = 1'b1;
				a = a-1;							
			end
			else if(a == -1) 	begin
					//刪除在000的障礙物
				for(j = 0; j < 4; j = j + 1) begin
					if(j <= R_CIL)
						LED8x8[a+1][j] = 1'b1;
				end
					
				for(i = 3; i < 8; i = i + 1) begin
					if(i >= R_GND)
						LED8x8[a+1][i] = 1'b1;
				end
					
				//產生新的障礙物亂數
				Barrier_Ground_random = (5*Barrier_Ground_random + 3)%16;
				R_GND = 3+ Barrier_Ground_random % 5;
					
				Barrier_Ceiling_random = (5*Barrier_Ceiling_random + 1)%16;
				R_CIL = Barrier_Ceiling_random % 3;
				if(R_GND == 3) R_CIL = 0;
				else if(R_GND == 4) R_CIL = R_CIL % 2;
				a = 7;
			end
				
			//產生無敵星星
			if(Star_generate == 1'b1) begin
				if(b == 7) begin
					Star_Position = b;
					star[b][R_STAR] = 1'b0;
					b = b-1;
					Star_Position = Star_Position - 1;
				end
				else if (b >= 0 && b < 7) begin
					star[b+1][R_STAR] = 1'b1;
					star[b][R_STAR] = 1'b0;
					b = b-1;
					Star_Position = Star_Position - 1;
				end
				else if(b == -1) begin
					star[b+1][R_STAR] = 1'b1;
					
					//產生新的無敵星星亂數
					Star_random = (5*Star_random + 2)%16;
					R_STAR = Star_random % 8;
					Star_generate = 1'b0;
					Star_Position = -1;
					b = 7;
				end
			end
		end			
		else begin//若死亡則產生死亡提示畫面
			LED8x8[0] = 8'b11111111;
			LED8x8[1] = 8'b01111101;
			LED8x8[2] = 8'b10111101;
			LED8x8[3] = 8'b11011111;
			LED8x8[4] = 8'b11011111;
			LED8x8[5] = 8'b10111101;
			LED8x8[6] = 8'b01111101;
			LED8x8[7] = 8'b11111111;
		end
	end

	//Bird Jump
	always @(posedge CLK_3Hz) begin
		if(Reset == 1'b1) begin
			Bird_Position = 4;
			Is_dead = 1'b0;
			Super_Star = 1'b0;
			Star_count = 0;
			
			Bird[0] = 8'b11101111;
			Bird[1] = 8'b11101111;
			Bird[2] = 8'b11111111;
			Bird[3] = 8'b11111111;
			Bird[4] = 8'b11111111;
			Bird[5] = 8'b11111111;
			Bird[6] = 8'b11111111;
			Bird[7] = 8'b11111111;

		end
	
		else begin
			//若吃到無敵星星
			if(Super_Star) begin
				Star_count = Star_count + 1;
				if(Star_count >= 6) begin
					Super_Star = 1'b0;// 2sec
					Star_count = 0;
				end
			end
			
			//Jump時Bird向上一格
			if(Bird_Jump == 1'b1 && Bird[0] != 8'b11111110) begin
				Bird[0] = Bird[0] >> 1;
				Bird[0] = {1'b1, Bird[0][6:0]};
				Bird[1] = Bird[1] >> 1;
				Bird[1] = {1'b1, Bird[1][6:0]};
				Bird_Position = Bird_Position - 1;
			end
			//Bird Falling
			else if (Bird[0] != 8'b01111111) begin
					Bird[0] = Bird[0] << 1;
					Bird[0] = {Bird[0][7:1], 1'b1};
					Bird[1] = Bird[1] << 1;
					Bird[1] = {Bird[1][7:1], 1'b1};
					Bird_Position = Bird_Position + 1;
			end
			
			//碰撞障礙物判定
			//若沒有無敵星星，且Bird與障礙物碰撞，則判定死亡
			if(Super_Star == 1'b0 && (LED8x8[1][Bird_Position] == 1'b0 || LED8x8[0][Bird_Position] == 1'b0))
				Is_dead = 1'b1;
			//若Bird與無敵星星發生碰撞，則代表吃到無敵星星
			else if(star[1][Bird_Position] == 1'b0 || star[0][Bird_Position] == 1'b0)
				Super_Star = 1'b1;
		end
	end
	
	//輸出至LED matrix
	reg [2:0] count;
	reg Bird_count;
	always @(posedge CLK_1kHz) begin
		if(Reset == 1'b1) begin
			DATA_R = 8'b11111111;
			DATA_B = 8'b11111111;
			DATA_G = 8'b11111111;
			LED_COMM = 4'b1000; 
		end
		if(count == 3'b111)
			count <= 3'b000;
		else
			count <= count + 1'b1;
				
		LED_COMM = {1'b1, count};
		if(Is_dead == 1'b0) begin
			//if(Super_Star == 1'b0) begin
				DATA_G <= Bird[count];
				DATA_B <= LED8x8[count];
				DATA_R <= star[count];
			//end
			/*else begin
				DATA_R <= LED8x8[count];
				DATA_G <= LED8x8[count];
				DATA_B <= LED8x8[count];
						
				DATA_G <= Bird[count];
				DATA_R <= star[count];	
			end*/
		end
				
		else begin
			DATA_R <= LED8x8[count];
			DATA_G <= 8'b11111111;
			DATA_B <= 8'b11111111;		
		end	
	end
	
	
	//segment進位
	always @(posedge CLK_100Hz)
		begin	
		
			if(Reset) begin
				BCD_0 = 4'b0000;
				BCD_1 = 4'b0000;
				BCD_2 = 4'b0000;
				BCD_3 = 4'b0000;
			end
			else if(Is_dead == 1'b0)
				begin
				 if(BCD_0 == 9 && BCD_1 == 9 && BCD_2 == 9 && BCD_3 == 9)
					begin
						BCD_0 <= 0;
						BCD_1 <= 0;
						BCD_2 <= 0;
						BCD_3 <= 0;
					end
				else if(BCD_0 == 9 && BCD_1 == 9 && BCD_2 == 9)
					begin						
						BCD_3 <= BCD_3 + 1'b1;
						BCD_2 <= 0;
						BCD_1 <= 0;
						BCD_0 <= 0;
					end
				else if(BCD_0 == 9 && BCD_1 == 9)
					begin
						BCD_2 <= BCD_2 + 1'b1;
						BCD_1 <= 0;
						BCD_0 <= 0;
					end
				else if(BCD_0 == 9)
					begin
						BCD_1 <= BCD_1 + 1'b1;
						BCD_0 <= 0;
					end
				else
					BCD_0 <= BCD_0 + 1'b1;
			end
		end
		
	//segment 視覺暫留
	reg [2:0] seg_display_count = 2'b0;
	always @(posedge CLK_1kHz) begin
		seg0 = {G0,F0,E0,D0,C0,B0,A0};
		seg1 = {G1,F1,E1,D1,C1,B1,A1};
		seg2 = {G2,F2,E2,D2,C2,B2,A2};
		seg3 = {G3,F3,E3,D3,C3,B3,A3};
		if(seg_display_count > 4)
			seg_display_count <= 0;
			
		if(seg_display_count == 1) begin
				seg_COMM <= 4'b1110; // display seg0
				seg_output <= seg0;
		end
		else if(seg_display_count == 2) begin
				seg_COMM <= 4'b1101; // display seg1
				seg_output <= seg1;
		end
		else if(seg_display_count == 3) begin
				seg_COMM <= 4'b1011; // display seg2
				seg_output <= seg2;
		end
		else if(seg_display_count == 4) begin
				seg_COMM <= 4'b0111; // display seg3
				seg_output <= seg3;
		end		
		seg_display_count <= seg_display_count + 1'b1;
	end
	
	
	//若吃到無敵星星，用LED燈閃爍提示
	always @(posedge CLK_8Hz) begin
		if(Is_dead) Star_indication = 0;
		if(Super_Star)
			Star_indication = ~Star_indication;
		else
			Star_indication = 0;
	end
	
endmodule

module seven_seg(output reg a, b, c, d, e, f, g, input A, B, C, D);
	always @(A, B, C, D)
		case({A, B, C, D})
			4'b0000:	{a,b,c,d,e,f,g} = 7'b0000001;
			4'b0001: {a,b,c,d,e,f,g} = 7'b1001111;
			4'b0010:	{a,b,c,d,e,f,g} = 7'b0010010;
			4'b0011:	{a,b,c,d,e,f,g} = 7'b0000110;
			4'b0100:	{a,b,c,d,e,f,g} = 7'b1001100;
			4'b0101:	{a,b,c,d,e,f,g} = 7'b0100100;
			4'b0110:	{a,b,c,d,e,f,g} = 7'b0100000;
			4'b0111:	{a,b,c,d,e,f,g} = 7'b0001111;
			4'b1000:	{a,b,c,d,e,f,g} = 7'b0000000;
			4'b1001:	{a,b,c,d,e,f,g} = 7'b0000100;
			default: {a,b,c,d,e,f,g} = 7'b1111111;
		endcase
endmodule

//除頻器
module divfreq(output reg CLK_div, input [24:0] Herz, input CLK);
	reg [24:0] count = 25'b0;
	always @(posedge CLK)
		if(count > Herz) begin
				count <= 25'b0;
				CLK_div = ~CLK_div;
		end
		else
			count <= count + 1'b1;
endmodule
