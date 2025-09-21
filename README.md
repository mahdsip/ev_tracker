# EV Tracker for Home Assistant

A comprehensive Electric Vehicle tracking system for Home Assistant that monitors trips and charging sessions for Volkswagen ID.Buzz vehicles.

## Purpose

This project provides detailed tracking of:
- **Trip Monitoring**: Distance, energy consumption, locations, and timestamps
- **Charging Sessions**: Energy added, costs, locations, and duration
- **Historical Data**: JSON logs of last 50 trips and charging sessions
- **Aggregated Statistics**: Daily, weekly, and monthly summaries

## Features

### Trip Tracking
- Automatic trip detection when vehicle starts/stops moving
- Records start/end locations, times, distance, and energy consumption
- Calculates efficiency (kWh/100km)
- Stores trip history in JSON format

### Charging Tracking
- Monitors charging sessions via charging switch state
- Location-based pricing (Endesa: €0.31/kWh, Iberdrola: €0.32/kWh, Default: €0.45/kWh)
- Tracks energy added and total cost
- Records charging location and duration

### Data Storage
- Trip data: `/config/ev_trips.json`
- Charging data: `/config/ev_charging.json`
- Maintains last 50 records for each type
- Compatible with Home Assistant dashboards

## Implementation Details

### Required Sensors
Your Home Assistant must have these entities from the Volkswagen integration:
- `binary_sensor.idbuzz_vehicle_moving` - Vehicle movement status
- `sensor.idbuzz_odometer` - Vehicle odometer reading
- `sensor.idbuzz_battery_level` - Battery percentage
- `switch.idbuzz_charging` - Charging status
- `device_tracker.idbuzz_position` - Vehicle location

### File Structure
```
packages/
├── trip_tracker.yaml       # Trip monitoring configuration
├── charging_tracker.yaml   # Charging session configuration
ev_tracker/
└── README.md               # This file
```

### Helpers Created
**Trip Tracking:**
- `idbuzz_previous_odometer` (km)
- `idbuzz_previous_battery_percent` (%)
- `idbuzz_last_trip_distance` (km)
- `idbuzz_last_trip_kwh` (kWh)
- `idbuzz_trip_start_location`
- `idbuzz_trip_end_location`
- `idbuzz_trip_start_time`
- `idbuzz_trip_end_time`

**Charging Tracking:**
- `idbuzz_charging_start_battery` (%)
- `idbuzz_last_charge_kwh` (kWh)
- `idbuzz_last_charge_cost` (€)
- `idbuzz_charging_location`
- `idbuzz_charging_start_time`
- `idbuzz_charging_end_time`

## Installation Steps

### 1. Prerequisites
- Home Assistant with Volkswagen integration configured
- File editor access (Studio Code Server, File Editor, etc.)
- SSH access or terminal access to Home Assistant

### 2. Install Files
1. Create directory: `/homeassistant/packages/`
2. Copy `trip_tracker.yaml` and `charging_tracker.yaml` to this directory

### 3. Configure Home Assistant
Add to your `configuration.yaml` in the `homeassistant:` section:
```yaml
homeassistant:
  packages: !include_dir_named packages
```

### 4. Restart Home Assistant
- Go to **Settings** → **System** → **Restart**
- Wait for restart to complete

### 5. Verify Installation
Check that these entities are created:
- All helpers listed above
- `sensor.ev_trip_history`
- `sensor.ev_charging_history`
- Utility meters for daily/weekly/monthly aggregation

### 6. Configure Charging Costs (Optional)
To get accurate charging costs based on different charging locations:

**Step 1: Create Zones in Home Assistant**
1. Go to **Settings** → **Areas & zones** → **Zones**
2. Click **Add Zone** and create zones for your charging locations:
   - Name: `endesa1` (for Endesa charging stations)
   - Name: `iberdrola1` (for Iberdrola charging stations)
   - Set the location and radius for each charging station

**Step 2: Modify Charging Costs**
Edit `charging_tracker.yaml` and update the `kwh_price` section with your actual prices:
```yaml
kwh_price: >
  {% if 'endesa1' in location %}
    0.31
  {% elif 'iberdrola1' in location %}
    0.32
  {% elif 'your_home_zone' in location %}
    0.25
  {% else %}
    0.45
  {% endif %}
```

**Current Default Prices:**
- **endesa1 zone**: €0.31/kWh
- **iberdrola1 zone**: €0.32/kWh
- **Other locations**: €0.45/kWh (default)

**Note**: The system detects charging location when charging starts and applies the corresponding price when calculating costs.

## Dashboard Integration

### Trip History Table
```yaml
type: markdown
content: |
  ## Recent EV Trips
  
  | Date | Start | End | Distance | Energy | Efficiency |
  |------|-------|-----|----------|--------|------------|
  {%- set trips = state_attr('sensor.ev_trip_history', 'trips') %}
  {%- if trips %}
    {%- for trip in trips[:10] %}
  | {{ trip.start_time[:16] }} | {{ trip.start_location }} | {{ trip.end_location }} | {{ trip.distance_km }}km | {{ trip.kwh_consumed }}kWh | {{ (trip.kwh_consumed / trip.distance_km * 100) | round(1) }}kWh/100km |
    {%- endfor %}
  {%- endif %}
```

### Charging History Table
```yaml
type: markdown
content: |
  ## Recent Charging Sessions
  
  | Date | Location | Energy | Cost | Price |
  |------|----------|--------|------|-------|
  {%- set sessions = state_attr('sensor.ev_charging_history', 'sessions') %}
  {%- if sessions %}
    {%- for session in sessions[:10] %}
  | {{ session.start_time[:16] }} | {{ session.location }} | {{ session.kwh_charged }}kWh | {{ session.charge_cost_euros }}€ | {{ session.kwh_price }}€/kWh |
    {%- endfor %}
  {%- endif %}
```

## Troubleshooting

### Common Issues
1. **Entities not created**: Check `configuration.yaml` includes and restart HA
2. **No trip data**: Verify `binary_sensor.idbuzz_vehicle_moving` is working
3. **No charging data**: Verify `switch.idbuzz_charging` state changes
4. **File errors**: Check `/config/` directory permissions

### Log Files
Monitor these for errors:
- Home Assistant logs: **Settings** → **System** → **Logs**
- Check automation traces in **Settings** → **Automations**

## Customization

### Modify Battery Capacity
Change `0.77` in calculations to your vehicle's actual kWh capacity.

### Adjust Pricing
Edit charging costs in `charging_tracker.yaml`:
```yaml
kwh_price: >
  {% if 'your_zone' in location %}
    0.25
  {% elif 'another_zone' in location %}
    0.30
  {% else %}
    0.45
  {% endif %}
```

### Change Data Retention
Modify `sessions[:50]` and `trips[:50]` to keep more/fewer records.

## Support

For issues or questions:
1. Check Home Assistant logs
2. Verify all required sensors exist
3. Ensure Volkswagen integration is working
4. Test individual automations manually

## License

This project is provided as-is for personal use with Home Assistant and Volkswagen vehicle integrations.