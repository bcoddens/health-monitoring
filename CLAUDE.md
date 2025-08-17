# Health Monitoring System with Home Assistant, Node-RED, Garmin Connect, and Intervals.icu

## Introduction
This document outlines the integration of **Home Assistant**, **Node-RED**, **Garmin Connect**, and **Intervals.icu** to track, monitor, and optimize various health metrics. By combining these tools, you can create a comprehensive system to monitor physical, mental, and emotional health, and take actionable steps to improve overall well-being.

---

## Health Metrics to Track

### 1. **Diet and Lifestyle**
   - **Healthy Diet**: Track diet-related metrics using smart scales and manual input to monitor weight, body composition, and body fat percentage.
   - **Avoid Smoking & Limit Alcohol**: Use Home Assistant to set reminders for healthy lifestyle choices.
   - **Daily Movement**: Track steps, walking, and general activity using Garmin devices or a fitness tracker integrated with Home Assistant.
   - **Nutritional Periodization (Cycling-Specific)**: Monitor macronutrient ratios (carbs, protein, fat) tailored to training volume and phase (base vs. race prep), as recommended by Alan Couzens. Adjust carb intake based on ride duration/intensity to optimize cycling performance.
   
### 2. **Physical Health Metrics**
   - **Heart Rate**: Track resting, recovery, and training heart rates using Garmin Connect integration with Home Assistant.
   - **VO2max**: Monitor cardio fitness improvements using Garmin data automatically synced to Home Assistant.
   - **Blood Pressure**: If you have a smart blood pressure cuff, integrate it with Home Assistant for automatic logging and alerts.
   - **Body Composition**: Use Home Assistant to track weight and body fat percentage from a smart scale.
   - **Grip Strength**: If you use a fitness tracker that measures grip strength, you can integrate this into your health dashboard.
   - **Flexibility & Mobility**: Track daily stretching sessions, range of motion exercises, and balance training using manual input helpers or yoga/stretching app integrations.
   - **Oral Health**: Monitor dental hygiene habits including brushing frequency, flossing, and dental check-up reminders through Home Assistant automation schedules.
   
   #### **Cycling-Specific Metrics**
   - **Cadence**: Track high-cadence (100-120+ RPM) sessions to improve neuromuscular efficiency using power meter data synced through Garmin Connect.
   - **Power/Torque**: Monitor power output and torque during strength-endurance rides (e.g., low-cadence hill climbs) for cycling-specific fitness development.
   - **Lactate Threshold Zones**: Track heart rate zones between LT1 and LT2 for Norwegian-style pyramidal training periodization.

### 3. **Recovery & Sleep Monitoring**
   - **Sleep**: Monitor sleep duration and quality using Garmin data automatically synced to Home Assistant for pattern analysis and alerts.
   - **Recovery Metrics**: Track HRV (Heart Rate Variability), recovery time, and stress levels from Garmin Connect integration with Home Assistant.
   - **Sleep Environment**: Monitor bedroom temperature, humidity, and noise levels using smart sensors integrated with Home Assistant.
   - **Sleep Hygiene**: Set automated reminders for consistent bedtimes and create lighting automations to support circadian rhythms.
   - **HRV-Based Readiness**: Use HRV 7-day rolling averages to assess cycling readiness and automatically adjust training load recommendations, following Alan Couzens' methodology for data-driven recovery monitoring.

### 4. **Mental & Emotional Health**
   - **Stress Levels**: Monitor stress indicators from Garmin Connect (stress score, body battery) automatically imported into Home Assistant.
   - **Mood Tracking**: Use Home Assistant input helpers to manually log daily mood and energy levels with trend analysis.
   - **Meditation & Mindfulness**: Track meditation sessions and mindfulness practice duration using timer automations in Home Assistant.
   - **Social Connection**: Monitor social activity patterns, track frequency of social interactions, and set reminders for maintaining strong relationships and social network engagement.
   - **Work-Life Balance**: Track work hours, break frequency, and time spent on leisure activities using presence detection and calendar integration.
   - **Sense of Purpose**: Use daily reflection tracking to monitor goal alignment, life satisfaction, and personal meaning metrics through Home Assistant input helpers.

### 5. **Environmental Health Factors**
   - **Air Quality**: Monitor indoor and outdoor air quality using smart sensors integrated with Home Assistant for health alerts.
   - **Temperature & Humidity**: Track optimal living conditions and receive notifications when levels affect health and comfort.
   - **Light Exposure**: Monitor natural light exposure and UV levels using weather data and smart light sensors.
   - **Noise Pollution**: Track ambient noise levels in living and working spaces using smart sound sensors.
   - **Water Quality**: If available, integrate smart water quality monitors to track hydration source quality.

---

## System Architecture & Data Flow

### Core Components Integration
- **Home Assistant**: Central hub receiving data from all sources and managing automations
- **Garmin Connect**: Primary source for fitness, sleep, and health metrics, including cycling-specific metrics (cadence, power, heart rate zones) via Home Assistant integration
- **Node-RED**: Data processing, custom flows, and advanced automation logic for cycling training and recovery analysis
- **Intervals.icu**: Training analysis and performance tracking integration, syncing cycling activities for detailed analytics and Alan Couzens methodology implementation
- **Smart Devices**: Scales, sensors, power meters, and IoT devices for comprehensive data collection
- **Power Meters**: ANT+ or Bluetooth power meters for real-time cycling performance data

### Data Flow Overview
1. **Data Collection**: Garmin devices (with power meters) → Garmin Connect → Home Assistant integration
2. **Data Processing**: Home Assistant sensors → Node-RED flows → Processed metrics (cadence trends, LT zones, HRV analysis)
3. **Data Storage**: Home Assistant recorder → InfluxDB (optional) → Long-term analytics
4. **Data Visualization**: Home Assistant dashboards → Grafana (optional) → Advanced cycling performance charts
5. **External Sync**: Training data → Intervals.icu → Cycling performance analysis and Alan Couzens methodology tracking
6. **External Integrations**: Support for external analytics and training platforms for enhanced data analysis

---

## Implementation Setup Guide

### 1. Home Assistant Setup
```yaml
# Required integrations in configuration.yaml
homeassistant:
  # Add to customize.yaml for friendly names
  customize: !include customize.yaml

# Garmin Connect Integration
garmin_connect:
  username: !secret garmin_username
  password: !secret garmin_password

# Input helpers for manual tracking
input_number:
  daily_mood:
    name: "Daily Mood (1-10)"
    min: 1
    max: 10
    step: 1
    icon: mdi:emoticon-happy

input_boolean:
  meditation_session:
    name: "Meditation Session"
    icon: mdi:meditation
  
  brushed_teeth:
    name: "Brushed Teeth Today"
    icon: mdi:tooth
  
  flossed_teeth:
    name: "Flossed Today"
    icon: mdi:tooth-outline
  
  stretching_session:
    name: "Daily Stretching"
    icon: mdi:human-handsup

# Additional input helpers for comprehensive health tracking
input_number:
  social_interactions:
    name: "Social Interactions Today"
    min: 0
    max: 20
    step: 1
    icon: mdi:account-group

  purpose_satisfaction:
    name: "Sense of Purpose (1-10)"
    min: 1
    max: 10
    step: 1
    icon: mdi:target

  flexibility_score:
    name: "Flexibility Rating (1-10)"
    min: 1
    max: 10
    step: 1
    icon: mdi:human-handsup


# Template sensors for health metrics
template:
  - sensor:
      - name: "Health Score"
        state: >
          {% set sleep = states('sensor.garmin_sleep_score') | float(0) %}
          {% set stress = 100 - states('sensor.garmin_stress_level') | float(0) %}
          {% set steps = (states('sensor.garmin_steps') | float(0) / 10000) * 100 %}
          {{ ((sleep + stress + steps) / 3) | round(1) }}
        unit_of_measurement: "%"
        icon: mdi:heart-pulse
```

### 2. Node-RED Health Flows
Install Node-RED add-on in Home Assistant and create flows for:
- **Health Data Processing**: Aggregate multiple metrics into composite scores
- **Trend Analysis**: Calculate moving averages and trend detection
- **Alert Management**: Send notifications based on health thresholds
- **Data Export**: Sync processed data to external services like Intervals.icu

### 3. Garmin Connect Integration
The Home Assistant Garmin Connect integration provides these entities:
- `sensor.garmin_steps` - Daily step count
- `sensor.garmin_sleep_score` - Sleep quality score (0-100)
- `sensor.garmin_stress_level` - Current stress level
- `sensor.garmin_body_battery` - Energy level indicator
- `sensor.garmin_resting_heart_rate` - RHR measurement
- `sensor.garmin_vo2_max` - Cardio fitness level

#### Cycling-Specific Garmin Entities
- `sensor.garmin_power_avg` - Average power output from recent rides
- `sensor.garmin_cadence_avg` - Average cadence from cycling activities
- `sensor.garmin_training_stress_score` - TSS for workout intensity tracking
- `sensor.garmin_power_zones` - Current power zone distribution
- `sensor.garmin_lactate_threshold_heart_rate` - LT heart rate for zone training

### 4. Intervals.icu Integration
Set up webhook automation to sync training data:
```yaml
# automation.yaml
- alias: "Sync Garmin to Intervals.icu"
  trigger:
    - platform: state
      entity_id: sensor.garmin_activity_completed
  action:
    - service: rest_command.intervals_icu_sync
      data:
        activity_id: "{{ trigger.to_state.attributes.activity_id }}"
```

---

## Dashboard & Visualization Strategy

### Home Assistant Dashboard Cards
```yaml
# health_dashboard.yaml
type: vertical-stack
cards:
  - type: gauge
    entity: sensor.health_score
    name: "Overall Health Score"
    min: 0
    max: 100
    severity:
      green: 80
      yellow: 60
      red: 0

  - type: history-graph
    entities:
      - sensor.garmin_resting_heart_rate
      - sensor.garmin_sleep_score
      - sensor.garmin_stress_level
    hours_to_show: 168
    refresh_interval: 60

  - type: statistics-graph
    entities:
      - sensor.garmin_steps
    stat_types:
      - mean
      - max
    period: day
```

#### Cycling-Specific Dashboard Cards
```yaml
# cycling_performance_dashboard.yaml
type: vertical-stack
cards:
  - type: horizontal-stack
    cards:
      - type: gauge
        entity: sensor.garmin_power_avg
        name: "Average Power (7-day)"
        min: 0
        max: 400
        unit: "W"
      - type: gauge
        entity: sensor.garmin_cadence_avg
        name: "Average Cadence"
        min: 60
        max: 120
        unit: "RPM"
        severity:
          green: 90
          yellow: 80
          red: 70

  - type: history-graph
    entities:
      - sensor.garmin_power_avg
      - sensor.garmin_training_stress_score
      - sensor.garmin_lactate_threshold_heart_rate
    hours_to_show: 336  # 2 weeks
    refresh_interval: 300

  - type: custom:mini-graph-card
    entities:
      - entity: sensor.garmin_hrv_7_day_avg
        name: "HRV Readiness"
        color: '#1db954'
    hours_to_show: 168
    points_per_hour: 1
    line_width: 3
    show:
      name: true
      state: true
      legend: false

  - type: entities
    entities:
      - entity: sensor.training_zone_distribution
        name: "Current Training Zones"
      - entity: sensor.weekly_tss
        name: "Weekly TSS"
      - entity: sensor.carb_periodization_target
        name: "Today's Carb Target"
```

### Advanced Analytics (Optional)
- **InfluxDB**: Store long-term health metrics for advanced analysis
- **Grafana**: Create detailed health dashboards with correlations
- **Jupyter Notebooks**: Custom analysis scripts for health data mining

---

## Alerting & Notification Systems

### Health Threshold Alerts
```yaml
# automations.yaml
- alias: "Low Sleep Quality Alert"
  trigger:
    - platform: numeric_state
      entity_id: sensor.garmin_sleep_score
      below: 60
  condition:
    - condition: time
      after: "08:00:00"
  action:
    - service: notify.mobile_app
      data:
        title: "Poor Sleep Detected"
        message: "Sleep score: {{ states('sensor.garmin_sleep_score') }}%. Consider adjusting tonight's routine."

- alias: "High Stress Alert"
  trigger:
    - platform: numeric_state
      entity_id: sensor.garmin_stress_level
      above: 80
      for: "00:30:00"
  action:
    - service: script.stress_management_routine
    - service: notify.mobile_app
      data:
        title: "High Stress Detected"
        message: "Consider taking a break. Stress level: {{ states('sensor.garmin_stress_level') }}"
```

### Proactive Health Reminders
```yaml
- alias: "Hydration Reminder"
  trigger:
    - platform: time_pattern
      hours: "/2"
  condition:
    - condition: state
      entity_id: person.user
      state: "home"
  action:
    - service: tts.speak
      data:
        message: "Time for a glass of water!"

- alias: "Movement Break"
  trigger:
    - platform: numeric_state
      entity_id: sensor.garmin_steps_last_hour
      below: 100
      for: "01:00:00"
  action:
    - service: notify.mobile_app
      data:
        title: "Movement Break"
        message: "You've been inactive for an hour. Time for a quick walk!"
```

---

## API Configuration & Authentication

### Garmin Connect API Setup
```yaml
# secrets.yaml
garmin_username: "your_garmin_email@example.com"
garmin_password: "your_garmin_password"

# configuration.yaml
garmin_connect:
  username: !secret garmin_username
  password: !secret garmin_password
  exclude:
    - floors  # Exclude if not needed to reduce sensor count
```

### Intervals.icu API Integration
```yaml
# secrets.yaml
intervals_icu_api_key: "your_intervals_icu_api_key"
intervals_icu_athlete_id: "your_athlete_id"

# rest_command configuration
rest_command:
  intervals_icu_sync:
    url: "https://intervals.icu/api/v1/athlete/{{ intervals_icu_athlete_id }}/activities"
    method: POST
    headers:
      Authorization: "API_KEY {{ intervals_icu_api_key }}"
      Content-Type: "application/json"
    payload: >
      {
        "external_id": "{{ activity_id }}",
        "name": "{{ activity_name }}",
        "start_date_local": "{{ start_time }}",
        "moving_time": {{ duration }},
        "distance": {{ distance }}
      }
```

### Optional External APIs
```yaml
# Weather data for environmental tracking
weather:
  - platform: openweathermap
    api_key: !secret openweather_api_key
    latitude: !secret home_latitude
    longitude: !secret home_longitude

# Air quality monitoring
air_quality:
  - platform: waqi
    token: !secret waqi_token
    locations:
      - "Your City"
```

---

## Goal Setting & Progress Tracking

### Health Goals Configuration
```yaml
# input_number helpers for goals
input_number:
  daily_steps_goal:
    name: "Daily Steps Goal"
    min: 5000
    max: 20000
    step: 500
    initial: 10000
    icon: mdi:walk

  weekly_exercise_goal:
    name: "Weekly Exercise Sessions"
    min: 1
    max: 10
    step: 1
    initial: 5
    icon: mdi:dumbbell

  sleep_hours_goal:
    name: "Nightly Sleep Goal (hours)"
    min: 6
    max: 10
    step: 0.5
    initial: 8
    icon: mdi:sleep

  # Cycling-specific goals
  weekly_tss_goal:
    name: "Weekly TSS Goal"
    min: 200
    max: 1000
    step: 50
    initial: 400
    icon: mdi:bike

  lt1_percentage_goal:
    name: "LT1 Zone Percentage (Norwegian Model)"
    min: 60
    max: 85
    step: 5
    initial: 80
    icon: mdi:heart-pulse

  high_cadence_sessions_goal:
    name: "High Cadence Sessions per Week"
    min: 1
    max: 5
    step: 1
    initial: 2
    icon: mdi:speedometer

# Template sensors for goal tracking
template:
  - sensor:
      - name: "Steps Goal Progress"
        state: >
          {% set steps = states('sensor.garmin_steps') | float(0) %}
          {% set goal = states('input_number.daily_steps_goal') | float(10000) %}
          {{ (steps / goal * 100) | round(1) }}
        unit_of_measurement: "%"

      - name: "Weekly Exercise Progress"
        state: >
          {% set sessions = states('counter.weekly_exercise_sessions') | float(0) %}
          {% set goal = states('input_number.weekly_exercise_goal') | float(5) %}
          {{ (sessions / goal * 100) | round(1) }}
        unit_of_measurement: "%"

      # Cycling-specific progress tracking
      - name: "Weekly TSS Progress"
        state: >
          {% set current_tss = states('sensor.weekly_total_tss') | float(0) %}
          {% set goal = states('input_number.weekly_tss_goal') | float(400) %}
          {{ (current_tss / goal * 100) | round(1) }}
        unit_of_measurement: "%"
        icon: mdi:bike

      - name: "LT1 Zone Adherence"
        state: >
          {% set lt1_time = states('sensor.weekly_lt1_zone_time') | float(0) %}
          {% set total_time = states('sensor.weekly_training_time') | float(1) %}
          {% set percentage = (lt1_time / total_time * 100) | round(1) %}
          {% set goal = states('input_number.lt1_percentage_goal') | float(80) %}
          {{ (percentage / goal * 100) | round(1) }}
        unit_of_measurement: "%"
        icon: mdi:heart-pulse

      - name: "Norwegian Model Compliance"
        state: >
          {% set lt1_pct = states('sensor.lt1_zone_adherence') | float(0) %}
          {% set lt2_sessions = states('sensor.weekly_lt2_sessions') | float(0) %}
          {% set vo2_sessions = states('sensor.weekly_vo2_sessions') | float(0) %}
          {% if lt1_pct >= 80 and lt2_sessions >= 1 and vo2_sessions >= 1 %}
            Compliant
          {% else %}
            Non-Compliant
          {% endif %}
        icon: mdi:check-circle
```

### Progress Tracking Automations
```yaml
# Weekly progress report with cycling metrics
- alias: "Weekly Health & Cycling Report"
  trigger:
    - platform: time
      at: "09:00:00"
    - platform: time
      weekday:
        - sun
  action:
    - service: notify.mobile_app
      data:
        title: "Weekly Health & Training Summary"
        message: >
          Health: Steps {{ states('sensor.weekly_avg_steps') }}, Sleep {{ states('sensor.weekly_avg_sleep') }}h
          Training: TSS {{ states('sensor.weekly_total_tss') }}, LT1 {{ states('sensor.lt1_zone_adherence') }}%
          Norwegian Model: {{ states('sensor.norwegian_model_compliance') }}
          HRV Readiness: {{ states('sensor.garmin_hrv_7_day_avg') }}

# Norwegian pyramidal training compliance check
- alias: "Norwegian Training Model Alert"
  trigger:
    - platform: state
      entity_id: sensor.norwegian_model_compliance
      to: "Non-Compliant"
      for: "24:00:00"
  action:
    - service: notify.mobile_app
      data:
        title: "Training Model Alert"
        message: >
          Your training is not following the Norwegian pyramidal model. 
          Current LT1%: {{ states('sensor.lt1_zone_adherence') }}% (Target: 80%+)
          Consider adjusting this week's intensity distribution.
          
# Goal achievement celebrations
- alias: "Daily Steps Goal Achieved"
  trigger:
    - platform: numeric_state
      entity_id: sensor.steps_goal_progress
      above: 100
  action:
    - service: script.celebration_routine
    - service: notify.mobile_app
      data:
        title: "🎉 Steps Goal Achieved!"
        message: "Great job! You've reached your daily steps goal."
```

---

## Data Management, Privacy & Security

### Data Storage Strategy
```yaml
# recorder.yaml - Control what data is stored
recorder:
  purge_keep_days: 365
  include:
    entities:
      - sensor.garmin_*
      - sensor.health_score
      - input_number.daily_mood
  exclude:
    entities:
      - sensor.garmin_floors  # Exclude unnecessary data
    event_types:
      - call_service
```

### Privacy Considerations
- **Local Storage**: All health data stays within your Home Assistant instance
- **Data Encryption**: Enable encryption for Home Assistant database
- **Network Security**: Use HTTPS and secure your Home Assistant installation
- **Access Control**: Configure user accounts with appropriate permissions
- **Data Sharing**: Only share anonymized data with external services when necessary

### Security Best Practices
```yaml
# Enable recorder encryption
recorder:
  db_url: "sqlite:////config/home-assistant_v2.db?cipher=aes256&password=your_encryption_key"

# Secure secrets management
# Use secrets.yaml and never commit credentials to version control
```

### Backup Strategy
```yaml
# Google Drive Backup add-on configuration
name: "Health Data Backup"
folders:
  - config
  - addons/local
  - share
  - ssl
  - media
password: !secret backup_password
keep_days: 30
```

---

## System Monitoring & Health Checks

### System Health Sensors
```yaml
# Monitor system performance
template:
  - sensor:
      - name: "Database Size"
        state: "{{ (states.sensor.home_assistant_database_size.state | float / 1024) | round(2) }}"
        unit_of_measurement: "GB"

      - name: "Garmin Integration Status"
        state: >
          {% if states('sensor.garmin_steps') != 'unavailable' %}
            Online
          {% else %}
            Offline
          {% endif %}

# System health monitoring
- alias: "System Health Check"
  trigger:
    - platform: time_pattern
      hours: "/6"
  action:
    - service: system_health.info
    - condition: template
      value_template: "{{ states('sensor.garmin_integration_status') == 'Offline' }}"
    - service: notify.admin
      data:
        title: "Health System Alert"
        message: "Garmin integration is offline. Check credentials and connection."
```

### Performance Optimization
```yaml
# Optimize recorder for health data
recorder:
  auto_purge: true
  purge_keep_days: 90
  commit_interval: 5
  include:
    domains:
      - sensor
      - input_number
      - input_boolean
    entities:
      - automation.daily_health_summary
```

---

## Troubleshooting & Maintenance

### Common Issues

#### Garmin Connect Integration Issues
1. **Authentication Errors**
   - Verify credentials in secrets.yaml
   - Check if 2FA is enabled (may require app passwords)
   - Restart Home Assistant after credential changes

2. **Missing Data**
   - Ensure Garmin device is syncing to Garmin Connect app
   - Check integration logs in Home Assistant
   - Verify entity names match your device model

3. **Slow Updates**
   - Garmin data typically updates every 15-30 minutes
   - Force sync by opening Garmin Connect app
   - Check network connectivity

#### Node-RED Flow Issues
1. **Flow Not Triggering**
   - Verify Home Assistant WebSocket connection
   - Check entity state changes in Home Assistant
   - Review Node-RED debug logs

2. **Data Processing Errors**
   - Validate input data types in function nodes
   - Check for null/undefined values
   - Review template syntax

### Maintenance Schedule

#### Daily
- Review health dashboards for anomalies
- Check system alerts and notifications

#### Weekly
- Review health trends and patterns
- Update goals based on progress
- Check system health sensors

#### Monthly
- Review database size and performance
- Update integrations and add-ons
- Backup configuration files
- Review and optimize automations

#### Quarterly
- Analyze long-term health trends
- Update health goals and targets
- Review privacy settings
- Clean up unused entities

---

## Cost Analysis & Optimization

### Free Tier Components
- **Home Assistant**: Free open-source platform
- **Garmin Connect**: Free with Garmin device purchase
- **Node-RED**: Free Home Assistant add-on
- **Basic Dashboards**: Included with Home Assistant

### Optional Paid Services
- **Intervals.icu**: $8/month for premium features
- **Home Assistant Cloud**: $6.50/month for remote access
- **Weather APIs**: $0-10/month depending on usage
- **Cloud Storage**: $5-15/month for backups

### Hardware Costs
- **Garmin Device**: $200-800 (one-time purchase)
- **Smart Scale**: $30-200 (optional)
- **Smart Sensors**: $20-100 each (air quality, temperature, etc.)
- **Server Hardware**: $100-500 (Raspberry Pi to dedicated server)

### Optimization Tips
1. **Use Local Processing**: Keep data processing local to avoid cloud costs
2. **Selective Monitoring**: Only track metrics you actively use
3. **Efficient Storage**: Configure recorder to only store necessary data
4. **Free Alternatives**: Use open-source tools where possible
5. **Bundled Services**: Consider device ecosystems for better integration

### ROI Considerations
- **Health Improvements**: Early detection of health issues
- **Habit Formation**: Data-driven behavior changes
- **Time Savings**: Automated tracking vs manual logging
- **Peace of Mind**: Continuous health monitoring
- **Long-term Savings**: Preventive health management

---

## Alan Couzens Training Methodology Integration

### HRV-Based Training Load Management
```yaml
# Template sensors for Alan Couzens HRV methodology
template:
  - sensor:
      - name: "HRV Training Readiness"
        state: >
          {% set hrv_current = states('sensor.garmin_hrv_7_day_avg') | float(0) %}
          {% set hrv_baseline = states('input_number.hrv_baseline') | float(50) %}
          {% set deviation = ((hrv_current - hrv_baseline) / hrv_baseline * 100) %}
          {% if deviation >= 5 %}
            High Readiness
          {% elif deviation >= -5 %}
            Normal Readiness
          {% else %}
            Low Readiness
          {% endif %}
        icon: mdi:heart-pulse

      - name: "Training Load Recommendation"
        state: >
          {% set readiness = states('sensor.hrv_training_readiness') %}
          {% set weekly_tss = states('sensor.weekly_total_tss') | float(0) %}
          {% set target_tss = states('input_number.weekly_tss_goal') | float(400) %}
          {% if readiness == 'High Readiness' %}
            {{ (target_tss * 1.1) | round(0) }}
          {% elif readiness == 'Normal Readiness' %}
            {{ target_tss | round(0) }}
          {% else %}
            {{ (target_tss * 0.8) | round(0) }}
          {% endif %}
        unit_of_measurement: "TSS"
        icon: mdi:trending-up

# Nutritional periodization based on training load
      - name: "Carb Periodization Target"
        state: >
          {% set training_load = states('sensor.training_load_recommendation') | float(400) %}
          {% set body_weight = states('sensor.smart_scale_weight') | float(70) %}
          {% if training_load > 500 %}
            {{ (body_weight * 8) | round(0) }}
          {% elif training_load > 300 %}
            {{ (body_weight * 6) | round(0) }}
          {% else %}
            {{ (body_weight * 4) | round(0) }}
          {% endif %}
        unit_of_measurement: "g"
        icon: mdi:food-apple

# Automation for HRV-based training adjustments
automation:
  - alias: "HRV Training Load Adjustment"
    trigger:
      - platform: time
        at: "06:00:00"
    condition:
      - condition: state
        entity_id: sensor.hrv_training_readiness
        state: "Low Readiness"
    action:
      - service: notify.mobile_app
        data:
          title: "Training Adjustment Recommended"
          message: >
            HRV suggests low readiness. Consider reducing today's training load.
            Recommended TSS: {{ states('sensor.training_load_recommendation') }}
            Current carb target: {{ states('sensor.carb_periodization_target') }}g

  - alias: "Nutritional Periodization Alert"
    trigger:
      - platform: state
        entity_id: sensor.carb_periodization_target
    action:
      - service: input_number.set_value
        target:
          entity_id: input_number.daily_carb_target
        data:
          value: "{{ states('sensor.carb_periodization_target') }}"
      - service: notify.mobile_app
        data:
          title: "Nutrition Update"
          message: "Today's carb target updated to {{ states('sensor.carb_periodization_target') }}g based on training load"
```

### Pyramidal Training Distribution Tracking
```yaml
# Weekly training zone distribution sensors
template:
  - sensor:
      - name: "Zone 1 Percentage"
        state: >
          {% set z1_time = states('sensor.weekly_zone1_time') | float(0) %}
          {% set total_time = states('sensor.weekly_training_time') | float(1) %}
          {{ (z1_time / total_time * 100) | round(1) }}
        unit_of_measurement: "%"

      - name: "Zone 2 Percentage"
        state: >
          {% set z2_time = states('sensor.weekly_zone2_time') | float(0) %}
          {% set total_time = states('sensor.weekly_training_time') | float(1) %}
          {{ (z2_time / total_time * 100) | round(1) }}
        unit_of_measurement: "%"

      - name: "Pyramidal Distribution Score"
        state: >
          {% set z1_pct = states('sensor.zone_1_percentage') | float(0) %}
          {% set z2_pct = states('sensor.zone_2_percentage') | float(0) %}
          {% set z3_plus_pct = 100 - z1_pct - z2_pct %}
          {% set ideal_z1 = 80 %}
          {% set ideal_z2 = 15 %}
          {% set ideal_z3_plus = 5 %}
          {% set score = 100 - (abs(z1_pct - ideal_z1) + abs(z2_pct - ideal_z2) + abs(z3_plus_pct - ideal_z3_plus)) %}
          {{ score | round(1) }}
        unit_of_measurement: "%"
        icon: mdi:chart-pyramid
```

### Integration with Intervals.icu for Advanced Analytics
```yaml
# Enhanced Intervals.icu sync with Couzens methodology
rest_command:
  intervals_icu_hrv_sync:
    url: "https://intervals.icu/api/v1/athlete/{{ intervals_icu_athlete_id }}/wellness"
    method: POST
    headers:
      Authorization: "API_KEY {{ intervals_icu_api_key }}"
      Content-Type: "application/json"
    payload: >
      {
        "id": "{{ now().strftime('%Y-%m-%d') }}",
        "hrvRmssd": {{ states('sensor.garmin_hrv_rmssd') }},
        "readiness": "{{ states('sensor.hrv_training_readiness') }}",
        "trainingReadiness": {{ states('sensor.training_load_recommendation') }},
        "sleepQuality": {{ states('sensor.garmin_sleep_score') }},
        "fatigue": {{ states('sensor.garmin_stress_level') }}
      }

automation:
  - alias: "Sync HRV Data to Intervals.icu"
    trigger:
      - platform: time
        at: "08:00:00"
    condition:
      - condition: template
        value_template: "{{ states('sensor.garmin_hrv_rmssd') not in ['unknown', 'unavailable'] }}"
    action:
      - service: rest_command.intervals_icu_hrv_sync
```

---

## Advanced Cycling Analytics: Xert Integration

### Xert Power Analysis Integration

#### API Configuration for Xert
```yaml
# secrets.yaml
xert_username: "your_xert_username"
xert_password: "your_xert_password"
xert_client_id: "xert_public"
xert_client_secret: "xert_public"

# rest_command configuration for Xert API
rest_command:
  xert_get_training_info:
    url: "https://www.xertonline.com/oauth/training_info"
    method: GET
    headers:
      Authorization: "Bearer {{ states('sensor.xert_access_token') }}"
      Content-Type: "application/json"
    
  xert_get_fitness_signature:
    url: "https://www.xertonline.com/oauth/training_info"
    method: GET
    headers:
      Authorization: "Bearer {{ states('sensor.xert_access_token') }}"
      Content-Type: "application/json"

# Template sensors for Xert advanced metrics
template:
  - sensor:
      - name: "Xert Fitness Signature FTP"
        state: "{{ state_attr('sensor.xert_training_data', 'signature')['ftp'] | round(0) }}"
        unit_of_measurement: "W"
        icon: mdi:lightning-bolt

      - name: "Xert Lower Threshold Power"
        state: "{{ state_attr('sensor.xert_training_data', 'signature')['ltp'] | round(0) }}"
        unit_of_measurement: "W"
        icon: mdi:gauge-low

      - name: "Xert High Intensity Energy"
        state: "{{ state_attr('sensor.xert_training_data', 'signature')['hie'] | round(0) }}"
        unit_of_measurement: "kJ"
        icon: mdi:battery-high

      - name: "Xert Peak Power"
        state: "{{ state_attr('sensor.xert_training_data', 'signature')['pp'] | round(0) }}"
        unit_of_measurement: "W"
        icon: mdi:flash

      - name: "Xert Training Status"
        state: "{{ state_attr('sensor.xert_training_data', 'status') }}"
        icon: >
          {% set status = state_attr('sensor.xert_training_data', 'status') %}
          {% if status == 'Very Fresh' %}
            mdi:battery-plus
          {% elif status == 'Fresh' %}
            mdi:battery-high
          {% elif status == 'Tired' %}
            mdi:battery-medium
          {% elif status == 'Very Tired' %}
            mdi:battery-low
          {% else %}
            mdi:battery-unknown
          {% endif %}

      - name: "Xert Training Load Peak"
        state: "{{ state_attr('sensor.xert_training_data', 'tl')['peak'] | round(1) }}"
        unit_of_measurement: "XSS"
        icon: mdi:trending-up

      - name: "Xert Training Recommendation"
        state: >
          {% set status = states('sensor.xert_training_status') %}
          {% set target_xss = state_attr('sensor.xert_training_data', 'targetXSS')['total'] %}
          {% if status == 'Very Fresh' %}
            High Intensity ({{ target_xss | round(0) }} XSS)
          {% elif status == 'Fresh' %}
            Moderate Training ({{ target_xss | round(0) }} XSS)
          {% elif status == 'Tired' %}
            Easy Recovery ({{ (target_xss * 0.5) | round(0) }} XSS)
          {% else %}
            Rest Day
          {% endif %}
        icon: mdi:run-fast
```

#### Xert Automation for Training Optimization
```yaml
# Automation to fetch Xert data daily
automation:
  - alias: "Update Xert Training Data"
    trigger:
      - platform: time
        at: "07:00:00"
    action:
      - service: rest_command.xert_get_training_info
      - delay: "00:00:05"
      - service: notify.mobile_app
        data:
          title: "Xert Training Update"
          message: >
            Status: {{ states('sensor.xert_training_status') }}
            FTP: {{ states('sensor.xert_fitness_signature_ftp') }}W
            Recommendation: {{ states('sensor.xert_training_recommendation') }}

  - alias: "Xert Fitness Breakthrough Detection"
    trigger:
      - platform: state
        entity_id: sensor.xert_fitness_signature_ftp
    condition:
      - condition: template
        value_template: >
          {{ (trigger.to_state.state | float) > (trigger.from_state.state | float) }}
    action:
      - service: notify.mobile_app
        data:
          title: "🚀 Fitness Breakthrough Detected!"
          message: >
            Your FTP increased from {{ trigger.from_state.state }}W to {{ trigger.to_state.state }}W!
            New HIE: {{ states('sensor.xert_high_intensity_energy') }}kJ
            Peak Power: {{ states('sensor.xert_peak_power') }}W

  - alias: "Adaptive Training Load Warning"
    trigger:
      - platform: state
        entity_id: sensor.xert_training_status
        to: "Very Tired"
    action:
      - service: notify.mobile_app
        data:
          title: "⚠️ Recovery Needed"
          message: >
            Xert shows you're Very Tired. Consider a rest day or easy recovery ride.
            Current training load: {{ states('sensor.xert_training_load_peak') }} XSS
```

### Enhanced Dashboard for Advanced Analytics

#### Xert Dashboard Cards
```yaml
# advanced_cycling_dashboard.yaml
type: vertical-stack
cards:
  # Xert Fitness Signature Section
  - type: custom:mushroom-title-card
    title: "Xert Fitness Signature"
    
  - type: horizontal-stack
    cards:
      - type: gauge
        entity: sensor.xert_fitness_signature_ftp
        name: "FTP"
        min: 150
        max: 400
        unit: "W"
        severity:
          green: 300
          yellow: 250
          red: 200
      - type: gauge
        entity: sensor.xert_high_intensity_energy
        name: "HIE"
        min: 15
        max: 35
        unit: "kJ"
        severity:
          green: 25
          yellow: 20
          red: 15

  - type: entities
    entities:
      - entity: sensor.xert_training_status
        name: "Training Status"
      - entity: sensor.xert_peak_power
        name: "Peak Power"
      - entity: sensor.xert_training_recommendation
        name: "Today's Recommendation"

  - type: history-graph
    entities:
      - sensor.xert_fitness_signature_ftp
      - sensor.xert_training_load_peak
      - sensor.weekly_total_tss
    hours_to_show: 168
    refresh_interval: 300

  # Combined Performance Overview
  - type: custom:apexcharts-card
    header:
      title: "Performance Integration"
    series:
      - entity: sensor.xert_fitness_signature_ftp
        name: "FTP"
        type: line
        color: blue
      - entity: sensor.weekly_total_tss
        name: "Weekly TSS"
        type: column
        color: green
      - entity: sensor.xert_training_load_peak
        name: "Training Load"
        type: line
        color: orange
    span:
      start: week
```

### Integration Benefits

#### Performance Optimization
- **Real-time Fitness Tracking**: Xert's dynamic fitness signature updates
- **Advanced Power Analysis**: Xert's dynamic fitness signature and training recommendations
- **Adaptive Training**: Automated workout adjustments based on freshness
- **Recovery Optimization**: Combined HRV + Xert status for rest day decisions

#### Data-Driven Insights
- **Breakthrough Detection**: Automatic FTP/power curve updates
- **Training Load Analysis**: Peak, high, and low intensity tracking
- **Performance Correlation**: Power output vs. training status analysis
- **Predictive Analytics**: Training readiness forecasting

#### Automation Enhancements
- **Smart Workout Selection**: Xert AI-generated workouts based on current status
- **Adaptive Training Load**: Dynamic intensity adjustments based on freshness status
- **Recovery Alerts**: Multi-metric fatigue detection (HRV + Xert + Sleep)
- **Performance Notifications**: Real-time breakthrough and achievement alerts

---


## Conclusion

This comprehensive health monitoring system provides a robust foundation for tracking, analyzing, and improving your overall well-being. By integrating Garmin Connect with Home Assistant and leveraging Node-RED for advanced automation, you can create a personalized health dashboard that grows with your needs.

The system's modular design allows you to start with basic metrics and gradually add more sophisticated monitoring and analysis capabilities.

Regular maintenance and optimization ensure the system remains efficient and provides valuable insights for years to come.

Remember that this system is a tool to support your health journey - the most important factor is taking action on the insights it provides to improve your daily habits and long-term wellness.