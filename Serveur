#include <NTPClient.h>
#include <SPI.h>
#include <SD.h>
#include <ESP8266WiFi.h>
#include <WiFiUdp.h>
#include <C:\Users\maintenance\Documents\Arduino\espServer_3\secrets.h>

WiFiUDP udp;
WiFiUDP NTP_udp;

NTPClient timeClient(NTP_udp, "europe.pool.ntp.org", 7200, 60000);

int previousMillis = 0;
String previousHours = "0";

String code1 = "xx";
String code2 = "xx";

void setup() {
  Serial.begin(115200);
  Serial.println();
  
  pinMode(CS_Pin, OUTPUT);
  if (SD.begin(CS_Pin) != 1) {
    Serial.println("Erreur avec la carte SD !");
    return;
  } else {
    Serial.println("Connexion avec la carte SD établie");
  }
  
  udp.begin(localPort);

  WiFi.mode(WIFI_AP_STA);
  Serial.print("Configuration du Point d'Accès : ");
  Serial.println(WiFi.softAPConfig(local_IP, gateway, subnet) ? "OK" : "Erreur !");
  Serial.print("Point d'Accès WiFi : ");
  Serial.println(WiFi.softAP(ssid, password) ? "OK" : "Erreur !");

  Serial.print("Connexion au WiFi : ");
  Serial.println(WiFi.begin(STAssid, STApassword) ? "OK" : "Erreur !");

  Serial.print(F("Serveur UDP : "));
  Serial.println(WiFi.softAPIP());

  timeClient.begin();

}
void loop() {
  int currentMillis = millis();
  int packetSize = udp.parsePacket();
  bool error = false;

  timeClient.update();
  
  //Fonction quand on reçoit un packet
  if(packetSize > 0){

    String ID;
    String Data;
    String param_Value[20];
    String param[20];
    String value[20];

    IPAddress remoteIP = udp.remoteIP();

    //Lecture du packet
    if(packetSize) {
      int len = udp.read(packetBuffer, 255);
      if(len > 0) packetBuffer[len] = 0;
    }
    
    //udp.stop();
    String _packetBuffer = String(packetBuffer);

    //Fonction de séparation des éléments du message
    if(_packetBuffer.indexOf('?') > -1){
      ID = _packetBuffer.substring(0, _packetBuffer.indexOf('?'));
      Data = _packetBuffer.substring(_packetBuffer.indexOf('?') + 1);
    } else if(_packetBuffer.indexOf('!') > -1){
      String _packetBuffer = String(packetBuffer);
      ID = _packetBuffer.substring(0, _packetBuffer.indexOf('!'));
      Data = _packetBuffer.substring(_packetBuffer.indexOf('!') + 1);
    } else {
      Serial.println("----Code : 0xFF----");
      code1 = "0x";
      code2 = "FF";
      error = true;
      return;
    }

    int dataIndexed = 0;
    for(int i = 0; Data.indexOf('&') != Data.lastIndexOf('&'); i++){
      Data.remove(0, 1);
      param_Value[i] = Data.substring(0, Data.indexOf('&'));
      Data.remove(0, Data.indexOf('&'));
      dataIndexed++;
    }

    for(int i = 0; i < dataIndexed; i++){
      param[i] = param_Value[i].substring(0, param_Value[i].indexOf('='));
      value[i] = param_Value[i].substring(param_Value[i].indexOf('=') + 1);
    }

    Serial.print(" Client : ");
    Serial.print(ID);
    Serial.print(" / ");
    Serial.println(remoteIP);

//-------------------------------------Traitement du packet-------------------------------------
    String filepath;
    File cFile;
    //Fonction quand le client s'initialise auprès du serveur
    if(param[0] == "payload" && param[1] == NULL && _packetBuffer.indexOf('?') > -1 && _packetBuffer.indexOf('!') == -1 && error == false){
      Serial.println(" Initialisation du Client");
      int payload_INT = 0;
      String payload[20];
      for(int i = 0; value[0].indexOf('|') != value[0].lastIndexOf('|'); i++){
        value[0].remove(0, 1);
        payload[i] = value[0].substring(0, value[0].indexOf('|'));
        value[0].remove(0, value[0].indexOf('|'));
        payload_INT++;
      }
      if(!SD.exists(ID)){
        Serial.println(" Création du dossier Client");
        SD.mkdir(ID);
        filepath = ID + "/payload";
        SD.mkdir(filepath);
        filepath = ID + "/IP.txt";
        cFile = SD.open(filepath, FILE_WRITE);
        cFile.print(remoteIP.toString());
        cFile.close();
        for(int i = 0; i < payload_INT; i++){
          filepath = ID + "/payload/" + payload[i];
          SD.mkdir(filepath);
          filepath += "/recipients.txt";
          cFile = SD.open(filepath, FILE_WRITE);
          cFile.close();
        }
        Serial.println("----Code : 1xC0----");
        code1 = "1x";
        code2 = "C0";
      } else {
        Serial.println(" Mise à jour du dossier Client");

        filepath = ID + "/payload";
        if(SD.exists(filepath)) SD.mkdir(filepath);

        filepath = ID + "/IP.txt";
        if(SD.exists(filepath)) SD.remove(filepath);

        cFile = SD.open(filepath, FILE_WRITE);
        cFile.print(remoteIP);
        cFile.close();

        for(int i = 0; i < payload_INT; i++){
          filepath = ID + "/payload/" + payload[i];
          if(!SD.exists(filepath)){
            Serial.print(" Dossier manquant : ");
            Serial.println(payload[i]);

            SD.mkdir(filepath);
            filepath += "/recipients.txt";
            cFile = SD.open(filepath, FILE_WRITE);
            cFile.close();
          }
        }
        Serial.println("----Code : 1xC1----");
        code1 = "1x";
        code2 = "C1";
      }
    } else if(param[0] != "payload" && _packetBuffer.indexOf('!') > -1 && _packetBuffer.indexOf('?') == -1 && error == false) {
      Serial.println(" Requête d'action Client");
      if(!SD.exists(ID)){
        Serial.println("----Code : 0xF0----");
        code1 = "0x";
        code2 = "F0";
        error = true;
        goto errorOut;
      }
      for(int i = 0; i < dataIndexed; i++){
        String filepath1 = ID + "/payload/" + param[i];
        String filepath2 = ID + "/payload/" + param[i] + "/recipients.txt";
        if(!SD.exists(filepath1) || !SD.exists(filepath2)){
          Serial.println("----Code : 0xFF----");
          code1 = "0x";
          code2 = "FF";
          error = true;
          goto errorOut;
        }
      }
      for(int i = 0; i < dataIndexed; i++){
        filepath = ID + "/payload/" + param[i] + "/recipients.txt";
        cFile = SD.open(filepath, FILE_READ);
        String c = "";
        while(cFile.available() > 0){
          c += cFile.readString();
        }

        String recipients[20];
        int recipients_INT = 0;
        for(int j = 0; c.indexOf('|') != c.lastIndexOf('|'); j++){
          c.remove(0, 1);
          recipients[j] = c.substring(0, c.indexOf('|'));
          c.remove(0, c.indexOf('|'));
          recipients_INT++;
        }

        for(int u = 0; u < recipients_INT; u++){
          String pSend = param[i];
          String vSend = value[i];
          filepath = recipients[u] + "/IP.txt";
          cFile = SD.open(filepath, FILE_READ);
          String h = "";
          while(cFile.available() > 0){
            h += cFile.readString();
          }
          Serial.println(h);
          IPAddress ip;
          ip.fromString(h);
          udp.beginPacket(ip, localPort);
          udp.print("Server?");
          udp.write("?");
          udp.write("&");
          udp.print(pSend);
          udp.write("=");
          udp.print(vSend);
          udp.write("&");
          udp.write("\r\n");
          udp.endPacket();
        }
      }
      Serial.println("----Code : 1xC2----");
      code1 = "1x";
      code2 = "C2";
    }

    errorOut:
    udp.beginPacket(remoteIP, localPort);
    udp.print("Server?");
    udp.print(code1);
    udp.print(code2);
    udp.write("\r\n");
    udp.endPacket();
  }

  
  String filepath;
  String day = String(timeClient.getDay());
  String hours = String(timeClient.getHours());
  hours += String(timeClient.getMinutes());


  if(previousHours != hours){
    previousHours = hours;
    Serial.println(hours);

    filepath = "scenario/" + day + "/" + hours;
    if(SD.exists(filepath)){
      File cFile;
      for(int i = 1; SD.exists(("scenario/" + day + "/" + hours + "/" + String(i) + ".txt")); i++){
        String ID;
        String charge;
        filepath = "scenario/" + day + "/" + hours + "/" + String(i) + ".txt";
        cFile = SD.open(filepath, FILE_READ);
        String c = "";
        while(cFile.available()){
          c += cFile.readString();
        }
        cFile.close();

        c.remove(0,1);
        ID = c.substring(0, c.indexOf("|"));
        c.remove(0, c.indexOf("|"));
        charge = c.substring(c.indexOf("|"), c.lastIndexOf("|"));
        charge.remove(0,1);

        filepath = ID + "/IP.txt";
        c = "";
        if(!SD.exists(filepath)){
          Serial.println("----Code : 2xF1----");
        }
        cFile = SD.open(filepath, FILE_READ);
        while(cFile.available()){
          c += cFile.readString();
        }
        cFile.close();

        IPAddress IP = IP.fromString(c);
        Serial.println(udp.beginPacket(IP, localPort) ? "OK" : "Erreur !");
        udp.print("Server");
        udp.print("!")
        udp.print(charge);
        udp.write("\r\n");
        Serial.println(udp.endPacket() ? "OK" : "Erreur !");
      }
    }
  }
}
