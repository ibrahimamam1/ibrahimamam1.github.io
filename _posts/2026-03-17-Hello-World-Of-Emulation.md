# Building a CHIP-8 Interpreter: The Hello World of Emulation

A few weeks ago i started reading the legendary book [Computer Systems a Programmer's Perspective](https://www.amazon.com/Computer-Systems-Programmers-Perspective-3rd/dp/013409266X). It's one of those books that have legendary status on the internet so i thought i should at least read it and see why for myself (spoiler: it is indeed a legendary book). In chapter four the authors designed their own instruction set architecture to teach how the CPU actually works. This inspired me to go on a quest of designing my own CPU then emulating it and hopefully get c programs running through my emulator. Before doing that i thought i need to learn how emulators are built first and practice with some toy ones and that's how i learned about the hello world of emulation: CHIP 8.

CHIP 8 was actually an interpreter written to run games on the ancient hardware of the 1970s. It is not really an emulator in the strict sense because there is no CHIP 8 hardware to emulate, but the working principle is just the same as an actual emulator so the difference is just a linguistic one.

The first thing to do when making an emulator is to read the specification and turn it into a context within your code. CHIP 8 has the following specifications:

**Memory:** 4KB (4,096 bytes) of RAM, addressed from `0x000` to `0xFFF`. The first 512 bytes (`0x000–0x1FF`) are reserved for the interpreter itself. Most CHIP-8 programs start at address `0x200`.

**Registers:**
- 16 general-purpose 8-bit registers, referred to as `V0` through `VF`. The `VF` register doubles as a flag used by certain instructions (carry, borrow, collision, etc.) and should not be used directly by programs.
- A 16-bit register called `I`, generally used to store memory addresses (only the lowest 12 bits are typically used).
- Two special-purpose 8-bit registers for the delay timer (DT) and sound timer (ST). When non-zero, they are automatically decremented at 60 Hz.
- A 16-bit program counter (PC) storing the currently executing address, and an 8-bit stack pointer (SP) pointing to the topmost level of the stack.

**Stack:** The interpreter must guarantee sufficient stack space for up to twelve successive subroutine calls, though modern implementations may allocate more.

**Timers:** The delay timer does nothing more than count down at 60 Hz until it reaches 0. The sound timer also decrements at 60 Hz, but as long as its value is greater than zero, the CHIP-8 buzzer sounds. The tone has only one frequency, decided by the author of the interpreter.

**Display:** 64×32 pixels, monochrome (black or white). The pixel at [0,0] is the top-left corner and [63,31] is the bottom-right. Graphics are rendered using sprites: a sprite is a group of bytes as a binary representation of a picture, up to 15 bytes in size (8×15 pixels). The interpreter area of memory (`0x000–0x1FF`) stores built-in font sprites for the hexadecimal digits 0–F, each 5 bytes tall (8×5 pixels). Sprites are XOR'd onto the display — if this causes any pixels to be erased, `VF` is set to 1, otherwise 0. If a sprite is positioned so part of it is outside the display coordinates, it wraps around to the opposite side.

**Input:** The interpreter accepts input from a 16-key keypad, with each key corresponding to a hexadecimal digit (0–F). Modern implementations typically simulate this by mapping a standard keyboard to the hex layout.

**Opcodes:** The original specification includes 36 instructions covering math, graphics, and flow control. All instructions are 2 bytes long, stored most-significant-byte first, and the first byte of each instruction should be located at an even address.

**Clock:** A standard speed of around 700 CHIP-8 instructions per second fits well for most programs.

---

When translated to code we get the following struct:

```cpp
constexpr std::size_t CHIP8_MEM_SIZE = 4096; // 4KB memory
constexpr short NUM_REGS = 16;               // 16 gp registers
constexpr short DISPLAY_HEIGHT = 32;
constexpr short DISPLAY_WIDTH = 64;
constexpr short TIMER_FREQ = 60;
constexpr short CLOCK_SPEED = 700;
//
typedef struct chip8_context {
  std::vector<unsigned char> mem; // 4096 bytes of memory
  std::stack<uint16_t> st;        // 16 bit stack
  std::vector<uint8_t> regs;      // 8 bit general purpose registers
  uint16_t pc;                    // progam counter = index into memory
  uint16_t ir;                    // 16 bit index register
  uint8_t delay_timer;            // 8bit delay timer register
  uint8_t sound_timer;            // 8 bit sound timer register
  std::vector<std::vector<unsigned char>> dis_buff; // display buffer
  chip8_context()
      : mem(CHIP8_MEM_SIZE), regs(NUM_REGS), pc(0), ir(0), delay_timer(0),
        sound_timer(0),
        dis_buff(DISPLAY_HEIGHT, std::vector<unsigned char>(DISPLAY_WIDTH, 0)) {
  }
} chip8_context;
```

The next thing i did was define the fonts used for display and load those into memory. To handle the GUI i decided to use raylib for it's ease of use.

After initialising the display all you need is a fetch, decode, execute loop. This is the core part and where most of the code is written. Fetching means simply reading the instruction at the memory address pointed to by the pc. In decoding we read the first nibble of the instruction to determine it's type and once we know what instruction it is we execute it. Executing means writing the logic for whatever that instruction is suppose to do. The final things needed are just handling fps and clock speed which is just making sure we render the screen and execute instructions at a particular frequency. That all it takes to make a chip 8 interpreter. it's simple enough to be completed within a few hours.

You can find my full code [here](https://github.com/ibrahimamam1/yaci.git). The excellent guide i read and to which i have really nothing to add was: [Guide to making a CHIP-8 emulator](https://tobiasvl.github.io/blog/write-a-chip-8-emulator/). Give it a try. Thank you and see you soon.
