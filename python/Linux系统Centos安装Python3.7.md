```shell
wget https://www.python.org/ftp/python/3.7.3/Python-3.7.3.tgz

tar -xvzf Python-3.7.3.tgz

cd Python-3.7.3

./configure --prefix=/usr/python

make && make install

ln -s /usr/local/Python-3.7.3 /usr/bin/python

```

