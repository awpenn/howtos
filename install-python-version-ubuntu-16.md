# Setting up different-than-default python on ubuntu 16 (ie. VM)
- following this: https://serverfault.com/a/919064

# Install requirements
sudo apt-get install -y build-essential \
checkinstall \
libreadline-gplv2-dev \
libncursesw5-dev \
libssl-dev \
libsqlite3-dev \
tk-dev \
libgdbm-dev \
libc6-dev \
libbz2-dev \
zlib1g-dev \
openssl \
libffi-dev \
python3-dev \
python3-setuptools \
wget

# Prepare to build
mkdir /tmp/Python37
mkdir /tmp/Python37/Python-3.7.10
cd /tmp/Python37/

# Pull down Python 3.7.10, build, and install
wget https://www.python.org/ftp/python/3.7.10/Python-3.7.10.tar.xz
tar xvf Python-3.7.10.tar.xz -C /tmp/Python37
cd /tmp/Python37/Python-3.7.10/
./configure --enable-optimizations
sudo make altinstall

- now `python3.7 -version` returns correct response
- can now use `python3.7 -m venv .venv` to create venv and use regular `python` command to use 3.7


