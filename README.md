Making my Home Assistant Automations available for others to use and take inspiration from.

My hardware:
- Feyree EV Charger integrated using Tuya Local
- Fronius Smart Meter installed at grid connection

feyree-charger-target-matching.yaml is used to slowly ramp down amperage. If you suddenly drop from 32A to 16A the Feyree EV Charger will detect an overcurrent draw and go into fault protection mode. The only way to recover it is a power cycle. So the feyree-charger-target-matching.yaml automation will drop current by 1A every 5 seconds until it reaches the target amperage. Slowly ramping is not necessary when increasing amperage so the automation will just set the feyree charger to the greater target amperage when needed.

I use a template helper to calculate available power. This is my grid meters net power reading plus the charger's current power draw. I'm using amps X voltage because the power sensor precision on the Feyree charger is only 1 decimal and multiplying it by 1000 is not very accurate (eg. 2.3kW = 2300W). With my hardware it looks like this:
{{ states('sensor.smart_meter_63a_real_power')|int(0)*-1 + ((states('sensor.feyree_ev_charger_current')|float(0))*(states('sensor.feyree_ev_charger_voltage')|float(0)))}}

I then use a statistic helper to calculate the average (linear) of available power for the last 5 minutes. This prevents the charger from turning off/on when a cloud passes over and power dips for a short while.

I use a buffer variable because the car's onboard charger has fixed current steps (eg. 8A, 10A, 16A, 20A, etc). Even if the automation sets the charger to 19A, the car cannot draw the full available current.

This is what charging looks like on a cloudy day. There are some gaps in charging when the car was being used.
<img width="1416" height="490" alt="image" src="https://github.com/user-attachments/assets/d7f9912a-99ac-4f13-864f-ee6e5c4f04c9" />

