# HomeAssistant_Mail

Card, Method and iOS Notification for getting UPS, USPS, and FedEx delivery information in Home Assistant

Note: This is not a lovelace UI card, it will be converted in a later version.

<img src="https://github.com/moralmunky/HomeAssistant_Mail_Card/blob/master/mail_card_screenshot.jpg" alt="Preview of the custom mail card" width="505" height="552" />

<img src="https://github.com/moralmunky/HomeAssistant_Mail_Card/blob/master/notification_screenshot.jpg" alt="Preview of the custom mail card" width="505" height="411" />

## Credits:

* Mail Card based on Bram_Kragten work at [Home Assistant Community: Custom UI weather state card, with a question](https://community.home-assistant.io/t/custom-ui-weather-state-card-with-a-question/23008)
* Mail.py script based on @skalavala work at [skalavala blog](https://blog.kalavala.net/usps/homeassistant/mqtt/2018/01/12/usps.html)

## Issues:

* In Transit currently shows the count of delivered items for USPS and FedEx. The Broken official [USPS](https://www.home-assistant.io/components/usps/) and [FedEx](https://www.home-assistant.io/components/sensor.fedex/) components used to provide in transit data. 
* USPS Informed Delivery GIF: A browser cache reset needs to be performed to see the correct updated image in the frontend.
Home Assistant Configuration

## Requirements:

paho-mqtt and imagemagick PIP packages are installed within the Home Assistant environment
```
sudo pip install paho-mqtt
sudo apt-get install imagemagick
```
The Home Assistant configuration.yaml must have [packages:](https://www.home-assistant.io/components/mqtt/) and [mqtt:](https://www.home-assistant.io/components/mqtt/) defined.

Example:
```
packages: !include_dir_named includes/packages
mqtt:
  password: !secret mqtt_pass
```

## Upload Files

Upload the files into inside the Home Assistant .homeassistant/ folder as structured in the repository
```
.homeassistant/www/mail_card/custom-mail-card.html
.homeassistant/www/mail_card/nomail.gif
.homeassistant/www/mail_card/todays_mail.gif
.homeassistant/includes/mail.py
.homeassistant/includes/packages/mail_package.yaml
```
## Setup Mail.py file

Enter the details for the following variables used in the file

MQTT Details
```
Line 18 MQTT_SERVER
Line 19 MQTT_SERVER_PORT
Line 22 MQTT_USERNAME
Line 23 MQTT_PASSWORD
```

Interval to check email
```
Line 31 SLEEP_TIME_IN_SECONDS
```

Email account details
```
Line 33 HOST
Line 34 PORT
Line 35 USERNAME
Line 36 PASSWORD
Line 37 FOLDER
```

Mail image path
```
Line 41 IMAGE_OUTPUT_PATH
```

## mail_package.yaml

Line 133 Change the path to the full image path defined in the mail.py line 41 IMAGE_OUTPUT_PATH.
Line 165, 166 #Sign up for UPS MyChoice and add your credentials
Line 292 Define the notify component setup in Home Assistant
Line 298 Change the URL to match Home Assistants and the location of the todays_mail.gif

## Running the above Python Program as a Service/Daemon

The mail.py needs to run in the background to check and publish information. Create a systemd daemon to allow the background run and restart on errors. These instruction may vary depending on system setup.

Create a file called homeassistant_mail_check.service in /etc/systemd/system folder:
```
cd /etc/systemd/system
sudo vim homeassistant_mail_check.service
```
Enter insert mode ( type i) then type the following:
```
[Unit]
Description=Home Assistant Mail Count Retriever
After=network.target
Requires=network.target

[Service]
User=homeassistant
ExecStart=/srv/homeassistant/bin/python3 /home/homeassistant/.homeassistant/includes/mail.py
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

Exit insert mode (hit the esc keyboard button) then save it (type :wq!). Run the following commands to load it:
```
sudo systemctl --system daemon-reload
sudo systemctl enable homeassistant_mail_check.service
sudo systemctl start homeassistant_mail_check.service
```

Service Commands
```
Check status
sudo systemctl status homeassistant_mail_check.service

Stop
sudo systemctl stop homeassistant_mail_check.service

Restart
sudo systemctl restart homeassistant_mail_check.service
```
