### Simple Password Application

To help understand its usage, I created a simple password application. Of course, it doesn't reflect real life very well. I aimed for it to be not too complex, so that beginners can also grasp it easily.

I will explain in more detail below.👇

| WRONG | CORRECT |
|---------|---------|
| ![Resim 1](https://github.com/arastaskiran/RustyKeypad/blob/main/.platform/assets/wrong_password.gif?raw=true) | ![Resim 2](https://github.com/arastaskiran/RustyKeypad/blob/main/.platform/assets/correct_password.gif?raw=true) |

## PIN Configuration Example (4x3 Keypad Matrix)
> [!NOTE]
> The benefits listed here are the default settings. Be sure to configure your own keypad settings within the setup() method using the logic shown below.
```cpp
/**
 * Configuration for the rows of the keypad matrix.
 * (GPIO PIN NUMBERS)
 */
uint8_t rows[MAX_KEYPAD_MATRIX_SIZE] = {2U, 3U, 4U, 5U};

/**
 * Configuration for the columns of the keypad matrix.
 * (GPIO PIN NUMBERS)
 */
uint8_t cols[MAX_KEYPAD_MATRIX_SIZE] = {6U, 7U, 8U};

/**
 * Maximum number of rows in the keypad matrix.
 */
uint8_t max_row_length=4;

/**
 * Maximum number of columns in the keypad matrix.
 */
uint8_t max_col_length=3;

/**
 * Keypad layout mapping.
 */
const char *keypadFactoryMap[MAX_KEYPAD_MATRIX_SIZE][MAX_KEYPAD_MATRIX_SIZE] = {
    {"1.,?!'\"-()@/:_", "2ABCabc",   "3DEFdef"  },
    {"4GHIghiİ",        "5JKLjkl",   "6MNOmnoÖö"},
    {"7PQRSpqrsŞş",     "8TUVtuvÜü", "9WXYZwxyz"},
    {"*",                "0 +",      "#"},
};

/**
 * Setup the keypad with the specified configuration.
 */
RustyKeypad::keyboardSetup(
    keypadFactoryMap,
    rows,
    cols,
    max_row_length,
    max_col_length,
    INPUT_PULLUP
);

```
### Information About the Demo Hardware
> [!TIP]
> The internal structure of the hardware I used in the demo application. This may differ from yours. By understanding the logic of the electrical schematic shown above, you can locate the pins with your multimeter, or you can review the documentation of the keypad if it is available.
<pre>
+---------------------------------------+
|                  Keypad               |
|             +---+---+---+             |
|             | 1 | 2 | 3 |             |
|             +---+---+---+             |
|             | 4 | 5 | 6 |             |
|             +---+---+---+             |
|             | 7 | 8 | 9 |             |
|             +---+---+---+             |
|             | * | 0 | # |             |
|             +---+---+---+             |
+---------------------------------------+
  |   |   |   |   |   |   |   |   |
 (1) (2) (3) (4) (5) (6) (7) (8) (9)
  |   |   |   |   |   |   |   |   |
(N/A) |   |   |   |   |   |   | (N/A)
      |   |   |   |   |   |   |
     C2   R1  C1  R4  C3  R3  R2
</pre>
> [!TIP]
> The keypad and Arduino connections in the demo application are made as follows.
<pre>
+-------------+                       +-------------+
|             |                       |             |
|             |-(2)-------------(D7)--|             |
|   KEYPAD    |-(3)-------------(D2)--|   ARDUINO   |
|             |-(4)-------------(D6)--|    NANO     |
|             |-(5)-------------(D5)--|             |
|             |-(6)-------------(D8)--|             |
|             |-(7)-------------(D4)--|             |
|             |-(8)-------------(D3)--|             |
|             |                       |             |
|             |                       |             |
+-------------+                       +-------------+
</pre>

## Libraries

```cpp
#include <Arduino.h>
#include <rusty_keypad.h>
#include <LiquidCrystal.h>
```

* Arduino.h: This is the core library for the Arduino platform. It provides basic functionalities for input/output operations, timing, and more.

* rusty_keypad.h: This includes a custom library for managing keypad functionalities. It allows you to interact with a keypad.

* LiquidCrystal.h: This library is used to control an LCD screen. It provides functions to display text on the LCD.
  
## LCD Initialization
```cpp
LiquidCrystal LCD(A2, A1, A3, 10, 11, 12, 13);
```
* This creates a LiquidCrystal object named LCD. The pins A2, A1, A3, 10, 11, 12, and 13 are the connections to the LCD.

## A) Native Functions

#### 1. setup Function
```cpp
void setup()
{
  Serial.begin(115200);
  pinMode(A1, OUTPUT);
  pinMode(A2, OUTPUT);
  pinMode(A3, OUTPUT);
  LCD.begin(16, 2);
  RustyKeypad::addTextChangeListener(textChange);
  RustyKeypad::addEnterActionListener(textEnter);
  RustyKeypad::setEnterKey('#');
  RustyKeypad::useDeleteKey('*');
  RustyKeypad::setType(RKP_T9);
  RustyKeypad::enableBuzzer(9, 10UL);
  RustyKeypad::enable();
  waitPassword();
}
```
* Serial.begin(115200): Initializes serial communication at a baud rate of 115200.

* pinMode(...): Sets the specified pins as outputs.

* LCD.begin(16, 2): Initializes the LCD dimensions (16 columns and 2 rows).

* The RustyKeypad functions configure the keypad:
  * addTextChangeListener: Adds the textChange function to listen for text changes on the keypad.

  * addEnterActionListener: Calls the textEnter function when the Enter key is pressed.

  * setEnterKey and useDeleteKey: Assign the Enter and Delete keys.

  * setType(RKP_T9): Sets the keypad to work in T9 mode.

  * enableBuzzer: Configures the buzzer.

* Calls waitPassword() to prompt the user for a password.

> [!IMPORTANT]
> If you want your password to be displayed with a * mask, please configure the following in the setup section.
```cpp
 RustyKeypad::setPasswordMask(true);
```

#### 2. loop Function
```cpp
void loop()
{
  RustyKeypad::scan();
}
```
* The loop() function runs continuously.
  
* RustyKeypad::scan(): Checks the inputs from the keypad.


## B) Implementation Functions

#### 1. waitPassword
```cpp
void waitPassword()
{
  LCD.clear();
  LCD.setCursor(0, 0);
  LCD.print("PLEASE PASSWORD:");
}
```
* Clears the LCD screen.
  
* Displays "PLEASE PASSWORD:" on the first line, prompting the user to enter a password.

#### 2. wrongPassword
```cpp
void wrongPassword()
{
  LCD.clear();
  LCD.setCursor(0, 0);
  LCD.print("WRONG");
  LCD.setCursor(0, 1);
  LCD.print("PASSWORD");
  delay(2000);
  waitPassword();
}
```

* This function is called when the password entered is incorrect.
  
* It clears the screen and shows "WRONG" and "PASSWORD".

* Waits for 2 seconds and then calls waitPassword() to prompt the user for the password again.

#### 3. correctPassword
```cpp
void correctPassword()
{
  LCD.clear();
  LCD.setCursor(0, 0);
  LCD.print("CORRECT");
  LCD.setCursor(0, 1);
  LCD.print("PASSWORD");
  delay(2000);
  waitPassword();
}
```
* This function is called when the password entered is correct.
  
* It clears the screen and shows "CORRECT" and "PASSWORD".
  
* Waits for 2 seconds and then calls waitPassword() to prompt for the password again.

#### 4. clearSecondRow
```cpp
void clearSecondRow()
{
  LCD.setCursor(0, 1);
  LCD.print("                ");
}
```
* Clears the second row of the LCD by writing 16 spaces.


#### 5. textChange
```cpp
void textChange(String x)
{
  clearSecondRow();
  LCD.setCursor(0, 1);
  LCD.print(x);
}
```
* This function is called when there is a change in the keypad input.

* It clears the second row and prints the new text x in that row.

#### 6. textEnter
```cpp
void textEnter(String text)
{
  if (RustyKeypad::isKeypadEqual("6789"))
  {
    correctPassword();
    return;
  }
  wrongPassword();
}
```
* This function is called after the user enters the password.

* If the entered password is "6789", it calls correctPassword(); otherwise, it calls wrongPassword().

