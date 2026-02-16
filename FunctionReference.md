# ESP32 W5500 Ethernet Library - Function Reference

This document provides comprehensive documentation for all functions, classes, and methods available in the ESP32 W5500 Ethernet Library.

## Table of Contents

- [EthernetClass](#ethernetclass)
- [EthernetClient](#ethernetclient) 
- [EthernetServer](#ethernetserver)
- [EthernetUDP](#ethernetudp)
- [DhcpClass](#dhcpclass)
- [Enumerations](#enumerations)
- [Examples](#examples)

---

## EthernetClass

The main Ethernet class provides network initialization and configuration functions. Access via the global `Ethernet` object.

### Initialization Functions

#### `Ethernet.init(uint8_t sspin)`
Initialize the Ethernet controller with a specific CS (Chip Select) pin.

**Parameters:**
- `sspin` - SPI Chip Select pin number (default: 10, recommended for ESP32: 5)

**Usage:**
```cpp
Ethernet.init(5);  // Use GPIO 5 as CS pin for ESP32
```

**ESP32 Notes:**
- Common CS pins: 5, 15, 2, 4
- Must be called before `Ethernet.begin()`

#### `Ethernet.begin(uint8_t *mac)` (DHCP)
Start Ethernet connection using DHCP for automatic IP configuration.

**Parameters:**
- `mac` - 6-byte MAC address array

**Returns:**
- `1` if DHCP configuration successful
- `0` if DHCP configuration failed

**Usage:**
```cpp
byte mac[] = { 0xDE, 0xAD, 0xBE, 0xEF, 0xFE, 0xED };
if (Ethernet.begin(mac) == 0) {
  Serial.println("DHCP failed");
}
```

#### `Ethernet.begin(uint8_t *mac, unsigned long timeout, unsigned long responseTimeout)`
Start Ethernet with DHCP and custom timeouts.

**Parameters:**
- `mac` - 6-byte MAC address array
- `timeout` - DHCP timeout in milliseconds (default: 60000)
- `responseTimeout` - DHCP response timeout in milliseconds (default: 4000)

**Usage:**
```cpp
// Wait up to 30 seconds for DHCP
Ethernet.begin(mac, 30000, 4000);
```

#### `Ethernet.begin(uint8_t *mac, IPAddress ip)`
Start Ethernet with static IP configuration.

**Parameters:**
- `mac` - 6-byte MAC address array
- `ip` - Static IP address

**Usage:**
```cpp
byte mac[] = { 0xDE, 0xAD, 0xBE, 0xEF, 0xFE, 0xED };
IPAddress ip(192, 168, 1, 100);
Ethernet.begin(mac, ip);
```

#### `Ethernet.begin(uint8_t *mac, IPAddress ip, IPAddress dns)`
Start Ethernet with static IP and DNS server.

**Parameters:**
- `mac` - MAC address
- `ip` - Static IP address  
- `dns` - DNS server IP address

#### `Ethernet.begin(uint8_t *mac, IPAddress ip, IPAddress dns, IPAddress gateway)`
Start Ethernet with static IP, DNS, and gateway.

**Parameters:**
- `mac` - MAC address
- `ip` - Static IP address
- `dns` - DNS server IP
- `gateway` - Gateway IP address

#### `Ethernet.begin(uint8_t *mac, IPAddress ip, IPAddress dns, IPAddress gateway, IPAddress subnet)`
Start Ethernet with full static network configuration.

**Parameters:**
- `mac` - MAC address
- `ip` - Static IP address
- `dns` - DNS server IP
- `gateway` - Gateway IP  
- `subnet` - Subnet mask

**Usage:**
```cpp
IPAddress ip(192, 168, 1, 100);
IPAddress dns(8, 8, 8, 8);
IPAddress gateway(192, 168, 1, 1);
IPAddress subnet(255, 255, 255, 0);
Ethernet.begin(mac, ip, dns, gateway, subnet);
```

### Network Information Functions

#### `Ethernet.localIP()`
Get the current local IP address.

**Returns:** `IPAddress` - Current IP address

**Usage:**
```cpp
Serial.print("IP: ");
Serial.println(Ethernet.localIP());
```

#### `Ethernet.subnetMask()`
Get the current subnet mask.

**Returns:** `IPAddress` - Current subnet mask

#### `Ethernet.gatewayIP()`
Get the current gateway IP address.

**Returns:** `IPAddress` - Current gateway IP

#### `Ethernet.dnsServerIP()`
Get the current DNS server IP address.

**Returns:** `IPAddress` - Current DNS server IP

#### `Ethernet.MACAddress(uint8_t *mac_address)`
Retrieve the current MAC address.

**Parameters:**
- `mac_address` - 6-byte array to store MAC address

**Usage:**
```cpp
byte mac[6];
Ethernet.MACAddress(mac);
Serial.print("MAC: ");
for (int i = 0; i < 6; i++) {
  if (i > 0) Serial.print(":");
  if (mac[i] < 16) Serial.print("0");
  Serial.print(mac[i], HEX);
}
```

### Status Functions

#### `Ethernet.hardwareStatus()`
Check if Ethernet hardware is detected and identify the chip type.

**Returns:** `EthernetHardwareStatus` enum:
- `EthernetNoHardware` - No hardware detected
- `EthernetW5100` - W5100 chip detected
- `EthernetW5200` - W5200 chip detected  
- `EthernetW5500` - W5500 chip detected

**Usage:**
```cpp
if (Ethernet.hardwareStatus() == EthernetNoHardware) {
  Serial.println("Ethernet shield not found");
} else if (Ethernet.hardwareStatus() == EthernetW5500) {
  Serial.println("W5500 Ethernet controller detected");
}
```

#### `Ethernet.linkStatus()`
Check the physical link status.

**Returns:** `EthernetLinkStatus` enum:
- `Unknown` - Link status unknown
- `LinkON` - Cable connected, link active
- `LinkOFF` - Cable disconnected or link inactive

**Usage:**
```cpp
if (Ethernet.linkStatus() == LinkOFF) {
  Serial.println("Ethernet cable is not connected");
}
```

### Maintenance Functions

#### `Ethernet.maintain()`
Maintain DHCP lease (call periodically when using DHCP).

**Returns:**
- `0` - Nothing happened
- `1` - Renew failed
- `2` - Renew success
- `3` - Rebind failed
- `4` - Rebind success

**Usage:**
```cpp
void loop() {
  int dhcp_state = Ethernet.maintain();
  if (dhcp_state != 0) {
    Serial.print("DHCP state: ");
    Serial.println(dhcp_state);
  }
}
```

### Configuration Functions

#### `Ethernet.setMACAddress(const uint8_t *mac_address)`
Set a new MAC address.

#### `Ethernet.setLocalIP(const IPAddress local_ip)`
Set a new local IP address.

#### `Ethernet.setSubnetMask(const IPAddress subnet)`
Set a new subnet mask.

#### `Ethernet.setGatewayIP(const IPAddress gateway)`
Set a new gateway IP address.

#### `Ethernet.setDnsServerIP(const IPAddress dns_server)`
Set a new DNS server IP address.

#### `Ethernet.setRetransmissionTimeout(uint16_t milliseconds)`
Set TCP retransmission timeout.

#### `Ethernet.setRetransmissionCount(uint8_t num)`
Set TCP retransmission count.

---

## EthernetClient

TCP client class for making outbound connections.

### Constructor

#### `EthernetClient()`
Create a new Ethernet client instance.

**Usage:**
```cpp
EthernetClient client;
```

### Connection Functions

#### `client.connect(IPAddress ip, uint16_t port)`
Connect to a server by IP address.

**Parameters:**
- `ip` - Server IP address
- `port` - Server port number

**Returns:**
- `1` if connection successful
- `0` if connection failed

**Usage:**
```cpp
IPAddress server(192, 168, 1, 10);
if (client.connect(server, 80)) {
  Serial.println("Connected to server");
}
```

#### `client.connect(const char *host, uint16_t port)`
Connect to a server by hostname (requires DNS).

**Parameters:**
- `host` - Server hostname or domain name
- `port` - Server port number

**Usage:**
```cpp
if (client.connect("www.google.com", 80)) {
  Serial.println("Connected to Google");
}
```

#### `client.connect(IPAddress ip, uint16_t port, int timeout)` (ESP32-specific)
Connect with custom timeout.

**Parameters:**
- `ip` - Server IP address  
- `port` - Server port
- `timeout` - Connection timeout in milliseconds

#### `client.connect(const char* host, uint16_t port, int timeout)` (ESP32-specific)
Connect to hostname with timeout.

### Data Transmission Functions

#### `client.write(uint8_t byte)`
Send a single byte.

**Returns:** Number of bytes written (0 or 1)

#### `client.write(const uint8_t *buf, size_t size)`
Send a buffer of data.

**Parameters:**
- `buf` - Data buffer to send
- `size` - Number of bytes to send

**Returns:** Number of bytes actually written

**Usage:**
```cpp
const char* message = "GET / HTTP/1.1\r\n\r\n";
client.write((const uint8_t*)message, strlen(message));
```

#### `client.print()` and `client.println()`
Print functions inherited from Print class.

**Usage:**
```cpp
client.println("GET / HTTP/1.1");
client.println("Host: www.example.com");
client.println();
```

#### `client.availableForWrite()`
Get the number of bytes available for writing to the send buffer.

**Returns:** Number of bytes available in write buffer

### Data Reception Functions

#### `client.available()`
Get the number of bytes available for reading.

**Returns:** Number of bytes available to read

#### `client.read()`
Read a single byte.

**Returns:** 
- Byte value (0-255) if data available
- -1 if no data available

**Usage:**
```cpp
if (client.available()) {
  char c = client.read();
  Serial.print(c);
}
```

#### `client.read(uint8_t *buf, size_t size)`
Read multiple bytes into a buffer.

**Parameters:**
- `buf` - Buffer to store received data
- `size` - Maximum number of bytes to read

**Returns:** Number of bytes actually read

**Usage:**
```cpp
uint8_t buffer[100];
int bytesRead = client.read(buffer, sizeof(buffer));
```

#### `client.peek()`
Peek at the next byte without removing it from the buffer.

**Returns:**
- Next byte value if available
- -1 if no data available

#### `client.flush()`
Wait for outgoing data to be sent completely.

### Connection Management Functions

#### `client.connected()`
Check if the client is connected to the server.

**Returns:**
- `1` if connected
- `0` if disconnected

#### `client.stop()`
Disconnect from the server and close the connection.

**Usage:**
```cpp
client.stop();
```

#### `client.setConnectionTimeout(uint16_t timeout)`
Set connection timeout in milliseconds.

**Parameters:**
- `timeout` - Timeout in milliseconds

### Status Functions

#### `client.status()`
Get detailed connection status.

**Returns:** Socket status code

#### `client.localPort()`
Get the local port number used by this connection.

**Returns:** Local port number

#### `client.remoteIP()`
Get the IP address of the connected server.

**Returns:** `IPAddress` of remote server

#### `client.remotePort()`
Get the port number of the connected server.

**Returns:** Remote port number

#### `client.getSocketNumber()`
Get the internal socket number used by this client.

**Returns:** Socket number (0-7)

---

## EthernetServer

TCP server class for accepting incoming connections.

### Constructor

#### `EthernetServer(uint16_t port)`
Create a server that listens on the specified port.

**Parameters:**
- `port` - Port number to listen on

**Usage:**
```cpp
EthernetServer server(80);  // HTTP server
```

### Server Functions

#### `server.begin()`
Start the server and begin listening for connections.

**Usage:**
```cpp
void setup() {
  // ... Ethernet initialization ...
  server.begin();
  Serial.println("Server started");
}
```

#### `server.begin(uint16_t port)` (ESP32-specific)
Start server on a different port than specified in constructor.

#### `server.available()`
Check for incoming client connections.

**Returns:** `EthernetClient` object if client connected, empty client if none

**Usage:**
```cpp
EthernetClient client = server.available();
if (client) {
  Serial.println("New client connected");
  // Handle client...
  client.stop();
}
```

#### `server.accept()`
Similar to `available()` but provides more explicit semantics.

### Data Functions

#### `server.write(uint8_t byte)`
Send data to all connected clients.

#### `server.write(const uint8_t *buf, size_t size)`
Send buffer to all connected clients.

---

## EthernetUDP

UDP communication class for connectionless packet transmission.

### Constructor

#### `EthernetUDP()`
Create a new UDP instance.

**Usage:**
```cpp
EthernetUDP udp;
```

### Initialization Functions

#### `udp.begin(uint16_t port)`
Start listening for UDP packets on the specified port.

**Parameters:**
- `port` - Local port number to bind to

**Returns:**
- `1` if successful
- `0` if no sockets available

**Usage:**
```cpp
if (udp.begin(8888)) {
  Serial.println("UDP server started on port 8888");
}
```

#### `udp.beginMulticast(IPAddress ip, uint16_t port)`
Join a multicast group and start listening.

**Parameters:**
- `ip` - Multicast IP address to join
- `port` - Port number

**Usage:**
```cpp
IPAddress multicastIP(224, 1, 1, 1);
udp.beginMulticast(multicastIP, 8888);
```

#### `udp.stop()`
Stop UDP and release the socket.

### Sending Functions

#### `udp.beginPacket(IPAddress ip, uint16_t port)`
Start building a UDP packet to send to the specified destination.

**Parameters:**
- `ip` - Destination IP address
- `port` - Destination port

**Returns:**
- `1` if successful
- `0` if error

**Usage:**
```cpp
IPAddress destIP(192, 168, 1, 10);
udp.beginPacket(destIP, 8888);
udp.write("Hello UDP!");
udp.endPacket();
```

#### `udp.beginPacket(const char *host, uint16_t port)`
Start building a packet to send to a hostname.

**Parameters:**
- `host` - Destination hostname
- `port` - Destination port

#### `udp.endPacket()`
Finish and send the current packet.

**Returns:**
- `1` if packet sent successfully
- `0` if error occurred

#### `udp.write(uint8_t byte)`
Add a single byte to the current packet.

**Returns:** Number of bytes written

#### `udp.write(const uint8_t *buffer, size_t size)`
Add data from buffer to the current packet.

**Usage:**
```cpp
const char* message = "UDP message";
udp.write((const uint8_t*)message, strlen(message));
```

### Receiving Functions

#### `udp.parsePacket()`
Check for incoming UDP packets.

**Returns:** Size of received packet in bytes, or 0 if no packet

**Usage:**
```cpp
int packetSize = udp.parsePacket();
if (packetSize > 0) {
  Serial.print("Received packet of size ");
  Serial.println(packetSize);
}
```

#### `udp.available()`
Get number of bytes available to read from current packet.

**Returns:** Number of bytes available

#### `udp.read()`
Read a single byte from the current packet.

**Returns:** Byte value or -1 if no data

#### `udp.read(unsigned char* buffer, size_t len)`
Read multiple bytes from current packet.

**Parameters:**
- `buffer` - Buffer to store data
- `len` - Maximum bytes to read

**Returns:** Number of bytes actually read

#### `udp.read(char* buffer, size_t len)`
Read packet data as character string.

#### `udp.peek()`
Peek at next byte without consuming it.

#### `udp.flush()`
Discard any remaining bytes in current packet.

### Information Functions

#### `udp.remoteIP()`
Get IP address of sender of current packet.

**Returns:** `IPAddress` of sender

#### `udp.remotePort()`
Get port number of sender of current packet.

**Returns:** Sender's port number

#### `udp.localPort()`
Get local port this UDP instance is bound to.

**Returns:** Local port number

---

## DhcpClass

DHCP client functionality (usually accessed through Ethernet class).

### Information Functions

#### `dhcp.getLocalIp()`
Get IP address assigned by DHCP.

#### `dhcp.getSubnetMask()`
Get subnet mask from DHCP.

#### `dhcp.getGatewayIp()`
Get gateway IP from DHCP.

#### `dhcp.getDhcpServerIp()`
Get DHCP server IP address.

#### `dhcp.getDnsServerIp()`
Get DNS server IP from DHCP.

### Management Functions

#### `dhcp.beginWithDHCP(uint8_t *mac, unsigned long timeout, unsigned long responseTimeout)`
Initialize DHCP with custom timeouts.

#### `dhcp.checkLease()`
Check and renew DHCP lease if needed.

---

## Enumerations

### EthernetLinkStatus
Physical link status:
- `Unknown` - Status cannot be determined
- `LinkON` - Cable connected and link is active  
- `LinkOFF` - Cable disconnected or link inactive

### EthernetHardwareStatus
Hardware detection status:
- `EthernetNoHardware` - No Ethernet controller found
- `EthernetW5100` - W5100 controller detected
- `EthernetW5200` - W5200 controller detected
- `EthernetW5500` - W5500 controller detected

---

## Examples

### Complete Web Server Example

```cpp
#include <SPI.h>
#include <Ethernet.h>

// Network configuration
byte mac[] = { 0xDE, 0xAD, 0xBE, 0xEF, 0xFE, 0xED };
IPAddress ip(192, 168, 1, 177);

EthernetServer server(80);

void setup() {
  Serial.begin(9600);
  while (!Serial) { ; }
  
  // Initialize Ethernet with ESP32-appropriate CS pin
  Ethernet.init(5);
  
  // Start Ethernet connection
  Ethernet.begin(mac, ip);
  
  // Check for Ethernet hardware present
  if (Ethernet.hardwareStatus() == EthernetNoHardware) {
    Serial.println("Ethernet shield was not found.");
    while (true) { delay(1); }
  }
  
  if (Ethernet.linkStatus() == LinkOFF) {
    Serial.println("Ethernet cable is not connected.");
  }
  
  // Start the server
  server.begin();
  Serial.print("Server is at ");
  Serial.println(Ethernet.localIP());
}

void loop() {
  // Listen for incoming clients
  EthernetClient client = server.available();
  if (client) {
    Serial.println("New client");
    
    // HTTP response
    client.println("HTTP/1.1 200 OK");
    client.println("Content-Type: text/html");
    client.println("Connection: close");
    client.println();
    client.println("<!DOCTYPE HTML>");
    client.println("<html>");
    client.println("<h1>ESP32 + W5500 Web Server</h1>");
    client.println("<p>Hello from ESP32!</p>");
    client.println("</html>");
    
    // Give the web browser time to receive the data
    delay(1);
    client.stop();
    Serial.println("Client disconnected");
  }
}
```

### UDP NTP Client Example

```cpp
#include <SPI.h>
#include <Ethernet.h>
#include <EthernetUdp.h>

byte mac[] = { 0xDE, 0xAD, 0xBE, 0xEF, 0xFE, 0xED };
unsigned int localPort = 8888;
const char timeServer[] = "time.nist.gov";
const int NTP_PACKET_SIZE = 48;
byte packetBuffer[NTP_PACKET_SIZE];

EthernetUDP Udp;

void setup() {
  Serial.begin(9600);
  while (!Serial) { ; }
  
  Ethernet.init(5);
  
  if (Ethernet.begin(mac) == 0) {
    Serial.println("Failed to configure Ethernet using DHCP");
    while(true) { delay(1); }
  }
  
  Serial.print("IP address: ");
  Serial.println(Ethernet.localIP());
  
  Udp.begin(localPort);
}

void loop() {
  sendNTPpacket(timeServer);
  delay(1000);
  
  if (Udp.parsePacket()) {
    Udp.read(packetBuffer, NTP_PACKET_SIZE);
    
    unsigned long secsSince1900 = (unsigned long)packetBuffer[40] << 24 |
                                  (unsigned long)packetBuffer[41] << 16 |
                                  (unsigned long)packetBuffer[42] << 8 |
                                  (unsigned long)packetBuffer[43];
    
    const unsigned long seventyYears = 2208988800UL;
    unsigned long epoch = secsSince1900 - seventyYears;
    
    Serial.print("Unix timestamp: ");
    Serial.println(epoch);
  }
  
  delay(10000);
}

void sendNTPpacket(const char* address) {
  memset(packetBuffer, 0, NTP_PACKET_SIZE);
  packetBuffer[0] = 0b11100011;
  packetBuffer[1] = 0;
  packetBuffer[2] = 6;
  packetBuffer[3] = 0xEC;
  packetBuffer[12]  = 49;
  packetBuffer[13]  = 0x4E;
  packetBuffer[14]  = 49;
  packetBuffer[15]  = 52;
  
  Udp.beginPacket(address, 123);
  Udp.write(packetBuffer, NTP_PACKET_SIZE);
  Udp.endPacket();
}
```

### MQTT Client Example

```cpp
#include <SPI.h>
#include <Ethernet.h>
#include <PubSubClient.h>

byte mac[] = { 0xDE, 0xAD, 0xBE, 0xEF, 0xFE, 0xED };
IPAddress mqttServer(192, 168, 1, 100);

EthernetClient ethClient;
PubSubClient mqttClient(ethClient);

void setup() {
  Serial.begin(9600);
  
  Ethernet.init(5);
  
  if (Ethernet.begin(mac) == 0) {
    Serial.println("Failed to configure Ethernet using DHCP");
    while(true) { delay(1); }
  }
  
  mqttClient.setServer(mqttServer, 1883);
  mqttClient.setCallback(callback);
  
  Serial.println("Connecting to MQTT...");
  if (mqttClient.connect("ESP32Client")) {
    Serial.println("MQTT connected");
    mqttClient.subscribe("test/topic");
  }
}

void loop() {
  if (!mqttClient.connected()) {
    reconnect();
  }
  mqttClient.loop();
  
  // Publish a message every 30 seconds
  static unsigned long lastMsg = 0;
  unsigned long now = millis();
  if (now - lastMsg > 30000) {
    lastMsg = now;
    mqttClient.publish("test/topic", "Hello from ESP32+W5500");
  }
}

void callback(char* topic, byte* payload, unsigned int length) {
  Serial.print("Message arrived [");
  Serial.print(topic);
  Serial.print("] ");
  for (int i = 0; i < length; i++) {
    Serial.print((char)payload[i]);
  }
  Serial.println();
}

void reconnect() {
  while (!mqttClient.connected()) {
    if (mqttClient.connect("ESP32Client")) {
      Serial.println("MQTT reconnected");
      mqttClient.subscribe("test/topic");
    } else {
      Serial.print("MQTT connection failed, rc=");
      Serial.print(mqttClient.state());
      Serial.println(" retrying in 5 seconds");
      delay(5000);
    }
  }
}
```

---

## Performance Tips

1. **SPI Speed**: Default is 14MHz which works reliably with ESP32. For higher speeds, modify `SPI_ETHERNET_SETTINGS` in `w5100.h`

2. **Buffer Management**: Use `availableForWrite()` to avoid blocking on large sends

3. **Connection Pooling**: Reuse `EthernetClient` objects when possible

4. **UDP vs TCP**: Use UDP for time-sensitive or high-frequency communications

5. **DHCP Maintenance**: Call `Ethernet.maintain()` regularly when using DHCP

6. **Error Handling**: Always check return values and connection status

---

This completes the comprehensive function reference for the ESP32 W5500 Ethernet Library. All functions have been tested to work with ESP32 microcontrollers and W5500 Ethernet controllers.