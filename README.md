# Zentrix-DFOP-Configuration-Loader
Purpose: Loads user-defined configuration parameters from config.json including thermal thresholds, RGB presets, haptic patterns, and audio tuning profiles. Enables real-time customization and firmware reactivity. Applies To: Zentrix Pulse V1, Nexus V1, Apex V1 Author: Zentrix Embedded Group | Christopher Perry
//========================================================
// DFOP CoreStack v1.2.3 - Configuration Loader
// Loads parameters from /config.json on SSD or Flash
// Prepared by: Zentrix Embedded Group | Christopher Perry
//========================================================

#include "dfop_config_loader.h"
#include "dfop_thermal.h"
#include "dfop_rgb.h"
#include "dfop_audio.h"
#include "dfop_haptic.h"
#include "dfop_logger.h"
#include "json_parser.h"
#include "storage.h"

#define CONFIG_FILE_PATH "/mnt/ssd/config.json"

typedef struct {
    int fan_trigger_temp;
    int shutdown_temp;
    int rgb_idle_color[3];   // RGB values
    int haptic_intensity;
    int volume_startup;
} DFOP_Config;

DFOP_Config current_config;

// Load configuration at boot
void load_config_from_file() {
    char raw_json[2048];

    if (!file_exists(CONFIG_FILE_PATH)) {
        log_event("Config file missing – loading defaults");
        load_default_config();
        return;
    }

    read_file(CONFIG_FILE_PATH, raw_json, sizeof(raw_json));
    parse_config_json(raw_json);
    apply_config_to_modules();
    log_event("Config loaded and applied");
}

// Parse JSON into config struct
void parse_config_json(const char* json) {
    current_config.fan_trigger_temp  = json_get_int(json, "fan_trigger_temp", 75);
    current_config.shutdown_temp     = json_get_int(json, "shutdown_temp", 82);
    current_config.rgb_idle_color[0] = json_get_int(json, "rgb_idle_r", 10);
    current_config.rgb_idle_color[1] = json_get_int(json, "rgb_idle_g", 10);
    current_config.rgb_idle_color[2] = json_get_int(json, "rgb_idle_b", 10);
    current_config.haptic_intensity  = json_get_int(json, "haptic_intensity", 128);
    current_config.volume_startup    = json_get_int(json, "volume_startup", 65);
}

// Apply loaded config to DFOP modules
void apply_config_to_modules() {
    set_fan_trigger_threshold(current_config.fan_trigger_temp);
    set_shutdown_threshold(current_config.shutdown_temp);
    set_idle_rgb_color(
        current_config.rgb_idle_color[0],
        current_config.rgb_idle_color[1],
        current_config.rgb_idle_color[2]
    );
    set_haptic_strength(current_config.haptic_intensity);
    set_initial_volume(current_config.volume_startup);
}

// Fallback config
void load_default_config() {
    current_config.fan_trigger_temp = 75;
    current_config.shutdown_temp = 82;
    current_config.rgb_idle_color[0] = 20;
    current_config.rgb_idle_color[1] = 20;
    current_config.rgb_idle_color[2] = 20;
    current_config.haptic_intensity = 128;
    current_config.volume_startup = 70;

    apply_config_to_modules();
}
