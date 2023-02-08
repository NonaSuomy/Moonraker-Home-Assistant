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

Name: 3D Printer Camera

MJPEG URL: `<moonraker-ip-address>/webcam/?action=stream`

Still Image URL: `<moonraker-ip-address>/webcam/?action=snapshot`

Uncheck Verify SSL certificate.

Click SUBMIT.

If you need to flip your camera uncomment this section in the yaml

```
#    card_mod:
#      style: |
#        ha-card {
#          transition: transform 0s;
#          transform: rotate(180deg);
#        }
```

## Add The Thumbnail 

Click Settings -> Devices and Services -> + Add Integration

Type Generic Camera and click it.

In the Generic Camera → Still image URL box put:

`http://<moonraker-ip-address>:7125/server/files/gcodes/{{ states("sensor.3d_printer_object_thumbnails") }}`

![image](https://user-images.githubusercontent.com/1906575/217470789-4a640fd2-a8d8-438b-9f31-33a660c91538.png)

Then click submit.

Click check box on "This image looks good" then submit again.

```
Success!
Options successfully saved.
```

Click Finish.

It will unfortunatly add a camera with its name as the IP address.

Click Settings -> Entities

Find the camera.IP in the "Entity ID" column you made.

Click the Entity ID of it and rename it to: camera.3d_printer_thumbnail

Click the "Name" of it and rename it to: 3D Printer Thumbnail

![image](https://user-images.githubusercontent.com/1906575/217494326-3ec35b96-fd07-422f-a75f-90b1df8985ca.png)

Click Update.

*Note:* Depending on how many thumbnails your slicer creates you may have to change this array value from [2] to [1] in two places in the mooonraker.yaml mine generates 3

```
[0] 32x32
[1] 64x64
[2] 400x300
```

```
  - platform: rest
    scan_interval: 15
    name: klipper_preview_path
    unique_id: "<moonraker-ip-address>fd0ae36c-0d51-4392-a18d-89861d536ba4"
    resource_template: "http://<moonraker-ip-address>:7125/server/files/metadata?filename={{ states(('sensor.3d_printer_current_print')) | urlencode }}"
    json_attributes_path: "$.result.thumbnails.[2]"
    json_attributes:
      - relative_path
      - width
      - height
      - size
    value_template: "OK"
```

`json_attributes_path: "$.result.thumbnails.[2]"` -> `json_attributes_path: "$.result.thumbnails.[1]"`

```
    - name: 3d_printer_object_thumbnails
      unique_id: "<moonraker-ip-address>59b37837-b751-4d31-98c2-516a52edf833"
      state: >
        {% set dir = states('sensor.3d_printer_current_print') %}
        {% set img = states.sensor.printer_3d_file_metadata.attributes["thumbnails"][2]["relative_path"] %}
        {{ (dir.split('/')[:-1] + [img]) | join('/') }}
      availability: >
        {% set items = ['sensor.printer_3d_file_metadata'] %}
        {{ expand(items)|rejectattr('state','in',['unknown','unavailable'])
          |list|count == items|count }}
      icon: mdi:image
      attributes:
        friendly_name: "Object Thumbnails"
```

`{% set img = states.sensor.printer_3d_file_metadata.attributes["thumbnails"][2]["relative_path"] %}` -> `{% set img = states.sensor.printer_3d_file_metadata.attributes["thumbnails"][1]["relative_path"] %}`

## Lovelace cards

Click Edit Dashboard

Click + Add Card
  
Scroll to the bottom and click Manual

Paste the yaml content below into the Manual cards.

> **Grid Card**

Showing model preview window with file name, live camera and buttons to stop, FW reset, pause, resume and cancel. [Card code](https://github.com/NonaSuomy/Moonraker-Home-Assistant/blob/main/grid.yaml)

> **Glance Card (Info and Sonoff Power Plug)**

Showing info and button to turn on and off the Sonoff S26 Power Plug. [Card code](https://github.com/NonaSuomy/Moonraker-Home-Assistant/blob/main/glance-card.yaml)

> **Gauge Card (printing progress)**

Showing currently % of printing progress. Changes color depending on % of progress [Code card](https://github.com/NonaSuomy/Moonraker-Home-Assistant/blob/main/gauge-card.yaml)

> **Temperature Graph (hotend and bed)**

Using mini-graph-card component, to show graph of hotend and bed temp. [Code card](https://github.com/NonaSuomy/Moonraker-Home-Assistant/blob/main/mini-graph-card.yaml)

If you want to use the more acurate layer counter "Actual Layer" you must add to your slicer (Prusa/Super):

In the "Start" GCode

```
SET_PRINT_STATS_INFO TOTAL_LAYER=[total_layer_count]
SET_PRINT_STATS_INFO CURRENT_LAYER=0
```
  
In the “Before layer change" GCode

```
SET_PRINT_STATS_INFO CURRENT_LAYER=[layer_num]
```
  
## Thank you!

This is comprised of all three of these code bases below

- **@denkyem**'s [Home Assistant Moonraker](https://github.com/denkyem/home-assistant-moonraker)
- **@SirGoodenough**'s [Home Assistant Moonraker](https://github.com/SirGoodenough/DEV-Moonraker-HA)
- **@mSarheed**'s [Home Assistant Moonraker](https://github.com/mSarheed/home-assistant-moonraker)

## Few added tweaks
  
- Added more directions to readme and updated picture.
- Larger quality thumbnails.
- Fixed urlencode, multi, and no directory thumbnails Thanks @davisgoodman @petro @NSX
- Added Actual Layer Sensor to show a more accurate layer counter. @davisgoodman idea
- BTT Smart Filament Runout Sensor.
- Chamber Temperature.
- Flipped Webcam.
- Removed synthwave theme.
