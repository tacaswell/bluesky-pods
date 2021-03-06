# Pods for bluesky(-adaptive)

This is a set of buildah and podman scripts that will stand up a pod that
can run a Bluesky session and an out-of-core adaptive plan

## Build the containers

```sh
# this is fedora + some heavy weight Python
bash build_bluesky_base_image.sh
# installs the rest of our stack on top of the base image
bash build_bluesky_image.sh
# build an image with caproto installed
bash build_caproto_image.sh
```

## run the pod

```sh
# this sarts up caproto, mongo, zmqproxy, and redis
bash start_core_pod.sh
```

## .. manually

Run

```sh
bash launch_bluesky.sh
```

in a terminal


## ...and watch from the outside

On your host machine run:

```bash
python kafka_echo_consumer.py
```


##  ...adaptively

Start the adaptive server:

```sh
bash start_adaptive_server.py
```


Running in the shell should

```python
from ohpyd.sim import *
RE(adaptive_plan([det], {motor: 0}, to_brains=to_brains, from_brains=from_brains))
```

should now take 17 runs stepping the motor by 1.5.  The data flow is

```
  | ---> kafka to the edge  --- /exposed ports on edge/ --> external consumers
  | ---> mongo
  | ---> live table
  |
  ^
  RE ---- kafka broker -----> adaptive_server
  ^                                  |
  | < -------- redis --------<-----< |

```

Maybe redis should be replaced by kafka?

The extra imports are because the motor and det that are set up by 00-base.py do not have
the keys that the adatptive code is expecting.


## ...queuely

and run

```python
RE(queue_sever_plan())
```

On your host machine run:

```bash
http POST localhost:60606/add_to_queue 'plan:={"plan":"scan", "args":[["pinhole"], "motor_ph", -10, 10, 25]}'
```

and watch the scans run!

The data flow is

```
  | ---> kafka to the edge ----------- /exposed ports on edge/ ---> external consumers
  | ---> mongo                                                                 |
  | ---> live table                                                            |
  ^                                                                            ↓
  RE < --- http --- queueserver < ---- / http from edge / <-------- http POST {json}


```
