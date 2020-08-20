# MyRocks with Key-value Tracer

Tested on Ubuntu 16.04.7 LTS

## 1. Setup

### 1.1 install required development tools

Make sure the required development tools are installed:

```sh 
sudo apt-get update 
sudo apt-get -y install g++ cmake libbz2-dev libaio-dev bison \
zlib1g-dev libsnappy-dev libboost-all-dev
sudo apt-get -y install libgflags-dev libreadline6-dev libncurses5-dev \
libssl-dev liblz4-dev gdb git libzstd-dev libzstd0 libcap-dev
```

In MyRocks repository, update submodules.

```sh
git clone https://github.com/facebook/mysql-5.6.git
cd mysql-5.6
git submodule init
git submodule update
```

### 1.2 build & install MyRocks

Run cmake 

```sh
make . -DCMAKE_BUILD_TYPE=RelWithDebInfo -DWITH_SSL=system \
-DWITH_ZLIB=bundled -DMYSQL_MAINTAINER_MODE=0 -DENABLED_LOCAL_INFILE=1 \
-DENABLE_DTRACE=0 -DCMAKE_CXX_FLAGS="-march=native" \
-DWITH_ZSTD=/usr
```

Then compile

```sh
make -j8
```

After compile is done, this will install MyRocks in `/usr/local/mysql`

```sh
sudo make install
```


### 1.3 Initialize & Run MyRocks

Go to `/usr/local/mysql` and initialize MyRocks with following commands.

```sh
sudo ./scripts/mysql_install_db
```

Then, Run MyRocks with RocksDB Plugin.

```sh
sudo ./bin/mysqld --user=root --skip-innodb --rocksdb --default-storage-engine=rocksdb --default-tmp-storage-engine=MyISAM
```

## 2. Check & Trace

During mysqld loads RocksDB plugin, it will shows

```
NewFileTraceWriter Status: 0, 0
StartTrace Status: 0, 0
```

Each number means (code, subcode).
If numbers are 0, it means tracing running successfully.
If not, there is an error while starting tracing.


To check, RocksDB loaded successfully, run mysql client.

```sh
./bin/mysql -u root
```

And check default storage engine is RocksDB.

```SQL
SHOW ENGINES;
```

Then do your benchmark and shutoff mysqld.

When mysqld unloads RocksDB plugin, following output have to be printed.

```
EndTrace Status: 0, 0
```

Now, you can use the trace file `/usr/local/mysql/trace_test` to analyze it.
