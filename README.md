# HA-multifamily-OpenEVSE
Home Assistant (HA) scripts, automations, helpers, etc. to enable management of multiple OpenEVSE chargers: user management, cost logging, etc. 

We have 25 OpenEVSE chargers in a shared parking lot for a community of 30 homes. This Home Assistant implementation allows all users to use any charger, with several charging options. User charging session costs are computed live (based on automatically-updated electricity rates); sessions and costs are logged for billing purposes. Many decisions are particular to our application (e.g. letting all users see who is charging where). Also, neither of us who wrote this is an accomplished programmer, so apologies if things are clunky.

This is not a plug-and-play HA "integration." You'll need to have someone who is willing to dive into the yaml files and their Jinja2 and javascript templates. If anyone wants to create a more robust and transportable system, we're happy to help if we can.

Our chargers are actually former JuiceBox chargers with their control boards replaced with OpenEVSE drop-in replacements ([version 1--metal box](https://store.openevse.com/collections/all-products/products/replacement-electronics-for-juicebox-v1-metal-black-and-orange); [version 2--plastic box](https://store.openevse.com/collections/all-products/products/replacement-electronics-for-juicebox-v2-plastic-grey-and-white)). Also get a [temperature sensor](https://store.openevse.com/collections/all-products/products/temperature-sensor) since the replacement board doesn't have one onboard. The brain-transplant chargers behave exactly like a standard OpenEVSE build, as far as I can tell.

Thirty-three users have been using the system for a few weeks now, after a month of more limited testing, and it's SO much more stable and functional than Juicenet ever was. Plus our data for billing is much better. 

The box you hang on the wall is called an EVSE: Electric Vehicle Supply Equipment. In this document we use the less-accurate but more common term "charger."

## Functionality

### User interface
Users control chargers through the HA mobile app. The default (Overview) dashboard has a button for each charger. The button's color and icon give a little info about the charger: Green = available; Blue = charge initiated; zzz = charge complete but car still plugged in.

<img src="https://github.com/user-attachments/assets/6cdb4aa1-43b5-41cf-9fc4-e49926401187" alt="Screenshot_20250421-213927" width="250">

Tapping one of those buttons takes you to the charger's dashboard:

<img src="https://github.com/user-attachments/assets/0db4b4ec-df25-448c-95ed-a3ba31bc1ac8" alt="Screenshot_20250421-213944" width="250">
<img src="https://github.com/user-attachments/assets/0c17b198-1dd9-4ac5-800a-3d7a642fde3c" alt="Screenshot_20250421-215655" width="250">

Three buttons on the charger dashboard (left side) start a charge. The 3 types: 
* "normal," avoiding expensive 4:00–9:00 p.m. charging,
* "eco," which starts a normal charge if the current time is between 9:00 a.m and 2:00 p.m.; otherwise waits until 9:00 a.m. to start charging and then initiates a normal charge (here in California, we have so much solar on the grid that the lowest-carbon electricity is mid-day)
* "override," which starts a charge immediately and ignores our usual 4:00–9:00 p.m. lockout (this button has a confirmation pop up for the user to confirm).

Each charge type runs a check for a connected vehicle, and waits for about 45 seconds before prompting the user "no car? Try again" and setting the charger back to Available.

The Stop Charge button also has a pop-up confirmation. We don't restrict access to stop a charge to the user who started it (or to someone else with the same account), but it wouldn't be hard to do.

The right side of the charger dashboard shows various status info for the charger: charge type, user, charging time, plugged-in status, current, energy dispensed this session, present electricity cost, and session cost.

When a user plugs in then taps one of those buttons, the charger button on the Overview dashboard changes to blue with a charging icon, the charge type and user name update in the status column on the right, and the start_charge script is initated, which uses the type of charge selected to set up charging.

When the user unplugs, the session cost (and other info, including user name, user account, time stamp) is logged, and the session cost resets to $0.00. We haven't yet built a way for users to see their charging history, but it's on the to-do list.

### Admin interface
Administrators have a similar Overview dashboard with the addition of "Admin Tools," 

<img src="https://github.com/user-attachments/assets/fb4e73cb-6a7a-4eda-bbcb-3645a6c79b76" alt="Screenshot_20250421-214030" width="250">
<img src="https://github.com/user-attachments/assets/483e77e1-8e90-4480-94a4-6b8a58b7d7e3" alt="Screenshot_20250421-214040" width="250">

The Admin Tools charger button for a charger leads to an Admin dashboard which gives access to most charger controls and sensors, including a rudimentary "Disable charger" button that makes the charger's button on the (user) Overview dashboard inoperable.


<img src="https://github.com/user-attachments/assets/aa5bdec9-a722-4674-a018-e77d1527f777" alt="Screenshot_20250421-214058" width="250">
<img src="https://github.com/user-attachments/assets/f8d44614-f756-4e6a-9030-d2ea9d448376" alt="Screenshot_20250421-214127" width="250">

## General HA setup

For this to be useful, your users' phones will all have to be able to connect to the HA server. We have a site-wide local network so we have not opened up any remote access. Our users just know that they have to connect to wifi in the parking lot or at home to connect and interact with the chargers. It is possible to set up your HA to allow remote access via the [Home Assistant Cloud](https://www.nabucasa.com/). However, your HA server and chargers do need to be on the same local network.

### Integrations required:
* [MIDAS](https://github.com/MattDahEpic/ha-midas) (California-specific, for automatically updating utility rates)
* [OpenEVSE](https://github.com/firstof9/openevse/) Component to integrate with OpenEVSE chargers. There are two integrations with this name; be sure you get the one by firstof9.
* [HACS](https://www.hacs.xyz/)
* [File](https://www.home-assistant.io/integrations/file/) for logging charging sessions (file set to /config/ev_logs/charge_sessions)
  * We are considering switching to [Google Sheets](https://www.home-assistant.io/integrations/google_sheets/) integration to save directly to an (off-site) spreadsheet
* [Time and Date](https://www.home-assistant.io/integrations/time_date/)
* [Browser Mod](https://github.com/thomasloven/hass-browser_mod)
* [Integral](https://www.home-assistant.io/integrations/integration/)
* [Custom Button-card](https://github.com/custom-cards/button-card?tab=readme-ov-file#installation)
* (not sure this got used in final version) [Config-template-card](https://github.com/iantrich/config-template-card)
* [Decluttering card](https://github.com/custom-cards/decluttering-card)
* [Vertical Stack in Card](https://github.com/ofekashery/vertical-stack-in-card)

We also use the integrations below, though they are not specific to the OpenEVSE project
* [Mobile App](https://www.home-assistant.io/integrations/mobile_app/) (auto-enabled)
* [Backup](https://www.home-assistant.io/integrations/backup/)
* [Google Drive](https://www.home-assistant.io/integrations/google_drive/) (for storing backups)

### Dashboards
All dashboards are in yaml mode so that we can use !include statements. We are running a Home Assistant Green. On our machine, /homeassistant is the root directory for all HA files, but it's referenced as /config when !include'ing files.

Charger dashboards are all identical files, with the charger number determined simply from the URL for the dashboard. So, for instance, the dashboard ```/homeassistant/dashboards/08/fs-08.yaml```, in total, is below. This allows us to change all charger dashboards at once by just changing the !include'd file. Same idea for the admin charger dashboards. We make ample use of HA's custom:button-card (to get javascript templating) and decluttering templates.

Our scripts and automations hard-code in the prefix "FS-" so you'll have to modify for however you name your chargers.

```
# The only value that needs to be changed from dashboard to dashboard is the
# charger number, which we get via Javascript and the dashboard URL

decluttering_templates: !include /config/decluttering_templates/charger_dashboard_decluttering_templates.yaml

views: !include /config/dashboards/user_charger_master.yaml
```

### Sequence for a charger
Sample sequence of events when a user "ben" wants to charge at charger FS-09:
1. User plugs in car 
2. Presses "Normal Charge" on the FS-09 user dashboard
3. ```script.start_charge``` initiated with data ```user_name: ben```, ```charge_type: normal```, ```charger_id: 09```
4. text helper ```input_text.09_user_name``` set to "ben"; text helper ```input_text.09_charge_type``` set to "normal"
5. ```script.save_charge_event``` runs to save event to log file (CSV) with data
```
   * ev_username: "{{ user_name }}"
   * chargerid: "{{ charger_id }}"
   * eventtype: "{{ charge_type }}"
   * totalusage: "{{ start_usage }}" # charger-reported total kWh delivered to date
   * totalcost: "{{ start_total_cost }}" # running total of $ value of charging delivered to date 
   * user_unit: "{{ user_unit }}" # unit number associated to the user; our version of "account"
```
6. ```script.start_charge``` turns off "override" on charger, so charger controls charging (charger has 16:00 disable/21:00 enable schedulers set). In the case of an "eco" charge, script just sets ```charge_type``` to "eco waiting" and then an automation at 9:00 turns the charger override off. In the case of "override," script turns charger override on, state "enabled," to ignore charger's schedulers.
7. ```script.start_charge``` exits

The charge continues under charger control until the car says "no more" or the charger's "Stop" button on its dashboard is pressed. When charger current drops to 0, the no_current automation runs script.save_charge_event to log an event, and sets the charge_type helper to "charge complete" (which has the effect of switching the icon on the Overview dashboard to "zzz").

Finally, when the user unplugs, the ```end_charge_unplug``` automation runs ```script.save_charge_event```, and this time includes the ```session_charge```. It then disables the charger, resets ```session_charge``` to $0.00, clears ```user_name```, and sets ```charge_type``` to "available."

### Automations
* ```eco_9am_start```: Runs at 9:00 a.m., checks all charge_type helpers, and any with the value "eco waiting" run ```script.start_charge``` to initate a charge
* ```end_charge_no_current```: If the ```charging_status``` sensor for a charger changes from "charging" to anything else, check that it's not because of the 4pm-9pm charging pause, and then save charge event and set ```charge_type``` to "charge complete"
* ```end_charge_unplug```: When a charger's ```vehicle_connected``` sensor changes to "unplugged," this disables the charger, logs the event (including ```session_cost```), resets ```session_cost``` to $0, clears ```user_name```, and sets ```charge_type``` to "available"
* ```plug_in_save_event```: Logs charger plug-ins. No user available, so just saves ```charger_id``` and total $ and kWh on that charger to date
* ```reload_charger```: We have one charger that is having trouble staying connected to HA, so this reloads a charger if it becomes unavailable
* ```update_off_peak_price/super/peak```: These update the electricity tariffs once a day. California-specific tool.

### Scripts
* ```end_charge```: called by other automations and scripts. Receives ```user_name```, ```charger_id```, ```event_type``` from calling entity, runs ```script.save_charge_event```, disables charger, sets ```charge_type``` to "available", clears ```user_name```
* ```save_charge_event```: uses File integration to save a line to a comma-separated values file. Fields saved include ```user_unit```, ```time```, ```user_name```, ```charger_id```, ```event_type```, ```total_usage```, ```total_cost```, and ```session_cost``` (```session_cost``` only saved for unplug events)
* ```start_charge```: receives ```user_name```, ```charger_id```, ```charge_type``` from button press. Sets ```charge_type``` helper and ```user_name``` helper, saves start event, starts appropriate charge (see Sequence section above)

### Helpers
Helpers are in the directory config/charger_helpers. Each charger uses these helpers (using name for charger FS-01)
* ```input_number.fs_01_which_button``` : controls display of additional info on the charger dashboard
* ```input_number.01_last_saved_total_cost``` : saves the "total_cost" value at the end of the previous charging session
* ```input_text.01_user_name```
* ```input_text.01_charge_type```
* ```sensor.01_cost_per_hour```: calculates current $ rate of charging by multiplying ```sensor.openevse_fs_01_charging_current``` by ```sensor.electricity_rate``` by ```sensor.openevse_fs_01_charging_voltage``` (units: (amps) * ($/kWh) * (volts)/1000 = ($ * watts)/(kW * 1000 * hr) = $/hr) 
* ```sensor.01_total_cost```: Uses the integral integration to integrate (in the calculus sense) the ```cost_per_hour sensor```. This gives accumulation of cost.
* ```number.01_session_cost_2```: Just the difference between the charger's ```total_cost``` and ```last_saved_total_cost``` to give the current session cost to display on the charger page (and record when charging is done)
* I previously tried a [utility meter](https://www.home-assistant.io/integrations/utility_meter/) for session cost but got weird results, so switched to the simple model above.

#### Calculating costs
A little more detail: The OpenEVSE integration provides the ```sensor.openevse_fs_01_charging_current``` sensor. Multiplying this by the present electricity rate and adjusting units correctly gives a cost per hour sensor. The integral (in the calculus sense) over time of "cost per hour" is "cost" for the period of time that was integrated.

### UI-controlled entities
We still have a few entities that are built through the UI and we haven't bothered to move them to yaml.
* ```input_number.bev_1_off_peak_price``` stores the off-peak electricity price (set by automation, above)
* ```input_number.bev_1_peak_price``` stores the peak price
* ```input_number.bev_1_super_off_peak_price``` stores the super off-peak price
* ```input_text.charger_list``` list of our charger IDs. Still have to move some scripts/automations to use this list rather than hard-code in all chargers.
* ```binary_sensor.eco_charge_time```: time-based sensor; "On" during super-off-peak pricing, currently 9:00 a.m. to 2:00 p.m. for us. "Off" otherwise
* ```sensor.electricity_rate```: template sensor that returns the current electricity rate. Unit of measurement is $/kWh; template is
  ```
  {% set current_hour = strptime(states('sensor.time'), "%H:%M").hour %}
          {% if  16 <= current_hour < 21 %}
            {{ states('input_number.bev_1_peak_price') }}
          {% elif 9 <= current_hour < 14 %}
            {{ states('input_number.bev_1_super_off_peak_price') }}
          {% else%}
            {{ states('input_number.bev_1_off_peak_price') }}
          {% endif %}
  ```
* ```binary_sensor.peak_charge_time```: time-based sensor; "On" during peak time (4:00-9:00 p.m. for us); "Off" otherwise
  

## Installing a charger
Get everything working for one charger first.

An OpenEVSE charger runs its own web server for charger setup (and for charger control, but we don't use the web interface for control). Use OpenEVSE's setup instructions to get the charger connected to your wifi. We created a separate SSID just for chargers to use, so the password isn't out in the wild as it's a pain to change on 25 chargers.

In the OpenEVSE setup process, or in the web interface, set the following on the charger:
* Max Current (Settings --> EVSE). Ours are 50 amp circuits, so 40 amp max
* Host Name: Decide on your pattern and be consistent. We use FS-XX(a)—e.g. FS-01, FS-02, FS-02a. The pairs with an "a" variant share a circuit. The scripts and automations depend on being eable to extract the unique part of the name (e.g. 17, 17a) by stripping out the FS-
* Time zone
* Schedulers for when you want the charger to charge in case of a "normal" charging session
* User name and Password for http interface. Make them the same for all chargers. Important to set so that users can't connect directly to chargers and initiate charging without HA control (and record-keeping)

Assign the charger a fixed IP address in your DHCP server. The charger needs to be in the same local network as the HA server.

In HA, go to the integrations page and then the OpenEVSE page. At the bottom of the "Integration Entries" list, "Add entry." The alias here is important, as all entity names are derived from it. For our naming convention, enter (for example) ```openevse_fs_01``` here. You'll also enter the charger's IP address and user name and password that you assigned on the charger. Leave the other optional entries blank, scroll to the bottom, and click Submit. If successful, around 60 entities will be created. 

## Future plans
### Circuit sharing
Ten of our chargers are on shared circuits, in pairs. They are presently set to 20 amp max so as not to exceed the 40 amp max continuous load on our 50 amp circuits. We intend to insert a load balancing routine in the start_charge script, and again in the end_charge automations, that will set the max_current controls on the chargers to safe values--probably 20/20 when two charges are happening, 40/0 when just one.

### User charging history
With a python script we could return all lines associated with the user from the log file, formatted in a table. Because we log so many events we might be more selective than that.

If we switch to logging in a Google Sheet, we might just share their history there.
