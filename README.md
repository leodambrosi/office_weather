# why?

Measuring Co2 and Temperature in closed environments (office, home).

People are sensitive to high levels of Co2 in closed envorinments, so we want to have some numbers to monitor it.

# requirements

## hardware

1) [TFA-Dostmann AirControl Mini CO2 Messgerät](http://www.amazon.de/dp/B00TH3OW4Q) -- 80 euro

2) [Raspberry PI 3]() -- 40 euro

3) case, 5v power supply, microSD card

## software

2) Install on your Raspberry PI 3 [InfluxDB & Grafana](https://www.circuits.dk/install-grafana-influxdb-raspberry/).

1) download [Raspbian](https://www.raspberrypi.org/downloads/) and [install it on the microSD](https://www.raspberrypi.org/documentation/installation/installing-images/README.md). We used [this version](https://github.com/wooga/office_weather/blob/0da94b4255494ecbcf993ec592988503c6c72629/.gitignore#L2) of raspbian.

# installation on the raspberry

0) Boot the raspberry with the raspbian. You'll need a USB-keyboard, monitor and ethernet for this initial boot. After overcoming the initial configuration screen, you can login into the box using ssh.

1) install python libs
```
sudo apt-get install python-pip python-dev libyaml-dev
sudo pip install pyyaml
sudo pip install requests
```

2) create `config.yaml`:
```
prefix: office.floor3
```

2b) (optional) You can configure this bot to automatically post to a Slack channel.
Just add an "Incoming Webhook" to your Slack team's integrations and add a `slack` hash to the config file.

```yaml
slack:
    webhook: 'https://hooks.slack.com/services/TXXXXXX/XXXXXX/xxxxxxx'
    channel: '#general'   # optional - this is the default value
    botname: 'CO2bot'     # optional - this is the default value
    icon: ':monkey_face:' # optional - this is the default value
    upper_threshold: 800  # optional - this is the default value
    lower_threshold: 600  # optional - this is the default value
```

3) fix socket permissions
```
sudo chmod a+rw /dev/hidraw0
```

4) run the script
```
./monitor.py /dev/hidraw0
```

5) run on startup

To get everything working on startup, need to add 2 crontabs, one for root
and the other for the pi user:

Roots:

```
SHELL=/bin/bash
* * * * * if [ $(find /dev/hidraw0 -perm a=rw | wc -l) -eq 0 ] ; then chmod a+rw /dev/hidraw0 ; fi
```

Pi:

```
SHELL=/bin/bash
* * * * * /usr/bin/python /home/pi/monitor.py /dev/hidraw0 [ **optional:** /home/pi/my_config.yaml ]  > /dev/null 2>&1
```

The script will default to using "config.yaml" (residing in the same directory as the
monitor.py script - /home/pi in the example) for the librato credentials.
You can optionally override this by passing a custom configuration file path as a second parameter.

# ansible deployment on the raspberry

1) In the project folder create the `config.yaml` file as described in the chapter for the manual installation process. Replace the line containing the `prefix` assignment by:

`prefix: {{ name }}`


# credits

based on code by [henryk ploetz](https://hackaday.io/project/5301-reverse-engineering-a-low-cost-usb-co-monitor/log/17909-all-your-base-are-belong-to-us)
based on code by [Wooga](https://github.com/wooga/office_weather)

# license

[MIT](http://opensource.org/licenses/MIT)
