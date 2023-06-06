// # NodeMcu
//Controlling LED with wifi on NodeMcu
 #include <ESP8266WiFi.h>

const char* ssid = "WIFI_SSID";
const char* password = "WIFI_PASSWORD";

const int ledPin = 2; // ESP8266 üzerindeki GPIO pin numarasını belirleyin

WiFiServer server(80);

void setup() {
  pinMode(ledPin, OUTPUT);
  digitalWrite(ledPin, LOW);

  Serial.begin(115200);

  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
    Serial.println("WiFi bağlantısı başarısız. Yeniden deniyor...");
  }

  server.begin();
  Serial.println("Server başlatıldı!");
  Serial.print("IP Adresi: ");
  Serial.println(WiFi.localIP());
}

void loop() {
  WiFiClient client = server.available();
  
  if (client) {
    Serial.println("Yeni bir istemci bağlandı!");
    while (client.connected()) {
      if (client.available()) {
        String request = client.readStringUntil('\r');
        Serial.println(request);
        
        if (request.indexOf("/LED=ON") != -1) {
          digitalWrite(ledPin, HIGH);
        }
        else if (request.indexOf("/LED=OFF") != -1) {
          digitalWrite(ledPin, LOW);
        }
        
        client.flush();
        
        client.println("HTTP/1.1 200 OK");
        client.println("Content-Type: text/html");
        client.println("");
        client.println("<!DOCTYPE HTML>");
        client.println("<html>");
        
        client.print("LED Durumu: ");
        if (digitalRead(ledPin)) {
          client.print("AÇIK");
        }
        else {
          client.print("KAPALI");
        }
        
        client.println("<br><br>");
        client.println("<a href=\"/LED=ON\">LED'i AÇ</a>&nbsp;");
        client.println("<a href=\"/LED=OFF\">LED'i KAPAT</a><br/>");
        
        client.println("</html>");
        break;
      }
    }
    
    delay(1);
    client.stop();
    Serial.println("İstemci bağlantısı kesildi!");
  }
}
