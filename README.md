#include <WiFi.h>
#include <WiFiClientSecure.h>
#include <UniversalTelegramBot.h>
#include <Adafruit_Sensor.h>
#include <Adafruit_BME680.h>
#include <HTTPClient.h>
#include <ArduinoJson.h>
#include <Wire.h>

// --- 1. YOUR CREDENTIALS ---
const char* ssid = "RedmiA3x";
const char* password = "Kvng22hh"; 
const char* botToken = "7694550682:AAHKr_Y239toYKuP9hq6uQ5IdN_wI8enegQ";
const char* chatId = "8588235670";
const char* owmApiKey = "528c0f0810576f7f5fc7ccb5deef5b65";

// --- 2. MINI APP SETTINGS ---
const char* miniAppURL = "https://kvnghh22.github.io/Greenguard/"; 

// --- 3. HARDWARE & THRESHOLDS ---
#define BATTERY_PIN 34    
const float TEMP_HIGH = 28.0; 
const float TEMP_LOW = 3.0;   

Adafruit_BME680 bme;
WiFiClientSecure client;
UniversalTelegramBot bot(botToken, client);

// --- 4. TIMERS ---
unsigned long lastSensorRead = 0;
const long sensorInterval = 30000; 
unsigned long lastForecastCheck = 0;
const long forecastInterval = 3600000; 

// --- 5. SETUP (THE BRAIN STARTUP) ---
void setup() {
  Serial.begin(115200);
  delay(1000); 
  Serial.println("\n--- GREENGUARD DEBUG BOOT ---");
  
  Wire.begin(41, 40); 
  if (!bme.begin(0x77)) {
    Serial.println("FATAL: BME680 Sensor Not Found!");
    while (1);
  }
  Serial.println("Sensor Online.");

  bme.setTemperatureOversampling(BME680_OS_8X);
  bme.setHumidityOversampling(BME680_OS_2X);
  bme.setPressureOversampling(BME680_OS_4X);
  bme.setIIRFilterSize(BME680_FILTER_SIZE_3);
  bme.setGasHeater(320, 150); 

  WiFi.begin(ssid, password);
  client.setInsecure(); 
  
  Serial.print("Connecting to WiFi");
  while (WiFi.status() != WL_CONNECTED) { 
    delay(500); 
    Serial.print("."); 
  }
  Serial.println("\nWiFi CONNECTED!");

  // Test Telegram
  String welcome = "✅ Greenguard System Online!\nClick below for your dashboard.";
  String keyboardJson = "[[\"{\\\"text\\\": \\\"Open Dashboard\\\", \\\"web_app\\\": {\\\"url\\\": \\\"" + String(miniAppURL) + "\\\"}}\"]]";
  
  if (bot.sendMessageWithReplyKeyboard(chatId, welcome, "", keyboardJson, true)) {
    Serial.println("Telegram Message Sent Successfully!");
  } else {
    Serial.println("Telegram Message FAILED.");
  }
}

// --- 6. LOOP (THE MAIN WATCHMAN) ---
void loop() {
  int numNewMessages = bot.getUpdates(bot.last_message_received + 1);
  while (numNewMessages) {
    handleNewMessages(numNewMessages);
    numNewMessages = bot.getUpdates(bot.last_message_received + 1);
  }

  if (millis() - lastSensorRead > sensorInterval) {
    monitorEnvironment();
    lastSensorRead = millis();
  }

  if (millis() - lastForecastCheck > forecastInterval) {
    getWeatherForecast();
    lastForecastCheck = millis();
  }
  delay(1000);
}

// --- 7. HELPER FUNCTIONS (THE WORKERS) ---

void monitorEnvironment() {
  if (!bme.performReading()) return;
  float temp = bme.temperature;
  Serial.print("Current Temp: "); Serial.println(temp);
  
  if (temp > TEMP_HIGH) bot.sendMessage(chatId, "🔥 ALERT: Too Hot! " + String(temp) + "°C", "");
  if (temp < TEMP_LOW) bot.sendMessage(chatId, "❄️ ALERT: Frost! " + String(temp) + "°C", "");
}

void handleNewMessages(int numNewMessages) {
  for (int i=0; i<numNewMessages; i++) {
    String text = bot.messages[i].text;
    if (text == "/status") {
      bme.performReading();
      String msg = "📊 *Greenguard Status*\nTemp: " + String(bme.temperature) + "°C\nHum: " + String(bme.humidity) + "%\nLocation: Preston";
      bot.sendMessage(chatId, msg, "Markdown");
    }
  }
}

void getWeatherForecast() {
  HTTPClient http;
  String url = "http://api.openweathermap.org/data/2.5/forecast?q=Preston,GB&units=metric&appid=" + String(owmApiKey);
  http.begin(url);
  if (http.GET() > 0) {
    JsonDocument doc;
    deserializeJson(doc, http.getStream());
    float rainProb = doc["list"][0]["pop"];
    if (rainProb > 0.7) bot.sendMessage(chatId, "🌧️ Rain forecast for Preston!", "");
  }
  http.end();
}
