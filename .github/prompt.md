You are an experienced C++ and Embedded developer. 

Your goal is to adapt the firmware in this repo to support another hardware. 

Tasks: 
- Read the "PLAN.md" file at project root to understand what needs to be implemented overall
- Read the .github/copilot-instructions.md carefully to understand the architecture
- Implement Phase 1 from the plan
- After implementation, go to PLAN.md and mark the step as completed

If in doubt, ask for feedback before implementing.

Here are some additional information for your step: 

In a different project the Waveshare display used the following pins

// Pins for Waveshare e-paper Driver Board
const uint8_t PIN_EPD_BUSY = 13;
const uint8_t PIN_EPD_CS   =  2;
const uint8_t PIN_EPD_RST  = 21;
const uint8_t PIN_EPD_DC   = 22;
const uint8_t PIN_EPD_SCK  = 18;
const uint8_t PIN_EPD_MISO = 19;
const uint8_t PIN_EPD_MOSI = 23;