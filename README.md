Role Name
=========

semuadmin.octoprint

Ansible role to deploy octoprint and/or mjpg_streamer on Raspberry Pi running stock Raspberry
Pi OS 32-bit Lite or Full (Lite is recommended). Should also work on other Debian distributions
though webcam support may depend on the specific hardware.

By default:
- octoprint service will be available on: http://host_ip:5000.
- mjpg_streamer service will be available on: http://host_ip:8080/?action=stream.

(mjpg_streamer can be installed independently of octoprint if required)

The user will be prompted to go through the Setup Wizard on accessing the service for the
first time, but a pre-configured config.yaml template is included with standard configuration
items already set up. These can be amended via the Octoprint admin console.

Based on instructions here:
https://community.octoprint.org/t/setting-up-octoprint-on-a-raspberry-pi-running-raspbian/2337

Support for Python3 (>= 3.7) has been added since Octoprint 1.4, but *note that* many 
Octopi plugins still only work with Python2. The easiest way to install under Python3 is
to add a directive in your Ansible hosts file to point to the Python3 executable on your
target host. For Raspberry Pi OS this is typically `/usr/bin/python3`.

Requirements
------------

Raspberry Pi model 3 or 4 with SSH enabled. If installing on a fresh 'headless' Raspberry
Pi server (e.g. Raspberry Pi OS Lite), add an empty file named 'ssh' to the boot directory 
of the SD card to enable remote SSH access.

Role Variables
--------------

- `install_octoprint`: true
- `install_mjpg_streamer`: true
- `uninstall_services` : false
- `uninstall_dependencies` : false
        
- `host_ip`: 127.0.0.1
- `octoprint_user`: pi (this is the linux user under which the service runs)
- `install_dir`: "/home/{{ octoprint_user }}"
- `octoprint_port`: 5000

- `webcam_user`: mjpg_streamer
- `webcam_dir`: "{{ install_dir}}/mjpg-streamer/mjpg-streamer-experimental"
- `webcam_port`: 8080
- `webcam_type` : "usb" (options are 'usb', 'raspi' or 'custom')
- `webcam_width`: 640
- `webcam_height` : 480
- `webcam_fps`: 10 (frames per second)
- `custom_input` : "input_uvc.so" (input parameters to be used with 'custom' webcam type)

See https://community.octoprint.org/t/available-mjpg-streamer-configuration-options/1106
for available custom options.

Dependencies
------------

To install under Python2:
- Python >=2.7.9<3

To install under Python3:
- Python >=3.7

Example Playbook
----------------

To install octoprint and mjpg_streamer for raspberry pi camera:

```yaml

    - name: Provision octoprint on raspbian
      hosts: your_octoprint_hostname
      remote_user: pi
      become: true
      
      vars:
        host_ip: 192.168.0.18
        webcam_type: raspi

      roles:
      - semuadmin.octoprint
```

By default, Ansible will use the Python2 interpreter on Raspberry Pi OS.
To install under Python3, use the same playbook as above but add the
following entry to your ansible hosts file:

```yaml

[your_octoprint_hostname:vars]
ansible_python_interpreter=/usr/bin/python3
```

See https://docs.ansible.com/ansible/latest/reference_appendices/python_3_support.html 
for more details.

To install just mjpg_streamer with a custom uvc camera configuration:

```yaml

    - name: Provision octoprint on raspbian
      hosts: your_octoprint_hostname
      remote_user: pi
      become: true
      
      vars:
        install_octoprint: false
        install_mjpg_streamer: true
        host_ip: 192.168.0.18
        webcam_type: custom
        custom_input: "input_uvc.so -d /dev/video12 -r 1920x1080 -f 15 -q 50"

      roles:
      - semuadmin.octoprint
```

To uninstall octoprint, mjpg_streamer and all package dependencies:

```yaml

    - name: Uninstall octoprint on raspbian
      hosts: your_octoprint_hostname
      remote_user: pi
      become: true
      
      vars:
        uninstall_services: true
        uninstall_dependencies: true

      roles:
      - semuadmin.octoprint
```

License
-------

BSD

Author Information
------------------

semuadmin@noreply.user.github.com
