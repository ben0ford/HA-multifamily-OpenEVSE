# HA-multifamily-OpenEVSE
Home Assistant (HA) scripts, automations, helpers, etc. to enable management of multiple OpenEVSE chargers: user management, cost logging, etc. 

We have 25 OpenEVSE chargers in a shared parking lot for a community of 30 homes. This Home Assistant implementation allows all users to use any charger, with several charging options. User charging session costs are computed live, based on automatically-updated electricity rates; sessions and costs are logged for billing purposes.

## Functionality

### User interface
Users control chargers through the HA mobile app. The default (Overview) dashboard has a button for each charger. The button's color and icon give a little info about the charger: Green = available; Blue = charge initiated; zzz = charge complete but car still plugged in.
![Screenshot_20250421-213927](https://github.com/user-attachments/assets/6cdb4aa1-43b5-41cf-9fc4-e49926401187)

Tapping one of those buttons takes you to the charger's dashboard:
![Screenshot_20250421-215655](https://github.com/user-attachments/assets/0c17b198-1dd9-4ac5-800a-3d7a642fde3c)

Three buttons on the charger dashboard (left side) start a charge. The 3 types: 
* "normal," avoiding expensive 4:00–9:00 p.m. charging,
* "eco," which starts a normal charge if the current time is between 9:00 a.m and 2:00 p.m.; otherwise waits until 9:00 a.m. to start charging and then initiates a normal charge
* "override," which starts a charge immediately and ignores our usual 4:00–9:00 p.m. lockout.

The right side of the charger dashboard shows various status info for the charger: charge type, user, charging time, plugged-in status, current, energy dispensed this session, present electricity cost, and session cost.

When a user plugs in then hits one of those buttons, the charger button changes to blue with a charging icon on the Overview dashboard, the charge type and user name update in the status column on the right, and the start_charge script is initated, which uses the type of charge selected to set up charging.

### Admin interface
Administrators have a similar Overview dashboard with the addition of "Admin Tools," 
![Screenshot_20250421-214030](https://github.com/user-attachments/assets/fb4e73cb-6a7a-4eda-bbcb-3645a6c79b76)![Screenshot_20250421-214040](https://github.com/user-attachments/assets/483e77e1-8e90-4480-94a4-6b8a58b7d7e3)

The Admin Tools charger buttons lead to an Admin dashboard which gives access to most charger controls and sensors, including a rudimentary "Disable charger" button that makes the charger's button on the (user) Overview dashboard inoperable.
![Screenshot_20250421-214058](https://github.com/user-attachments/assets/aa5bdec9-a722-4674-a018-e77d1527f777)![Screenshot_20250421-214127](https://github.com/user-attachments/assets/f8d44614-f756-4e6a-9030-d2ea9d448376)






## Integrations required:
* [MIDAS](https://github.com/MattDahEpic/ha-midas) (California-specific, for automatically updating utility rates)
* [OpenEVSE](https://github.com/firstof9/openevse/) Component to integrate with OpenEVSE chargers. There are two integrations with this name; be sure you get the one by firstof9.
* HACS
* [File](https://www.home-assistant.io/integrations/file/) for logging charging sessions (file set to /config/ev_logs/charge_sessions)
  * We are considering switching to [Google Sheets](https://www.home-assistant.io/integrations/google_sheets/) integration to save directly to an (off-site) spreadsheet
* [Time and Date](https://www.home-assistant.io/integrations/time_date/)
* [Browser Mod](https://github.com/thomasloven/hass-browser_mod)

We also use the integrations below, though they are not specific to the OpenEVSE project
* [Mobile App](https://www.home-assistant.io/integrations/mobile_app/) (auto-enabled)
* [Backup](https://www.home-assistant.io/integrations/backup/)
* [Google Drive](https://www.home-assistant.io/integrations/google_drive/) (for storing backups)
