# 软件工程导论小组展示作业
---
## 一个基于红外热释电与声、光传感器的遥控感应灯
## 所有硬件均来Arduino中级套件
代码如下：
```c
#include <IRremote.h>

int ledPin=13;         //LED灯的引脚为13
int RECV_PIN=10;      //红外接受模块引脚为10
int buttonPin=5;     // 设红外感受器的引脚为5
int voice = 2;  //设声音传感器的引脚为2
int lightPin = 0;        //设光敏传感器的引脚为9

boolean ledState=HIGH;           //ledState用来存储LED的状态
boolean funcOn[3] = {false, false, false},alwaysMode = false;     //funcON数组用于存储三个函数的开闭状态,alwaysMode存储常亮模式开启状态
boolean voiceState = false, lightState = false, warmState = false;
IRrecv irrecv(RECV_PIN);
decode_results results;

int val=0;       //将初始值设为0， 在安静环境下熄灭

//判断
int isLightOn();
int iswarmon();

//感应器函数
int voicedetect();
int lightdetect();
int looptempturedetect();

void setup() {
  // put your setup code here, to run once:
  Serial.begin(9600);
  irrecv.enableIRIn();
  pinMode(ledPin,OUTPUT);         //设置LED为输出状态
  pinMode(buttonPin,INPUT);        //button输入
}

void loop() {
  // put your main code here, to run repeatedly:
  if(irrecv.decode(&results)){
    Serial.println(results.value,HEX);
    if(results.value==0xFD00FF){       //如果接受到的是电源键的信号，常亮模式反转,所有函数开闭状态随灯反转
      digitalWrite(ledPin, LOW);
      alwaysMode = !alwaysMode;
      Serial.print("alwaysMode: ");
      Serial.println(alwaysMode);
    }else if(results.value==0xFD08F7){       //如果接受到的是按键1的信号，开闭声感应模式
      voiceState = false;
      funcOn[0] = !funcOn[0];
      Serial.print("voicedetect: ");
      Serial.println(funcOn[0]);
    }else if(results.value==0xFD8877){       //如果接受到的是按键2的信号，开闭光感应模式
      lightState = false;
      funcOn[1] = !funcOn[1];
      Serial.print("lightdetect: ");
      Serial.println(funcOn[1]);
    }else if(results.value==0xFD48B7){       //如果接受到的是按键3的信号，开闭红外感应模式
      warmState = false;
      funcOn[2] = !funcOn[2];
      Serial.print("looptempturedetect: ");
      Serial.println(funcOn[2]);
    }
    irrecv.resume();
  }else{
    
  }
   if(alwaysMode){                         //如果常亮模式开启，关闭其他函数，打开led
    digitalWrite(ledPin, HIGH);
  }else{
    if(funcOn[0]){
      voiceState = voicedetect();
      Serial.print("Voice Value: ");
      Serial.println(val);   //在串口观测声音值
      delay(100);
    }
    if(funcOn[1]){
      lightState = lightdetect();
      Serial.print("Light Value: ");
      Serial.println(analogRead(lightPin));        //串口监视器查看当前光敏传感器返回的模拟信号值，测试用，可删去
      delay(100);
    }
    if(funcOn[2]){
      warmState = looptempturedetect();
      Serial.print("Warm Value: ");
      Serial.println(digitalRead(buttonPin));
      delay(100);
    }
    if(voiceState || lightState || warmState){
      digitalWrite(ledPin, HIGH);
      delay(2000);
    }else{
      digitalWrite(ledPin, LOW);
    }
  }
}




//光感
int lightdetect()
{
  if(analogRead(lightPin) > 10){             //当光敏传感器传入模拟信号大于10
    return 1;                               //即有光时，返回1，否则返回0
  }else{
    return 0;
  }
}

//红外探测器
int looptempturedetect(){;
  if(digitalRead(buttonPin)>0)
  {
    return 1;                             
  }
  return 0;
}

//声音探测
int voicedetect() {
  val = analogRead(voice);
  if(val > 50)
  {
    return 1;    //如果声音大于三十（可修改），灯亮              
  }
  return 0;
}
```