# Moonraker For Home Assistant

Integration between Moonraker and Home Assistant 

![image](https://user-images.githubusercontent.com/1906575/215313907-ba5221d9-6ccb-47f4-8656-ffac2d450bb8.png)

## Setup

### Requirements

- HACS (for installing custom component)
- **@thosmasloven**'s [Lovelace Card Mod](https://github.com/thomasloven/lovelace-card-mod)
- **@kalkih**'s [Mini Graph Card](https://github.com/kalkih/mini-graph-card)
- **Sonoff S26** with **ESPHome** for powering the 3D printer on/off

### HACS Install

SSH into your Docker host and navigate to the folder location where you have the Home Assistant /config volume.

Run the command below.

```
wget -O - https://get.hacs.xyz | bash -
```

The files will now be downloaded and put into the directory where the Home Assistant volume is mapped.

Restart the docker

```
sudo docker restart homeassistant
```

Log in to Home Assistant.

After Home Assistant connects, select Settings, then Devices & Services.

Make a folder in your home assistant directory

In the bottom right, select Add Integration.

Search for HACS and select it.

If you agree with everything, select all options and then Submit.

Copy the code that Home Assistant provides and then select the link to sign into GitHub.

Sign in to GitHub, and then paste in the code from the previous step.

If you’d like to proceed, select Authorize HACS.

HACS is now installed! It’s best to reboot your Docker host at this point.

```
sudo docker restart homeassistant
```

After Home Assistant loads back up, HACS will be fully configured and ready to use!

### HA Packages Configuration

```
mkdir homeassistant/config/packages
```

Add packages directory to configuration.yaml

```
homeassistant:
  packages: !include_dir_named packages
```

Put the moonraker.yaml in

```
cp moonraker.yaml homeassistant/config/packages/moonraker.yaml
```

Replace <moonraker-ip-address> with your 3D Printer IP in the moonraker.yaml file.

```
sed -i 's/<moonraker-ip-address>/10.0.0.69/g' homeassistant/config/packages/moonraker.yaml
```

Restart Home Assistant through the WebUI then you should see the entities of your 3D Printer show up.

## Install HACS Items

Click HACS on the left column -> Frontend -> + Explore & Download Repositories

Search at the top for card-mod and mini-graph-card and then install them.

In your configurations.yaml add this for card-mod

```
frontend:
  extra_module_url:
    - /community/lovelace-card-mod/card-mod.js
```

## Add The MJPEG Camera

Click Settings -> Devices and Services -> + Add Integration

Type MJPEG IP Camera and click it.

Name: 3d printer camera

MJPEG URL: <moonraker-ip-address>/webcam/?action=stream

Still Image URL: <moonraker-ip-address>/webcam/?action=snapshot

Uncheck Verify SSL certificate.

Click SUBMIT.

## Lovelace cards

Click Edit Dashboard

Click + Add Card
  
Scroll to the bottom and click Manual

Paste the yaml below into the Manual cards.

> **Grid Card**

Showing model preview window with file name, live camera and buttons to stop, FW reset, pause, resume and cancel. [Card code](https://github.com/NonaSuomy/Moonraker-Home-Assistant/blob/main/grid.yaml)

> **Glance Card (Info and Sonoff Power Plug)**

Showing info and button to turn on and off the Sonoff S26 Power Plug. [Card code](https://github.com/NonaSuomy/Moonraker-Home-Assistant/blob/main/glance-card.yaml)

> **Gauge Card (printing progress)**

Showing currently % of printing progress. Changes color depending on % of progress [Code card](https://github.com/NonaSuomy/Moonraker-Home-Assistant/blob/main/gauge-card.yaml)

> **Temperature Graph (hotend and bed)**

Using mini-graph-card component, to show graph of hotend and bed temp. [Code card](https://github.com/NonaSuomy/Moonraker-Home-Assistant/blob/main/mini-graph-card.yaml)

## Thank you!

This is comprised of all three of these code bases below

- **@denkyem**'s [Home Assistant Moonraker](https://github.com/denkyem/home-assistant-moonraker)
- **@SirGoodenough**'s [Home Assistant Moonraker](https://github.com/SirGoodenough/DEV-Moonraker-HA)
- **@mSarheed**'s [Home Assistant Moonraker](https://github.com/mSarheed/home-assistant-moonraker)

## Few added tweaks
  
- Added more directions to readme and updated picture.
- Larger quality thumbnails.
- BTT Smart Filament Runout Sensor.
- Chamber Temperature.
- Flipped Webcam.
- Removed synthwave theme.
