# Nabaztag in Python for Raspberry Pi
[ [:fr:](README.md) | [:gb:](README_en.md) | [:us:](README_en.md) ] 

[![Build Status](https://travis-ci.org/nabaztag2018/pynab.svg?branch=master)](https://travis-ci.org/nabaztag2018/pynab)
[![Style code: black](https://img.shields.io/badge/code%20style-black-000000.svg)](https://github.com/psf/black)
[![Total alerts](https://img.shields.io/lgtm/alerts/g/nabaztag2018/pynab.svg?logo=lgtm&logoWidth=18)](https://lgtm.com/projects/g/nabaztag2018/pynab/alerts/)
[![Language grade: Python](https://img.shields.io/lgtm/grade/python/g/nabaztag2018/pynab.svg?logo=lgtm&logoWidth=18)](https://lgtm.com/projects/g/nabaztag2018/pynab/context:python)
[![codecov](https://codecov.io/gh/nabaztag2018/pynab/branch/master/graph/badge.svg)](https://codecov.io/gh/nabaztag2018/pynab)

# Cards

This system is designed for two cards:
- A card made for Maker Faire 2018, which only works with Nabaztag v1 (without microphone or RFID).
- A new version of the card, offered via the Ulule campaign in May 2019, which works with the Nabaztag v1 and v2 (the microphones are on the card, suddenly the Nabaztag v1 also benefit from voice recognition).

The schematics and manufacturing files for these two cards are in the repository [hardware](https://github.com/nabaztag2018/hardware), respectively [`RPI_Nabaztag`](https://github.com/nabaztag2018/hardware/ blob / master / RPI_Nabaztag.PDF) (2018) and [`tagtagtag_V2.0`](https://github.com/nabaztag2018/hardware/tree/master/tagtagtag_V2.0) (2019).

# Images

The [releases](https://github.com/nabaztag2018/pynab/releases) are images of Raspbian Buster Lite 2019-09-26 with pynab pre-installed. They have the same settings as [Raspbian](https://www.raspberrypi.org/downloads/raspbian/).

Current releases (0.7.x) only work on 2019 cards (see # 44)

# Installation on Raspbian (for developers!)

0. Make sure the system is up to date

The installation script now requires a Raspbian with buster, to benefit from Python 3.7.
The headers from the apt package must match the kernel version.

```
sudo apt update
sudo apt upgrade
```

1. Configure the sound card, ears and RFID reader and restart.

Maker Faire 2018:
https://support.hifiberry.com/hc/en-us/articles/205377651-Configuring-Linux-4-x-or-higher

Ulule 2019:
https://github.com/pguyot/wm8960/tree/tagtagtag-sound

The two cards:
https://github.com/pguyot/tagtagtag-ears

Nabaztag: tag only (not required on Nabaztag, but installed by updates)
https://github.com/pguyot/cr14

2. Install PostgreSQL and the required packages

```
sudo apt-get install postgresql libpq-dev git python3 python3-venv python3-dev gettext nginx openssl libssl-dev libffi-dev libmpg123-dev libasound2-dev libatlas-base-dev libgfortran3 libopenblas-dev liblapack-dev gfortran
```

3. Retrieve the code

```
git clone https://github.com/nabaztag2018/pynab.git
cd pynab
```

4. Run the installation script that does the rest, including installing and starting the services via systemd.

```
bash install.sh
```

or, for Maker Faire 2018 cards:

```
bash install.sh --makerfaire2018
```

# Update

A priori, it works via the web interface.
If necessary, it can be done on the command line with:
```
cd pynab
bash upgrade.sh
```

# Nabblockly

[Nabblockly](https://github.com/pguyot/nabblockly), a block programming interface for rabbit choreographies, is installed on the release images since 0.6.3b and works on port 8080. Installation is possible on port 80 by modifying the configuration of Nginx.

# Architecture

See the document [PROTOCOL.md](PROTOCOL_en.md)

- nabd: daemon that manages the rabbit (i/o, choreographies)
- nab8balld: daemon for the guru service
- nabairqualityd: daemon for air quality service
- nabclockd: daemon for the clock service
- nabsurprised: daemon for the surprise service
- nabtaichid: daemon for taichi service
- nabmastodond: daemon for the mastodon service
- nabweatherd: daemon for the weather service
- nabweb: web interface for configuration
