# How to use YOLOv7 on Jetson Nano

Here is most up-to-date tutorial on how to run YOLOv7 model on Jetson Nano. Befeore starting with this tutorial, make sure to install latest Nvidia Developer Kit SDK. You can follow official tutorial from Nvidia at: https://developer.nvidia.com/embedded/learn/get-started-jetson-nano-devkit#intro. Follow the steps to section Next Steps and you will be ready to start with this tutorial.

## Jetson Nano Setup

First create a folder for the YOLO project and clone yolov7 repo:

```bash
mkdir yolo
cd yolo
git clone https://github.com/WongKinYiu/yolov7
```

Then use virtual environment to install most of the required python packages inside. We will use virtual environment so we can for example always choose another PyTorch package version.
You have to use SUDO. Note: we can't install OpenCV as a local package (more on that later).

```bash
sudo pip3 install virtualenv virtualenvwrapper
```

Now we installed virtualenvironment. Next change .bashrc file using nano editor (we have to install it first):

```bash
sudo apt update
sudo apt install nano
nano ~/.bashrc
```

Add this to the end of .bashrc file (under all the lines already there):

```bash
# virtualenv and virtualenvwrapper
export WORKON_HOME=$HOME/.virtualenvs
export VIRTUALENVWRAPPER_PYTHON=/usr/bin/python3
source /usr/local/bin/virtualenvwrapper.sh
```

To activate the changes the following command must be executed:

```bash
source ~/.bashrc
```

Now it's time to create our virtual environment, where most of our Python packages will be stored.

To create a virtual environment run:

```bash
mkvirtualenv yolov7 -p python3 # yolov7 is the name of our virtual environment. You can use any other name
```

If you want to activate environment just type:

```bash
workon yolov7 # yolov7 is the name of our virtual environment
```

If you want to deactivate virtual environment type:

```bash
deactivate
```

Because OpenCV has to be installed system wide (comes preinstalled with Nvidia developer pack Ubuntu 18.04), we have to create a symbolic link from global to our virtual environment. Otherwise we won't be able to access it from our virtual environment.

```bash
cd ~/.virtualenvs/xxxx/lib/python3.6/site-packages/

ln -s /usr/lib/python3.6/dist-packages/cv2/python-3.6/cv2.cpython-36m-aarch64-linux-gnu.so cv2.so
```

Note: xxxx is your environment folder name (in our case it's yolov7). EXAMPLE:

```bash
cd ~/.virtualenvs/yolov7/lib/python3.6/site-packages/

ln -s /usr/lib/python3.6/dist-packages/cv2/python-3.6/cv2.cpython-36m-aarch64-linux-gnu.so cv2.so
```

Move back to the folder you created in the beginning (yolo in our example).

We have to install all required packages. But here comes a problem I've encountered. I couldn't install matplotlib library just using pip install matplotlib. It will take us a few more steps to install matplotlib on Jetson Nano. First enter these two commands:

```bash
sudo apt install libfreetype6-dev
sudo apt-get install python3-dev
```

I found that it's best to install numpy and matplotlib first:

```bash
pip3 install --upgrade pip setuptools wheel
pip3 install numpy==1.19.4
pip3 install matplotlib
```

Now move to yolov7 repository you cloned at the beginning (if you are not already there).

We have to comment out two packages that we will install manually (PyToch and TorchVision). Change requirements.txt using nano editor:

```bash
nano requirements.txt
```

Comment out these two lines with # before package name:

```nano
# torch>=1.7.0,!=1.12.0
# torchvision>=0.8.1,!=0.13.0
```

Save and exit (ctr+s, y, enter).

Install all required packages from requirements.txt. Make sure you are in a yolov7 cloned repository where requirements.txt file is located.

```bash
pip install -r requirements.txt
```

### Install PyTorch and TorchVision

We are ready to install PyTorch. I've tested YOLOv7 algorithm with PyTorch 1.8 and 1.9 and both worked fine. We will install PyTorch version 1.8 in this tutorial, because it's officialy provided by Nvidia.

Use these commands from official Nvidia tutorial (reference: https://forums.developer.nvidia.com/t/pytorch-for-jetson/72048).

```bash
pip3 install -U future psutil dataclasses typing-extensions pyyaml tqdm seaborn
pip3 install Cython

wget https://nvidia.box.com/shared/static/p57jwntv436lfrd78inwl7iml6p13fzh.whl -O torch-1.8.0-cp36-cp36m-linux_aarch64.whl

pip3 install torch-1.8.0-cp36-cp36m-linux_aarch64.whl
```

If you get error:

```bash
OSError: libmpi_cxx.so.20: cannot open shared object file: No such file or directory
```

Run this command and try again:

```bash
sudo apt-get install libopenblas-base libopenmpi-dev
```

To check if everything is installed correctly run:

```bash
python -c "import torch; print(torch.__version__)"
```

You should see the result:

```bash
1.8.0
```

Now the tricky part. Because there is no pre-built wheel for torchvision for Jetson Nano, we have to clone it from GitHub. Then we build the right version. After this we install it in our virtual environment.

TorchVision version needs to be compatible with PyTorch. We used PyTorch version 1.8.0, so we will choose TorchVision version 0.9.0.

```bash
sudo apt install libjpeg-dev zlib1g-dev libpython3-dev libavcodec-dev libavformat-dev libswscale-dev
pip3 install --upgrade pillow
git clone --branch v0.9.0 https://github.com/pytorch/vision torchvision
cd torchvision
export BUILD_VERSION=0.9.0
python3 setup.py bdist_wheel # Build a wheel. This will take a couple of minutes.
cd dist/
pip3 install torchvision-0.9.0-cp36-cp36m-linux_aarch64.whl
cd ..
cd ..
sudo rm -r torchvision # Now remove the branch you cloned
```

To test if TorchVision is installed successfuly type:

```bash
python -c "import torchvision; print(torchvision.__version__)"
```

You should see the result:

```bash
0.9.0
```

### Try YOLOv7

Congrats, we are almost ready. Before we run YOLOv7 on Jetson Nano for the first time, we have to download trained weights frst. We can choose between normal and tiny version. We used tiny version for this tutorial, because it's optimized for edge devices like Nano.

Then download trained weights (choose between tiny or normal) into yolov7 folder:

```bash
# Download tiny weights
wget https://github.com/WongKinYiu/yolov7/releases/download/v0.1/yolov7-tiny.pt

# Download regular weights
wget https://github.com/WongKinYiu/yolov7/releases/download/v0.1/yolov7.pt
```

Now we are ready to run YOLOv7 algorithm for the first time.

```bash
python3 detect.py --weights yolov7-tiny.pt --conf 0.25 --img-size 640 --source inference/images/horses.jpg
```

If there is no error, YOLO should detect objects in cca. 30 second. You can open the folder runs/detect/ and choose your latest experiment folder (for example: exp3). There will be an image with bounding boxes around recognized objets.

Congrats! Now you are ready to build awesome stuff with Jetson Nano and YOLOv7 algorithm.
