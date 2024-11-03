# Electricity-using-SSB```
#include <iostream>
#include <cmath>

// Constants
const float speed_breaker_capacity = 1000;  // Watts
const float piezo_sensor_efficiency = 0.8;  // 80% efficient
const float pcu_efficiency = 0.9;  // 90% efficient
const float ssb_capacity = 5000;  // Wh (5kWh)
const float load_power = 500;  // Watts (LED lighting)

// Variables
float speed;
float energy_generated;
float sensor_energy;
float pcu_energy;
float ssb_soc;

// Function prototypes
float calculate_energy_generated(float speed);
float calculate_sensor_energy(float energy);
float calculate_pcu_energy(float sensor_energy);
void update_ssb_soc(float pcu_energy, float load_power);

int main() {
    // Initialize variables
    speed = 0;
    ssb_soc = 0;

    // Simulation loop
    for (int i = 0; i < 3600; i++) {  // 1-hour simulation
        speed = 30 + 10 * sin(2 * 3.14 * i / 3600);  // Simulated speed profile
        energy_generated = calculate_energy_generated(speed);
        sensor_energy = calculate_sensor_energy(energy_generated);
        pcu_energy = calculate_pcu_energy(sensor_energy);
        update_ssb_soc(pcu_energy, load_power);

        // Print results
        std::cout << "Time: " << i << "s, Speed: " << speed << " km/h, SSB SOC: " << ssb_soc << " Wh" << std::endl;
    }

    return 0;
}

// Function definitions
float calculate_energy_generated(float speed) {
    return speed_breaker_capacity * sin(2 * 3.14 * speed / 60);
}

float calculate_sensor_energy(float energy) {
    return energy * piezo_sensor_efficiency;
}

float calculate_pcu_energy(float sensor_energy) {
    return sensor_energy * pcu_efficiency;
}

void update_ssb_soc(float pcu_energy, float load_power) {
    ssb_soc += pcu_energy - load_power / 3600;
    if (ssb_soc > ssb_capacity) {
        ssb_soc = ssb_capacity;
    } else if (ssb_soc < 0) {
        ssb_soc = 0;
    }
}
```

*Arduino Code (for hardware implementation)*
```
c++
const int piezo_pin = A0;  // Piezoelectric sensor pin
const int pcu_pin = 9;  // Power conversion unit pin
const int ssb_pin = 10;  // Solid-state battery pin
const int load_pin = 11;  // Load pin

float speed_breaker_capacity = 1000;  // Watts
float piezo_sensor_efficiency = 0.8;  // 80% efficient
float pcu_efficiency = 0.9;  // 90% efficient
float ssb_capacity = 5000;  // Wh (5kWh)
float load_power = 500;  // Watts (LED lighting)

void setup() {
    pinMode(piezo_pin, INPUT);
    pinMode(pcu_pin, OUTPUT);
    pinMode(ssb_pin, OUTPUT);
    pinMode(load_pin, OUTPUT);
    Serial.begin(9600);
}

void loop() {
    int piezo_reading = analogRead(piezo_pin);
    float speed = map(piezo_reading, 0, 1023, 0, 60);  // Map piezo reading to speed
    float energy_generated = speed_breaker_capacity * sin(2 * 3.14 * speed / 60);
    float sensor_energy = energy_generated * piezo_sensor_efficiency;
    float pcu_energy = sensor_energy * pcu_efficiency;

    // Update SSB SOC
    float ssb_soc = calculate_ssb_soc(pcu_energy, load_power);

    // Control PCU and load
    analogWrite(pcu_pin, map(pcu_energy, 0, 1000, 0, 255));
    digitalWrite(load_pin, ssb_soc > 0 ? HIGH : LOW);

    // Print results
    Serial.print("Speed: ");
    Serial.print(speed);
    Serial.print(" km/h, SSB SOC: ");
    Serial.print(ssb_soc);
    Serial.println(" Wh");
    delay(1000);
}

float calculate_ssb_soc(float pcu_energy, float load_power) {
    static float ssb_soc = 0;
    ssb_soc += pcu_energy - load_power / 3600;
    if (ssb_soc > ssb_capacity) {
        ssb_soc = ssb
```
