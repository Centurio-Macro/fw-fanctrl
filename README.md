# fw-fanctrl

This is a very simple Python service for Linux that drives the Framework Laptop's fan speed according to a configurable speed/temp curve.
Its default configuration targets very silent fan operation, but it's easy to configure it for a different comfort/performance trade-off.
Its possible to specfify two separate fan curves depending on whether the Laptop is charging/discharging.
Under the hood, it uses [fw-ectool](https://github.com/DHowett/fw-ectool) to change parameters in FrameWork's embedded controller (EC).
This is a fork of the [original version](https://github.com/TamtamHero/fw-fanctrl.git) of fw-fanctrl, which provides the same functionality without the automatic switching.

# Install

## Dependancies
This tool depends on `lm-sensors` to fetch CPU temperature:
```
sudo apt install lm-sensors
yes | sudo sensors-detect
```

To communicate with the embedded controller the `fw-ectool` is needed. You can either use the pre-compiled executable of `fw-ectool` in this repo, or recompile one from [this repo](https://github.com/DHowett/fw-ectool) and copy it in `./bin`.

The charging status of the battery is fetched from the following file:
`/sys/class/hwmon/hwmon2/device/status`
Make sure the charging status is stored under the same path on your machine. Otherwise edit the `fw-fanctrl.py` script.

Then, simply run:
```
sudo ./install.sh
```

This bash script is going to create and enable a service that runs this repo's main script, `fanctrl.py`.
It will copy `fanctrl.py` and `./bin/ectool` to `/usr/local/bin` and create a config file in `/home/<user>/.config/fw-fanctrl/config.json`

# Uninstall
```
sudo ./install.sh remove
```

# Configuration

There is a single `config.json` file where you can configure the service. You need to run the install script again after editing this config, or you can directly edit the installed config at `/home/<user>/.config/fw-fanctrl/config.json` and restart the service with:

```
sudo service fw-fanctrl restart
```

It contains different strategies, ranked from the most silent to the noisiest. You can add new strategies, and if you think you have one that deserves to be shared, feel free to make a PR to this repo :)

The strategy active by default is the one specified in the `defaultStrategy` entry. Additionally a separate strategy can be defined, which is only active during discharge of the battery. This one is optional and specified `strategyOnDischarging` entry. If none is specified only the `defaultStrategy` is used.

Strategies can be configured with the following parameters:

- **SpeedCurve**:

    This is the curve points for `f(temperature) = fan speed`

    `fw-fanctrl` measures the CPU temperature, compute a moving average of it, and then find an appropriate `fan speed` value by interpolation on the curve.

- **FanSpeedUpdateFrequency**:

    Time interval between every update to the fan's speed. `fw-fanctrl` measures temperature every second and add it to its moving average, but the actual update to fan speed is made every 5s by default. This is for comfort, otherwise the speed is changed too often and it is noticeable and annoying, especially at low speed.
    For a more reactive fan, you can lower this setting.

- **MovingAverageInterval**:

    Number of seconds on which the moving average of temperature is computed. Increase it, and the fan speed will change more gradually. Lower it, and it will gain in reactivity. Defaults to 30 seconds.
