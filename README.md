# Welcome to DonkeyCar on balena!  

A [DonkeyCar](https://www.donkeycar.com) is a small autonomous vehicle project built on an RC Car chassis, that uses computer vision and machine learning techniques to convert the platform into a self-driving vehicle.  It uses a Raspberry Pi 3 or Jetson Nano, a camera, a motor driver, and of course the DonkeyCar application to perform this task.  There is also a "virtual" version that you can use to get started, prior to even building the physical version.  A "virtual" DonkeyCar looks rather like a video game, and it connects to a DonkeyCar Simulator.  Just like the physical version, the virtual Donkey can attempt to drive itself, in the virtual world.

![](https://docs.donkeycar.com/assets/sim_screen_shot.png)

To cover these different scenarios there are 4 main Repositories for the project:

 - [DonkeyCar - Physical Build - Raspberry Pi](https://github.com/dtischler/balena-DonkeyCar-Physical)
 - [DonkeyCar - Physical Build - UpBoard](https://github.com/dtischler/balena-DonkeyCar-Physical-UpBoard)
 - [DonkeyCar - Virtual Build, Raspberry Pi](https://github.com/dtischler/balena-DonkeyCar-Virtual) 
 - [DonkeyCar Simulator - Intel NUC / x86](https://github.com/dtischler/balena-DonkeyCar-Simulator)

This Readme and Repo cover the **`DonkeyCar Simulator`** version of the project.  You can refer to the other GitHub repos linked above, for other components or flavors of DonkeyCar.

Also note, the DonkeyCar project has their own [detailed documentation available](https://docs.donkeycar.com), that goes into more advanced topics and gives thorough descriptions of the architecture, machine learning and training scenarios, performance and tuning of models, and many more topics.  This Readme is simply intended to make it easy to get started, and once you have the core concepts down, be sure to refer to their documentation for more advanced guidance. 


## Intro

Waymo, Tesla, Cruise, and other companies already have self-driving vehicles deployed out in the world around us.  With this project, it is possible to build a miniature version of an autonomous vehicle, intended to race around a track, which is perfect for learning how the core concepts of computer vision and machine learning play a role in self-driving vehicles!  Keep in mind this Repo is specifically devoted to setting up the DonkeyCar Simulator, which will create a virtual world that can be used by virtual DonkeyCars to drive, train, and race in.  This repo does not involve any physical construction and does **NOT** require an RC Car Chassis.  This repo requires an x86-based device like a spare PC or laptop, that you will use to install the Simulator on.  If you are looking to build a "real" DonkeyCar, simply check out [this repo](https://github.com/dtischler/balena-DonkeyCar-Physical) to get started.

Before diving in, let's cover a few basics.  A DonkeyCar (whether real or virtual) can be driven through a web browser, using the keyboard, a Bluetooth gamepad, or even via the accelerometer on your cell phone.  While driving, the DonkeyCar will record data about the car's throttle position, steering position, and what it "sees" in the camera.  When you are finished recording, all of the data is collected into a folder called a "Tub".  This metadata is then used as the input to create or "train" an AI model.  Once complete, the output of that process (the resulting *model* file) can then be used by the DonkeyCar to attempt to drive itself.  The Raspberry Pi loads the model, and attempts to navigate and move around the track on its own!

The process in the middle - the Training - takes a very long time on a Raspberry Pi or even an x86 device like we will use for this Simulator.  Analyzing all of the data that got recorded during driving, creating a neural network, and iterating on that data over and over through what are called epochs can easily take hours on the Pi.  And once complete, there is no guarantee the DonkeyCar will even be able to drive safely and accurately!  So, to improve that feedback loop and to reduce the iteration time, most people find it better to drive the DonkeyCar and collect the data, but then transfer that resulting "Tub" of data to a cloud server or a PC with a GPU, and perform the training there.  Once it completes (hopefully much quicker!), the resulting output of the process is the *model* file just like mentioned above, and that model can be transferred back to the Pi.  Then, the DonkeyCar can try to drive using it (hopefully it drives good!).


## Build

There is no physical construction of a DonkeyCar in this Repo, so we can move right into deploying the software stack onto our target device.  As mentioned, this repo requires a spare PC, laptop, Intel NUC, or similar **[WARNING:  WE ARE GOING TO OVERWRITE IT'S DRIVE.  BACKUP ANY DATA YOU NEED]**.  Make you sure whatever your device is, that it can boot from a USB stick.

We are going to diverge a bit from the official DonkeyCar Simulator install workflow, which normally walks you through installation of the simulator software on an Ubuntu linux machine.  However, this repo has been altered to the balena workflow and has bundled all of those bits into a Dockerfile, so rather than performing all of those manual installation steps you can instead just click this button to launch a build in the cloud, provision a balena device, download a USB stick image, flash your device, and end up with the same result:

[![balena deploy button](https://www.balena.io/deploy.svg)](https://dashboard.balena-cloud.com/deploy?repoUrl=https://github.com/dtischler/balena-DonkeyCar-Simulator)

More specifically:
1. Click on the Blue Button just above.
2. Log in to balenaCloud, or create an account if you do not already have one.  (It is free :-) )
3. Create a name for the application in the pop-up modal, and choose "Generic X86-64" from the drop down list (Or Intel NUC if you are using a NUC).
4. Click Create and deploy.
5. Click Add Device.
6. You will come to the Summary page.  Here, click "Add Device".
7. While developing, it is probably best to choose a Development variant of the operating system, and you can enter your WiFi credentials as well.
8. At the bottom of the modal, click "Download balenaOS".
9. After download completes, flash the file to a USB stick with Etcher.
10. Hook up a monitor, keyboard, and mouse to your device.  (Unnecessary on a laptop, of course)

**[WARNING:  WE ARE GOING TO OVERWRITE YOUR DEVICE.  BACKUP ANY DATA YOU NEED FIRST]**

11. Insert the USB stick into your device, power on, and if necessary choose to boot from USB.  The PC will boot up, copy the contents of the USB drive to the device's internal storage, then power off.
12. Remove the USB stick, and power the device back on.  Wait a few minutes for it to register itself with balenaCloud and come online.
13. After another moment, the PC or laptop will begin downloading the pre-built DonkeyCar Simulator container, which will take some time.  Get a cup of tea while this occurs.

Once the device has finished downloading the container, we are ready to launch our virtual world, so it is time to move on to the next section!


## Simulator

Once the container is running on your device, it should launch the simulator, which will be sitting at the main menu:

![](/images/img1.png)

Click on a track, in this case, Mountain Track is selected:

![](/images/img2.png)

This is great, but, there are no cars driving around yet.  Time to move on to the next section.


## Drive

### Option 1, the Easy Way

With your Simulator up and running, it's time to go for a virtual drive!  In balenaCloud, on the Device Details page, open up an SSH session to the DonkeyCar container with the Terminal interface at the bottom right portion of the screen.  

Type in `cd mysim && python3 manage.py drive` and press Enter. The script will launch, and take a moment to complete, but will eventually reach `Starting vehicle at 20 Hz`. 

![](/images/img3.png)

Check the IP address of your device in the balenaCloud dashboard. Make note of this IP. Open a web browser, and go to `http://ip-address-of-your-device/drive`. In my example, this would be `http://192.168.0.196/drive`.

![](/images/img4.png)

In the throttle and steering applet on that page, click and drag just a tiny bit up from center, and your virtual car should start to move!  Alternatively, you can use the `I, K, J, and L` keys on the keyboard to nudge the car forward, backward, left, and right.  Remember, this is the exact same software stack as the real DonkeyCar, so if you had built a physical racer, it would start driving as well.  As mentioned earlier, the official DonkeyCar documentation is very thorough, so you can refer to those Docs for more detailed usage. But keep in mind these important details:

- DonkeyCar is meant to be used on a racetrack, and will attempt to learn your driving technique.
- Using the web portal for driving (use a Bluetooth controller for a much better experience than driving via keyboard or phone), drive a few practice laps around the track. When you have a good feel for driving and are ready to capture data, click the "Start Recording" button, and at this point DonkeyCar will begin storing the throttle, steering, and camera feed for later use in the Training process.
- Drive at least 10 laps around the track, preferably more, while Recording.
- Once you have completeled 10 laps (or more), you can stop the Recording.
- Over in the balenaCloud Dashboard, in that Terminal window, press `Control-C` on the keyboard to exit out of the DonkeyCar application.
- You will see all of the data get written out in a table, and the raw files are stored in the `data` directory inside of that `mycar` folder.

![](/images/img5.png)

![](/images/img7.png)

### Option 2, The Extra Device Way

Because this DonkeyCar Simulator is now serving up connections, you can actually build a secondary device using a Raspberry Pi as is discussed in this repo:  [https://github.com/dtischler/balena-DonkeyCar-Virtual](https://github.com/dtischler/balena-DonkeyCar-Virtual)

If you follow the steps in that repo, you end up with a virtual DonkeyCar that can connect to your newly provisioned Donkey Server.  Of course, other users can also do the same, and your Server can be used to host races, allow external users to drive and practice, capture data for training, etc.  Your DonkeyCar Simulation Server could be a valuable resource for your Autonomous Vehicle club, Robotics club, or more!

![](/images/img6.png)

![](/images/img8.png)


## Train

Back in balenaCloud and inside the terminal session we opened, now that we have a bit of sample data recorded and saved, it's time to begin training our model. Remember, as mentioned above, it is NOT very efficient to train directly on a Raspberry Pi or x86 CPU, and using a cloud server or a desktop PC with a GPU will be MUCH faster. However, simply for learning purposes and to keep things organized and in one place, we will in this situation train directly on the device. It could literally take 8 to 10 hours or more, so, grab a cup of tea, and sip it VERY slowly. Or do something else in the meantime.

In the terminal session in balenaCloud, and still within the DonkeyCar container, run `donkey train --tub ./data --model ./models/myawesomepilot.h5`. Now go do something else.

Fast forward 10 or so hours, and returning to balenaCloud, you should see that process has completed. The output of all that hard work is the model file. Double check that everything completed successfully, and you should have a file sitting in the `models` directory called `myawesomepilot.h5`

### Speeding Up Training

Knowing full well that training on this device is not ideal, we simply want to demonstrate functionality in this GitHub repo. If you are interested in offloading the Tub data and training on a GPU or Cloud server, have a look at the official DonkeyCar docs here: https://docs.donkeycar.com/guide/train_autopilot/#transfer-data-from-your-car-to-your-computer. That will help immensely. :-)


## Drive Autonomously

With the model now ready (hope you slept well), you can try to let the DonkeyCar now navigate the virtual racetrack autonomously. Fair warning, mine did not drive very well with only 10 laps of data, so, be ready to watch it crash!

- In balenaCloud dashboard, open up a terminal session to the DonkeyCar container.
- Enter `cd mysim && python3 manage.py drive --model models/myawesomepilot.h5`
- Navigate to `http://ip-address-of-your-device/drive` once again.
- On the left, click the dropdown menu for Mode and Pilot, and choose Local Pilot.
- The DonkeyCar should begin to make it's way around the track (hopefully).


## Conclusions

This repo is intended to demonstrate the containerization of the DonkeyCar Simulator, and help you to quickly deploy the software stack that they are gracious enough to build and maintain. Also included is the virtual DonkeyCar application, which is a virtual clone using the exact same software that the physical one would use.  The training process on the device is not ideal, but there are methods to help accelerate the process by offloading the data.  As you get better at driving and training your model, be sure to also check out and compete in the online races the community hosts: [https://www.meetup.com/DIYRobocars](https://www.meetup.com/DIYRobocars)
