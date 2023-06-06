lvconnect
=========

<!-- spell-checker:ignore (jargon) sitename (names) AzureWebSites Dynos HerokuApp LibreView LVAPI LVConnect Nightscout -->

[![Deploy to Heroku](https://www.herokucdn.com/deploy/button.svg)](https://heroku.com/deploy?template=https://github.com/skobkars/lvconnect)

lvconnect copies your CGM data from LibreView web services to a [Nightscout](https://github.com/Nightscout/cgm-remote-monitor) website. The tool runs as `node.js index.js`, or as a Nightscout plugin, and will attempt to connect to your LibreView account, fetch data and upload to your Nightscout website via its REST API.

### Setup

* Connect to [LibreView](https://www.libreview.com) with a web browser and login to your account. (Is saving the device token required?)
* Inspect local storage and retrieve the value of the first key within the 'trustedDeviceTokens' object. This will be the `LVCONNECT_TRUSTED_DEVICE_TOKEN` value. This *TOKEN* value should be viable for 14 days.
* Use `env API_SECRET="123456789012" LVCONNECT_USER_NAME="Roy.Ivy.III@gmail.com" LVCONNECT_PASSWORD="*PASSWORD*" LVCONNECT_TRUSTED_DEVICE_TOKEN="*TOKEN*" node index.js`.
* Use the "login" argument to verify login...
  ```text
  lvconnect: attempt to login # 1
  logging successful: b1087484-2df9-11ed-b7a7-0242ac110008
  patient is the user
  saving session...
  ```
* Use the "fetch" argument to verify data fetch (note: will only fetch *new* data since last login/fetch combination)...
  ```text
  reading stored session...
  lvconnect: attempt to login and fetch data # 1
  current token is valid until 2023-06-04T20:58:40.797Z
  saving data...
   ...
  saving session...
  ```

### Prerequisites

* A working LibreView account, either Personal or Pro. NOTE: In case of a Pro account any connected patient data can be fetched, so please make sure that the correct Nightscout URL and API_SECRET are set, or otherwise data will be uploaded and merge to the wrong Nightscout.
* A working Nightscout website

### Environment

`VARIABLE` (default) - description

#### Required

* `API_SECRET` - A secret passphrase that must be at least 12 characters long, and must match the `API_SECRET` from your Nightscout website
* `WEBSITE_HOSTNAME` - The hostname for your Nightscout website.  Example: sitename.herokuapp.com or sitename.azurewebsites.net. Do not include http:// or https://
* `LVCONNECT_SERVER` (api.libreview.io)- LibreView api website. ['EU', 'US', or any actual hostname]. Blank value can be used to get redirect LibreView servers in the US. Set to (EU) to fetch from servers outside of US from the beginning.
* `LVCONNECT_USER_NAME` - Your personal account email for the Pro LibreView service. Overrides LVCONNECT_PRO_USER_NAME if set. Can be left empty if Pro account is used (see below).
* `LVCONNECT_PASSWORD` - Your personal password for the Pro LibreView service. Overrides LVCONNECT_PRO_PASSWORD if set. Can be left empty if Pro account is used (see below).
* `LVCONNECT_TRUSTED_DEVICE_TOKEN` - Trusted device token for 2FA verification

#### Optional

* `LVCONNECT_PRO_USER_NAME` - Account email for the Pro LibreView service.
* `LVCONNECT_PRO_PASSWORD` - Password for the Pro LibreView service.
* `LVCONNECT_PRO_TRUSTED_DEVICE_TOKEN` - Pro account's trusted device token for 2FA verification
* `LVCONNECT_PATIENT_ID` - LibreView Pro connected patient UUID. Used with Pro accounts to access one of the connected patients data. Ignored for personal LibreView accounts.
* `LVCONNECT_INTERVAL` (3600000) - The time (in milliseconds) to wait between each update. Default is 1 hour
* `LVCONNECT_MAX_FAILURES` (3) -The maximum number of attempts to connect to the LibreView server.
* `LVCONNECT_FIRST_FULL_DAYS` (90) - The number of days to search for data on the first update only.
* `NS` - A fully-qualified Nightscout URL (e.g. `https://sitename.herokuapp.com`) which overrides `WEBSITE_HOSTNAME`
* `LVCONNECT_TIME_OFFSET_MINUTES` - Time difference in MINUTES between UTC and your location. LibreView treats your local time as UTC and doesn't report the actual time differences, so data will be shown with wrong timestamps if this parameter is not set.

### More information

Before sending over and sensitive user information lvconnect checks if the `LVCONNECT_SERVER` appears to be a valid Lvapi server, then logs in and fetches historical glucose data through the Daily Log report. This information is then uploaded to the user's specified `WEBSITE_HOSTNAME` or `NS`.

If called without any parameters as a stand-alone tool, or as a plugin lvconnect will fetch data accumulated since previous call every `LVCONNECT_INTERVAL` ms, except for the the first run when it will request all data in the last `LVCONNECT_FIRST_FULL_DAYS`. Only essential step information is printed out.

The following three command line parameters are used for development and debugging:

* `login` - forces starting of a new login session and fetches new authentication token.
* `fetch` - prevents fetched data from being uploaded to Nightscout, and saves it to `fetched.json` instead.
* `run` - runs one full authorize/fetch/upload cycle.

In development mode lvconnect only fetches data once per call, and its current session is saved to a `session.json` file and is reused for next calls. It also re-uses authentication tokens until they expire, or obtains new ones as required.

Deletion of the `session.json` file will enforce new session start.

### How to deploy to Heroku

Click 'Deploy to Heroku' button above and configure all of the variables as per descriptions, then click 'Deploy app'.

After the app is built and started click 'Manage app' button or go to Overview tab, then click 'Configure Dynos' link. Disable 'web' and enable 'worker' processes, confirm changes. Lvconnect does not have web interface when used as a standalone tool, and having 'web' process will cause app crashes.

**Make sure that you correctly set `LVCONNECT_TIME_OFFSET_MINUTES` variable, as LibreView is not using timezones internally and therefore there is no way for lvconnect to know what your local time is. `LVCONNECT_TIME_OFFSET_MINUTES` is set in minutes to accommodate 1/2 hour timezones, not in hours!**

### Disclaimer

The code is licensed under [GNU General Public License](https://www.gnu.org/licenses/#GPL).

This project is not approved by any national Health Authority, not recommended for therapy, and not
related to, or approved by [LibreView / Abbott](https://www.libreview.com/regulatoryInformation).
