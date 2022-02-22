# RawLoRa-Portenta-RaspberryPi
Raw LoRa between Arduino Portenta + Vision Shield &amp; Raspberry Pi + Dragino 

## Portenta + Vision Shield

### 1. Setting environment to program Portenta

<aside>
💻 I firmly recommend using a Linux machine for this project, there are some more advance stuff that didn’t worked for my using my Macbook Pro 16” (2019). ~~~~

</aside>

1. Install Arduino on your Linux machine:
    
    [How to Install Arduino IDE on Ubuntu 20.04](https://linoxide.com/how-to-install-arduino-ide-on-ubuntu-20-04/)
    
2. Open Arduino as root:
    
    ```bash
    sudo arduino
    ```
    
3. Connect the Portenta + Vision Shield to the computer
4. Install Arduino Mbed OS Portenta Boards (Pop up when connecting Portenta for the first time)
5. Select Port and Board: Tools/Board/Arduino Mbed OS Portenta Boards/Arduino Portenta H7 (M7 Core) and Tools/Ports/[your port] (for me it was “/dev/ttyACM0”)

### 2. Communicate using Raw LoRa (default Settings)

<aside>
💡 Note: I’ve tried this step with a MacOS machine and it worked fine, but it’s a little finicky in some cases recognizing the Ports when you connect the Portenta to the computer...

</aside>

Works if you are only using two Portentas and don’t want to change the Frequency or anything else (just a demo...). The defaults settings are: 

```arduino
LoRaRadio.begin(915000000);
LoRaRadio.setFrequency(915000000);
LoRaRadio.setTxPower(1);    //default 14
LoRaRadio.setBandwidth(LoRaRadio.BW_125);
LoRaRadio.setSpreadingFactor(LoRaRadio.SF_7);
LoRaRadio.setCodingRate(LoRaRadio.CR_4_5);
LoRaRadio.setLnaBoost(true);
```

### Install Portenta Pro Community Solutions and libraries to make it work

1. Tools/Manage Libraries → Search: Portenta Pro Community Solutions (by Jeremy Ellis, big thanks Jeremy!)
2. Install and then select Install all (other needed libraries...)
3. Now search for MKRWAN and install it too

### Firmware Update for the Portenta (STM32 bootloader protocol) and communication

1. Go to File/Examples/Portenta-pro-community-solutions/dot3-portenta-vision-shields/dot34-pure-lora
    1. Open dot341-lora-update-standalone and then repeat the last point and open in other window the dot341-serial-lora
    2. Flash the Portenta:
        1. First go to the dot341-lora-update-standalone 
        2. Try compiling it to check that everything is working ok
        3. Double press the boot button on the Portenta and it should start flashing the green light (bootloader mode)
        4. Check that the Port is selected in Tools/Ports
        5. Click on Upload and wait until it says: Done uploading.
    3. Let’s talk:
        1. Open the dot342-serial-lora code and upload it to the Portenta without entering the flash mode (no green light flashing needed...)
        2. If it uploads without problem, you should be able to open the Serial Monitor (Tools/Serial Monitor) and if you write something on the bar at the top and press enter it should be sending it over LoRa and receiving other messages on the same frequency and default settings. If you can match the default settings shown above, you should be able to communicate between the Portenta and any raw LoRa device, but I recommend you to follow the next steps because you should be capable of changing at least the Frequency and some other settings of the LoRa communication in order for it to be really useful and have the capabilities to use it in any context or region... (915 is not legal in Europe...)

### 3.  Setting ****Arduino Core for STM32L0 based boards****

<aside>
🔥 This step is setting-up a thing that only works with a Linux machine, let’s focus on making it work on our Raspberry Pi!

</aside>

This instructions are based on Jeremy Ellis’ instructions over here:

[GitHub - hpssjellis/ArduinoCore-stm32l0 at vision_shield](https://github.com/hpssjellis/ArduinoCore-stm32l0/tree/vision_shield)

But I’ve adapted them a little for the context of this tutorial:

1. In the Arduino IDE, go to File/Preferences or press Ctrl+Comma
2. Press the little windows at the right of “Additional Boards Manager URLs:” 
3. Paste this in the window that will Pop up: [https://grumpyoldpizza.github.io/ArduinoCore-stm32l0/package_stm32l0_boards_index.json](https://grumpyoldpizza.github.io/ArduinoCore-stm32l0/package_stm32l0_boards_index.json) and press “OK” to close the pop-up window
4. Below the “Additional Boards Manager URLs” should said: “More preferences can be edited directly in the file: and below it a path like this:
    
    > /root/.arduino15/preferences.txt
    > 
    
    Press this path and it will open the folder. Leave it open because we will use it later...
    
5. Press “OK” again to close the Preferences window
6. Go to Tools/Board and open the Boards Manager
7. Search for "Tlera Corp STM32L0 Boards” and install it
8. Go to the folder that we open earlier and select “Open in Terminal” (after right clicking on it). For me it was easier with this method, but you should be able to access this folder using the terminal (sudo su and then looking for the path)
9. In the terminal type:
    
    ```bash
    cd packages/TleraCopr/hardware/stm32l0/<VERSION>/drivers/linux/
    ```
    
10. Continue in the terminal and type:
    
    ```bash
    sudo cp *.rules /etc/udev/rules.d
    ```
    
11. Then reboot the computer
    
    ```bash
    sudo reboot
    ```
    
12. Once rebooted, open the Arduino IDE again as root, typing:
    
    ```bash
    sudo arduino
    ```
    
13. Navigate to your Arduino sketch folder. 
    
    Mine was located on: /root/Arduino. The easiest way to access it is to follow the step 4. again and then navigate to it and open the terminal (I access it directly from the terminal...)
    
14. Create a folder named hardware. You can do this using the using your mouse or in terminal with:
    
    ```bash
    mkdir hardware
    ```
    
15. Change directories to it
    
    ```bash
    cd hardware
    ```
    
16. Clone this repo: https://github.com/hpssjellis/ArduinoCore-stm32l0.git (In the instructions they mention the original repo for this code, but we need the specific vision_shield branch, so you should type this instead:
    
    ```bash
    git clone https://github.com/hpssjellis/ArduinoCore-stm32l0.git --branch vision_shield --single-branch TleraCorp/stm32l0 
    
    ```
    
17. Restart the Arduino IDE: Close the Arduino IDE going to your initial terminal window and pressing Ctrl+C (Or you can close everything one by one...) and then type again sudo arduino 
18. Now if you go to Tools/Board: You should be able to see a new option called Tlera Corp STM32L0 Boards (in sketchbook) and when opening it you should be able to see a list of boards, if one is called **Arduino Portenta Vision Shield,** congratulations! We should have all setup in order for it to work...

### 4. Portenta-Murata Bridge and Murata-LoRa (LoRa Settings!)

<aside>
🔥 This works only with a Linux machine

</aside>

With everything that we have already done until this part, this should be easy and work ok (I hope!)

### 1. Accessing the examples:

At last, we need to access the examples from Jeremy Ellis

[portenta-pro-community-solutions/examples/dot3-portenta-vision-shields/dot34-pure-lora/dot343-extras at main · hpssjellis/portenta-pro-community-solutions](https://github.com/hpssjellis/portenta-pro-community-solutions/tree/main/examples/dot3-portenta-vision-shields/dot34-pure-lora/dot343-extras)

You can do it as you please, I did it cloning the whole repo and then accessing it from the Arduino IDE opening the sketches, here are the steps:Clone the repo somewhere on your Raspberry:

1. In your Arduino IDE go to File/Examples:
    
    > portenta-pro-community-solutions/dot3-portenta-vision-shields/dot34-pure-lora/dot343-extras/
    > 
2. Open the following examples (one by one):
    - dot3432-portenta-murata-bridge
    - dot3433-murata-lora-grumpyoldpizza
    - dot3434-portenta-serial

### 2. **Portenta LoRa Murata model raw programming using GrumpyOldPizza STM32IO core**

<aside>
🎬 This part follows the same steps that Jerey Ellis goes through this video:

[https://www.youtube.com/watch?v=d2lkL6oCsgg&t=108s](https://www.youtube.com/watch?v=d2lkL6oCsgg&t=108s)

</aside>

1. Go to dot3432-portenta-murata-bridge:
    
    We will use the dot3432-portenta-murata-bridge.ino code to “start the communication” with the LoRA Murata module in order to change the LoRa settings as we please.
    
    1. Select Port and Board: Tools/Board/Arduino Mbed OS Portenta Boards/Arduino Portenta H7 (M7 Core) and Tools/Ports/[your port] (for me it was “/dev/ttyACM0”)
    2. Double press the boot button on the Portenta and it should start flashing the green light (bootloader mode)
    3. Upload the code to the Portenta
    4. If it uploads ok (Done uploading), congrats again, we are almost over!
2. Go to dot3433-murata-lora-grumpyoldpizza (let’s talk with the Murata module!)
    
    This code is where the LoRa Settings are located, you can go and change them, they are located between lines 120 and 126 and they use the library LoRaRadio, you can check it here: 
    
    [ArduinoCore-stm32l0/libraries/LoRaRadio at vision_shield · hpssjellis/ArduinoCore-stm32l0](https://github.com/hpssjellis/ArduinoCore-stm32l0/tree/vision_shield/libraries/LoRaRadio)
    
    For now I changed the frequency (line 121: LoRaRadio.setFrequency(XXXXX);) to **869525000** because I’m in Europe, but you can edit the settings to fit your needs.
    
    Once ready, let’s Upload this code with this steps:
    
    1. Go to T**ools/Board/Tlera Corp STM32L0 Boards (in sketchbook)** and select Arduino Portenta Vision Shield
    2. Upload the code (Do not double press button into bootloader mode, must just be in regular mode)
3. Go to dot3434-portenta-serial:
    
    This code will help us send messages using the serial port
    
    1. Go to **Tools/Boad/Arduino Mbed OS Portenta Boards** and select Arduino Portenta H7 (M7 Core)
    2. Check the port is still selected on **Tools/Port**
    3. If everything is ok Upload the code and wait for the “Done Uploading” message. 
    4. Go to **Tools/Serial Monitor** and write something and press Enter (or send in the UI), it should say: 
        
        ```
        Sending with length: <message length>
        <your message>
        ```
        
        
        If so, we are ready!
        
    

## Raspberry + Dragino LoRa Shield

1. Follow the first two steps
    
    [https://github.com/pmanzoni/raspi-lmic](https://github.com/pmanzoni/raspi-lmic)
    
2. Go to raspi-lmic/src/lmic open the config.h and uncomment the line 66:
    
    ```cpp
    #define DISABLE_INVERT_IQ_ON_RX
    ```
    
3. Change LoRa settings (if needed):
    1. Go to raspi-lmic/examples/raw and open raw.cpp
    2. Go to line 128 to find the setup function and change the settings that you need (It should communicate by default with the Portenta as it is).
4. Build the executable:
    
    ```bash
    cd raspi-lmic/examples/raw
    make
    ```
    
5. Run:
    
    ```bash
    sudo ./raw
    ```
    

## Let’s talk!

While running the Raspberry code and with a serial monitor window opened in the Raspberry IDE, you should start receiving “Hello word!” from the raspberry onto the Arduino Portenta.

If you type something in the serial monitor and press enter, it should arrive onto the Raspberry Pi terminal
