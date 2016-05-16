# arduino-keypad-lock
Arduino Keypad Lock with Password Changing
#include <Keypad.h>
#include <Password.h>
 
String newPasswordString; //hold the new password
char newPassword[6]; //charater string of newPasswordString
 
//initialize password to 1234
//you can use password.set(newPassword) to overwrite it
Password password = Password( "1234" );
Password passwordset = Password( "" );
boolean change = false;// Are we changing the code?
byte maxPasswordLength = 4; // Self-explanatory
byte currentPasswordLength = 0;//^ Same as above ^
const byte ROWS = 4; // Four rows
const byte COLS = 4; // Four columns
 
//Define the keymap
char keys[ROWS][COLS] = {
{'1','2','3','A'},
{'4','5','6','B'},
{'7','8','9','C'},
{'*','0','#','D'}
};// Set yours, they can often be different
 
//// Connect keypad ROW0, ROW1, ROW2 and ROW3 to these Arduino pins.
byte rowPins[ROWS] = {6,7,8,9}; //connect to row pinouts
 
// Connect keypad COL0, COL1, COL2 and COL3 to these Arduino pins.
byte colPins[COLS] = {2,3,4,5}; //connect to column pinouts
 
// Create the Keypad
Keypad keypad = Keypad( makeKeymap(keys), rowPins, colPins, ROWS, COLS );
 
void setup(){
   Serial.begin(9600);// Set up Serial
}
 
void loop(){
  if(change=false){// If we aren't changing password
   char key = keypad.getKey();// Get the key
   if (key != NO_KEY){// If there is a key being pressed
      delay(60); // Wait a bit
      switch (key){// Evaluate the key pressed
      case '#': checkPassword(); break;
      case '*': change=true; break;
      default: processNumberKey(key);
      }
   }
  }
  else {// Same as above, but is changing password instead of
        // entering the code
    char key = keypad.getKey();
   if (key != NO_KEY){
      delay(60); 
      switch (key){
      case '#': /* nothing */ break;
      case '*': changePassword(); change=false; break;
      default: processChange(key);
      }
   }
  }
}
 
void processChange(char key) {
   Serial.print(key);
   currentPasswordLength++;
   passwordset.append(key);
   if (currentPasswordLength == maxPasswordLength) {
      password.set(passwordset);// Set the passcode
   } 
}
 
void processNumberKey(char key) {
   Serial.print(key);
   currentPasswordLength++;
   password.append(key);
   if (currentPasswordLength == maxPasswordLength) {
      checkPassword();// Validate the entered code
   } 
}

void checkPassword() {// Validation
   if (password.evaluate()){
      Serial.println(" OK.");
   } else {
      Serial.println(" Wrong password!");
   } 
   resetPassword();
}

void resetPassword() {// Reset password. Duh :)
   password.reset(); 
   currentPasswordLength = 0; 
}

void changePassword() {// Change the password
   newPasswordString = "123";
   newPasswordString.toCharArray(newPassword, newPasswordString.length()+1); //convert string to char array
password.set(newPassword);
   resetPassword();
   Serial.print("Password changed to ");
   Serial.println(newPasswordString);
}
