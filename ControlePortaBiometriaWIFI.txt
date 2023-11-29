#include <Adafruit_Fingerprint.h>
#include <ESP8266WiFi.h>

// Configuração do sensor de impressão digital
#define FINGERPRINT_SENSOR_RX 2
#define FINGERPRINT_SENSOR_TX 3
Adafruit_Fingerprint finger = Adafruit_Fingerprint(&Serial, FINGERPRINT_SENSOR_RX, FINGERPRINT_SENSOR_TX);

// Configuração do módulo de relé
#define RELAY_PIN 4

// Configuração da rede Wi-Fi
const char *ssid = "Sua Rede WiFi";
const char *password = "Sua Senha WiFi";

void setup() {
  Serial.begin(115200);
  delay(100);

  // Inicializa o sensor de impressão digital
  if (!finger.begin()) {
    Serial.println("Não foi possível encontrar o sensor de impressão digital. Verifique a conexão!");
    while (1);
  }

  // Conecta-se ao Wi-Fi
  Serial.println();
  Serial.println("Conectando ao WiFi...");
  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
    Serial.print(".");
  }

  Serial.println("");
  Serial.println("Conectado ao WiFi");

  // Configuração do pino do relé
  pinMode(RELAY_PIN, OUTPUT);
  digitalWrite(RELAY_PIN, LOW); // Inicialmente, mantenha o relé desligado
}

void loop() {
  // Aguarda uma impressão digital válida
  Serial.println("Coloque o dedo no sensor...");
  if (finger.getImage()) {
    int fingerprintID = finger.fingerID();

    // Verifica se a impressão digital é autorizada
    if (isAuthorized(fingerprintID)) {
      Serial.println("Acesso autorizado. Abrindo a porta...");
      digitalWrite(RELAY_PIN, HIGH); // Liga o relé para abrir a porta
      delay(5000); // Mantém a porta aberta por 5 segundos
      digitalWrite(RELAY_PIN, LOW); // Desliga o relé para fechar a porta
    } else {
      Serial.println("Impressão digital não autorizada. Acesso negado.");
    }
  }

  delay(1000); // Aguarda um segundo antes de verificar novamente
}

bool isAuthorized(int fingerprintID) {
  // Aqui você pode implementar lógica para verificar se a impressão digital é autorizada
  // Neste exemplo, consideraremos que apenas um ID específico é autorizado
  int authorizedID = 1; // Substitua pelo ID autorizado

  return fingerprintID == authorizedID;
}
