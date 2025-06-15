#include <Adafruit_GFX.h>
#include <MCUFRIEND_kbv.h>
#include <TouchScreen.h>

MCUFRIEND_kbv tft;

// Touchscreen
#define MINPRESSURE 200
#define MAXPRESSURE 1000
#define XP 8
#define XM A2
#define YP A3
#define YM 9
TouchScreen ts = TouchScreen(XP, YP, XM, YM, 300);

// Kalibrierung
const int TS_LEFT = 248;
const int TS_RT   = 873;
const int TS_TOP  = 929;
const int TS_BOT  = 213;

// Farben
#define BLACK   0x0000
#define BLUE    0x001F
#define RED     0xF800
#define GREEN   0x07E0
#define CYAN    0x07FF
#define MAGENTA 0xF81F
#define YELLOW  0xFFE0
#define WHITE   0xFFFF

#define BOXSIZE 40
#define PENRADIUS 3

int currentcolor = RED;
int oldcolor = RED;
bool started = false;

int pixel_x, pixel_y;

// START-Button Position links
#define BTN_X 20
#define BTN_Y 130
#define BTN_W 160
#define BTN_H 60

bool Touch_getXY() {
  TSPoint p = ts.getPoint();
  if (p.z < MINPRESSURE || p.z > MAXPRESSURE) return false;
  pinMode(YP, OUTPUT);
  pinMode(XM, OUTPUT);
  digitalWrite(YP, HIGH);
  digitalWrite(XM, HIGH);
  pixel_x = map(p.x, TS_LEFT, TS_RT, 0, tft.width());
  pixel_y = map(p.y, TS_TOP, TS_BOT, 0, tft.height());
  return true;
}

void drawStartScreen() {
  tft.fillScreen(BLACK);

  // HD Robotics links oben
  tft.setTextColor(WHITE);
  tft.setTextSize(3);
  tft.setCursor(20, 30);
  tft.print("HD Robotics");

  // Paint darunter
  tft.setTextSize(2);
  tft.setCursor(20, 70);
  tft.print("Paint");

  // START-Button links
  tft.fillRoundRect(BTN_X, BTN_Y, BTN_W, BTN_H, 8, BLUE);
  tft.drawRoundRect(BTN_X, BTN_Y, BTN_W, BTN_H, 8, WHITE);
  tft.setTextColor(WHITE);
  tft.setTextSize(3);
  tft.setCursor(BTN_X + 30, BTN_Y + 15); // Text im Button leicht eingerückt
  tft.print("START");

  // Webseite unten links klein
  tft.setTextSize(1);
  tft.setCursor(20, 230);
  tft.print("www.hdrobotics.de");
}

void drawColorPalette() {
  tft.fillRect(0, 0, BOXSIZE, BOXSIZE, RED);
  tft.fillRect(BOXSIZE, 0, BOXSIZE, BOXSIZE, YELLOW);
  tft.fillRect(BOXSIZE * 2, 0, BOXSIZE, BOXSIZE, GREEN);
  tft.fillRect(BOXSIZE * 3, 0, BOXSIZE, BOXSIZE, CYAN);
  tft.fillRect(BOXSIZE * 4, 0, BOXSIZE, BOXSIZE, BLUE);
  tft.fillRect(BOXSIZE * 5, 0, BOXSIZE, BOXSIZE, MAGENTA);
  tft.drawRect(0, 0, BOXSIZE, BOXSIZE, WHITE);
}

void setup() {
  Serial.begin(9600);
  uint16_t ID = 0x9341;
  tft.begin(ID);
  tft.setRotation(0);
  drawStartScreen();
}

void loop() {
  if (!started) {
    if (Touch_getXY()) {
      if (pixel_x >= BTN_X && pixel_x <= BTN_X + BTN_W &&
          pixel_y >= BTN_Y && pixel_y <= BTN_Y + BTN_H) {
        started = true;
        tft.fillScreen(BLACK);
        drawColorPalette();
      }
    }
    return;
  }

  TSPoint p = ts.getPoint();
  if (p.z > MINPRESSURE && p.z < MAXPRESSURE) {
    pinMode(YP, OUTPUT);
    pinMode(XM, OUTPUT);
    digitalWrite(YP, HIGH);
    digitalWrite(XM, HIGH);

    int x = map(p.x, TS_LEFT, TS_RT, 0, tft.width());
    int y = map(p.y, TS_TOP, TS_BOT, 0, tft.height());

    if (y < BOXSIZE) {
      oldcolor = currentcolor;
      if (x < BOXSIZE) {
        currentcolor = RED;
        tft.drawRect(0, 0, BOXSIZE, BOXSIZE, WHITE);
      } else if (x < BOXSIZE * 2) {
        currentcolor = YELLOW;
        tft.drawRect(BOXSIZE, 0, BOXSIZE, BOXSIZE, WHITE);
      } else if (x < BOXSIZE * 3) {
        currentcolor = GREEN;
        tft.drawRect(BOXSIZE * 2, 0, BOXSIZE, BOXSIZE, WHITE);
      } else if (x < BOXSIZE * 4) {
        currentcolor = CYAN;
        tft.drawRect(BOXSIZE * 3, 0, BOXSIZE, BOXSIZE, WHITE);
      } else if (x < BOXSIZE * 5) {
        currentcolor = BLUE;
        tft.drawRect(BOXSIZE * 4, 0, BOXSIZE, BOXSIZE, WHITE);
      } else if (x < BOXSIZE * 6) {
        currentcolor = MAGENTA;
        tft.drawRect(BOXSIZE * 5, 0, BOXSIZE, BOXSIZE, WHITE);
      }

      if (oldcolor != currentcolor) {
        if (oldcolor == RED) tft.fillRect(0, 0, BOXSIZE, BOXSIZE, RED);
        else if (oldcolor == YELLOW) tft.fillRect(BOXSIZE, 0, BOXSIZE, BOXSIZE, YELLOW);
        else if (oldcolor == GREEN) tft.fillRect(BOXSIZE * 2, 0, BOXSIZE, BOXSIZE, GREEN);
        else if (oldcolor == CYAN) tft.fillRect(BOXSIZE * 3, 0, BOXSIZE, BOXSIZE, CYAN);
        else if (oldcolor == BLUE) tft.fillRect(BOXSIZE * 4, 0, BOXSIZE, BOXSIZE, BLUE);
        else if (oldcolor == MAGENTA) tft.fillRect(BOXSIZE * 5, 0, BOXSIZE, BOXSIZE, MAGENTA);
      }
    }

    if (y > tft.height() - 10) {
      tft.fillRect(0, BOXSIZE, tft.width(), tft.height() - BOXSIZE, BLACK);
    }

    if (y > BOXSIZE && y < tft.height() - 10) {
      tft.fillCircle(x, y, PENRADIUS, currentcolor);
    }
  }
}
