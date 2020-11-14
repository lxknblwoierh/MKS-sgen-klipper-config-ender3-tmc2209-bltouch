# MKS-sgen-klipper-config
Klipper config for mks sgen, ender3 and bltouch and tmc2209
```
#### custom Ender 3 build
##
##  - MKS S_GEN
##  - stock steppers
##  - TMC2209 drivers in UART mode
##  - glass bed
##  - BLTouch V3
##  - Ender3 stock display
##  - top-down LED lighting
##
########################################
## Originally posted on https://www.reddit.com/r/klippers/comments/csjzid/sample_config_for_ender_3_bltouch_v30_skr_v13/
## (c) VonThing 2019
## Edited by MatejZ90
## Licensed under CC-BY-SA-4.0 https://choosealicense.com/licenses/cc-by-sa-4.0/
## Redistributions MUST include this copyright notice and the same license terms.

[bltouch]
sensor_pin: ^P1.25   # Pull-up (^ symbol) needed in open drain mode
control_pin: P1.23
# Some BLTouch V3 and many clones apparently require this
pin_up_touch_mode_reports_triggered: False
x_offset: -53
y_offset: -11
# this is for ABS z_offset: -0.05
z_offset: -0.05
speed: 5.0
samples: 1

# I went with pretty safe values for run/hold currents:
[tmc2209 stepper_x]
uart_pin: P1.1
microsteps: 32
run_current: 0.640
hold_current: 0.520
stealthchop_threshold: 250

[tmc2209 stepper_y]
uart_pin: P1.8
microsteps: 32
run_current: 0.640
hold_current: 0.520
stealthchop_threshold: 250

[tmc2209 stepper_z]
uart_pin: P1.10
microsteps: 32
run_current: 0.520
hold_current: 0.380
stealthchop_threshold: 100

# Stealthchop disabled on extruder. It's useless.
[tmc2209 extruder]
uart_pin: P1.15
microsteps: 32
run_current: 0.800
hold_current: 0.640
stealthchop_threshold: 0

#[tmc2209 extruder1]
#uart_pin: P1.17
#tx_pin: P4.29
#microsteps: 32
#run_current: 0.800
#hold_current: 0.500
#stealthchop_threshold: 5

[stepper_x]
step_pin: P2.2
dir_pin: !P2.3
enable_pin: !P2.1
#64step_distance: .0031353859660124
step_distance: .00627077193
endstop_pin: ^P1.29  # ^P1.28 for X-max
position_endstop: 0
position_max: 235
homing_speed: 100

[stepper_y]
step_pin: P0.19
dir_pin: !P0.20
enable_pin: !P2.8
#64step_distance: .0031327339369067
step_distance: .00626546787
endstop_pin: ^P1.27  # ^P1.26 for Y-max
position_endstop: 0
position_max: 235
homing_speed: 100

[stepper_z]
step_pin: P0.22
dir_pin: P2.11
enable_pin: !P0.21
#64step_distance: .00062300249
step_distance: .00124600498
# You need this to be below zero because with a Z probe, the Z axis may need to go below zero (due to probe offset):
position_min: -5
position_max: 250
endstop_pin: probe:z_virtual_endstop  # ^P1.24 for Z-max

[extruder]
max_extrude_only_distance: 100
step_pin: P2.13
dir_pin: !P0.11
enable_pin: !P2.12
# Tune your Esteps just like in marlin except when you get a % under/over extrusion just multiply this number with that percentage (0.xx for under/1.xx for overextrusion)
#64step_distance: 0.00056120186
step_distance: .00112240372
nozzle_diameter: 0.400
filament_diameter: 1.750
heater_pin: P2.7
sensor_type: EPCOS 100K B57560G104F
sensor_pin: P0.23
# These get overrridden when you tune PID from Klipper. These are my defaults for a 200 C target.
#control: pid
#pid_kp = 23.975
#pid_ki = 0.951
#pid_kd = 151.044
min_temp: 0
max_temp: 300

# Ender 3 stock heater fan wires into the second extruder heater output marked 'HE1'
#[heater_fan e0_fan]
#pin: P2.4
#heater: extruder
#heater_temp: 50.0
#fan_speed: 1.0

# Ender 3 stock heater bed
[heater_bed]
heater_pin: P2.5
sensor_type: EPCOS 100K B57560G104F
sensor_pin: P0.24
control: pid
pid_Kp: 54.027
pid_Ki: 0.770
pid_Kd: 948.182
min_temp: 0
max_temp: 130

# This is the part cooling fan that plugs right in to the fan output marked 'FAN'
[fan]
pin: P2.4

# This is the part where you customize
[mcu]
serial: /dev/serial/by-id/usb-Klipper_lpc1768_1610000802094AAF57805E5DC12000F5-if00

[printer]
kinematics: cartesian
max_velocity: 300
max_accel: 1500
max_z_velocity: 5
max_z_accel: 100

# This adds a bed mesh calibrate GCode
[bed_mesh]
speed: 200
horizontal_move_z: 5
mesh_min: 5,5
mesh_max: 180,220
probe_count: 3,3

# Z-safe homing that (1) moves the probe up 15mm (2) home XY (3) move to center of bed (4) home Z
# Edit the X166 Y120 for your own center of bed
[homing_override]
axes: z
gcode:
    G90
    G1 Z15
    G28 X Y
    G1 X60 Y15 F6000
    G28 Z
set_position_z: 0.0

# Custom G29 that does (1) home all (2) get bed mesh (3) move nozzle to corner so it doesnt ooze on the bed while heating up.
[gcode_macro G29]
gcode:
    G28
    BED_MESH_CALIBRATE
    G0 X10 Y10 Z10 F6000

# This adds a screw tilt adjust command that probes certain points on the bed and recommends new bed screw positions (turn clockwise this much etc)
[screws_tilt_adjust]
screw1: 50,50
screw1_name: Front left screw
screw2: 50,215
screw2_name: Rear left screw
screw3: 215,50
screw3_name: Front right screw
screw4: 215,215
screw4_name: Rear right screw
horizontal_move_z: 10
speed: 250
screw_thread: CW-M3

# Filament switch sensor made out of the leftover Z-stop after BLTouch install
# Wired to X-max stop but you can pick any pin you want.
#[filament_switch_sensor filament_sensor]
#pause_on_runout: True
#switch_pin: ^!P1.28

[gcode_macro T0]
gcode:
    G28
    BED_MESH_CALIBRATE
    G0 X10 Y10 Z10 F6000
    SAVE_CONFIG

[gcode_macro M80]
gcode:
    RESPOND TYPE=command MSG=action:poweron
    G4 P1500
    FIRMWARE_RESTART

[gcode_macro M108]
gcode:
    TURN_OFF_HEATERS

# This adds a bed screws adjust GCode, that moves the nozzle around for you so you can paper adjust.
[bed_screws]
screw1: 31,37
screw1_name: Front left screw
screw2: 31,208
screw2_name: Rear left screw
screw3: 201,208
screw3_name: Rear right screw
screw4: 201,37
screw4_name: Front right screw

# This adds pause/resume support
[pause_resume]

# This adds the 'respond' G-Code that you can use to send commands back to OctoPrint
[respond]
default_type: echo

# This enables a 'force_move' command ignoring all homing, Z-stops etc. Useful in debugging.
[force_move]
enable_force_move: True

#*# <---------------------- SAVE_CONFIG ---------------------->
#*# DO NOT EDIT THIS BLOCK OR BELOW. The contents are auto-generated.
#*#
#*# [extruder]
#*# control = pid
#*# pid_kp = 23.975
#*# pid_ki = 0.951
#*# pid_kd = 151.044
#*#
#*# [bltouch]
#*#
#*# [bed_mesh default]
#*# version = 1
#*# points =
#*# 	  -0.013706, -0.351996, -0.632971
#*# 	  0.186901, -0.032396, -0.425511
#*# 	  -0.469121, -0.796820, -1.181213
#*# tension = 0.2
#*# min_x = 5.0
#*# algo = lagrange
#*# y_count = 3
#*# mesh_y_pps = 2
#*# min_y = 5.0
#*# x_count = 3
#*# max_y = 220.0
#*# mesh_x_pps = 2
#*# max_x = 180.0
```
