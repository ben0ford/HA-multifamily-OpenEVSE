# HA-multifamily-OpenEVSE
Home Assistant scripts, automations, helpers, etc. to enable management of multiple OpenEVSE chargers: user management, cost logging, etc. 

We have 25 OpenEVSE chargers in a shared parking lot for a community of 30 homes. This Home Assistant implementation allows all users to use any charger, with several charging options. User charging session costs are computed live, based on automatically-updated electricity rates; sessions and costs are logged for billing purposes.

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
