# Healthcare device:Measuring Body Temperature and Heartbeat

People have always tried to develop devices which are useful to mankind. The reason is obvious since the past,
man has been very successful in using the devices developed to reduce the amount of physical labor needed to do many
tasks. With the introduction of the computer, it became a possibility that devices could also reduce the amount of mental
effort needed for many tasks. But the problem of interchanging data between human beings and computing devices is a
challenging one. In reality, it is very difficult to achieve 100% accuracy. Even humans too will make mistakes when
coming to pattern recognition. Even those who need to program a computer can do their job more effectively with a better
understanding of how a computer system functions, what its various components are, and what are its capabilities and
limitations.

Nowadays there is a demand for an embedded device which contain hardware as well as software. Hardware can
work according to a software instruction. Such as any hardware(sensor, input device) get the value using the software in
the backend and give appropriate result.

### Tools and technology used

* Programming language: Arduino
* Operating system (Development): ​ Linux ubuntu
* IDE: ​ Arduino IDE (Arduino Software)
* Device:​ Arduino UNO R3, Android device
* Other tools: ​ HeartBeat sensor (Part no-KY-039),
Temperature sensor (Part no-DS18B20),
Bluetooth module (Part no-HC-05)
* App inventor: ​ MIT App Inventor 2
* Library: ​ DallasTemperature, OneWire

### Temperature sensor(Part no-DS18B20) :

DSB18B20 is a digital temperature sensor. It Provides 9 to 12 bit temperature reading. It communicates
over one wire bus which means it requires only one data line for communication. It can drive power
directly from the data line. It has the following features:
* Power supply range is 3.3V to 5.0V
* It can measure temperature ranging from -55°C to 125°C
* Unique one wire interface
* ±0.5°C accuracy from –10°C to +85°C
* It converts 12-bit temperature to digital word in 750 ms (max.).
* Applications include thermostatic controls,industrial systems, consumer products,thermometers,
or any thermally sensitive system.

<img src="https://raw.githubusercontent.com/jaysuthar743/Healthcare-device-Measuring-Body-Temperature-and-Heartbeat/master/t.png" alt="alt text" width="250" height="250">


### Heartbeat sensor (Part no-KY-039) :

KY-039 is an analog sensor. KY-039 uses a bright infrared (IR) LED and a phototransistor to detect the
reduced IR transmission or increased IR reflection during the a beat or pulse. It has 3 pins named VCC,
GND and signal. The sensor works as follows: the LED is the light side of the finger and the photo
transistor on the other side of the finger. The photo transistor collects the intensity of the light after
passing through our finger. We choose a very high resistance R1,because most of the light through the
finger is absorbed, It is desirable phototransistor sensitive enough.

<img src="https://raw.githubusercontent.com/jaysuthar743/Healthcare-device-Measuring-Body-Temperature-and-Heartbeat/master/hb.png" width="250" height="250">


### Bluetooth module (Part no-HC-05):

HC-05 Bluetooth Module is an easy to use Bluetooth SPP (Serial Port Protocol) module, designed for
transparent wireless serial connection setup.It's Communications via serial communication which makes
an easy way to interface with controller or PC. HC-05 Bluetooth module provides switching mode
between master and slave mode which means it able to use neither receiving nor transmitting data.

<img src="https://raw.githubusercontent.com/jaysuthar743/Healthcare-device-Measuring-Body-Temperature-and-Heartbeat/master/b.png" width="300" height="350">

### Module specification
The process of health device is mainly divided into following steps:
1. Getting option from app.
2. Getting value from appropriate sensor.
3. Display output on app.

* Getting option from app:

A user will select the options button which are displayed on the app. Options are temperature logo and heartbeat
logo. The app is developed using MIT App Inventor. The app will communicate with the help of Bluetooth.

* Getting value from appropriate sensor:
Device user select temperature or heartbeat from the android app and command is sent to Arduino board using the
Bluetooth. Board will take value from the sensor, do some operation on it to get an output based on the command.
Temperature is measure in celsius and heartbeat is measure in BPM The output will send to an app using the
Bluetooth.

Temperature sensor is connected to arduino in following manner:
```VDD -> 3.3V
GND->GND
S->Digital 5
```

Heartbeat sensor is connected to arduino in following manner:
```VDD->5.0v
GND->GND
S->Analog(A0)
```
Bluetooth module is connected to arduino in following manner:
```VDD->5.0v
GND->GND
RX->TX
TX->RX
```
* Display output on app:

Android app which is developed in MIT app inventor is using to display the value of body temperature and
heartbeat of your body.App will notify the user that his/her body temperature and heartbeat is low, normal or high
using color notifier.

### Code
```

#include <OneWire.h>
#include <DallasTemperature.h>
#define samp_siz 4
#define rise_threshold 4
int temp_sensor = 5;       // Pin DS18B20 Sensor is connected (temperature sensor pin)
int hb_sensorPin=0; // Pin KY-039 sensor is connected (heartbeat sensor pin)
int ans =0;
String voice="";
OneWire oneWirePin(temp_sensor);
DallasTemperature sensors(&oneWirePin);
void setup(void)
{
  Serial.begin(9600);
  sensors.begin();
}

void loop()
{
  float temperature = 0,heartbeat=0;      //Variable to store the temperature and heartbeat
  int m=0;
  // variable used in heartbeat
  float reads[samp_siz],sum,last,reader,start,first,second,third,before,print_val;
  long int now,ptr,last_beat;
  bool rising;
  int rise_count,n;
   while(Serial.available())
  {
    char c = Serial.read();
    voice+=c; 
    Serial.println(voice); 
  }
  long st=millis();
  long et = st;
  if (voice == "Temp")
  {
 while((et-st)<30000)
  {
    sensors.requestTemperatures(); 
    temperature  += sensors.getTempCByIndex(0);
    m++;
    et=millis();
  }
  ans = temperature/m;
  Serial.println(ans);  
  voice="";
  }
 
  if (voice=="Heart")
  {
      for (int i = 0; i < samp_siz; i++){
        reads[i] = 0;
    }
    sum = 0;
    ptr = 0;

    while((et-st)<30000)
    {
      // calculate an average of the sensor
      // during a 20 ms period (this will eliminate
      // the 50 Hz noise caused by electric light
      n = 0;
      start = millis();
      reader = 0.;
      do
      {
        reader += analogRead (hb_sensorPin);
        n++;
        now = millis();
      }
      while (now < start + 20);  
      reader /= n;  // we got an average
      
      // Add the newest measurement to an array
      // and subtract the oldest measurement from the array
      // to maintain a sum of last measurements
      sum -= reads[ptr];
      sum += reader;
      reads[ptr] = reader;
      last = sum / samp_siz;
      if (last > before)
      {
        rise_count++;
        if (!rising && rise_count > rise_threshold)
        {
          rising = true;
          first = millis() - last_beat;
          last_beat = millis();
          heartbeat += 60000. / (0.4 * first + 0.3 * second + 0.3 * third);
          m++;
          third = second;
          second = first;
          
        }
      }
      else
      {
        // Ok, the curve is falling
        rising = false;
        rise_count = 0;
      }
      before = last;
      
      
      ptr++;
      ptr %= samp_siz;
      et=millis();
    }
  ans = heartbeat/m;
  Serial.println(ans);  
  voice=""; 
  }
}
```

### Circuit diagram

<img src="https://raw.githubusercontent.com/jaysuthar743/Healthcare-device-Measuring-Body-Temperature-and-Heartbeat/master/cd.png" width="500" height="500">
