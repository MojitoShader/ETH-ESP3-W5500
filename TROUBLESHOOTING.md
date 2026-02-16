# ESP32 W5500 Ethernet Library - Installation & Troubleshooting Guide

## Quick Installation

### Arduino IDE
1. Download this repository as ZIP
2. Arduino IDE → Sketch → Include Library → Add .ZIP Library
3. Select the downloaded ZIP file
4. Restart Arduino IDE

### PlatformIO
Add to your `platformio.ini`:
```ini
[env:esp32dev]
platform = espressif32
board = esp32dev
framework = arduino
lib_deps = 
    https://github.com/yourusername/ETH-ESP3-W5500.git
```

## Hardware Wiring Guide

### Standard ESP32 DevKit to W5500 Module

```
ESP32 Pin    W5500 Pin    Function
---------    ---------    --------
3.3V     →   VCC/3.3V     Power Supply
GND      →   GND          Ground
GPIO 18  →   SCLK         SPI Clock
GPIO 19  →   MISO         SPI Master In Slave Out  
GPIO 23  →   MOSI         SPI Master Out Slave In
GPIO 5   →   CS/SS        Chip Select (configurable)
```

### Alternative CS Pin Options
- GPIO 5 (recommended, used in examples)
- GPIO 15 (alternative)
- GPIO 2 (alternative) 
- GPIO 4 (alternative)

**Important**: Update your code with the correct CS pin:
```cpp
Ethernet.init(5);  // Use your chosen CS pin
```

### Power Requirements
- **Voltage**: 3.3V (some modules support 5V)
- **Current**: 150-200mA typical, up to 300mA peak
- **Decoupling**: Add 100nF ceramic + 10µF electrolytic capacitors near VCC

## Common Issues & Solutions

### Issue: "Ethernet shield was not found"

**Symptoms:**
```
Ethernet shield was not found.
```

**Causes & Solutions:**

1. **Wrong CS Pin**
   ```cpp
   // Try different CS pins
   Ethernet.init(5);   // Most common for ESP32
   Ethernet.init(15);  // Alternative 1
   Ethernet.init(2);   // Alternative 2
   ```

2. **Wiring Problems**
   - Double-check all SPI connections
   - Verify 3.3V power supply
   - Check ground connections
   - Use short, quality jumper wires

3. **Power Issues**
   - Ensure adequate current supply (200mA+)
   - Add decoupling capacitors
   - Try external power supply if USB power insufficient

4. **Faulty Module**
   - Test with multimeter for power/ground continuity
   - Try a different W5500 module

### Issue: "Failed to configure Ethernet using DHCP"

**Symptoms:**
```
Failed to configure Ethernet using DHCP
```

**Solutions:**

1. **Check Network Connection**
   ```cpp
   if (Ethernet.linkStatus() == LinkOFF) {
     Serial.println("Ethernet cable is not connected");
   }
   ```

2. **Increase DHCP Timeout**
   ```cpp
   // Wait longer for DHCP response
   if (Ethernet.begin(mac, 60000, 10000) == 0) {
     Serial.println("DHCP failed");
   }
   ```

3. **Try Static IP First**
   ```cpp
   // Test with static IP to verify hardware
   IPAddress ip(192, 168, 1, 177);
   Ethernet.begin(mac, ip);
   ```

4. **Router/Network Issues**
   - Check if DHCP is enabled on router
   - Verify network cable
   - Try different Ethernet port on router

### Issue: Intermittent Connection Drops

**Solutions:**

1. **Power Stability**
   - Add better decoupling capacitors
   - Use external 3.3V regulator if needed
   - Check for voltage drops under load

2. **SPI Clock Speed**
   ```cpp
   // In w5100.h, try slower speed:
   #define SPI_ETHERNET_SETTINGS SPISettings(8000000, MSBFIRST, SPI_MODE0)
   ```

3. **Cable Quality**
   - Use quality Ethernet cable
   - Try shorter cable
   - Check for interference

4. **DHCP Lease Maintenance**
   ```cpp
   void loop() {
     Ethernet.maintain();  // Maintain DHCP lease
     // ... your code ...
   }
   ```

### Issue: Slow Network Performance

**Solutions:**

1. **Increase SPI Speed** (if stable)
   ```cpp
   // In w5100.h, try faster speed:
   #define SPI_ETHERNET_SETTINGS SPISettings(20000000, MSBFIRST, SPI_MODE0)
   ```

2. **Buffer Management**
   ```cpp
   // Check buffer availability before writing
   while (client.availableForWrite() < dataSize) {
     delay(1);
   }
   client.write(data, dataSize);
   ```

3. **Use Larger Buffers** (if memory permits)
   ```cpp
   // In Ethernet.h, uncomment:
   #define ETHERNET_LARGE_BUFFERS
   ```

### Issue: MQTT Connection Problems

**Solutions:**

1. **Keep-Alive Settings**
   ```cpp
   mqttClient.setKeepAlive(60);  // 60 seconds
   ```

2. **Connection Monitoring**
   ```cpp
   void loop() {
     if (!mqttClient.connected()) {
       reconnectMQTT();
     }
     mqttClient.loop();
   }
   ```

3. **QoS and Buffer Sizes**
   ```cpp
   // Use QoS 0 for better performance
   mqttClient.publish("topic", "message", false);
   ```

## Debugging Tips

### Enable Debug Output
```cpp
void debugEthernet() {
  Serial.print("Hardware: ");
  switch(Ethernet.hardwareStatus()) {
    case EthernetNoHardware:
      Serial.println("No hardware detected");
      break;
    case EthernetW5500:
      Serial.println("W5500 detected");
      break;
    default:
      Serial.println("Unknown hardware");
  }
  
  Serial.print("Link: ");
  switch(Ethernet.linkStatus()) {
    case LinkON:
      Serial.println("Connected");
      break;
    case LinkOFF:
      Serial.println("Disconnected");
      break;
    default:
      Serial.println("Unknown");
  }
  
  Serial.print("IP: ");
  Serial.println(Ethernet.localIP());
}
```

### Test Basic Connectivity
```cpp
void testPing() {
  EthernetClient client;
  if (client.connect("8.8.8.8", 53)) {  // Google DNS
    Serial.println("Basic connectivity OK");
    client.stop();
  } else {
    Serial.println("Cannot reach external server");
  }
}
```

### Memory Usage Check (ESP32)
```cpp
void checkMemory() {
  Serial.print("Free heap: ");
  Serial.print(ESP.getFreeHeap());
  Serial.println(" bytes");
}
```

## Performance Optimization

### For High-Speed Applications
```cpp
// In w5100.h, increase SPI speed:
#define SPI_ETHERNET_SETTINGS SPISettings(25000000, MSBFIRST, SPI_MODE0)

// In Ethernet.h, enable large buffers:
#define ETHERNET_LARGE_BUFFERS
```

### For Low-Memory Applications  
```cpp
// In Ethernet.h, reduce socket count:
#define MAX_SOCK_NUM 4
```

### For MQTT Applications
```cpp
// Optimize keep-alive and buffer settings
client.setKeepAlive(30);
client.setSocketTimeout(5);
```

## Version Compatibility

### ESP32 Core Versions
- **Recommended**: 2.0.0 or newer
- **Minimum**: 1.0.6
- **Note**: Some older cores may have SPI timing issues

### Arduino IDE Versions
- **Recommended**: 1.8.19 or newer
- **Minimum**: 1.8.13

### PlatformIO
- **Platform**: espressif32 @ 5.0.0 or newer
- **Framework**: arduino

## Getting Help

### Before Reporting Issues
1. Verify hardware connections
2. Test with provided examples
3. Check serial monitor output
4. Try different CS pins
5. Test with static IP configuration

### Information to Include in Bug Reports
- ESP32 board model and core version
- W5500 module/shield model
- Wiring diagram or connection details
- Complete error messages
- Minimal code example that reproduces the issue
- Serial monitor output

### Community Support
- Arduino Forum ESP32 section
- GitHub Issues (for this library)
- ESP32 community Discord/Reddit

---

This troubleshooting guide covers the most common issues encountered when using ESP32 with W5500 Ethernet controllers. Most problems are related to wiring, power supply, or configuration issues rather than software bugs.