# Sensirion SCD4x Component (ESP-IDF)

ESP-IDF component for the Sensirion SCD4x (SCD40/SCD41) CO2, temperature, and humidity sensor over I2C.

## Features

- Periodic and low-power periodic measurement modes
- Single-shot measurement (SCD41 only)
- Temperature offset and altitude compensation
- Forced recalibration and automatic self-calibration (ASC)
- Power-down / wake-up for low-power applications
- Built-in CRC verification and I2C error recovery

## Requirements

- ESP-IDF >= 5.2.0
- I2C master driver (`esp_driver_i2c`)

## Installation

From your project directory, use the [IDF Component Manager](https://docs.espressif.com/projects/idf-component-manager/en/latest/use/how_to_add_dependency.html) to add the dependency from Git:

```bash
idf.py add-dependency scd4x --git https://github.com/jef-sure/esp32-component-scd4x.git
```

Or add it manually to your project's `main/idf_component.yml`:

```yaml
dependencies:
  scd4x:
    git: https://github.com/jef-sure/esp32-component-scd4x.git
```

Alternatively, clone into your project's `components/` directory:

```bash
cd components
git clone https://github.com/jef-sure/esp32-component-scd4x.git scd4x
```

## Wiring

| SCD4x Pin | Connection |
|-----------|------------|
| SDA       | I2C data line (any GPIO) |
| SCL       | I2C clock line (any GPIO) |
| VDD       | 3.3V       |
| GND       | GND        |

The I2C address is fixed at `0x62`. Connect SDA/SCL to whichever GPIOs you configure for your I2C master bus.

## Quick Start

```c
#include "scd4x.h"

// After initializing the I2C bus and adding device at address SCD4X_I2C_ADDR:
scd4x_t *scd4x = scd4x_init(bus_handle, dev_handle);

scd4x_start_periodic_measurement(scd4x);

while (true) {
    scd4x_measurement_t m = {0};
    esp_err_t ret = scd4x_read_measurement(scd4x, &m);
    if (ret == ESP_OK) {
        printf("CO2=%.0f ppm  T=%.2f °C  RH=%.2f %%\n",
               m.co2, m.temperature, m.humidity);
    }
    vTaskDelay(pdMS_TO_TICKS(5000));
}
```

See [examples/example_main.c](examples/example_main.c) for a complete working example.

## API Reference

### Types

```c
typedef struct {
    i2c_master_bus_handle_t bus_handle;
    i2c_master_dev_handle_t i2c_dev;
    bool has_error;
    bool is_measuring;
} scd4x_t;

typedef struct {
    float co2;         // CO2 in ppm
    float temperature; // Temperature in °C
    float humidity;    // Relative humidity in %RH
} scd4x_measurement_t;
```

### Initialization

| Function | Description |
|----------|-------------|
| `scd4x_init()` | Initialize sensor, stop any active measurement, allocate device handle |

### Measurement

| Function | Description |
|----------|-------------|
| `scd4x_start_periodic_measurement()` | Start periodic measurement (5 s interval) |
| `scd4x_start_low_power_periodic_measurement()` | Start low-power periodic measurement (~30 s interval) |
| `scd4x_stop_periodic_measurement()` | Stop periodic measurement, return to idle |
| `scd4x_read_measurement()` | Read CO2, temperature, humidity. Returns `ESP_ERR_NOT_FINISHED` if not ready |
| `scd4x_data_ready()` | Check if new data is available |
| `scd4x_measure_single_shot()` | Single-shot CO2 + T + RH measurement (SCD41 only, blocks 5 s) |
| `scd4x_measure_single_shot_rht_only()` | Single-shot T + RH only (SCD41 only, blocks 50 ms) |

### Configuration

| Function | Description |
|----------|-------------|
| `scd4x_set_temperature_offset()` | Set temperature offset for compensation (0–175 °C) |
| `scd4x_get_temperature_offset()` | Read current temperature offset |
| `scd4x_set_sensor_altitude()` | Set altitude for pressure compensation |
| `scd4x_get_sensor_altitude()` | Read configured altitude |
| `scd4x_set_ambient_pressure()` | Set ambient pressure in hPa (can be sent during measurement) |
| `scd4x_persist_settings()` | Save configuration to EEPROM (limited write cycles) |

### Calibration

| Function | Description |
|----------|-------------|
| `scd4x_perform_forced_recalibration()` | Forced recalibration to known CO2 reference |
| `scd4x_set_automatic_self_calibration()` | Enable/disable ASC |
| `scd4x_get_automatic_self_calibration()` | Read ASC state |

### Utility

| Function | Description |
|----------|-------------|
| `scd4x_get_serial_number()` | Read 48-bit sensor serial number |
| `scd4x_perform_self_test()` | Run built-in self-test |
| `scd4x_perform_factory_reset()` | Reset all settings to factory defaults |
| `scd4x_reinit()` | Reload settings from EEPROM |
| `scd4x_power_down()` | Enter sleep mode |
| `scd4x_wake_up()` | Wake from sleep mode |

## License

MIT — see [LICENSE](LICENSE).
