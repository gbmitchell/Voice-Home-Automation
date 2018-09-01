# Python Code for Raspberry Pi

    // initialize global variables
    int AtticLightState = 0;
    int BedroomLightState = 0;
    int BathroomLightState = 0;
    int KitchenLightState = 0;
    int LivingroomLightState = 0;
    int aRelayState = 1;
    int kRelayState = 1;
    int beRelayState = 1;
    int baRelayState = 1;
    int lRelayState = 1;
    int i=0;
    int wait = 0;
    float tempValLiv=60.0;
    float tempValAttic = 60.0;
    float tempValBed = 60.0;
    char temp[2] = {0};
    char switchCase = 0;
    char n[4] = {0};
    int klightsens = 1;
    int belightsens = 1;
    int balightsens = 1;
    int alightsens = 1;
    int llightsens = 1;

    int betempsensor[30] = {0};
    int atempsensor[30] = {0};
    int ltempsensor[30]= {0};

    int betempsens = 0;
    int atempsens = 0;
    int ltempsens = 0;

    char dump = 0;

    float bevoltage = 0.0;
    float avoltage = 0.0;
    float lvoltage = 0.0;
    float betemperatureC = 0.0;
    float atemperatureC = 0.0;
    float ltemperatureC = 0.0;
    float betemperatureF = 0.0;
    float atemperatureF = 0.0;
    float ltemperatureF = 0.0;


    unsigned long startTempMillis = 0;
    unsigned long currentTempMillis = 0;
    unsigned long intervalTemp = 1000;


    // setup routine
    void setup() {
      //pin A2 Attic Temp
      //Pin A0 Living Room Temp
      //Pin A1 Bedroom Temp
      pinMode(13, OUTPUT); //fan ATTIC
      pinMode(12, INPUT); //light in LIVING ROOM
      pinMode(11, INPUT); //light in BATHROOM
      pinMode(10, INPUT); //light in KITCHEN
      pinMode(9, INPUT); //light in BEDROOM
      pinMode(8, INPUT); //light in ATTIC
      pinMode(7, OUTPUT); //light out BATHROOM
      pinMode(6, OUTPUT); //light out KITCHEN
      pinMode(5, OUTPUT); //light out LIVING ROOM
      pinMode(4, OUTPUT); //light out BEDROOM
      pinMode(3, OUTPUT); //light out ATTIC
      pinMode(2, OUTPUT); //fan LIVING ROOM
      pinMode(A3, OUTPUT); //resistor LIVING ROOM
      pinMode(A4, OUTPUT); //resistor ATTIC
      pinMode(A5, OUTPUT); //resistor BEDROOM

      // initialize serial communication at baud rate 9600
      Serial.begin(9600);
    }


    // main loop
    void loop(){
    //************************************START WITH ALL RELAYS OFF************************************
      digitalWrite(2, HIGH);
      digitalWrite(3, HIGH);
      digitalWrite(4, HIGH);
      digitalWrite(5, HIGH);
      digitalWrite(6, HIGH);
      digitalWrite(7, HIGH);
      digitalWrite(13, HIGH);
      digitalWrite(A3, HIGH);
      digitalWrite(A4, HIGH);
      digitalWrite(A5, HIGH);
      startTempMillis = millis();

      while(true){
    //**********************************************TEMP SENSOR********************************************
        currentTempMillis = millis();

        // check the temperature at an interval of 1 sencond
        // take the average of 30 temp sensor readings
        if((currentTempMillis - startTempMillis) >= intervalTemp){
          for(i = 0; i < 30; i++){
            betempsensor[i] = analogRead(A1);
            atempsensor[i] = analogRead(A2);
            ltempsensor[i] = analogRead(A0);
          }

          for(i = 0; i < 30; i++){
            betempsens = betempsens + betempsensor[i];
            atempsens = atempsens + atempsensor[i];
            ltempsens = ltempsens + ltempsensor[i];
          }
          betempsens = betempsens/30;
          atempsens = atempsens/30;
          ltempsens = ltempsens/30;

          // converting sensor readings to voltage
          bevoltage = betempsens * 5.0;
          avoltage = atempsens * 5.0;
          lvoltage = ltempsens * 5.0;
          bevoltage /= 1024.0;
          avoltage /= 1024.0;
          lvoltage /= 1024.0;

          // convert voltage to celcius
          betemperatureC = (bevoltage) * 100 ;  //converting from 10 mv per degree wit 500 mV offset
          atemperatureC = (avoltage) * 100 ;  //converting from 10 mv per degree wit 500 mV offset
          ltemperatureC = (lvoltage) * 100 ;  //converting from 10 mv per degree wit 500 mV offset
          //to degrees ((bevoltage - 500mV) times 100)

          // now convert to Fahrenheit
          betemperatureF = (betemperatureC * 9.0 / 5.0) + 32.0;
          atemperatureF = (atemperatureC * 9.0 / 5.0) + 32.0;
          ltemperatureF = (ltemperatureC * 9.0 / 5.0) + 32.0;

          // set the lower limit of the temperature to 60 deg F
          if(betemperatureF < 60){
            betemperatureF = 60;
          }

          if(atemperatureF < 60){
            atemperatureF = 60;
          }

          if(ltemperatureF < 60){
            ltemperatureF = 60;
          }

    //      // print temp values for debug
    //      Serial.println(betemperatureF);
    //      Serial.println(atemperatureF);
    //      Serial.println(ltemperatureF);
    //      Serial.println();
    //
    //      Serial.println(tempValBed);
    //      Serial.println(tempValAttic);
    //      Serial.println(tempValLiv);
    //      Serial.println();

        //*********************Turn on fan or turn on heater*************************************
          if(tempValLiv <= ltemperatureF){
            digitalWrite(A3, HIGH);
            digitalWrite(2, LOW);
          }else{
            digitalWrite(2, HIGH);
            digitalWrite(A3, LOW);
          }

          if(tempValBed <= betemperatureF){
            digitalWrite(A5, HIGH);
            digitalWrite(13, LOW);
          }else{
            digitalWrite(13, HIGH);
            digitalWrite(A5, LOW);
          }

          if(tempValAttic <= atemperatureF){
            digitalWrite(A4, HIGH); 
          }else{
            digitalWrite(A4, LOW);
          }
          startTempMillis = millis();
        }

    //****************************************read light switches************************************
        // pause to let Amazon server update finish
        delay(1000);

        // read light sensor values
        alightsens = digitalRead(8);
        belightsens = digitalRead(9);
        balightsens = digitalRead(11);
        klightsens = digitalRead(10);
        llightsens = digitalRead(12);

    //    // print light sensor readings for debug
    //    Serial.println(alightsens);
    //    Serial.println(belightsens);
    //    Serial.println(balightsens);
    //    Serial.println(klightsens);
    //    Serial.println(llightsens);
    //    Serial.println();

    //**************************LIGHT states based on state of switches********************

    //***************************************ATTIC*************************************
        if(alightsens==0 && AtticLightState==0 && aRelayState ==1){
          Serial.println('a');  // sending 'a' to make alexa change its state to 1
          AtticLightState = 1;  // 1 means alexa thinks light is on
          aRelayState = 1;      // 1 means relay is off 
          alightsens =0;

          // clear the serial buffer of automatic reply from Amazon
          if(Serial.available()){
            for(i = 0; i < 8; i++){
              while(Serial.available()){
                dump = Serial.read();
              }
            }
          }

        }else if(alightsens==1 && AtticLightState==1 && aRelayState==1){
          Serial.println('b');  // sending 'b' to make alexa change its state to 0
          AtticLightState = 0;
          aRelayState = 1;
          alightsens =1;

          if(Serial.available()){
            for(i = 0; i < 8; i++){
              while(Serial.available()){
                dump = Serial.read();
              }
            }
          }

        }else if(alightsens==1 && AtticLightState==1 && aRelayState==0){
          Serial.println('b');
          AtticLightState = 0;
          aRelayState = 0;
          alightsens =1;

          if(Serial.available()){
            for(i = 0; i < 8; i++){
              while(Serial.available()){
                dump = Serial.read();
              }
            }
          }


        }else if(alightsens==0 && AtticLightState==0 && aRelayState==0){
          Serial.println('a');
          AtticLightState = 1;
          aRelayState = 0;
          alightsens =0;

          if(Serial.available()){
            for(i = 0; i < 8; i++){
              while(Serial.available()){
                dump = Serial.read();
              }
            }
          }

        }else

    //***************************************KITCHEN*************************************
        if(klightsens==0 && KitchenLightState==0 && kRelayState ==1){
          Serial.println('c');    // sending 'c' to make alexa change its state to 1
          KitchenLightState = 1;  // 1 means alexa thinks light is on
          kRelayState = 1;        // 1 means relay is off 
          klightsens =0;

          // clear the serial buffer of automatic reply from Amazon
          if(Serial.available()){
            for(i = 0; i < 8; i++){
              while(Serial.available()){
                dump = Serial.read();
              }
            }
          }

        }else if(klightsens==1 && KitchenLightState==1 && kRelayState==1){
          Serial.println('d');    // sending 'd' to make alexa change its state to 0
          KitchenLightState = 0;
          kRelayState = 1;
          klightsens =1;

          if(Serial.available()){
            for(i = 0; i < 8; i++){
              while(Serial.available()){
                dump = Serial.read();
              }
            }
          }

        }else if(klightsens==1 && KitchenLightState==1 && kRelayState==0){
          Serial.println('d');
          KitchenLightState = 0;
          kRelayState = 0;
          klightsens =1;

          if(Serial.available()){
            for(i = 0; i < 8; i++){
              while(Serial.available()){
                dump = Serial.read();
              }
            }
          }

        }else if(klightsens==0 && KitchenLightState==0 && kRelayState==0){
          Serial.println('c');
          KitchenLightState = 1;
          kRelayState = 0;
          klightsens =0;

          if(Serial.available()){
            for(i = 0; i < 8; i++){
              while(Serial.available()){
                dump = Serial.read();
              }
            }
          }

        }else
    // *****************************************BEDROOM****************************************
        if(belightsens==0 && BedroomLightState==0 && beRelayState ==1){
          Serial.println('e');    // sending 'e' to make alexa change its state to 1
          BedroomLightState = 1;  // 1 means alexa thinks light is on
          beRelayState = 1;       // 1 means relay is off 
          belightsens =0;

          // clear the serial buffer of automatic reply from Amazon
          if(Serial.available()){
            for(i = 0; i < 8; i++){
              while(Serial.available()){
                dump = Serial.read();
              }
            }
          }

        }else if(belightsens==1 && BedroomLightState==1 && beRelayState==1){
          Serial.println('f');    // sending 'f' to make alexa change its state to 0
          BedroomLightState = 0;
          beRelayState = 1;
          belightsens =1;

          if(Serial.available()){
            for(i = 0; i < 8; i++){
              while(Serial.available()){
                dump = Serial.read();
              }
            }
          }

        }else if(belightsens==1 && BedroomLightState==1 && beRelayState==0){
          Serial.println('f');
          BedroomLightState = 0;
          beRelayState = 0;
          belightsens =1;

          if(Serial.available()){
            for(i = 0; i < 8; i++){
              while(Serial.available()){
                dump = Serial.read();
              }
            }
          }

        }else if(belightsens==0 && BedroomLightState==0 && beRelayState==0){
          Serial.println('e');
          BedroomLightState = 1;
          beRelayState = 0;
          belightsens =0;

          if(Serial.available()){
            for(i = 0; i < 8; i++){
              while(Serial.available()){
                dump = Serial.read();
              }
            }
          }

        }else
    // *****************************************BATHROOM****************************************
        if(balightsens==0 && BathroomLightState==0 && baRelayState ==1){
          Serial.println('g');    // sending 'g' to make alexa change its state to 1
          BathroomLightState = 1; // 1 means alexa thinks light is on
          baRelayState = 1;       // 1 means relay is off 
          balightsens =0;

          // clear the serial buffer of automatic reply from Amazon
          if(Serial.available()){
            for(i = 0; i < 8; i++){
              while(Serial.available()){
                dump = Serial.read();
              }
            }
          }

        }else if(balightsens==1 && BathroomLightState==1 && baRelayState==1){
          Serial.println('h');    // sending 'h' to make alexa change its state to 0
          BathroomLightState = 0;
          baRelayState = 1;
          balightsens =1;

          if(Serial.available()){
            for(i = 0; i < 8; i++){
              while(Serial.available()){
                dump = Serial.read();
              }
            }
          }

        }else if(balightsens==1 && BathroomLightState==1 && baRelayState==0){
          Serial.println('h');
          BathroomLightState = 0;
          baRelayState = 0;
          balightsens =1;

          if(Serial.available()){
            for(i = 0; i < 8; i++){
              while(Serial.available()){
                dump = Serial.read();
              }
            }
          }

        }else if(balightsens==0 && BathroomLightState==0 && baRelayState==0){
          Serial.println('g');
          BathroomLightState = 1;
          baRelayState = 0;
          balightsens =0;

          if(Serial.available()){
            for(i = 0; i < 8; i++){
              while(Serial.available()){
                dump = Serial.read();
              }
            }
          }

        }else
    // *****************************************LIVINGROOM****************************************
        if(llightsens==0 && LivingroomLightState==0 && lRelayState ==1){
          Serial.println('i');      // sending 'i' to make alexa change its state to 1
          LivingroomLightState = 1; // 1 means alexa thinks light is on
          lRelayState = 1;          // 1 means relay is off 
          llightsens =0;

          // clear the serial buffer of automatic reply from Amazon
          if(Serial.available()){
            for(i = 0; i < 8; i++){
              while(Serial.available()){
                dump = Serial.read();
              }
            }
          }

        }else if(llightsens==1 && LivingroomLightState==1 && lRelayState==1){
          Serial.println('j');      // sending 'j' to make alexa change its state to 0
          LivingroomLightState = 0;
          lRelayState = 1;
          llightsens =1;

          if(Serial.available()){
            for(i = 0; i < 8; i++){
              while(Serial.available()){
                dump = Serial.read();
              }
            }
          }

        }else if(llightsens==1 && LivingroomLightState==1 && lRelayState==0){
          Serial.println('j');
          LivingroomLightState = 0;
          lRelayState = 0;
          llightsens =1;

          if(Serial.available()){
            for(i = 0; i < 8; i++){
              while(Serial.available()){
                dump = Serial.read();
              }
            }
          }

        }else if(llightsens==0 && LivingroomLightState==0 && lRelayState==0){
          Serial.println('i');
          LivingroomLightState = 1;
          lRelayState = 0;
          llightsens =0;

          if(Serial.available()){
            for(i = 0; i < 8; i++){
              while(Serial.available()){
                dump = Serial.read();
              }
            }
          }

        }

    //**************************LIGHT states based on state of AWS shadow (Relays)******************
    //**************************************SERIAL INPUT FROM PI************************************
        //check for an update from Alexa (for lights and temperature)
        if(Serial.available()){
          i=0;
          switchCase = {0};

          for(i=0; i<4; i++){
            n[i] = Serial.read();
          }

          switchCase = n[0];

          switch (switchCase) {
    //***********************************************ATTIC**************************        
            case 'a':
              if(alightsens==1 && aRelayState==1){
                digitalWrite(3, LOW);
                AtticLightState = 1;
                aRelayState = 0;
                while(alightsens == 1){
                  alightsens = digitalRead(8);
                }
                //alightsens = 0;
                break;
              }
              if(alightsens==0 && aRelayState==1){
                digitalWrite(3, HIGH);
                AtticLightState = 1;
                aRelayState = 1;
                while(alightsens == 1){
                  alightsens = digitalRead(8);
                }
                //alightsens = 0;
                break;
              }
              if(alightsens==1 && aRelayState==0){
                digitalWrite(3, HIGH);
                AtticLightState = 1;
                aRelayState = 1;
                while(alightsens == 1){
                  alightsens = digitalRead(8);
                }
                //alightsens = 0;
                break;
              }
              if(alightsens==0 && aRelayState==0){
                digitalWrite(3, LOW);
                AtticLightState = 1;
                aRelayState = 0;
                while(alightsens == 1){
                  alightsens = digitalRead(8);
                }
                //alightsens = 0;
                break;
              }
              break;
            case 'b':
              if(alightsens==0 && aRelayState==1){
                digitalWrite(3, LOW);
                AtticLightState = 0;
                aRelayState =0;
                while(alightsens == 0){
                  alightsens = digitalRead(8);
                }
                //alightsens = 1;
                break;
              }
              if(alightsens==1 && aRelayState==1){
                digitalWrite(3, HIGH);
                AtticLightState = 0;
                aRelayState = 1;
                while(alightsens == 0){
                  alightsens = digitalRead(8);
                }
                //alightsens = 1;
                break;
              }
              if(alightsens==0 && aRelayState==0){
                digitalWrite(3, HIGH);
                AtticLightState = 0;
                aRelayState = 1;
                while(alightsens == 0){
                  alightsens = digitalRead(8);
                }
                //alightsens = 1;
                break;
              }
              if(alightsens==1 && aRelayState==0){
                digitalWrite(3, LOW);
                AtticLightState = 0;
                aRelayState = 0;
                while(alightsens == 0){
                  alightsens = digitalRead(8);
                }
                //alightsens = 1;
                break;
              }
              break;

    //***********************************************KITCHEN**************************        
            case 'c':
              if(klightsens==1 && kRelayState==1){
                digitalWrite(6, LOW);
                KitchenLightState = 1;
                kRelayState = 0;
                while(klightsens == 1){
                  klightsens = digitalRead(10);
                }
                break;
              }
              if(klightsens==0 && kRelayState==1){
                digitalWrite(6, HIGH);
                KitchenLightState = 1;
                kRelayState = 1;
                while(klightsens == 1){
                  klightsens = digitalRead(10);
                }
                break;
              }
              if(klightsens==1 && kRelayState==0){
                digitalWrite(6, HIGH);
                KitchenLightState = 1;
                kRelayState = 1;
                while(klightsens == 1){
                  klightsens = digitalRead(10);
                }
                break;
              }
              if(klightsens==0 && kRelayState==0){
                digitalWrite(6, LOW);
                KitchenLightState = 1;
                kRelayState = 0;
                while(klightsens == 1){
                  klightsens = digitalRead(10);
                }
                break;
              }
              break;
            case 'd':
              if(klightsens==0 && kRelayState==1){
                digitalWrite(6, LOW);
                KitchenLightState = 0;
                kRelayState =0;
                while(klightsens == 0){
                  klightsens = digitalRead(10);
                }
                break;
              }
              if(klightsens==1 && kRelayState==1){
                digitalWrite(6, HIGH);
                KitchenLightState = 0;
                kRelayState = 1;
                while(klightsens == 0){
                  klightsens = digitalRead(10);
                }
                break;
              }
              if(klightsens==0 && kRelayState==0){
                digitalWrite(6, HIGH);
                KitchenLightState = 0;
                kRelayState = 1;
                while(klightsens == 0){
                  klightsens = digitalRead(10);
                }
                break;
              }
              if(klightsens==1 && kRelayState==0){
                digitalWrite(6, LOW);
                KitchenLightState = 0;
                kRelayState = 0;
                while(klightsens == 0){
                  klightsens = digitalRead(10);
                }
                break;
              }
              break;

    //***********************************************BEDROOM**************************        
            case 'e':
              if(belightsens==1 && beRelayState==1){
                digitalWrite(4, LOW);
                BedroomLightState = 1;
                beRelayState = 0;
                while(belightsens == 1){
                  belightsens = digitalRead(9);
                }
                break;
              }
              if(belightsens==0 && beRelayState==1){
                digitalWrite(4, HIGH);
                BedroomLightState = 1;
                beRelayState = 1;
                while(belightsens == 1){
                  belightsens = digitalRead(9);
                }
                break;
              }
              if(belightsens==1 && beRelayState==0){
                digitalWrite(4, HIGH);
                BedroomLightState = 1;
                beRelayState = 1;
                while(belightsens == 1){
                  belightsens = digitalRead(9);
                }
                break;
              }
              if(belightsens==0 && beRelayState==0){
                digitalWrite(4, LOW);
                BedroomLightState = 1;
                beRelayState = 0;
                while(belightsens == 1){
                  belightsens = digitalRead(9);
                }
                break;
              }
              break;
            case 'f':
              if(belightsens==0 && beRelayState==1){
                digitalWrite(4, LOW);
                BedroomLightState = 0;
                beRelayState =0;
                while(belightsens == 0){
                  belightsens = digitalRead(9);
                }
                break;
              }
              if(belightsens==1 && beRelayState==1){
                digitalWrite(4, HIGH);
                BedroomLightState = 0;
                beRelayState = 1;
                while(belightsens == 0){
                  belightsens = digitalRead(9);
                }
                break;
              }
              if(belightsens==0 && beRelayState==0){
                digitalWrite(4, HIGH);
                BedroomLightState = 0;
                beRelayState = 1;
                while(belightsens == 0){
                  belightsens = digitalRead(9);
                }
                break;
              }
              if(belightsens==1 && beRelayState==0){
                digitalWrite(4, LOW);
                BedroomLightState = 0;
                beRelayState = 0;
                while(belightsens == 0){
                  belightsens = digitalRead(9);
                }
                break;
              }
              break;

    //***********************************************BATHROOM**************************        
            case 'g':
              if(balightsens==1 && baRelayState==1){
                digitalWrite(7, LOW);
                BathroomLightState = 1;
                baRelayState = 0;
                while(balightsens == 1){
                  balightsens = digitalRead(11);
                }
                break;
              }
              if(balightsens==0 && baRelayState==1){
                digitalWrite(7, HIGH);
                BathroomLightState = 1;
                baRelayState = 1;
                while(balightsens == 1){
                  balightsens = digitalRead(11);
                }
                break;
              }
              if(balightsens==1 && baRelayState==0){
                digitalWrite(7, HIGH);
                BathroomLightState = 1;
                baRelayState = 1;
                while(balightsens == 1){
                  balightsens = digitalRead(11);
                }
                break;
              }
              if(balightsens==0 && baRelayState==0){
                digitalWrite(7, LOW);
                BathroomLightState = 1;
                baRelayState = 0;
                while(balightsens == 1){
                  balightsens = digitalRead(11);
                }
                break;
              }
              break;
            case 'h':
              if(balightsens==0 && baRelayState==1){
                digitalWrite(7, LOW);
                BathroomLightState = 0;
                baRelayState =0;
                while(balightsens == 0){
                  balightsens = digitalRead(11);
                }
                break;
              }
              if(balightsens==1 && baRelayState==1){
                digitalWrite(7, HIGH);
                BathroomLightState = 0;
                baRelayState = 1;
                while(balightsens == 0){
                  balightsens = digitalRead(11);
                }
                break;
              }
              if(balightsens==0 && baRelayState==0){
                digitalWrite(7, HIGH);
                BathroomLightState = 0;
                baRelayState = 1;
                while(balightsens == 0){
                  balightsens = digitalRead(11);
                }
                break;
              }
              if(balightsens==1 && baRelayState==0){
                digitalWrite(7, LOW);
                BathroomLightState = 0;
                baRelayState = 0;
                while(balightsens == 0){
                  balightsens = digitalRead(11);
                }
                break;
              }
              break;

    //***********************************************LIVINGROOM**************************        
            case 'i':
              if(llightsens==1 && lRelayState==1){
                digitalWrite(5, LOW);
                LivingroomLightState = 1;
                lRelayState = 0;
                while(llightsens == 1){
                  llightsens = digitalRead(12);
                }
                break;
              }
              if(llightsens==0 && lRelayState==1){
                digitalWrite(5, HIGH);
                LivingroomLightState = 1;
                lRelayState = 1;
                while(llightsens == 1){
                  llightsens = digitalRead(12);
                }
                break;
              }
              if(llightsens==1 && lRelayState==0){
                digitalWrite(5, HIGH);
                LivingroomLightState = 1;
                lRelayState = 1;
                while(llightsens == 1){
                  llightsens = digitalRead(12);
                }
                break;
              }
              if(llightsens==0 && lRelayState==0){
                digitalWrite(5, LOW);
                LivingroomLightState = 1;
                lRelayState = 0;
                while(llightsens == 1){
                  llightsens = digitalRead(12);
                }
                break;
              }
              break;
            case 'j':
              if(llightsens==0 && lRelayState==1){
                digitalWrite(5, LOW);
                LivingroomLightState = 0;
                lRelayState =0;
                while(llightsens == 0){
                  llightsens = digitalRead(12);
                }
                break;
              }
              if(llightsens==1 && lRelayState==1){
                digitalWrite(5, HIGH);
                LivingroomLightState = 0;
                lRelayState = 1;
                while(llightsens == 0){
                  llightsens = digitalRead(12);
                }
                break;
              }
              if(llightsens==0 && lRelayState==0){
                digitalWrite(5, HIGH);
                LivingroomLightState = 0;
                lRelayState = 1;
                while(llightsens == 0){
                  llightsens = digitalRead(12);
                }
                break;
              }
              if(llightsens==1 && lRelayState==0){
                digitalWrite(5, LOW);
                LivingroomLightState = 0;
                lRelayState = 0;
                while(llightsens == 0){
                  llightsens = digitalRead(12);
                }
                break;
              }
              break;

    //***********************************************TEMP**************************
            case 't':
              temp[0]=n[2];
              temp[1]=n[3];

              if(n[1]=='l'){
                tempValLiv = atoi(temp);
              }
              if(n[1]=='b'){
                tempValBed = atoi(temp);
              }
              if(n[1]=='a'){
                tempValAttic = atoi(temp);
              }
              break;
          }
        }  
      }
    } 
  
  
#### [Voice Home Automation](https://gbmitchell.github.io/Voice-Home-Automation/main)
