import time
import requests
import json

# Firebase Configuration
FIREBASE_HOST = "smart-home-f5995-default-rtdb.firebaseio.com"
FIREBASE_AUTH = "AxKFBFQR4F4294UvzSy1Am34dR3oN8R9JsSlDkX"
FIREBASE_URL = f"https://{FIREBASE_HOST}/Variable/Value.json?auth={FIREBASE_AUTH}"

# WiFi Configuration (for reference - Python handles network differently)
WIFI_SSID = "0opsyProxy :-)"
WIFI_PASSWORD = "12345678"

class FirebaseSensor:
    def __init__(self):
        self.firebase_url = FIREBASE_URL
        
    def set_firebase_string(self, value):
        """Send string value to Firebase"""
        try:
            response = requests.put(
                self.firebase_url,
                data=json.dumps(value),
                headers={'Content-Type': 'application/json'}
            )
            if response.status_code == 200:
                print(f"Data sent successfully: {value}")
                return True
            else:
                print(f"Error: {response.status_code} - {response.text}")
                return False
        except Exception as e:
            print(f"Exception occurred: {e}")
            return False
    
    def read_analog_sensor(self):
        """
        Simulated analog read function.
        Replace this with actual sensor reading code based on your hardware:
        - For Raspberry Pi: Use ADC like MCP3008 with SPI
        - For Arduino-style boards: Use appropriate GPIO library
        """
        # Simulated sensor value (0-1023 range like Arduino)
        import random
        return random.randint(0, 1023)
    
    def setup(self):
        """Initialize the system"""
        print("Starting Firebase Sensor System...")
        print(f"Connecting to Firebase: {FIREBASE_HOST}")
        
        # Send initial value
        self.set_firebase_string("smart-home-f5995-default-rtdb")
        print("Setup complete!")
    
    def loop(self):
        """Main loop - reads sensor and sends to Firebase"""
        while True:
            try:
                # Read sensor value
                sensor_data = self.read_analog_sensor()
                sensor_string = str(sensor_data)
                
                print(sensor_string)
                
                # Send to Firebase
                self.set_firebase_string(sensor_string)
                
                # Wait 1 second
                time.sleep(1)
                
            except KeyboardInterrupt:
                print("\nProgram stopped by user")
                break
            except Exception as e:
                print(f"Error in main loop: {e}")
                time.sleep(1)

def main():
    """Main entry point"""
    sensor = FirebaseSensor()
    sensor.setup()
    sensor.loop()

if __name__ == "__main__":
    main()


# ============================================
# HARDWARE-SPECIFIC IMPLEMENTATIONS
# ============================================

# For Raspberry Pi with MCP3008 ADC:
"""
import busio
import digitalio
import board
import adafruit_mcp3xxx.mcp3008 as MCP
from adafruit_mcp3xxx.analog_in import AnalogIn

def read_analog_sensor_rpi(self):
    # Create the SPI bus
    spi = busio.SPI(clock=board.SCK, MISO=board.MISO, MOSI=board.MOSI)
    cs = digitalio.DigitalInOut(board.D5)
    mcp = MCP.MCP3008(spi, cs)
    
    # Create an analog input channel on pin 0
    chan = AnalogIn(mcp, MCP.P0)
    
    # Return the raw value (0-1023)
    return chan.value // 64  # Convert 16-bit to 10-bit
"""

# For systems with ADS1115 ADC:
"""
import board
import adafruit_ads1x15.ads1115 as ADS
from adafruit_ads1x15.analog_in import AnalogIn

def read_analog_sensor_ads1115(self):
    i2c = board.I2C()
    ads = ADS.ADS1115(i2c)
    chan = AnalogIn(ads, ADS.P0)
    
    # Scale to 0-1023 range
    return int((chan.value / 32767) * 1023)
"""
