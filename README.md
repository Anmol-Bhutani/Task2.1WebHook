#include <WiFiNINA.h>      
#include <ThingSpeak.h>    
#include <DHT.h>           

#define DHTPIN 2           // DHT sensor pin
#define DHTTYPE DHT11      // DHT sensor type

DHT dht(DHTPIN, DHTTYPE);

#define SECRET_SSID "inder's Galaxy"
#define SECRET_PASS "fesecurity"
#define SECRET_CH_ID 2625140
#define SECRET_WRITE_APIKEY "I4OACFZVVSD5NEZY"

char ssid[] = SECRET_SSID;
char pass[] = SECRET_PASS;

WiFiClient client;
unsigned long channelID = SECRET_CH_ID;
const char* apiKey = SECRET_WRITE_APIKEY;

void setup() {
  Serial.begin(9600);
  while (!Serial) {
    ; // Wait for serial port to connect
  }

  dht.begin();

  Serial.print("Connecting to ");
  Serial.print(ssid);
  WiFi.begin(ssid, pass);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("Connected to WiFi");

  ThingSpeak.begin(client);
}

void loop() {
  delay(2000);

  float humidity = dht.readHumidity();
  float temperature = dht.readTemperature();

  if (isnan(humidity) || isnan(temperature)) {
    Serial.println("Failed to read from DHT sensor!");
    return;
  }

  Serial.print("Humidity: ");
  Serial.print(humidity);
  Serial.print("%  Temperature: ");
  Serial.print(temperature);
  Serial.println("°C");

  ThingSpeak.setField(1, humidity);
  ThingSpeak.setField(2, temperature);

  int responseCode = ThingSpeak.writeFields(channelID, apiKey);
  if (responseCode == 200) {
    Serial.println("Channel update successful.");
  } else {
    Serial.print("Problem updating channel. HTTP error code ");
    Serial.println(responseCode);
  }
}
