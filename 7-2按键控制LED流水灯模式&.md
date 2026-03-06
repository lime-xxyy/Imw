# 按键控制LED流水灯模式

## 配置寄存器函数

打开stc-isp，在右栏找到**定时器计算器**，系统频率调为**12兆**，定时长度调为**1毫秒**，选择定时器为**定时器0**，定时器模式把16位重载调整为**16位**，定时器时钟选择**12T**，点击**生成C代码**并且**复制代码**

粘贴后删除下列代码

```
AUXR &= 0x7F;
```

再在后面补上下列代码

```
ET0=1;
EA=1;
PT0=0;
```

补上main函数

```
void main(){
	Timer0_Init();
	while(1){
		
	}
}
```

以及中断函数

```
unsigned int T0Count;
void Timer0_Routine() interrupt 1
{
	T0Count++;
	if(T0Count>=1000){
		T0Count=0;
		P2_0=~P2_0;
	}
}
```

最后在`T0Count++;`前补上（否则不会闪烁）

```
	TL0 = 0x18;		
	TH0 = 0xFC;		
```

然后将定时器模块化（具体步骤见5-1模块化编程笔记）

在main.c文件里面把`unsigned int T0Count;`挪入`void Timer0_Routine() interrupt 1`函数内部并且在下列代码之前

	TL0 = 0x18;
	TH0 = 0xFC;	

再改成`static unsigned int T0Count;`目的是下一次使用不丢失数值

*如果不加，值不会保存，下次使用就是另一个值了*

## 独立按键文件

独立按键也需要模块化

独立按键需要Delay函数，因此可以将之前的Delay文件复制过来（详见笔记5-1"模块化编程"）

以及独立按键代码原理以及操控LED灯代码原理详见笔记"独立按键控制LED灯显示二进制"

	unsigned char Key()
	{
		unsigned char KeyNumber=0;
		
	if(P2_1==0){Delay(20);while(P2_1==0);Delay(20);KeyNumber=1;}
	if(P2_0==0){Delay(20);while(P2_0==0);Delay(20);KeyNumber=2;}
	if(P2_2==0){Delay(20);while(P2_2==0);Delay(20);KeyNumber=3;}
	if(P2_3==0){Delay(20);while(P2_3==0);Delay(20);KeyNumber=4;}
	
	return KeyNumber;
	}
## 循环移位函数介绍

`_crol_(a,1)`为循环移位函数，存在于`<INTRINS.H>`函数库中

`a=_crol_(a,1)`可以把a向左移动一位，最左的位会被补到最右边

（相反，`a=_cror_(a,1)`会向右移一位）

使用该函数可以快捷实现流水灯

## 具体实现代码

```
unsigned char KeyNum,LEDMode;

void main(){
P2=0xFE;
	Timer0Init();
	while(1){
		KeyNum=Key();
		if(KeyNum!=0){
			if(KeyNum==1){
				LEDMode++;
				if(LEDMode>=2){LEDMode=0;}
			}
		}
	}
}
```

定义KeyNum（按键键码）和LEDMode（LED灯模式）

`KeyNum=Key();`用于收取键码

`if(KeyNum!=0)`收取键码后会检测出来

`if(LEDMode>=2){LEDMode=0;}`用于保证LEDMode的值为0或1

```
``void Timer0_Routine() interrupt 1`
`{`
	`static unsigned int T0Count;`
	`TL0 = 0x18;		//设置定时初值`
	`TH0 = 0xFC;		//设置定时初值`
	`T0Count++;`
	`if(T0Count>=500){`
		`T0Count=0;`
		`if(LEDMode==0){P2=_crol_(P2,1);}`
		`if(LEDMode==1){P2=_cror_(P2,1);}`
	}`
```

每次按下键都会改变LEDMode，`其中LEDMode==0`会使LED左移，`LEDMode==1`会使LED右移