#include <CAN_config.h>
#include <ESP32CAN.h>
#include "BluetoothSerial.h"

#if !defined(CONFIG_BT_ENABLED) || !defined(CONFIG_BLUEDROID_ENABLED)
#error Bluetooth is not enabled! Please run `make menuconfig` to and enable it
#endif

BluetoothSerial SerialBT;


// - - - - - - - - - - - - - - INIT  - - - - - - - - - - - - -
CAN_device_t CAN_cfg;
unsigned long previousMillis = 0;
const int rx_queue_size = 10;
uint8_t count = 0;

//Fréquence d'envoie des messages CAN en millisecondes
const int interval = 20;

int pinAccelerateur = 36;
int pinOut = 2;
int id_du_vesc = 0;

int CAN_EN = 21;

CAN_frame_t tx_frame;

bool start = 1;
bool start48 = 1;

bool A, B, C, D, E, F, G, H;

char envoi[100];

int STATE = 0;

// - - - - - - - - - - - - - STRUCTURES - - - - - - - - - - - -
struct message_status_1 {
  uint32_t ERPM_lu;     // 4 octets
  uint16_t Current_lu;  // 2 octets
  uint16_t Duty_lu;     // 2 octets
};
message_status_1 status48;
message_status_1 status79;

struct data {
  unsigned long int valeur_ERPM;
  unsigned long int valeur_ERPM_precedente;

  unsigned long int valeur_courant;
  unsigned long int valeur_courant_precedente;

  unsigned long int valeur_duty;
  unsigned long int valeur_duty_precedente;
};
data data_79;
data data_48;

// - - - - - - - - - - - - PROTOTYPAGE - - - - - - - - - - - -
void bluetooth_lecture_et_action();
void can_write_classique(int, int, unsigned long);
void can_receive_to_test();

// - - - - - - - - - - - - - SETUP - - - - - - - - - - - - - -
void setup() {
  Serial.begin(9600);

  // - - - - - - - Bluetooth

  SerialBT.begin("ESP32 - Techdemo V1.2");
  Serial.println("ESP32 LMX Bikes démarré, appairage possible \n");

  // - - - - - - - CAN
  Serial.println("Démarrage du CAN ... \n");
  CAN_cfg.speed = CAN_SPEED_500KBPS;
  CAN_cfg.tx_pin_id = GPIO_NUM_22;  // GPIO 22 = TX
  CAN_cfg.rx_pin_id = GPIO_NUM_4;   // GPIO 4 = RX

  CAN_cfg.rx_queue = xQueueCreate(rx_queue_size, sizeof(CAN_frame_t));
  ESP32Can.CANInit();
  Serial.println("Bus CAN lancé à 500 kbps \n");

  // - - - - - - - PINMODE
  pinMode(pinAccelerateur, INPUT);
  pinMode(pinOut, OUTPUT);
  pinMode(27, OUTPUT);

  // - - - - - - - INIT
  pinMode(CAN_EN, OUTPUT);
  digitalWrite(CAN_EN, LOW);

  status48.ERPM_lu = 0xFFFFFFFF;
  status48.Current_lu = 0xFFFF;
  status48.Duty_lu = 0xFFFF;

  status79.ERPM_lu = 0xFFFFFFFF;
  status79.Current_lu = 0xFFFF;
  status79.Duty_lu = 0xFFFF;

  data_79.valeur_ERPM_precedente = 0x0;
  data_79.valeur_ERPM = 0x0;
  data_79.valeur_courant_precedente = 0x0;
  data_79.valeur_courant = 0x1;
  data_79.valeur_duty_precedente = 0x0;
  data_79.valeur_duty = 0x1;

  data_48.valeur_ERPM_precedente = 0x0;
  data_48.valeur_ERPM = 0x1;
  data_48.valeur_courant_precedente = 0x0;
  data_48.valeur_courant = 0x1;
  data_48.valeur_duty_precedente = 0x0;
  data_48.valeur_duty = 0x1;
}

// - - - - - - - - - - - - -  LOOP - - - - - - - - - - - - - -
void loop() {
  check_data_et_envoi_bluetooth((uint32_t*)&(status48.ERPM_lu), &(status48.Current_lu), &(status48.Duty_lu), (uint32_t*)&(status79.ERPM_lu), &(status79.Current_lu), &(status79.Duty_lu));

  id_du_vesc = bluetooth_lecture_et_action(id_du_vesc);
}

// - - - - - - - - - - CONCAT VAL - - - - - - - - - - - - - -
void concatVal(int id_vesc, int id_data, unsigned long erpm, unsigned long courant, unsigned long duty) {
  sprintf(envoi, "%d,%d,%lu,%lu,%lu \n", id_vesc, id_data, erpm, courant, duty);
}

// - - - - - - - - - - - ENVOI BLUETOOTH - - - - - - - - - - - -
void envoi_BL_modif(int id_vesc, int id_data) {
  switch (id_data) {
    case 0x09:
      switch (id_vesc) {
        case 48:
          concatVal(id_vesc, id_data, data_48.valeur_ERPM, data_48.valeur_courant, data_48.valeur_duty);
          SerialBT.write((const uint8_t*)envoi, strlen(envoi));
          //Serial.println(envoi);
          break;
        case 79:
          concatVal(id_vesc, id_data, data_79.valeur_ERPM, data_79.valeur_courant, data_79.valeur_duty);
          SerialBT.write((const uint8_t*)envoi, strlen(envoi));
          //Serial.println(envoi);
          break;
      }
      break;
    case 0x0E:
      switch (id_vesc) {
        case 48:

          break;
        case 79:

          break;
      }
      break;
    case 0x0F:
      switch (id_vesc) {
        case 48:

          break;
        case 79:

          break;
      }
      break;
    case 0x10:
      switch (id_vesc) {
        case 48:

          break;
        case 79:

          break;
      }
      break;
    case 0x1B:
      switch (id_vesc) {
        case 48:

          break;
        case 79:

          break;
      }
      break;
    case 0x3A:
      switch (id_vesc) {
        case 48:

          break;
        case 79:

          break;
      }
      break;
  }
}

// - - - - - - - - - - - CHECK DATA CAN - - - - - - - - - - - - - -
void check_data_et_envoi_bluetooth(uint32_t* erpm_ptr48, uint16_t* current_ptr48, uint16_t* duty_ptr48, uint32_t* erpm_ptr79, uint16_t* current_ptr79, uint16_t* duty_ptr79) {

  int* trame_recue_fct = can_receive_par_adresse();

  //Message status 1, id VESC 0x4F//
  if ((trame_recue_fct[0] == 0x0000094F) && ((erpm_ptr79[0] != trame_recue_fct[2]) || (erpm_ptr79[1] != trame_recue_fct[3]) || (erpm_ptr79[2] != trame_recue_fct[4]) || (erpm_ptr79[3] != trame_recue_fct[5]) || (current_ptr79[0] != trame_recue_fct[6]) || (current_ptr79[1] != trame_recue_fct[7]) || (duty_ptr79[0] != trame_recue_fct[8]) || (duty_ptr79[1] != trame_recue_fct[9])) || (start == 1)) {
    start = 0;
    erpm_ptr79[0] = trame_recue_fct[2];
    erpm_ptr79[1] = trame_recue_fct[3];
    erpm_ptr79[2] = trame_recue_fct[4];
    erpm_ptr79[3] = trame_recue_fct[5];
    current_ptr79[0] = trame_recue_fct[6];
    current_ptr79[1] = trame_recue_fct[7];
    duty_ptr79[0] = trame_recue_fct[8];
    duty_ptr79[1] = trame_recue_fct[9];

    data_79.valeur_ERPM_precedente = data_79.valeur_ERPM;
    data_79.valeur_ERPM = trame_recue_fct[5] + (trame_recue_fct[4] << 8) + (trame_recue_fct[3] << 16) + (trame_recue_fct[2] << 24);

    data_79.valeur_courant_precedente = data_79.valeur_courant;
    data_79.valeur_courant = trame_recue_fct[7] + (trame_recue_fct[6] << 8);

    data_79.valeur_duty_precedente = data_79.valeur_duty;
    data_79.valeur_duty = trame_recue_fct[9] + (trame_recue_fct[8] << 8);

    if ((data_79.valeur_ERPM_precedente != data_79.valeur_ERPM) || (data_79.valeur_courant_precedente != data_79.valeur_courant) || (data_79.valeur_duty_precedente != data_79.valeur_duty)) {
      envoi_BL_modif(79, 9);
    }
  }

  //Message status 1, id VESC 0x30//
  else if ((trame_recue_fct[0] == 0x00000930) && ((erpm_ptr48[0] != trame_recue_fct[2]) || (erpm_ptr48[1] != trame_recue_fct[3]) || (erpm_ptr48[2] != trame_recue_fct[4]) || (erpm_ptr48[3] != trame_recue_fct[5]) || (current_ptr48[0] != trame_recue_fct[6]) || (current_ptr48[1] != trame_recue_fct[7]) || (duty_ptr48[0] != trame_recue_fct[8]) || (duty_ptr48[1] != trame_recue_fct[9])) || (start48 == 1)) {
    start48 = 0;
    erpm_ptr48[0] = trame_recue_fct[2];
    erpm_ptr48[1] = trame_recue_fct[3];
    erpm_ptr48[2] = trame_recue_fct[4];
    erpm_ptr48[3] = trame_recue_fct[5];
    current_ptr48[0] = trame_recue_fct[6];
    current_ptr48[1] = trame_recue_fct[7];
    duty_ptr48[0] = trame_recue_fct[8];
    duty_ptr48[1] = trame_recue_fct[9];

    data_48.valeur_ERPM_precedente = data_48.valeur_ERPM;
    data_48.valeur_ERPM = trame_recue_fct[5] + (trame_recue_fct[4] << 8) + (trame_recue_fct[3] << 16) + (trame_recue_fct[2] << 24);

    data_48.valeur_courant_precedente = data_48.valeur_courant;
    data_48.valeur_courant = trame_recue_fct[7] + (trame_recue_fct[6] << 8);

    data_48.valeur_duty_precedente = data_48.valeur_duty;
    data_48.valeur_duty = trame_recue_fct[9] + (trame_recue_fct[8] << 8);

    if ((data_48.valeur_ERPM_precedente != data_48.valeur_ERPM) || (data_48.valeur_courant_precedente != data_48.valeur_courant) || (data_48.valeur_duty_precedente != data_48.valeur_duty)) {
      envoi_BL_modif(48, 9);
    }
  }

  else {
  }
  free(trame_recue_fct);
}

// - - - - - - - - - - - LECTURE BLUETOOTH - - - - - - - - - - - -
int bluetooth_lecture_et_action(int id_VESC) {
  while (SerialBT.available()) {
    unsigned long valeur_BT;

    int dataBT = SerialBT.read();
    Serial.print("dataBT :");
    Serial.println(dataBT);

    if ((dataBT >= 0) && (dataBT <= 100)) {  // Valeur
      valeur_BT = map(dataBT, 0, 100, 850, 30000);
      Serial.print("Valeur de data réceptionnée : ");
      Serial.println(valeur_BT);
    }

    if ((dataBT >= 101) && (dataBT <= 199)) {  // id VESC
      id_VESC = dataBT - 100;
      Serial.print("id VESC réceptioné : ");
      Serial.println(id_VESC);
    }

    switch (dataBT) {
      case 200:  // mode accelerateur;
        { 
          int toujours_en_accel = 0;
          while (toujours_en_accel == 0) {
            can_write_classique(id_VESC, 0x08, map(analogRead(36), 000, 3000, 850, 10000));
            check_data_et_envoi_bluetooth((uint32_t*)&(status48.ERPM_lu), &(status48.Current_lu), &(status48.Duty_lu), (uint32_t*)&(status79.ERPM_lu), &(status79.Current_lu), &(status79.Duty_lu));
            int bluetooth_temp = SerialBT.read();
            if (bluetooth_temp == 200 || bluetooth_temp != -1) {
              toujours_en_accel = !toujours_en_accel;
            }
          }
          break;
        }

      case 201:  //mode input bluetooth;
        Serial.println("Mode 1 (non codé)");
        break;
      case 202:  //équivalent appui bouton smartled mode 2;
        Serial.println("Mode 2 (non codé)");
        break;
      case 203:  //équivalent appui bouton smartled mode 3;
        Serial.println("Mode 3 (non codé)");
        break;
      case 204:  //équivalent appui bouton smartled mode 4;
        Serial.println("Mode 4 (non codé)");
        break;
      case 205:  // Duty Cycle
        can_write_classique(id_VESC, 0x05, valeur_BT);
        Serial.print("Duty cycle envoyé : ");
        Serial.println(valeur_BT);
        Serial.print("Au VESC id : ");
        Serial.println(id_VESC);
        break;
      case 206:  // Courant
        /*can_write_classique(id_VESC, 0x06, valeur_BT);
        Serial.print("Courant envoyé : ");
        Serial.println(valeur_BT);
        Serial.print("Au VESC id : ");
        Serial.println(id_VESC);*/
        digitalWrite(27, STATE);
        STATE = !STATE;
        break;
      case 207:  //Brake courant
        can_write_classique(id_VESC, 0x07, valeur_BT);
        Serial.print("Brake envoyé : ");
        Serial.println(valeur_BT);
        Serial.print("Au VESC id : ");
        Serial.println(id_VESC);
        break;
      case 208:  // ERPM
        can_write_classique(id_VESC, 0x08, valeur_BT);
        Serial.print("ERPM envoyé : ");
        Serial.println(valeur_BT);
        Serial.print("Au VESC id : ");
        Serial.println(id_VESC);
        break;
      case 209:  // Power
        can_write_classique(id_VESC, 0x09, valeur_BT);
        Serial.print("Power envoyé : ");
        Serial.println(valeur_BT);
        Serial.print("Au VESC id : ");
        Serial.println(id_VESC);
        break;
      case 210:  // Hand brake courant
        can_write_classique(id_VESC, 0x0A, valeur_BT);
        Serial.print("Handbrake envoyé : ");
        Serial.println(valeur_BT);
        Serial.print("Au VESC id : ");
        Serial.println(id_VESC);
        break;
      case 211:  // Arret d'urgence
        can_write_classique(id_VESC, 0x08, 0);
        Serial.print("ARRET D'URGENCE CAN : 0 ERPM ENVOYE A VESC NUMERO : ");
        Serial.println(id_VESC);
        break;
    }

    if (dataBT > 211) {  //Call for get data
      Serial.print("Call for get data from CAN, valeur : ");
      Serial.println(dataBT);
    }
  }
  return id_VESC;
}

// - - - - - - - - - - - - - CAN DATA WRITE - - - - - - - - - - - - - -
void can_write_classique(int id_VESC, int hexa_mode, unsigned long valeur) {
  // ATTENTION : Ne pas ajouter de Serial.print dans cette fonction
  // Cela entraine un problème sur le cycle time qui se synchronise
  // avec le plotter.


  //Serial.println("entree fonction");

  unsigned long bytes_R = valeur;
  while (bytes_R > 256) {
    bytes_R = bytes_R - 256;
  };
  unsigned long bytes_M = valeur / 256;  //FF c'est ciao
  while (bytes_M > 256) {
    bytes_M = bytes_M - 256;
  };
  unsigned long bytes_L = valeur / 65536;  //FF FF c'est ciao
  //Serial.println("avant soustraction millis");

  unsigned long currentMillis = millis();
  if (currentMillis - previousMillis >= interval) {
    previousMillis = currentMillis;
    //digitalWrite(CAN_EN, LOW);
    //delay(10);
    //Serial.println("CAN_EN HIGH");
    tx_frame.FIR.B.FF = CAN_frame_ext;
    tx_frame.MsgID = (0x800 + id_VESC);
    tx_frame.FIR.B.DLC = 7;
    tx_frame.data.u8[0] = 0x02;  // A Vérifier
    tx_frame.data.u8[1] = 0x00;
    tx_frame.data.u8[2] = hexa_mode;  //MODE (7 IB, A HB, (6 I), (9 P), 5 D, 8 EPRM, )
    tx_frame.data.u8[3] = 0x00;
    //tx_frame.data.u8[4] = 0x00;
    //tx_frame.data.u8[5] = 0x13;
    //tx_frame.data.u8[6] = 0x88;
    tx_frame.data.u8[4] = bytes_L;
    tx_frame.data.u8[5] = bytes_M;
    tx_frame.data.u8[6] = bytes_R;
    ESP32Can.CANWriteFrame(&tx_frame);
    delay(20);

    //digitalWrite(CAN_EN, HIGH);
    //Serial.println("CAN_EN LOW");
  }
  //Serial.print("Trame CAN envoyee. ID : "); Serial.print(id_VESC); Serial.print(" Mode : "); Serial.print(hexa_mode); Serial.print(" Valeur : "); Serial.println(valeur);
}

// - - - - - - - - - - - - CAN DATA READ - - - - - - - - - - - - -
int* can_receive_par_adresse() {

  CAN_frame_t rx_frame;
  unsigned long currentMillis = millis();

  int* trame_recue = (int*)malloc(10 * sizeof(int));

  if (trame_recue == NULL) {
    //printf("Erreur lors de l'allocation de mémoire.\n");
    exit(EXIT_FAILURE);
  }

  if (xQueueReceive(CAN_cfg.rx_queue, &rx_frame, 3 * portTICK_PERIOD_MS) == pdTRUE) {
    if (rx_frame.FIR.B.FF == CAN_frame_std) {
      //New standard frame
    } else {
      //New extended frame
    }

    if (rx_frame.FIR.B.RTR == CAN_RTR) {

    } else {
      Serial.println("-------------------- Trame CAN reçue --------------------");
      trame_recue[0] = rx_frame.MsgID;
      trame_recue[1] = rx_frame.FIR.B.DLC;
      for (int i = 2; i < (rx_frame.FIR.B.DLC + 2); i++) {
        trame_recue[i] = rx_frame.data.u8[i - 2];
      }
    }
  }
  return trame_recue;
}
