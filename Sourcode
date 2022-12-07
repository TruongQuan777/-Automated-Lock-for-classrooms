#include <NTPClient.h>
#include <Servo.h>
#include <ESPAsyncTCP.h>
#include <ESPAsyncWebServer.h>
#include <WiFiUDP.h>
#include <FS.h>
const char* ssid = "yourssid";
const char* password =  "yourpassword";
AsyncWebServer server(80);
// Current time
unsigned long currentTime = millis();
// Previous time
unsigned long previousTime = 0; 
// Define timeout time in milliseconds (example: 2000ms = 2s)
const long timeoutTime = 2000;


/*NTPClient's variable*/
const long utcOffsetInSeconds = 3600;
char daysOfTheWeek[7][12] = {"Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"};
WiFiUDP ntpUDP;
NTPClient timeClient(ntpUDP, "pool.ntp.org");


/*Electronic devices*/
Servo Servo1;
int servoPin = D1;
const int Out = D2;
const int Trig = D5;
const int Echo = D7;


/*Variable Relating to the Schedule*/
boolean Schedule=false;
const int Close = 180;
const int Open = 0;
String input1="";
String nameinput1="input1";
String input2="";
String nameinput2="input2";
bool EmergencyOpen=false;
bool EmergencyClose=false;
String valuepassword="thepassword";
String namepassword="password";
String check;


/*A function to convert the NTPClient's returned day into a readable one*/
String reconstruct(String input)
{
  for(int i=0;i<=input.length()-1;i++)
  {
    if(input[i]=='-')
    {
        int s=0;
        for(int j=i+1;j<=input.length()-1;j++)
        {
          if(input[j]=='-')
          {
            break;
          }
          s++;
        }
        if(s==2)
        {
        }
        else
        {
        input=input.substring(0,i+1)+"0"+input.substring(i+1);
        }
    }
  }
  return input;
}
 
 
void setup(){
  // Begin the Serial Port
  Serial.begin(115200);
  // Initialize a NTPClient to get time
  timeClient.begin(); 
  // Set offset time in seconds to adjust for your timezone, for example:
  timeClient.setTimeOffset(25210);

  
  /* Set up the electronic devices*/
  pinMode(Out, INPUT);
  pinMode(Trig,OUTPUT);
  pinMode(Echo,INPUT);
  Servo1.attach(servoPin);


  /*Initiate WiFi Connection*/
  WiFi.begin(ssid, password);
  Serial.println("");
  // Wait for connection
  while (WiFi.status() != WL_CONNECTED) {
    delay(3000);
    Serial.print(".");
  }
  Serial.println("");
  Serial.print("Connected to ");
  //Print your WiFi's SSID (might be insecure)
  Serial.println(ssid);
  Serial.print("IP address: ");
  Serial.println(WiFi.localIP());

  
  /*Begin the SPIFFS file system*/
  if(!SPIFFS.begin())
  {
    // Serious problem
    Serial.println("SPIFFS Mount failed");
  } else {
    Serial.println("SPIFFS Mount succesfull");
  }


/*Format the response for the HTTP request*/
//Send index1.html file (Signin Page)
server.on("/",HTTP_GET, [](AsyncWebServerRequest *request){  
request->send(SPIFFS,"/index1.html",String());});
//send style1.css file (Signin Page)
server.on("/style1.css",HTTP_GET, [] (AsyncWebServerRequest *request){ 
request->send(SPIFFS,"/style1.css",String());});
// send image
server.on("/download.png", HTTP_GET, [] (AsyncWebServerRequest *request) {
request->send(SPIFFS, "/download.png", "image/png");});


/*Handle the /get request
*if the value send is for the input 1 => Update the new value to the input 1 & Send the index2.html (the Schedule page) file
*if the value send is for the input 1 => Update the new value to the input 2 & Send the index2.html (the Schedule page) file
*if the value send is for the password and is wrong => Still send the index1.html(Signin page) file
*if the value send is for the passowrd and is right => Send the index2.html(Schedule page) file
*/
server.on("/get",HTTP_GET, [] (AsyncWebServerRequest *request){
if (request->hasParam(nameinput1)) {
      input1 = request->getParam(nameinput1)->value();  
      input1.replace('T','-');
      input1. replace(':','-');
      Serial.println("input1 "+input1);
      request->send(SPIFFS,"/index2.html",String()); // Thoat ra khoi vong lap (password dung /input1/input2 => Truy cap file html2 giao dien
      }
else if(request->hasParam(nameinput2)){
      input2=request->getParam(nameinput2)->value();  //Value 2 => chinh gia tri
      input2.replace('T','-');
      input2.replace(':','-');
      Serial.println("input2 "+input2);
      request->send(SPIFFS,"/index2.html",String()); // Thoat ra khoi vong lap (password dung /input1/input2 => Truy cap file html2 giao dien
      }
else if(request->hasParam(namepassword)){
      if(request->getParam(namepassword)->value()!=valuepassword ){  //Password sai => dien form lai (file HTML1)
      request->send(SPIFFS,"/index1.html",String());
      
      }
      else {
      request->send(SPIFFS,"/index2.html",String());// Thoat ra khoi vong lap (password dung /input1/input2 => Truy cap file html2 giao dien
      
      }
  }

});  
// Send style2.css file
server.on("/style2.css", HTTP_GET, [] (AsyncWebServerRequest *request) { // Truy cập file CSS2 giao diện
request->send(SPIFFS, "/style2.css", "text/css");});


/*Handle the emergency buttons
*Emergency Open => Set the EmergencyOpen variable to true,...close to false
*Emergency Close => Set the EmergencyOpen variable to false,.... close to true
*Disable Emergency => Set both value to false
*/
server.on("/EmergencyOpen",HTTP_GET, [] (AsyncWebServerRequest *request){ // 3 button
  EmergencyOpen=true;
  EmergencyClose=false;
  request->send(SPIFFS,"/index2.html",String());
});
server.on("/EmergencyClose",HTTP_GET, [] (AsyncWebServerRequest *request){
  EmergencyClose=true;
  EmergencyOpen=false;
  request->send(SPIFFS,"/index2.html",String());
});
server.on("/DisableEmergency",HTTP_GET, [] (AsyncWebServerRequest *request){
  EmergencyClose=false;
  EmergencyOpen=false;
  request->send(SPIFFS,"/index2.html",String());
});


  //start web server
  server.begin();
  //Just stating things
  Serial.println("HTTP server started");
}
void loop() {


/*Handle the NTPClient*/
  timeClient.update();
  time_t epochTime = timeClient.getEpochTime();
  String formattedTime = timeClient.getFormattedTime();
  int currentHour = timeClient.getHours();
  int currentMinute = timeClient.getMinutes();
  struct tm *ptm = gmtime ((time_t *)&epochTime); 
  int monthDay = ptm->tm_mday;
  int currentMonth = ptm->tm_mon+1;
  int currentYear = ptm->tm_year+1900;
 // get the currentDate as a String
  String currentDate = String(currentYear)+"-"+ String(currentMonth)+"-"+String(monthDay) + "-" +String(currentHour)+"-"+String(currentMinute);
  currentDate=reconstruct(currentDate);
 // Print the currentDate through the serial monitor (Can be omitted)
  Serial.print("CurrentDate ");
  Serial.println(currentDate);
  
  
 //*Update schedule*//
  if(input1=="" ||input2=="")
  {
    Schedule=false;
  }
  else
  {
    if(currentDate.compareTo(input1) >=0 && currentDate.compareTo(input2) <0)
    {
      Schedule=true;
    }
    else
    {
      Schedule=false;
    }
  }
  // Print the values through the serial monitor (can be omitted)
  Serial.print("Start date: ");
  Serial.println(input1);
  Serial.print("End date ");
  Serial.println(input2);
  Serial.print("Schedule status: ");
  Serial.println(Schedule);
  Serial.print("Emergency Open: ");
  Serial.println(EmergencyOpen);
  Serial.print("Emergency Close: ");
  Serial.println(EmergencyClose);

  
//*Control the electronic devices//
//if EmergencyOpen is true => Open
if(EmergencyOpen){
  Servo1.write(Open);
}
//if EmergencyClose is true => Close
else if (EmergencyClose)
{
  Servo1.write(Close);
}
else
{
//if Schedule is false => The lock will work normally
  if(!Schedule) { 
digitalWrite(Trig,0);
delayMicroseconds(2);
digitalWrite(Trig,1);
delayMicroseconds(5);
digitalWrite(Trig,0);
if (pulseIn(Echo,HIGH)/2/29.412 <= 10){
  Servo1.write(Open); //Detect motion => Open
} else if (digitalRead(Out)==LOW){
  delay(1000);
  Servo1.write(Close); // No motion => Close
}
if (digitalRead(Out)==HIGH){
  Servo1.write(Open); //Door Open => Open the lock
}
} 
//if Schedule is false => The lock will work normally
else { 
  Servo1.write(Open);
}

}

}
