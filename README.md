# Automated Toggle Switch Light

I rent my place, so I didn’t want to cut into the walls or replace switches. But I still wanted to control my overhead light with Google Home. The overhead fixture is a built-in LED panel, so I couldn’t just swap the bulb for a smart one. Other smart options (like inline relays or smart wall switches) required electrical work I didn’t want to risk in a rental.

So I built a removable, servo-powered “robot finger” that presses the light switch for me. It’s cheap, uses parts I already had, and can be removed without damage. I also added a DHT11 temperature and humidity sensor just for fun. It's not exactly pretty but it works!

![alt text](20250922_013749.jpg)

---

## Programming Setup

1. **Connect ESP32 to Arduino Cloud**

   * Plug the ESP32 into your computer using a USB cable that supports data (not just charging).
   * Go to [Arduino Cloud](https://cloud.arduino.cc), log in, and add a new device.
   * My board is an **ESP-WROOM-32 DevKit V1**, so I chose **ESP32-WROOM-DA Module** from the compatible list.
   * Follow the prompts to create a new *Thing*, entering your Wi-Fi name, password, and the Secret Key that Arduino Cloud gives you.

2. **Set up Google Home variables**

   * Under your Thing, add a cloud variable named `Light`.
   * Choose **Google → Light** as the type.
   * Set it to **Read & Write** and **On Change**.
   * (Optional) Add `Temperature` and `Humidity` if you’re using a DHT11 sensor. Note: Arduino Cloud doesn’t have a “humidity” type for Google Home, so I just used the **CloudTemperatureSensor** type for both. Mark these as **Read Only** and **Periodic**.

3. **Load the code**

   * Copy the `automatic-light.ino` sketch into your Thing’s sketch file.
   * Remove the temperature/humidity parts if you don’t have a sensor.
   * Update the pin numbers to match your wiring.

4. **Tune the servos**

   * You’ll need to experiment with servo angles to get reliable switch presses.
   * Too little rotation → the switch doesn’t click.
   * Too much → the servo pushes itself loose or strains.

5. **Connect to Google Home**

    * Go to Google Home on your preferred device
    * Devices -> Add -> Works with Google Home -> Arduino -> Connect Account
    * Finally you can choose to add the Light (and Temperature and Humidity) to a room. 

---

## Physical Setup

### Servo Mounting

I used **SG90 micro-servos**. They come with plastic “horns” (arms) you can screw onto the shaft.

For mounting:

* You can 3D print a bracket if you want something clean.
* I went low-tech: heavy-duty wall-mount Velcro strips. A dab of glue between the Velcro pieces helped hold against servo torque.

---

### Wiring

#### Powering the Servos

* Each SG90 servo needs \~2100-250 mA (up to \~360 mA peak). Two can pull close to 1 A.
* The ESP32 can’t provide this from its 3.3 V or 5 V pins. If you try, the board will brown out or reset. I've found that they may work for a few uses, but they always eventually start whining, because of lack of power, instead of moving.
* Solution: use an external 5 V supply. I used an old USB-C phone charger cable (5 V, 2–3 A).

> 💡 Tip: Beginners may find it easier to buy a “USB breakout board” or “5 V wall adapter with screw terminals” instead of splicing wires.

If you do splice a USB cable:

1. Cut off the device end. Strip back the insulation until you find two main wires (sometimes red/black, sometimes other colors). If there are more than 2 wires, it's likely you used a data-capable charger. This is fine you'll just have to test the wires to see which 2 are ground and power. Then tape off the remaining wires.
2. Plug the USB end into a power source and use a multimeter to check which is 5 V and which is GND.

   * Set the multimeter to 20v
   * If the meter reads **+5 V**, the wire on the black probe is ground.
   * If it reads **-5 V**, swap the probes.
3. Connect these to jumper wires for breadboard use.
    
    * Strip a jumper wire with a male end. Twist the jumper wire copper up with the USB ground copper then wrap with electrical tape to prevent shorts.
    * Do the same for the power wire

#### Final Connections

1. Plug the external 5 V and GND into your breadboard rails.
2. Connect both servos’ red wires to 5 V and both brown/black wires to GND.
3. Connect the external **GND** to the ESP32 **GND**. This shared ground is essential for the servo signal to be understood. The servo compares the signal voltage to its own ground to know how much to turn by. So the signal pin and the servo need to have the same ground so that they have the same reference point. 
4. Connect the servos’ signal wires (orange usually) to your chosen ESP32 pins. (I used GPIO 26 and 27.)
5. If using a DHT11 sensor:

   * Connect its VCC to **3.3 V** or **5V** on the ESP32.
   * Data pin to GPIO 33 (or another free pin).
   * GND to GND.

---

![Birdseye view of entire setup](20250922_013322.jpg)
![Closeup of wires and their connections](20250922_013334.jpg)