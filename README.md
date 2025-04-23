# ar-hud-military-display.
Arduino code for AR-HUD project for military vehicles.
#include <U8g2lib.h>
#include <SPI.h>

// Initialize MicroOLED with SPI pins: CLK (D13), MOSI (D11), CS (D10), DC (D8), RESET (D9)
U8G2_SH1106_128X64_NONAME_F_4W_SW_SPI u8g2(U8G2_R0, 13, 11, 10, 8, 9);

// Struct to hold CAN-like data
struct CanData {
  uint8_t speed_byte;
  uint16_t rpm_bytes;
  uint8_t fuel_byte;
  uint8_t temp_byte;
  uint8_t direction_byte;
};

// Function to parse serial data from PC (format: speed,rpm,fuel,temp,direction\n)
CanData parse_serial_data(String input) {
  CanData data;
  int comma1 = input.indexOf(',');
  int comma2 = input.indexOf(',', comma1 + 1);
  int comma3 = input.indexOf(',', comma2 + 1);
  int comma4 = input.indexOf(',', comma3 + 1);
  data.speed_byte = input.substring(0, comma1).toInt() & 0xFF;
  data.rpm_bytes = input.substring(comma1 + 1, comma2).toInt() & 0xFFFF;
  data.fuel_byte = input.substring(comma2 + 1, comma3).toInt() & 0xFF;
  data.temp_byte = input.substring(comma3 + 1, comma4).toInt() & 0xFF;
  data.direction_byte = input.substring(comma4 + 1).charAt(0);
  return data;
}

void setup() {
  // Initialize MicroOLED
  u8g2.begin();
  
  // Initialize serial communication with PC at 9600 baud
  Serial.begin(9600);
  
  // Wait for serial port to connect (optional, useful for debugging)
  while (!Serial) {
    delay(100);
  }
  
  // Display a startup message on the MicroOLED
  u8g2.clearBuffer();
  u8g2.setFont(u8g2_font_ncenB08_tr);
  u8g2.setCursor(0, 10);
  u8g2.print("AR-HUD Starting...");
  u8g2.sendBuffer();
  delay(2000); // Show startup message for 2 seconds
}

void loop() {
  // Check if data is available from the PC via serial
  if (Serial.available()) {
    // Read the incoming string until newline
    String input = Serial.readStringUntil('\n');
    
    // Parse the serial data into CanData struct
    CanData data = parse_serial_data(input);
    
    // Clear the MicroOLED buffer
    u8g2.clearBuffer();
    
    // Set font for display
    u8g2.setFont(u8g2_font_ncenB08_tr);
    
    // Display Speed (top left)
    u8g2.setCursor(0, 10);
    u8g2.print("Speed: ");
    u8g2.print(data.speed_byte);
    u8g2.print(" km/h");
    
    // Display RPM (top right)
    u8g2.setCursor(70, 10);
    u8g2.print("RPM: ");
    u8g2.print(data.rpm_bytes);
    
    // Display Fuel (bottom left)
    u8g2.setCursor(0, 60);
    u8g2.print("Fuel: ");
    u8g2.print(data.fuel_byte);
    u8g2.print("%");
    
    // Display Temperature (bottom center)
    u8g2.setCursor(50, 60);
    u8g2.print("Temp: ");
    u8g2.print(data.temp_byte);
    u8g2.print("C");
    
    // Display Direction (bottom right)
    u8g2.setCursor(100, 60);
    u8g2.print(data.direction_byte);
    
    // Send the buffer to the MicroOLED
    u8g2.sendBuffer();
  }
  
  // Update display every 500 ms
  delay(500);
}
