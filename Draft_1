#include <FastLED.h>

#define LED_PIN  3

#define COLOR_ORDER GRB
#define CHIPSET     WS2812

#define BRIGHTNESS 64

// Helper functions for an two-dimensional XY matrix of pixels.
// Simple 2-D demo code is included as well.
//
//     XY(x,y) takes x and y coordinates and returns an LED index number,
//             for use like this:  leds[ XY(x,y) ] == CRGB::Red;
//             No error checking is performed on the ranges of x and y.
//
//     XYsafe(x,y) takes x and y coordinates and returns an LED index number,
//             for use like this:  leds[ XY(x,y) ] == CRGB::Red;
//             Error checking IS performed on the ranges of x and y, and an
//             index of "-1" is returned.  Special instructions below
//             explain how to use this without having to do your own error
//             checking every time you use this function.  
//             This is a slightly more advanced technique, and 
//             it REQUIRES SPECIAL ADDITIONAL setup, described below.


// Params for width and height
const uint8_t kmatrixH = 30;
const uint8_t kmatrixW = 7;

// Param for different pixel layouts
const bool    kMatrixSerpentineLayout = true;

uint16_t XY( uint8_t x, uint8_t y)
{
  uint16_t i;
  
  if( kMatrixSerpentineLayout == false) {
    i = (x * kmatrixH) + y;
  }

  if( kMatrixSerpentineLayout == true) {
    if( x & 0x01) {
      // Odd rows run backwards
      uint8_t reverseY = (kmatrixH - 1) - y;
      i = (x * kmatrixH) + reverseY;
    } else {
      // Even rows run forwards
      i = (x * kmatrixH) + y;
    }
  }
  
  return i;
}

#define NUM_LEDS (kmatrixH * kmatrixW)
CRGB leds_plus_safety_pixel[ NUM_LEDS + 1];
CRGB* leds( leds_plus_safety_pixel + 1);

CRGBPalette16 gPal;

uint16_t XYsafe( uint8_t x, uint8_t y)
{
  if( x >= kmatrixW) return -1;
  if( y >= kmatrixH) return -1;
  return XY(x,y);
}


/* Refresh rate. Higher makes for flickerier
   Recommend small values for small displays */
#define FPS 18
#define FPS_DELAY 1000/FPS

// Demo that USES "XY" follows code below

void loop()
{
    random16_add_entropy( random() ); // We chew a lot of entropy
  
  Fireplace();

  FastLED.show();
  FastLED.delay(FPS_DELAY); //
}

/* Rate of cooling. Play with to change fire from
   roaring (smaller values) to weak (larger values) */
#define COOLING 10  

/* How hot is "hot"? Increase for brighter fire */
#define HOT 125
#define MAXHOT HOT*kmatrixW

void Fireplace () {
  static unsigned int spark[kmatrixW]; // base heat
  CRGB stack[kmatrixW][kmatrixH];        // stacks that are cooler
 
  // 1. Generate sparks to re-heat
  for( int i = 0; i < kmatrixW; i++) {
    if (spark[i] < HOT ) {
      int base = HOT * 2;
      spark[i] = random16( base, MAXHOT );
    }
  }
  
  // 2. Cool all the sparks
  for( int i = 0; i < kmatrixW; i++) {
    spark[i] = qsub8( spark[i], random8(0, COOLING) );
  }
  
  // 3. Build the stack
  /*    This works on the idea that pixels are "cooler"
        as they get further from the spark at the bottom */
  for( int i = 0; i < kmatrixW; i++) {
    unsigned int heat = constrain(spark[i], HOT/2, MAXHOT);
    for( int j = kmatrixH-1; j >= 0; j--) {
      /* Calculate the color on the palette from how hot this
         pixel is */
      byte index = constrain(heat, 0, HOT);
      stack[i][j] = ColorFromPalette( gPal, index );
      
      /* The next higher pixel will be "cooler", so calculate
         the drop */
      unsigned int drop = random8(0,HOT);
      if (drop > heat) heat = 0; // avoid wrap-arounds from going "negative"
      else heat -= drop;
 
      heat = constrain(heat, 0, MAXHOT);
    }
  }
  
  // 4. map stacks to led array
  for( int i = 0; i < kmatrixW; i++) {
  for( int j = 0; j < kmatrixH; j++) {
     leds[(i*kmatrixH) + j] = stack[i][j];
  }
  }
  
}


void setup() {
  
  FastLED.addLeds<CHIPSET, LED_PIN, COLOR_ORDER>(leds, NUM_LEDS).setCorrection(TypicalSMD5050);
  FastLED.setBrightness( BRIGHTNESS );

    /* Set a black-body radiation palette
     This comes from FastLED */
  gPal = HeatColors_p;
  //gPal = CRGBPalette16( CRGB::Black, CRGB::Blue, CRGB::Aqua,  CRGB::White);
  
  /* Clear display before starting */
  FastLED.clear();
  FastLED.show();
  FastLED.delay(1500); // Sanity before start
}


