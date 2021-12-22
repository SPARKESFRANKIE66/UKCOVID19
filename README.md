# UKCOVID19
An application interfacing the Python UK-COVID19 SDK from the United Kingdom Health Security Agency into a Discord bot and 20✕4 Character LCD Display with 3 signal LEDs.

---
## Getting Started
### Required packages
The following packages are required from PyPI:
* `uk-covid19`
* `gpiozero`
* `iso3166`
* `asyncio`
* `discord.py`
* `emoji-country-flag`

Python 3.7 or greater will be required for this script to work.

Additional packages will be required to drive the Character LCD over the I2C protocol, these are included in the latest section.

### Required hardware
The folowing hardware is required:
* Raspberry Pi or of many clones that support GPIO outputs.
* A 20✕4 Character LCD with an I2C Display Adapter.
* One each of a Red, Yellow, and Green LED.
* 3✕82Ω resistors.
* Wire with Dupont connectors.

### Configuring the hardware
For information on how to configure the Pi for the I2C protocol and the Character LCD, see [this link](https://tutorials-raspberrypi.com/control-a-raspberry-pi-hd44780-lcd-display-via-i2c/).

Each LED connects to a GPIO Pin with one of the resistors. 82Ω is used to drop the 3.3V signal voltage from the GPIO pins to a more manageable voltage for the LEDs that doesn't cook them the femtosecond the script powers them. The LEDs connect to the GPIO pins specified in the following table:
|LED|GPIO Pin|
|---|--------|
|Red|GPIO14|
|Yellow|GPIO15|
|Green|GPIO18|

You can get the standard GPIO pinout of a Raspberry Pi [here](https://pinout.xyz).

Note: With this pin configuration, the Red LED will light briefly when the Pi boots up. This is because this pin is also used as UART TX. If this is an issue, you may move the LEDs to more suitable pins and modify the code accordingly.

### Files Required for Operation
The script requires two files before it may operate:
* `config.json`
* `variants.json`

The `config.json` file stores:
* `StartSearchingTime`: The time the script will begin requesting data from the API. Data usually will be published around 1600.
* `WaitTime`: When requesting data, the time in which the script will wait in between requests in seconds.
* `TimeoutTime`: The time in which the script will time out if no new data has been obtained. Must be earlier than the `StartSearchingTime`, ideally by 30 minutes or more.
* `ExcludedDays`: List of days in which the script will not check the API for new data at any time.
* `BotToken`: The token for the Discord bot.
* `ChannelID`: The ID number for the channel the bot will send and receive messages from.
* `AllData`: The file that contains all the formatted data from all time.
* `ErrorLogs`: The directory where fatal exception log files will be stored. These files are generated when the script enters an unrecoverable state and crashes.
* `Messages`: The file in which messages received from the API that have been sent are stored to ensure messages are not sent more than once, even between restarts. Also supports the entering of your own messages.
* `RollAvgPeaks`: The file which stores the dates and values of the 7-day rolling average peaks of Cases and Deaths.
* `RuntimeLogs`: The directory where standard operating logs are stored. This includes time-stampted messages about the current operation, whether it succeeded, and if it failed, the error details.
* `Variants`: The file which contains details about current variants of interest. Further information is provided below.
* `StatusMessages`: A list of web addresses that interface with the API to obtain relevant messages from the server.

You can create your own file using the following template:
```json
{
  "Configuration": {
    "StartSearchingTime": "",
    "WaitTime": 0,
    "TimeoutTime": "",
    "ExcludedDays": [

    ]
  },
  "Discord": {
    "BotToken": "",
    "ChannelID": ""
  },
  "Files": {
    "AllData": "",
    "ErrorLogs": "",
    "Messages": "",
    "RollAvgPeaks": "",
    "RuntimeLogs": "",
    "Variants": ""
  },
  "StatusMessages": {
    "BlueBannersAddresses": [
      "https://coronavirus.data.gov.uk/api/generic/log_banners/%DATE%/Cases/overview/United%20Kingdom",
      "https://coronavirus.data.gov.uk/api/generic/log_banners/%DATE%/Deaths/overview/United%20Kingdom",
      "https://coronavirus.data.gov.uk/api/generic/log_banners/%DATE%/Vaccinations/overview/United%20Kingdom"
    ],
    "YellowBannersAddress": "https://coronavirus.data.gov.uk/api/generic/announcements"
  }
}
```

The `variants.json` file is a regularly updated file. It combines the COVID-19 variants of Concern, Interest, and Observation from the [WHO list](https://www.who.int/en/activities/tracking-SARS-CoV-2-variants/) and associations from the [PANGO Lineage](https://cov-lineages.org) website.

Because neither of these services have any API or SDK to be utilised, this file needs to be updated manually at periodic intervals. I can provide these updates here for the life of the program here. However, if you wish to create your own file, use this template with the sample data as an example:
```json
{
  "Last Updated": "2021-12-13",
  "Associations": [
    {
      "Substring": "Q.",
      "References": [
        "B.1.1.7"
      ]
    }
  ],
  "Variants": [
    {
      "Ltr": "α",
      "Latin": "Alpha",
      "PANGO": [
        "B.1.1.7"
      ],
      "Variant of": "Concern",
      "Earliest Sample": "2020-09",
      "Nation": "GB"
    },
    {
      "Ltr": "ι",
      "Latin": "Iota",
      "PANGO": [
        "B.1.526"
      ],
      "Variant of": "Interest",
      "Earliest Sample": "2021-11",
      "Nation": "US"
    },
    {
      "PANGO": [
        "B.1.427",
        "B.1.429"
      ],
      "Variant of": "Observation",
      "Earliest Sample": "2020-03",
      "Nation": "US"
    }
  ]
}
```