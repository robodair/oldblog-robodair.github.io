---
layout: single
title: Twitter Controlled RPi DMX Light
excerpt: "Part 2 of my Twitter Moodlighting: Controlling the RPi DMX device from the internet."
categories:
  - Software
tags:
  - moodlighting
  - electronics
  - RaspberryPi
  - dmx
  - lighting
  - tutorial
  - Python
header:
  teaser: http://dair.io/images/rpi-dmx/market-day-stall.jpg
comments: true
---
### Part 1:
[Raspberry Pi “Native” DMX](/electronics/raspberrypi-native-dmx-twitter-moodlighting-part-1/)

UPDATE: I've not done anything more on this project for several months - Uni really gets in the way.
I had this mostly-finished post almost ready to go so I'm putting it up now.
I'm not sure if I'll revisit this project later on and expand it to my original plans or not, no promises.

## Intro
This is Part 2 of my Twitter controlled moodlighting project.

The idea is to create a number of RasperryPi's configured to output DMX, and provide some way to coordinate them remotely. From there I can build anything I like on top.

I used the first iteration of this project at my university market day stall for the Student Developer Organisation (Sudo).

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr"><a href="https://twitter.com/nicholasklomp">@nicholasklomp</a> drop by the <a href="https://twitter.com/sudo_uc">@sudo_uc</a> market day stall and I&#39;ll show you the <a href="https://twitter.com/hashtag/spectrumuc?src=hash">#spectrumuc</a> project we&#39;ve been working on!<br>purple <a href="https://t.co/d36YW2JvYz">pic.twitter.com/d36YW2JvYz</a></p>&mdash; Alisdair Robertson (@robodair) <a href="https://twitter.com/robodair/status/829130866320306176">February 8, 2017</a></blockquote>
<!--<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>-->

It also just so happened that Andrew Barr (Chief Minister of the ACT) showed up and changed the colour of our light! (To red, of course)

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">A huge shoutout to Andrew Barr at <a href="https://twitter.com/UniCanberra">@UniCanberra</a> Market Day today! Thank you for stopping by. <a href="https://t.co/duPgiEKEUR">https://t.co/duPgiEKEUR</a> <a href="https://t.co/SuHxbtAcTG">pic.twitter.com/SuHxbtAcTG</a></p>&mdash; UC Sudo. (@sudo_uc) <a href="https://twitter.com/sudo_uc/status/829208017296056320">February 8, 2017</a></blockquote>
<!--<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>-->

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">Great to visit the <a href="https://twitter.com/UniCanberra">@UniCanberra</a> Market Day today.  <a href="https://twitter.com/hashtag/StudyCanberra?src=hash">#StudyCanberra</a><a href="https://twitter.com/hashtag/CBR?src=hash">#CBR</a> <a href="https://t.co/Cxyh9K8jOd">pic.twitter.com/Cxyh9K8jOd</a></p>&mdash; Andrew Barr (@ABarrMLA) <a href="https://twitter.com/ABarrMLA/status/829215306103021568">February 8, 2017</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

## Bits and Pieces
1. One or more of the RaspberryPi's from the last post, with accompanying gear to get it outputting DMX
2. Python 3.5.2 (or higher, but I wrote the code to work with 3.5.2)
3. A [Firebase](https://firebase.google.com) account, I chose to use Firebase because they have a free tier and there are nice python libraries available, and it's easy to get a realtime feed, but you could use whatever backend works for you.

## Parts of the project
There's surprisingly few steps involved here.

1. Setup a Firebase account and create a Firebase instance
2. Setup a Twitter account & get API access
3. Code on the RPi that subscribes to colour changes in Firebase, to change the colour of the light
4. Twitter bot to write colours to Firebase

## Firebase
I won't bore you with setting up the firebase, but instead I'll jump straight into how I structured it.

I started with two user accounts, one for the node, and one for my twitter bot to be.

<img style="width:80%;" src="/images/spectrum/spectrum-firebase-users.jpg">

To keep things simple from the root node I created a 'nodes' branch, and under that an entry with the UID of the node account as the key. This is pretty standard practice with firebase as it allows you to use rules based on UID to restrict writes to the database.

While developing I haven't put additional restrictions on reads and writes other than being authenticated, although before we finish this I'm sure the rules will have become a lot more complex.

Under the node there's a name entry and a colour entry, representing RGB values, pretty straightforward:

<img style="width:80%;" src="/images/spectrum/spectrum-firebase-structure.jpg">

## Twitter
To access the Twitter API you need to create an app, do this by going to [apps.twitter.com](https://apps.twitter.com/) and following the steps there. Keep the tab open somewhere, we'll need to know those API keys later.

# The Fun Part - Code
Now we actually get into some coding - first up, code running on the RaspberryPi that subscribes to the colour key under it's UID in the Firebase, and issues the command to change the DMX output when the light changes. Followed by the really simple twitter bot.


## Lamp Node
To make this ridiculously easy to accomplish I decided to use [Pyrebase](https://github.com/thisbejim/Pyrebase), a really nice python module that is designed for communicating with Firebase.

To make this a little bit more complicated I decided that I wanted to make my Node send a heartbeat pulse to the server so I could tell if one ever went offline. To make this simple I decided to abstract the Firebase connection and heartbeat into one python module (`firebase.py`), and implement the colour changing in another (`lamp.py`).

Finally, because the image containing OLA that we wrote to the pi doesn't contain Python 3.5.X, you'll need to compile Python on it.

There is a good guide here: [http://bohdan-danishevsky.blogspot.com.au/2015/10/building-python-35-on-raspberry-pi-2.html](http://bohdan-danishevsky.blogspot.com.au/2015/10/building-python-35-on-raspberry-pi-2.html)

In short:

```bash
sudo apt-get update
sudo apt-get upgrade -y
sudo apt-get dist-upgrade
sudo apt-get install build-essential libncursesw5-dev libgdbm-dev libc6-dev
sudo apt-get install zlib1g-dev libsqlite3-dev tk-dev
sudo apt-get install libssl-dev openssl
cd ~
wget https://www.python.org/ftp/python/3.5.2/Python-3.5.2.tgz
tar -zxvf Python-3.5.2.tgz
cd Python-3.5.2
./configure
make
sudo make install
```

Here's `lamp.py`:

```python
#!/usr/bin/env python3
"""
lamp.py subscribes to this node's color in firebase,
then issues shell commands to change the color of the light.
"""
import os
from . import firebase as backend

__version__ = '0.0.1'

def main():
    """Initialise the backend and register a callback when the color changes"""
    database = backend.Database(config_file_path="firebase.cfg")
    database.initialise()
    database.register_callback(change_color)


def change_color(color):
    """Issue the command to change the color"""

    dmx_string = ",".join(str(x) for x in [color['red'], color['green'], color['blue']])
    command = "ola_streaming_client -d '{0}'".format(dmx_string)
    os.system(command)
```

Thanks to my abstraction of the connection to Firebase this code it pretty simple to comprehend. Now let's have a look at the `firebase.py` module:

```python
#!/usr/bin/env python3
"""firebase backend for lamp"""

import configparser
from threading import Thread
import sys
import os
import time
import copy
import pyrebase

HEARTBEAT_INTERVAL = 5 # seconds

class Database(object):
    """Encapsulation of the firebase connection in an object"""

    def __init__(self, config_file_path):
        """
        Create the database object
        param config_file_path: path to the desired config file
        """

        if not config_file_path or not os.path.exists(config_file_path):
            raise FileNotFoundError("Config File {0} Not Found".format(config_file_path))

        self.database = None
        self.user = None
        self.stream = None
        self.stop_heartbeat = False
        self.color = {}

        self.config = configparser.ConfigParser()
        self.config.optionxform = str
        self.config_file_path = config_file_path
        self.config.read_file(open(self.config_file_path))


    def initialise(self):
        """
        Initialise the connection to firebase, (authenticate), spawn a heartbeat thread
        """

        print('Initialise Backend')
        sys.stdout.flush()

        # Get the firebase configuration and init a connection
        self.database = pyrebase.initialize_app(self.get_firebase_config())

        auth = self.database.auth()
        self.user = auth.sign_in_with_email_and_password(*self.get_credentials())

        print("Authenticated")
        sys.stdout.flush()

        # Spawn a thread to issue heartbeat to the firebase
        self.spawn_heartbeat()

        print('Spawned Heartbeat')
        sys.stdout.flush()


    def get_firebase_config(self):
        """Returns the firebase_config object"""

        firebase_config = {}
        for key, value in self.config.items('firebase-config'):
            firebase_config[key] = value
        return firebase_config


    def get_credentials(self):
        """Returns the email and password from the firebase-credentials in config, as a tuple"""

        email = self.config.get('firebase-credentials', 'email')
        password = self.config.get('firebase-credentials', 'password')

        return email, password


    def get_online_leaf(self):
        """
        Return a reference to the leaf in the database,
        that is the online time/status of this node
        """
        return (
            self.database.database()
            .child('nodes')
            .child(self.user['localId'])
            .child('online')
        )


    def spawn_heartbeat(self):
        """Spawn and return heartbeat thread"""

        hb_thread = Thread(target=self.heartbeat_task, args=(self.get_online_leaf(),))
        hb_thread.start()

        return hb_thread


    def heartbeat_task(self, node_to_set):
        """Heartbeat thread, update this node's timestamp in the firebase at HEARTBEAT_INTERVAL"""
        while not self.stop_heartbeat:
            beat_response = node_to_set.set({".sv": "timestamp"})
            # TODO, check the response for errors
            time.sleep(HEARTBEAT_INTERVAL)

        self.stop_heartbeat = False


    def register_callback(self, callback):
        """
        Register a callback function to be called when the node color changes
        The callback will be called with a Color object with RGB properties
        """

        self.color = {
            "red": 0,
            "green": 0,
            "blue": 0
        }

        def stream_handler(message):
            """Handler method for the stream response from firebase"""

            new_color = copy.deepcopy(self.color)

            if message['path'] == '/':
                new_color['red'] = message['data']['red']
                new_color['green'] = message['data']['green']
                new_color['blue'] = message['data']['blue']

            elif message['path'] == '/red':
                new_color['red'] = message['data']

            elif message['path'] == '/green':
                new_color['green'] = message['data']

            elif message['path'] == '/blue':
                new_color['blue'] = message['data']
            else:
                print('No Valid Colors')
                print(message)
                return False

            self.color = new_color
            callback(self.color)
            return True

        print('Color change callback registered')
        sys.stdout.flush()

        # FIXME: Move getting the color node out to a different method
        self.stream = (
            self.database.database().child('nodes')
            .child(self.user['localId'])
            .child('color').stream(stream_handler)
        )

        print("Stream started, you're good to go!")
        sys.stdout.flush()
```

Overall this is pretty simple, the only two methods we really care about are `initialise()` and `register_callback()`, with `spawn_heartbeat()` also being interesting. The other methods are really just helpers and make unit testing easier.

The config file `firebase.cfg` looks like this:

```ini
[firebase-config]
apiKey=AIzaSyCG2-TArRw69Pkk4Pot1LY5ThFd6f0axI4
authDomain=spectrum.firebaseapp.com
databaseURL=https://spectrum.firebaseio.com
storageBucket=sudo-spectrum.appspot.com
messagingSenderId=521515407409

[firebase-credentials]
email=hostname@example.com
password=password
```

Hopefully it's pretty self explanatory what they do, but in case you're a bit lost, the major bits are:

### `initialise()`:
- Calls pyrebase to create the connection using the firebase API values we provide in the `firebase.cfg` file.
- Signs in with the user account we created earlier (we added the credentials to the config file)
- Spawns the heartbeat thread that updates the `online` key on firebase

### `register_callback()`:
- Accepts a function to be called with the color (a dictionary) when it changes
- Starts a stream from `/nodes/NODE_ID/color`
- When a change is received, it updates the old colour with whatever the changed value(s) were, and calls the callback function

### `spawn_heartbeat()`:
- Creates a new thread that calls `heartbeat_task()`, which updates the `online` key of the node entry in firebase every 5 seconds

I wrap these two files in a package called `lamp` and run it with a file `run-lamp.py` containing:

```python
#!/usr/bin/env python3
print('Starting. Sit tight, Importing dependencies may take a while...')
import lamp
lamp.main()
```

I've got that big print statement in there because it takes a while for Pyrebase to be imported fully and it looks like the program is hung for a few seconds.


## Twitter bot
For the Twitter bot side of things, there's two more libraries beside Pyrebase we'll need:
- [tweepy]() A twitter API library
- [webcolors](http://webcolors.readthedocs.io/) A library for dealing with CSS colours. It's an easy way to convert 147 colour names (CSS3 colours) to their RGB values

In the longer term I'll write a nice python module backend for my projects using spectrum, so they can just `import spectrum` and go with it, but for this one we're still going to be pretty tightly coupled.

All the twitter bot does is wait for new tweets on a hashtag, finds the colour word in the tweet, and stored the corresponding RGB colour in firebase.



There's a lot that could be done to make this more robust, but it's gone well for a small project that got some solid wow-factor.
