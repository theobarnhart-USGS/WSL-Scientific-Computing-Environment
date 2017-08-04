# wsl data science setup guide

# optional #
# setup cmder # 
# http://cmder.net/

## in cmd
lxrun /uninstall /full # remove the old WSL environment
lxrun /install # install WSL
lxrun /update # update WSL

bash # launch WSL ubuntu
sudo apt-get update # update ubuntu

# check ubuntu version
lsb_release -a

# download miniconda
wget --no-check-certificate https://repo.continuum.io/miniconda/Miniconda3-latest-Linux-x86_64.sh

# install miniconda
bash ./Miniconda3-latest-Linux-x86_64.sh

# fix the ssl usgs issue:
conda config --set ssl_verify False

# fix mkl error
echo "export KMP_AFFINITY=disabled" >> ~/.bashrc

conda install python==3.5
conda install jupyter
conda install -c jzuhone zeromq=4.1.dev0

# fix matplotlib
sudo apt-get install libqtgui4
conda install matplotlib=1.5.1

# this generates a working python 3.5 environment!

## install other packages
conda install pandas
conda install basemap
# conda install -c conda-forge gdal # doesn't work with WSL 

########################################################################################
########################################################################################
########################################################################################
########################################################################################
# Goal: create a scientific computing environment on the WSL
# Advantages: If you can work around the qwerky environment, there is a lot more freedom in the WSL environment b/c its essentially a virtual machine where you have root access that can interface with the windows system that you need to run word, etc. If this works, it will be more like working on a mac where you have access to all the standard software and and unix backend access to the file structure to work efficiently. 
# setup three environments py36, py27, and Renv
# it appears that much of this works because of conda forge, setting up new environments that use conda-forge is the key and trying to not mix and match too many channels keeps everything working.

# the main test of this interface is to load gdal correctly and plot something in matplotlib.
# those tools and pandas are my primary packages so thats what I want to be able to use. Ideally all the kernels for the environments show up in Jupyter when launched from the py36 environment. 

### ****untested portion*********
## in cmd
lxrun /uninstall /full # remove the old WSL environment
lxrun /install # install WSL
lxrun /update # update WSL

bash # launch WSL ubuntu
sudo apt-get update # update ubuntu

# check ubuntu version
lsb_release -a

# download miniconda
wget --no-check-certificate https://repo.continuum.io/miniconda/Miniconda3-latest-Linux-x86_64.sh

# install miniconda
bash ./Miniconda3-latest-Linux-x86_64.sh

# fix the ssl usgs issue:
conda config --set ssl_verify False

# fix mkl error
echo "export KMP_AFFINITY=disabled" >> ~/.bashrc

################# end untested portion

# try this all with a python 3.6 environment....
conda create -n py36
source activate py36
conda config --add channels conda-forge
conda install gdal python=3.6
conda install jupyter
conda install matplotlib pandas statsmodels basemap

## the above py36 environment and jupyter seem to work!

# python 2.7 environment
conda create -n py27
source activate py27
conda config --add channels conda-forge
conda install gdal python=2.7 matplotlib pandas statsmodels basemap
conda install ipykernel
python -m ipykernel install --user

## the above py2.7 environment seems to work as well!

# install google earth engine (in py27)
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
replace line 721 in ~/miniconda3/envs/py27/lib/python2.7/site-packages/ee/data.py "http = httplib2.Http(timeout=(_deadline_ms / 1000.0) or None)" with
"http = httplib2.Http(timeout=(_deadline_ms / 1000.0) or None,disable_ssl_certificate_validation=True)"
# save and try again
python -c "import ee; ee.Initialize()"
# should work now

###################
# build and R environment with Conda
conda create -n Renv
source activate Renv
conda install -c r r-essentials
R # then
IRkernel::installspec()
quit()

# kernel shows up in Jupyter but is not stable
# reinstalling the zmq fix in the Renv seems to fix this:
conda install -c jzuhone zeromq=4.1.dev0

# get CMDER working with arrows in vim
# settings > startup > tasks
# add a new predifined tast called {win_bash}
# under task parameters: /icon "%CMDER_ROOT%\icons\cmder.ico"
# in the large box below: cmd /k "%windir%\system32\bash.exe" -cur_console:p
# this seems to work
