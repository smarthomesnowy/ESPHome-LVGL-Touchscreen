# ESPHome LVGL Touchscreen
![2024-10-02 07_32_55-Window](https://github.com/user-attachments/assets/9f774921-8fe4-4fee-a19e-3b3520effec9) ![2024-10-02 07_24_13-Window](https://github.com/user-attachments/assets/3b58b561-aa74-425a-adb5-47c098e926d3)
## Idea
ESPHome 2024.8.0 included the LVGL graphics library which allows for much better styling options for ESPHome displays.

Clyde Barrow has but a LOT of work into making this work and Nagyrobi testing and making the documentation for it.

In a bid to get ahead of the curve I was testing LVGL in beta so I could get a good handle of what was going on with it from the start.
I am using this project to document the steps needed to make a LVGL display and to do it in a smart way that should help others.

This project is a new board (ESP32 S3) and a new (to me) 480X320 touchscreen.
The screen is an unbranded screen with a generic XPT2046 touchscreen. I had a lot of problems getting the config tweaked correctly to get the screen running then ran into an odd problem of the screen showing these random black blocks and crashing when the touchscreen was enabled.
Turns out the VCC needs 3.3V not 5V - bad documentation!!

So finally after much re wiring and juggling to get the display and touchscreen lined up in the right orientation it was working and I could begin coding...

## Parts used

- LilyGO T7-S3 ESP32-S3 Development Board
- Unbranded 480X320 Touchscreen

## Features
I already have 3 different displays with bluetooth speakers and RGB lighting with display layouts showing a lot of different information but with this LVGL layout I want a lot more from it.
I made a list of the features I would want in a touchscreen enabled device.

- Nice big display of time, date, day on the homescreen and a smaller time display on each page.
- Animated icons for the weather states.
- Display temperature sensor data in different colours based on the temperature.
- Football and F1 pages with all the relevant data.
- Short display of fixtures and countdowns to those fixtures for the homepage.
- Large nav panel with big buttons that can be "jabbed" at with a finger.
- Buttons to activate different HA scenes / media player / heating / cooling on the homepage.
- Team badges for football fixtures that dynamically change based on the fixture.

## Planning

This is really important when starting an LVGL display project. I drew on paper the type of design I wanted first, making a few pages and making a list in notepad++ of all the sensors I would have to import from Home Assistant and the layout of them on the screen.
One of the best tips I can give is to make !include files, at the start of the project I did not follow this path and literally hurt my finger scrolling all the time!, to make an LVGL layout you need to edit sensors as well as the widgets and labels etc so you find yourself forever scrolling or using "find" to jump up and down the file.
So plan ahead and make include files from the start! What I ended up doing was making these files -

- LVGL.yaml - to contain all the code from the lvgl: block
- images.yaml - to declare all the images used in the project
- fonts.yaml - to declare all the fonts used in the project
- colours.yaml - for all the colours
- sensors.yaml - sensors that are imported from Home Assistant and a few onboard sensors used for diagnostics

In the main yaml for the device I then included these yaml files into the config and had them open in VS Code so I could edit all of them at once. You can even go further and make yaml files for different widgets you are going to reuse in your project.
If you use VS Code for editing remember to install the ESPHome extensions so that you can use the OTA button to compile and upload to the device from the same window you are editing in.

A good layout tip is to make a photoshop or gimp file of the same size as the display and then you can experiment with the layout there faster and then use the rulers command to find out the x: and y: values that will correspond to the x: and y: on the display.

## LVGL 
LVGL uses a structure of Pages - Widgets - Objects to display things, if you look at any example of LVGL code you will see the structure over and over.
You will run into a lot of indentation problems while making LVGL pages so remember to highlight lines u need to move then use TAB and SHIFT TAB or [ and ] with SHIFT to move around blocks of code to get them in the right place
```yaml

pages:
  - id: main_page
    widgets:
      - obj:
          widgets:
            - label:
            - button:

  - id: second_page
    widgets:
      - obj:
          widgets:
            - label:
            - button:

```

That code would make two different pages with a label and a button on it. It would render these widgets in the default styles, you can define styling for anything in a style definition block or style it directly on the page/widget/object.

I found using the inbuilt grid layout system the best way to get things aligned with each other and in the right place, by editing the x: y: coords of the obj: the grid is sitting in you can move grids around.

In LVGL if you want to display an onboard sensor or one from Home Assistant you have to also update the LVGL label for that sensor.

```yaml
    - platform: homeassistant
        id: shelf_temperature
        entity_id: sensor.shelf_sensors_temperature
        on_value:
        - lvgl.label.update:
            id: livingroom_temp_label
            text:
                format: "%.0f°C"
                args: [x]
```

So this will pull the temperature from a Home Assistant sensor (actually on another ESPHome node) and then update the label: livingroom_temp_label - for labels I added _label to the sensor id to make the label: id.
The text: block is just telling LVGL the formatting of the output, make it to 0 decimal places so it rounds up and down to the next degree and adding °C after the last character.

So after the sensor is made and a new label id made and the code inplace for updating it if it changes, we need to make a label to be able to display it.

```yaml
    - label:
        id: livingroom_temp_label
        align: CENTER
        text_align: CENTER
        text_color: my_blue
        text_font: robo_25
```

This will display the temperature in the living room in the center of the screen with the text aligned to the center so there is an equal distance to the edge on both sides, the font is Roboto size 25px and light blue in colour. The fonts and colours are defined in their own yaml files.

On my display I have used grids, labels, buttons, and sliders, to make the layout. You an find out more examples for LVGL can be found here - https://esphome.io/cookbook/lvgl.html

## Construction

Not really any construction yet, the project is about getting a working LVGL display with all the features from my wishlist ( well as many as I can do ) in the display. Eventually it will have it's own 3D printed enclosure so that's where the construction will come into play as work out how to pack everything in and make it look nice.

## Ideas for the future
I want to eventually tweak the display to add a "controls" page with some sliders for sets of lights and also my Home Assistant media player control. I don't really use streaming services to a "media page" with now playing etc is not going to work.
A page for each football team and F1 for when there is a game on, I have made sensors in HA for this and made the new pages but cannot decide how to make it work.

## Enclosure
There will be another project coming soon in which I will add voice assistant to this board and design and print an enclosure for this.
I have the idea to make a vintage desktop PC and monitor to house everything in....

## Conclusion

This is my most complicated device so far and after using it for a while I can see some additions I could make such as making an "alarm" on the display and LED ring for certain triggers.
I am very happy with this project and will convert the other two clocks when the display boards arrive.


