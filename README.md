# air-monitor

미세먼지 알리미
===
![alt tag](https://github.com/nadir93/air-monitor/blob/master/res/IMG_20140418_145712.jpg?raw=true)
###개요 
*미세먼지 농도를 주기적으로 체크하고  미세먼지 주의나 경보상황 발생시 사용자에게 알려준다*

*먼지데이타 샘플링 : 10초*

*측정단위 : ug/m3*

###구성요소  
- arduino mega adk - [uno](http://arduino.cc/en/Main/arduinoBoardUno) or [leonardo](http://arduino.cc/en/Main/arduinoBoardLeonardo) 둘다 가능  
- [ethernet shield](http://arduino.cc/en/Main/ArduinoBoardEthernet)
- prototype shield - 빵판포함  
- dustSensor [GP2Y1010AU0F](http://www.aliexpress.com/item/GP2Y1010AU0F-100-NEW-SHARP-Optical-Dust-Sensor-GP2Y1010/1347390254.html) - *USD 7.38*
- [220 uF Capacitor](http://www.aliexpress.com/item/50-pcs-Aluminum-Radial-Electrolytic-Capacitor-220uF-25V/1143386595.html)
- 150 Ω Resistor
- push server - [mosquitto](http://mosquitto.org) on [Raspberry Pi](http://www.raspberrypi.org/)
- push client - [android](https://github.com/adflowweb/mqtt/tree/master/pushClient)

###설치 

- 회로도

![alt tag](http://arduinodev.woofex.net/wp-content/uploads/sharpFromDoc.png)

- 핀연결 
	 
| Sharp Dust Sensor | Attached To |
| -------------      | ------------- |
| 1 (V-LED)          | 5V Pin (150 Ohm in between)  |
| 2 (LED-GND)        | GND Pin |
| 3 (LED)	     | Digital Pin 2
| 4 (S-GND)	     | GND Pin
| 5 (Vo)             | Analog Pin A0
| 6 (Vcc)	     | 5V Pin (Direct)

###코드
- 먼지센서 

 [*먼지센서모듈*](https://github.com/Trefex/arduino-airquality/tree/master/Module_Dust-Sensor) 
```cpp
  digitalWrite(ledPower,LOW); // LED 전원인가, 측정시작
  delayMicroseconds(samplingTime); // 0.28ms 기다림 
  voMeasured = analogRead(measurePin); // 먼지데이타 읽기 
  delayMicroseconds(deltaTime); // 0.04ms 기다림 
  digitalWrite(ledPower,HIGH); // LED 전원중지  
  delayMicroseconds(sleepTime); // 최소 측정 주기가 10ms 이므로 9680microSeconds를 기다린다(측정에 320us사용함)
  // 0 - 5.0V mapped to 0 - 1023 integer values
  // recover voltage
  calcVoltage = voMeasured * (5.0 / 1024.0);
  // 먼지센서 스펙상 전압이 0.58 이상되어야함 
  if(calcVoltage > 0.58823529)
  {
    // linear eqaution taken from http://www.howmuchsnow.com/arduino/airquality/
    // Chris Nafis (c) 2012
    dustDensity = 0.17 * calcVoltage - 0.1;
  }
```

- mqtt client

 [*mqtt 아두이노 라이브러리*](https://github.com/knolleary/pubsubclient)

 **default message size : 128bytes**

 **default keepalive interval : 15초**
```cpp
// 먼지 측정 데이터를 푸시한다. 
client.publish((char*)"users/1c45de7cc1daa896bfd32dc", msg);
```
  
###문제
- *push message 크기가 230 바이트 넘어가면  오버플로우발생* 

###해결방안 
- *최소한의 raw data만 만들어서 node.js로 넘기고 푸시 로직은 node.js 에서 처리한다*

 *arduino는 먼지센싱만하고 푸시는 직접 하지않는다*

###추후작업
- cosm 연동
- node.js 연동 

 *arduino <--> node.js(publisher) --> mqttServer(mosquitto) --> androidClient(subscriber)*

 *node.js --> cosm*

###참고 
- *http://www.howmuchsnow.com/arduino/airquality/*
- *http://sensorapp.net/?p=479*
- *http://arduinodev.woofex.net/2012/12/01/standalone-sharp-dust-sensor/*
- *https://www.sparkfun.com/datasheets/Sensors/gp2y1010au_e.pdf*
- *http://knolleary.net/arduino-client-for-mqtt/*

