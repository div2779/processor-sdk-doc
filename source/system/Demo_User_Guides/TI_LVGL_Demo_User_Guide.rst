.. _TI-LVGL-Demo-User-Guide-label:

TI LVGL Demo - User Guide
=========================

Overview
--------

TI LVGL Demo is a simple app that showcases the capabilities of the `LVGL - Light and Versatile Graphics Library <https://lvgl.io/>`__.
It includes various demo applications, such as:

- EV Charging

- Smart Home

- Smart Meter

- Thermostat

- Security

All necessary equipment and step by step instructions are provided below:

.. note::

   Some of the features within each of these applications are not supported on all platforms. So there will be differences in feature
   enablement across platforms.

Hardware Prerequisites
----------------------

.. ifconfig:: CONFIG_part_variant in ('AM62LX')

   -  AM62L Evalaution Module: TMDS62LEVM

.. ifconfig:: CONFIG_part_variant in ('AM62X')

   -  TI AM62x SK / TI AM62x-LP SK / TI AM62xSIP SK / BeaglePlay

.. ifconfig:: CONFIG_part_variant in ('AM62P')

   -  TI |__PART_FAMILY_DEVICE_NAMES__| SK

-  PC (Windows or Linux, to use serial terminal console)

-  HDMI Monitor (to view the Demo on Display)

-  Ethernet Cable or CC33xx WiFi module (to connect |__PART_FAMILY_DEVICE_NAMES__| SK to internet)

-  Mouse or Touch Input from HDMI monitor (to control the TI LVGL Demo)

-  SD card (minimum 16GB)

-  Aux cable connected to earphone/speaker

.. ifconfig:: CONFIG_part_variant in ('AM62LX')

   -  Jumper cable (to connect potentiometer to ADC input) or some analog signal connected to ADC input. In the J11 header, pin D7 (A01)
      is the potentiometer out, which needs to be connected to pin D6 (A02) ADC in for channel 0.

      .. Image:: /images/ti-lvgl-demo-potentiometer-connection.png
         :height: 300


Launching the TI LVGL Demo
--------------------------

The demo will auto launch upon Linux booting on the EVM. Follow the below instructions to flash the SD card:

1. Flash an SD card with the :file:`tisdk-default-image`. User can download the :file:`tisdk-default-image` wic image from |__SDK_DOWNLOAD_URL__|.
   Please follow the instructions from here to :ref:`Flash an SD card <processor-sdk-linux-create-sd-card>`.

.. note::

   Images may vary between each platform based on the number of supported applications.

2. Insert the flashed SD card to the board, connect the display, mouse/touch-input, ethernet cable, aux cable, jumper wire and power on the SK-EVM.
   The TI LVGL Demo will launch automatically when the device is fully booted.

Using the TI LVGL Demo
----------------------

   The landing/home page of the LVGL demo looks like the following:

   .. Image:: /images/ti-lvgl-demo-home-page.gif
      :height: 500

   |

   - In the demo, scroll through the various widgets to launch different apps.
   - The date/time panel shows the UTC timezone (if device is connected to the internet)
   - Exit button on top left can be used to exit the demo. You can also use :file:`systemctl stop ti-lvgl-demo` to stop the demo.
   - Sytem button (Center button in top-right toolbar) shows FPS and CPU Utilization of the Demo.
   - You can switch between light annd dark theme from theme buttons in bottom-right toolbar.

   .. ifconfig:: CONFIG_part_variant in ('AM62LX')

      -  The temperature panel shows a value derived from the ADC data being input on ADC channel 0 pin.


Launching the EV Charging HMI
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

1. To launch the EV Charging HMI demo, click on the `EV Charging` widget in the apps scroll menu.
2. You can click on the `Start charging` button to emulate charging of EV. This will increment charge percentage at fixed time intervals.

   .. Image:: /images/ti-lvgl-demo-ev-charging1.png
      :height: 300
   .. Image:: /images/ti-lvgl-demo-ev-charging2.png
      :height: 300

Launching the Smart Home HMI
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

To launch the Smart Home HMI demo, click on the `Smart Home` widget in the apps scroll menu.

   .. Image:: /images/ti-lvgl-demo-smart-home1.png
      :height: 300
   .. Image:: /images/ti-lvgl-demo-smart-home2.png
      :height: 300

   |

*Understanding the MQTT setup of Smart Home*
   - All MQTT messages that are sent/received in this demo are to/from `broker.hivemq.com <https://broker.hivemq.com>`__ over port 8883.
   - You can find example MQTT communication between the demo HMI and online MQTT client in the following images:

      .. Image:: /images/ti-lvgl-demo-mqtt1.png
         :height: 300
      .. Image:: /images/ti-lvgl-demo-mqtt2.png
         :height: 300
   - Please note that the broker service used for this demo is free of charge, however, you can use other broker services as well and here
     are the necessary changes required in `ti-lvgl-demo/lv_port_linux/lvgl/demos/high_res/ <https://github.com/TexasInstruments/ti-lvgl-demo.git/>`__

      .. code-block:: c

         /* In mqtt_temp_pub_init() function in adc.c AND
            in mqtt_sub_init() function in mqtt_sub.c*/

         //Remove the following line if using unencrypted communication, else,
         //Update with path of the broker certificate you want to use.
         mosquitto_tls_set(mosq, "/usr/share/ti-lvgl-demo/cert/AmazonRootCA1.pem", NULL, NULL, NULL, NULL);

         //Replace 8883 with 1883 for unencrypted communication
         //Replace "broker.hivemq.com" with URL/IP of your broker
         rc = mosquitto_connect(mosq, "broker.hivemq.com", 8883, 60);

   - All messages transmitted/received are SSL-encrypted MQTT messages.
   - To interact with the HMI remotely, user can use any interface (mobile-app/web-app/website) that implements transmitting/receiving MQTT
     messages over internet. To do this, just set broker to `broker.hivemq.com` on that interface and subscribe to or publish on the below
     mentioned topics:

      - Remote device can subscribe to topic: `SmartHome/temp`
      - Remote device can subscribe to topic: `SmartHome/evCharge`
      - Remote device can publish to topic: `SmartHome/volume`
      - Remote device can publish to topic: `SmartHome/led`

   - Note that there is no restriction on who can publish/subscribe to the above topics. **That means that your device can receive data published
     from some other remote application apart from your own. Concurrent usage by other users on same MQTT topic may impact your GUI.** To minimize
     any issues, ensure that you use unique MQTT topic names, like *<unique ID>/SmartHome/<widget>*. The topics can be changed at following
     locations in `ti-lvgl-demo/lv_port_linux/lvgl/demos/high_res/ <https://github.com/TexasInstruments/ti-lvgl-demo.git/>`__:

      .. code-block:: c

         /* In  on_connect_vol_sub() function in mqtt_sub.c*/
         rc = mosquitto_subscribe(mosq, NULL, "SmartHome/volume", 1);
         :
         rc = mosquitto_subscribe(mosq, NULL, "SmartHome/led", 1);

         /* In  publish_sensor_data() function in adc.c*/
         rc = mosquitto_publish(mosq, NULL, "SmartHome/temp", strlen(payload), payload, 1, false);

         /* In  publish_evCharge_data() function in adc.c*/
         rc = mosquitto_publish(mosq, NULL, "SmartHome/evCharge", strlen(payload), payload, 1, false);

   .. note::

      By default CC33xx is configured at boot, so here are the steps that can be followed to enable it:

      .. code-block:: console

         $ systemctl stop ti-lvgl-demo
         $ cd /usr/share/cc33xx
         $ ./sta_start.sh
         $ ./sta_connect.sh -s WPA-PSK -n <SSID> -p <PASSWORD>
         $ udhcpc -i wlan0
         $ systemctl start ti-lvgl-demo

      For more details on how to enable CC33xx and connect to WiFi, visit :ref:`How to Enable M.2-CC33x1 in Linux <enable_m2cc3301>`

**As a part of this application, we have the following widgets:**

Widget#1: Climate Control
"""""""""""""""""""""""""

 .. note::

      This feature will work on platforms that have on-chip ADC. Following is supported on AM62Lx

 - The indoor temperature in this widget displays the digital data obtained from ADC and display (data/10) on the widget.
 - The outdoor temperature below the indoor teperature shows the value of (data/10)-6.5, if it is positive, else, shows zero.
 - The indoor temperature is sent over SSL-encrypted MQTT messages under the topic `SmartHome/temp`
 - For this demo, we are using channel 0 for ADC input. To change the channel being used, modify the following in
   `ti-lvgl-demo/lv_port_linux/lvgl/demos/high_res/adc.c <https://github.com/TexasInstruments/ti-lvgl-demo.git/>`__

      .. code-block:: c

         FILE *fp = popen("cat /sys/bus/iio/devices/iio\:device0/in_voltage0_raw", "r");

Widget#2: Charging
""""""""""""""""""

 - The data shown in this widget is the same that is shown in the EV Charging HMI app.
 - This percentage completion data is also sent over SSL-encrypted MQTT messages under the topic `SmartHome/evCharge`

Widget#3: Lock
""""""""""""""

   .. note::

      Lock screen feature has only been enabled on AM62Lx SK

   - Sliding the lock to the right will lock the screen (disable all screen input)
   - To unlock, press the SW5: User Button on the SK-AM62Lx EVM

         .. Image:: /images/ti-lvgl-demo-usr-button.png
            :height: 200

Widget#4: Speaker
"""""""""""""""""

   - The play/pause button will play noise in the McASP device.
   - You can play your own audio file by making necessary changes as shown below in
     `ti-lvgl-demo/lv_port_linux/lvgl/demos/high_res/audio.c <https://github.com/TexasInstruments/ti-lvgl-demo.git/>`__

      .. code-block:: c

         /* Copy your audio file in the SD file system */
         /* In audio_play() function in audio.c */

         //Modify the following as per the specification of your audio file
         rate     = 48000; //in Hz
         channels = 1;     //1: mono, 2:stereo
         seconds  = 1;     //duration of audio
         :
         :
         //Replace "/usr/share/sounds/alsa/Noise.wav" with path of your audio file
         char *filename_wav = "/usr/share/sounds/alsa/Noise.wav";

   - Volume can be controlled from the vertical slider on the right-side of the widget.
   - Volume can also be set by sending values between 1 to 100 over MQTT topic `SmartHome/volume` from remote device over internet.

Widget#5: Main Light
""""""""""""""""""""

   - The `Light temperature` slider will increase/decrease the User LED blinking frequency.
   - The left-end of slider (0 K) toggles LED every 250ms and the right-end of slider (20000 K) toggles LED every 20ms.
   - `Light temperature` can also be set by sending values between 0 to 20000 over MQTT topic `SmartHome/led` from remote device over internet.
   - In HMI, the slider `Light temperature` is not associated with any functionality but can be programmed be user to perform any function.

Widget#6: Sensor Controls
"""""""""""""""""""""""""

   - The buttons/sliders in this widget are not associated with any functionality but can be programmed by the user to perform any function.

Launching the Smart Meter HMI
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

To launch the Smart Meter HMI demo, click on the `Smart Meter` widget in the apps scroll menu.

   .. Image:: /images/ti-lvgl-demo-smart-meter.png
      :height: 300


Launching the Thermostat HMI
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

To launch the Thermostat HMI demo, click on the `Thermostat` widget in the apps scroll menu.

   .. Image:: /images/ti-lvgl-demo-thermostat.png
      :height: 300


Launching the Security HMI
^^^^^^^^^^^^^^^^^^^^^^^^^^

To launch the Security HMI demo, click on the `Security` widget in the apps scroll menu.

   .. Image:: /images/ti-lvgl-demo-security.png
      :height: 300

This widget contains a slide-show on Application Processor Security for AM6X devices.

Building the TI LVGL Demo
-------------------------

The TI LVGL Demo is enabled by default in yocto for |__PART_FAMILY_DEVICE_NAMES__| Linux. Note, that the binary itself does not have
asset images and slides built in it. :file:`tisdk-default-image` contains the required assets within :file:`/usr/share/ti-lvgl-demo/*`.
You will need to place any additional assets here while making any modifications. Yocto recipe for building this demo can be found at
`github: ti-lvgl-demo.bb <https://github.com/TexasInstruments/meta-tisdk/blob/scarthgap/recipes-demos/ti-lvgl-demo/ti-lvgl-demo.bb>`__

The source code is available at `TI LVGL Demo <https://github.com/TexasInstruments/ti-lvgl-demo.git/>`__ and can be re-compiled with the
following steps:

1. First clone the git repository and its submodules using:

   .. code-block:: console

      $ git clone --recurse-submodules https://github.com/TexasInstruments/ti-lvgl-demo.git

2. Create the docker environment and build the application:

   .. code-block:: console

      $ cd ti-lvgl-demo
      $ sudo ./scripts/docker_setup.sh --create-image
      $ sudo ./scripts/docker_setup.sh --build-app

3. Copy the compiled binary to :file:`/usr/bin` directory of the device

   .. code-block:: console

      $ scp lv_port_linux/bin/lvglsim root@<ip-addr-of-device>:/usr/bin/


