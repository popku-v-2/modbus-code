#include <ModbusRTU.h>
#include <driver/adc.h>

#define LDR_PIN 34        // ADC pin for LDR sensor (GPIO 34)
#define MODBUS_BAUD 9600  // Baud rate for Modbus communication

ModbusRTU mb;  // Create ModbusRTU object

uint16_t ldrValue = 0;    // Variable to hold LDR sensor value
uint16_t modbusRegisters[10]; // Modbus holding registers

// Callback function to read LDR value and store it in modbusRegisters
uint16_t readLDR(TRegister* reg, uint16_t val) {
    ldrValue = analogRead(LDR_PIN);  // Read LDR sensor value
    reg->value = ldrValue;           // Update the register value
    return ldrValue;                 // Return the new LDR value
}

void setup() {
  // Initialize serial for debugging
  Serial.begin(115200);

  // Setup LDR pin for ADC (Analog to Digital Conversion)
  adc1_config_width(ADC_WIDTH_BIT_12);           // 12-bit resolution
  adc1_config_channel_atten(ADC1_CHANNEL_6, ADC_ATTEN_DB_11); // LDR on GPIO34
  
  // Initialize Modbus RTU (using Serial2, RS485)
  Serial2.begin(MODBUS_BAUD, SERIAL_8N1, 16, 17); // UART2 on GPIO16 (TX) and GPIO17 (RX)
  
  mb.begin(&Serial2);           // Start Modbus with Serial2
  mb.slave(1);                  // Set Modbus server/slave ID to 1
  
  // Add Modbus holding register area (10 registers) and a callback function to read LDR
  mb.addHreg(0, 0, 10);        // Address 0 for holding registers, 10 registers in total
  mb.onGetHreg(0, readLDR);    // When register 0 is accessed, read LDR value
  
  Serial.println("Modbus RTU Server started, waiting for requests...");
}

void loop() {
  // Modbus RTU task, handles incoming requests from Modbus master
  mb.task();
  
  // Optionally print the current LDR value for debugging
  Serial.print("LDR value: ");
  Serial.println(ldrValue);
  
  delay(1000);  // Small delay for stability
}
