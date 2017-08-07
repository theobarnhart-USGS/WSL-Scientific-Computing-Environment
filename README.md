# wsl data science setup

Installing scientific computing software on a USGS computer can be difficult, but if you are on Windows 10 you have access to the Windows Subsystem for Linux (WSL), which is essentially an Unbuntu linux installation running on your PC. As WSL is essentially a virtualized environment within your Windows system you have root access to the WSL. This makes it a great place to put your scientific software. Another benefit of working via WSL is that your scripts will run on Windows via WSL, on OS X and on Linux machines and clusters. 

In this guide I provide what worked for me while setting up my computing environment on WSL.

## First, install the WSL environment
```
## in cmd
lxrun /uninstall /full # remove the old WSL environment
lxrun /install # install WSL
lxrun /update # update WSL
bash # launch WSL ubuntu
sudo apt-get update # update ubuntu
```

Now you have a working version of Ubuntu on your Windows PC!

## Get a working Python installation and the conda package/environment manager
```
# download miniconda
wget --no-check-certificate https://repo.continuum.io/miniconda/Miniconda3-latest-Linux-x86_64.sh

# install miniconda
bash ./Miniconda3-latest-Linux-x86_64.sh

# fix the ssl usgs issue:
conda config --set ssl_verify False

# fix mkl error
echo "export KMP_AFFINITY=disabled" >> ~/.bashrc

# fix matplotlib
sudo apt-get install libqtgui4
```

## Create Python and R Environments

Python has been migrating to version 3 so better to start here and also install a version 2 environment in case someting doesn't work with verison 3. We'll also install a R environment to greater flexibility.

### Python 3.6

```
conda create -n py36
source activate py36
conda config --add channels conda-forge
conda install gdal python=3.6
conda install jupyter
conda install matplotlib pandas statsmodels basemap
conda install -c jzuhone zeromq=4.1.dev0
```

### Python 2.7

```
conda create -n py27
source activate py27
conda config --add channels conda-forge
conda install gdal python=2.7 matplotlib pandas statsmodels basemap
conda install ipykernel
python -m ipykernel install --user
conda install -c jzuhone zeromq=4.1.dev0
```

### R
```
conda create -n Renv
source activate Renv
conda install -c r r-essentials
R # then
IRkernel::installspec()
quit()

# kernel shows up in Jupyter but is not stable
# reinstalling the zmq fix in the Renv seems to fix this:
conda install -c jzuhone zeromq=4.1.dev0
```

## Other Peculiarities

### Google Earth Engine Installation
```
source activate py27
pip install --index-url=http://pypi.python.org/simple/ --trusted-host pypi.python.org google-api-python-client

# check cryptography, hopefully no errors
python -c "from oauth2client import crypt"

pip install --index-url=http://pypi.python.org/simple/ --trusted-host pypi.python.org earthengine-api

# initialize EE
earthengine authenticate
# this brings up w3m, q out of w3m and copy and paste the URL into chrome, sign in and input the authorization token into the WSL command prompt.

# try 
python -c "import ee; ee.Initialize()"
#this should fail b/c of the usgs network
```
#### The Fix
Replace line 721 in ~/miniconda3/envs/py27/lib/python2.7/site-packages/ee/data.py 
```
http = httplib2.Http(timeout=(_deadline_ms / 1000.0) or None)"
```
with

```http = httplib2.Http(timeout=(_deadline_ms / 1000.0) or None,disable_ssl_certificate_validation=True)```

Save the file and try again
```python -c "import ee; ee.Initialize()"```
EE should work now.

### Getting CMDER working with arrows in vim

The arrow keys do not work when vim is opened in CMDER. Searched the net for a fix and found the tweak below.

- Navigate to: settings > startup > tasks
- Add a new predifined tast called {win_bash}
 - Under task parameters: /icon "%CMDER_ROOT%\icons\cmder.ico"
- In the large box below: cmd /k "%windir%\system32\bash.exe" -cur_console:p

This trick seems to work and make CMDER function more normally. 
