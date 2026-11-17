#include <DHT.h>

// Pinos dos sensores
#define DHTPIN 4
#define DHTTYPE DHT11

#define PINO_LUZ 32
#define PINO_SOLO 34

DHT dht(DHTPIN, DHTTYPE);

// Valores usados para verificar as condicoes da planta
const int SOLO_SECO = 3000;
const int SOLO_ATENCAO = 2500;
const float TEMP_ATENCAO = 30.0;
const float TEMP_ALTA = 34.0;

void setup() {
  Serial.begin(115200);
  dht.begin();

  analogReadResolution(12);

  Serial.println("================================");
  Serial.println("         AGRO ALERT");
  Serial.println("================================");

  delay(2000);
}

void loop() {

  // Pegando temperatura e umidade do ar
  float temperatura = dht.readTemperature();
  float umidadeAr = dht.readHumidity();

  // Fazendo a leitura dos sensores
  int luz = 0;
  int solo = 0;

  // Fazendo 10 leituras para ter uma media melhor
  for (int i = 0; i < 10; i++) {
    luz += analogRead(PINO_LUZ);
    solo += analogRead(PINO_SOLO);
    delay(10);
  }

  luz = luz / 10;
  solo = solo / 10;

  Serial.println("\n------ AGRO ALERT ------");

  // Mostrando os dados do DHT11
  if (isnan(temperatura) || isnan(umidadeAr)) {
    Serial.println("Erro na leitura do DHT11");
  } else {
    Serial.print("Temperatura: ");
    Serial.print(temperatura);
    Serial.println(" C");

    Serial.print("Umidade do ar: ");
    Serial.print(umidadeAr);
    Serial.println(" %");
  }

  // Mostrando a leitura de luz
  Serial.print("Luz: ");
  Serial.println(luz);

  // Mostrando a leitura do solo
  Serial.print("Umidade do solo: ");
  Serial.println(solo);

  // Verificando a situacao da planta
  bool soloSeco = solo >= SOLO_SECO;
  bool soloAtencao = solo >= SOLO_ATENCAO;

  bool sensorOk = !isnan(temperatura) && !isnan(umidadeAr);

  Serial.print("STATUS: ");

  if (!sensorOk) {
    Serial.println("VERIFICAR SENSOR");
  }
  else if (soloSeco && temperatura >= TEMP_ALTA) {
    Serial.println("POSSIVEL ESTRESSE");
  }
  else if (soloAtencao || temperatura >= TEMP_ATENCAO) {
    Serial.println("ATENCAO");
  }
  else {
    Serial.println("NORMAL");
  }

  Serial.println("------------------------");

  delay(3000);
}
