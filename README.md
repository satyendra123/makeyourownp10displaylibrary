# makeyourownp10displaylibrary
i have made my own p10 display library using esp32


















b) P10Display.cpp
#include "P10Display.h"

// Constructor
P10Display::P10Display(int dataPin, int clkPin, int latPin, int oePin, int aPin, int bPin, int cPin) {
  this->dataPin = dataPin;
  this->clkPin = clkPin;
  this->latPin = latPin;
  this->oePin = oePin;
  this->aPin = aPin;
  this->bPin = bPin;
  this->cPin = cPin;

  // Initialize the buffer
  clear();
}

// Initializes the P10 display
void P10Display::init() {
  pinMode(dataPin, OUTPUT);
  pinMode(clkPin, OUTPUT);
  pinMode(latPin, OUTPUT);
  pinMode(oePin, OUTPUT);
  pinMode(aPin, OUTPUT);
  pinMode(bPin, OUTPUT);
  pinMode(cPin, OUTPUT);

  digitalWrite(oePin, HIGH);  // Disable display during initialization
}

// Clears the display buffer
void P10Display::clear() {
  for (int row = 0; row < 4; row++) {
    for (int col = 0; col < 32; col++) {
      displayBuffer[row][col] = false;
    }
  }
}

// Draws a pixel at (x, y)
void P10Display::drawPixel(int x, int y, bool state) {
  if (x >= 0 && x < 32 && y >= 0 && y < 16) {
    int row = y / 4;  // Map y to buffer row (1/4 scan)
    int subY = y % 4;  // Map y to sub-row within the scan line
    displayBuffer[subY][x] = state;
  }
}

// Sends a byte of data to the display
void P10Display::sendData(uint8_t data) {
  for (int i = 0; i < 8; i++) {
    digitalWrite(dataPin, (data & (1 << (7 - i))) ? HIGH : LOW);
    digitalWrite(clkPin, HIGH);
    digitalWrite(clkPin, LOW);
  }
}

// Latches the data into the display
void P10Display::latch() {
  digitalWrite(latPin, HIGH);
  digitalWrite(latPin, LOW);
}

// Selects the row to display
void P10Display::selectRow(int row) {
  digitalWrite(aPin, row & 0x01);
  digitalWrite(bPin, row & 0x02);
  digitalWrite(cPin, row & 0x04);
}

// Turns the display ON or OFF
void P10Display::enable(bool state) {
  digitalWrite(oePin, state ? LOW : HIGH);
}

// Updates the display with the buffer content
void P10Display::update() {
  for (int row = 0; row < 4; row++) {
    for (int col = 0; col < 32; col++) {
      sendData(displayBuffer[row][col] ? 0xFF : 0x00);
    }
    latch();
    selectRow(row);
    enable(true);
    delayMicroseconds(200);
    enable(false);
  }
}

