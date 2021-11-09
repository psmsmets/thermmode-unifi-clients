# thermmode-uiclient
UniFi client montoring for geolocation-like functionality of the Netatmo Smart Thermostat.

UniFi clients of interest are monitored to automatically set the thermostat mode.
The thermostat is set to `mode=away` when all listed clients are disconnected longer
than the threshold `UI_CLIENT_OFFLINE_SECONDS` (defaults to 900s).
As soon as any of the listed clients reconnects the thermostat is set to `mode=schedule`.

Clients of interest are listed by their mac address (formatted 12:34:56:78:90:ab) and
corresponding connection details are retrieved from the UniFi's controller.

When the thermostat mode is set to frost guard (`mode=hg`) client monitoring is disabled.


## Preparation
Configure the UniFi controller and DirectAdmin accordingly. 
All related variables should either be defined as shell (environment) variables or provided in a configuration file.

### UniFi controller
Create a local user for the UniFi controller with read-only access.

1. Add user
1. Set role to `Limited Admin`
1. Set account type to `Local Access Only`
1. Complete the first name, last name, local username and local password fields. All other fields can be left blank.
1. In application permission, set the Unifi Network  to `View Only`. All other applications can either be `View Only` or `None`.

Add the UniFi Controller variables to your shell or the configuration file.
```
# UniFi controller configuration
UI_ADDRESS = https://url_or_ip_of_your_controller
UI_USERNAME = '...'
UI_PASSWORD = '...'
UI_SITENAME = default  # default value and optional
UI_CLIENTS = aa:aa:aa:aa:aa:aa bb:bb:bb:bb:bb:bb cc:cc:cc:cc:cc:cc # List mac addresses
UI_CLIENT_OFFLINE_SECONDS = 900 # default value and optional
```

### Netatmo Connect API

Add the Netatmo Connect variables to your shell or the configuration file.
```
# Netatmo connect configuration
NC_USER_ID    = ..
NC_HOME_ID    = ..
NC_USER_TOKEN = ..
```

## Usage
```
Usage:  thermmode-uiclient.sh <config_file>

Required config/environment variables:
  UI_ADDRESS UI_USERNAME UI_PASSWORD UI_SITENAME UI_CLIENTS UI_CLIENT_OFFLINE_SECONDS
  NC_USER_ID NC_HOME_ID NC_USER_TOKEN
```

Execute the script with a configuration file
```
bash thermmode-uiclient.sh /path/to/your/home.conf
```

Missing variables from the configuration file are assumed to be set in your shell (locally or as enviroment variables).

## Automatic trigger via Crontab
Trigger the Netatmo thermostat mode update every minute using crontab (`sudo crontab -e`) and log the output in syslog.
```
*/1 * * * * thermmode-uiclient.sh my-home.conf 2>&1 | /usr/bin/logger -t netatmo 
```
Make sure that the correct path to both the script and configuration are set.

Scan the syslog output
```
cat /var/log/syslog | grep netatmo
```