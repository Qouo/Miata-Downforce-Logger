# Miata-Downforce-Logger
Im broke, need to test aero, this might or might not work. 


<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
</head>
<body>
    <h1>Miata Downforce Measurement</h1>
    <p>This project provides a Python program to measure downforce in a Miata using load cells and a MegaSquirt ECU. Below are the instructions to set up and run the program.</p>
    
<h2>Requirements</h2>
    <ul>
        <li>Load cells</li>
        <li>Microcontroller (e.g., Arduino)</li>
        <li><code>pyserial</code> library for Python</li>
    </ul>

<h2>Setup</h2>
    <p>First, ensure you have the required libraries installed:</p>
    <pre><code>pip install pyserial</code></pre>
    
 <h2>Python Program</h2>
    <p>Create a Python script with the following code:</p>
    <pre><code>import serial

import time

# Replace 'COM3' with the correct port for your microcontroller
ser = serial.Serial('COM3', 9600, timeout=1)
time.sleep(2)  # Wait for the serial connection to initialize

def read_load_cells():
    ser.write(b'R')  # Command to read load cells (this will depend on your microcontroller's code)
    line = ser.readline().decode('utf-8').strip()
    if line:
        try:
            values = [float(x) for x in line.split(',')]
            return values
        except ValueError:
            print("Invalid data received: ", line)
            return None
    return None

def calculate_downforce(load_cell_data):
    if load_cell_data:
        # Assume load_cell_data is a list of force measurements from multiple load cells
        # The exact calculation will depend on your setup and calibration
        total_downforce = sum(load_cell_data)
        return total_downforce
    return 0

def send_to_megasquirt(downforce):
    # Format the data for MegaSquirt ECU
    data = f"D,{downforce}\n"
    ser.write(data.encode())

try:
    while True:
        load_cell_data = read_load_cells()
        downforce = calculate_downforce(load_cell_data)
        print(f"Downforce: {downforce} N")
        send_to_megasquirt(downforce)
        time.sleep(1)  # Adjust the frequency of measurements as needed

except KeyboardInterrupt:
    print("Measurement stopped by user")
    ser.close()
</code></pre>

<h2>Explanation</h2>
    <p>The script performs the following steps:</p>
    <ol>
        <li><strong>Serial Communication:</strong> Opens a serial connection to the microcontroller. Make sure the port (<code>COM3</code>) and baud rate (<code>9600</code>) match your microcontroller's configuration.</li>
        <li><strong>Reading Load Cells:</strong> The <code>read_load_cells</code> function sends a command to the microcontroller to read the load cells and returns the data.</li>
        <li><strong>Calculating Downforce:</strong> The <code>calculate_downforce</code> function sums up the values from the load cells to get the total downforce. This is a simplification; you may need to calibrate and adjust based on your specific setup.</li>
        <li><strong>Sending Data to MegaSquirt:</strong> The <code>send_to_megasquirt</code> function formats the data and sends it to the MegaSquirt ECU.</li>
    </ol>

<h2>Microcontroller Code</h2>
<p>Ensure your microcontroller (e.g., Arduino) is set up to read from the load cells and send the data via serial. Here’s a basic example in Arduino code:</p>
<pre><code>#include <HX711.h>

HX711 scale;

void setup() {
  Serial.begin(9600);
  scale.begin(DOUT, CLK);  // Connect your HX711 DOUT and CLK pins
  scale.set_scale(2280.f); // Adjust to match your calibration
  scale.tare();            // Reset the scale to 0
}

void loop() {
  if (Serial.available() > 0) {
    char command = Serial.read();
    if (command == 'R') {
      float load1 = scale.get_units(10);  // Read average of 10 readings
      float load2 = scale.get_units(10);  // Repeat for additional load cells if necessary
      Serial.print(load1);
      Serial.print(",");
      Serial.println(load2);
    }
  }
}
</code></pre>

<p>This is a basic example to get you started. You will need to adjust the program based on your specific hardware setup, including the number of load cells and their placement on your Miata. Additionally, you may need to calibrate the load cells to get accurate measurements.</p>
</body>
</html>
