.. Trollcast documentation master file, created by
   sphinx-quickstart on Wed Nov 14 12:57:53 2012.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Welcome to Trollcast's documentation!
=====================================

To the `source code page <http://github.com/mraspaud/trollcast>`_.

Trollcast is a tool to exchange polar weather satellite data. It aims at
providing near real time data transfer between peers, and should be adaptable
to any type of data that is scan-based. At the moments it works on 16-bits hrpt
minor frame data (both big and little endian).

The protocol it uses is loosely based on bittorrent.

.. warning::
  This is experimental software, use it at your own risk!

.. Contents:

  .. toctree::
     :maxdepth: 2

Installing trollcast
--------------------

Download trollcast from the `source code page
<http://github.com/mraspaud/trollcast>`_ and run::

  python setup.py install

Setting up trollcast
--------------------

A trollcast config file describes the different parameters one needs for
running both the client and the server.

.. code-block:: ini
  
  [local_reception]
  localhost=nimbus
  remotehosts=safe
  data=hrpt
  data_dir=/data/hrpt
  file_pattern={utctime:%Y%m%d%H%M%S}_{platform:4s}_{number:2s}.temp
  max_connections=2
  station=norrköping
  coordinates=16.148649 58.581844 0.02
  tle_files=/var/opt/2met/data/polar/orbitalelements/*.tle
  schedule_file=/var/opt/2met/data/polar/schedule/schedule.txt
  schedule_format=scisys
  mirror=my_receiver_server
  output_file=/tmp/{utctime:%Y%m%d%H%M%S}_{platform:4s}_{number:2s}.trollcast.hmf
  publisher=trollcast_receiver


  [safe]
  hostname=172.29.0.236
  pubport=9333
  reqport=9332
  
  [nimbus]
  hostname=172.22.8.16
  pubport=9333
  reqport=9332

The `local_reception` section
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

 - `localhost` defines the name of the host the process is going to run on
   locally. This name will be user further down in the configuration file as a
   section which will hold information about the host. More on this later.
 - `remotehosts` is the list of remote hosts to communicate with.
 - `data` give the type of data to be exchanged. Only *hrpt* is available at
   the moment.
 - `data_dir` is the place where streaming data from the reception station is
   written. 
 - `file_pattern` is the pattern (trollsift syntax) to use to detect the file
   that the reception station writes to. Trollcast will watch this file to
   stream the data to the network in real time.
 - `max_connections` tells how many times the data can be sent. This is usefull
   for avoiding too many clients retrieving the data from the same server,
   putting unnecessary load on it. Instead, clients will spread the data among
   each other, creating a more distributed load.
 - `station`: name of the station
 - `coordinates`: coordinates of the station. Used for the computation of
   satellite elevation. Lon/lats in degrees, altitude in kilometers.
 - `tle_dir`: directory holding the latest TLE data. Used for the computation
   of satellite elevation.
 - `schedule_file`: schedule file to give trollcast server the knowledge of
   passes to come.
 - `schedule_format`: the format of the schedule file. Supported at the moment:
   `scisys` and `kongsberg_metno`.
 - `mirror`: the hostname or ip address of the trollcast server the current
   server has to mirror.
 - `output_file`: the file to write data to, in trollsift syntax.
 - `publisher`: the name under which to publish new incoming files. If you don't
   want to publish anything (which is usually the case when you don't have a
   posttroll based trigger to handle messages), just omit this option.

The host sections
~~~~~~~~~~~~~~~~~
   
 - `hostname` is the hostname or the ip address of the host.
 - `pubport` on which publishing of messages will occur.
 - `reqport` on which request and transfer of data will occur.

Modes of operation
------------------

Server mode, giving out data to the world
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The server mode is used to serve data to remote hosts.

It is started with::

   trollcast_server my_config_file.cfg

This will start a server that watches a given file, as specified in the
configuration file. Some options are also available::


 usage: trollcast_server [-h] [-l LOG] [-m MAIL] [-v] config_file

 positional arguments:
   config_file

 optional arguments:
   -h, --help            show this help message and exit
   -l LOG, --log LOG     File to log to.
   -m MAIL, --mail MAIL  Mail to log to.
   -v, --verbose         Print out debug messages also.



.. note::

   In the eventuality that you want to start a sever in gateway mode, that is
   acting as a gateway to another server, add
   ``mirror=name_of_the_primary_server`` in your configuration file.

.. note::
   
   Don't forget to prepend "nohup" to the command if you want to make sure the
   process doesn't shut down when you loggout fro the server.

Client mode, retrieving data
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The client mode retrieves data. 

Here is the usage of the client::

 usage: trollcast_client [-h] [-t TIMES TIMES] [-o OUTPUT] -f CONFIG_FILE [-v]
                         satellite [satellite ...]
 
 positional arguments:
   satellite             eg. noaa_18

 optional arguments:
   -h, --help            show this help message and exit
   -t TIMES TIMES, --times TIMES TIMES
                         Start and end times, <YYYYMMDDHHMMSS>
   -o OUTPUT, --output OUTPUT
                         Output file (used only in conjuction with -t)
   -f CONFIG_FILE, --config_file CONFIG_FILE
                         eg. sattorrent_local.cfg
   -l LOG, --log LOG     File to log to.

   -v, --verbose

There are two ways of running the client:
 - The first way is to retrieve a given time interval of data. For example, to
   retrieve data from NOAA 18 for the 14th of November 2012, between 14:02:23
   and 14:15:00, the client has to be called with::

     trollcast_client -t 20121114140223 20121114141500 -o noaa18_20121114140223.hmf -f config_file.cfg noaa_18
 - The second way is to retrieve all the data possible data and dump it to
   files::

     trollcast_client -f config_file.cfg noaa_15 noaa_16 noaa_18 noaa_19

   In this case, only new data will be retrieved though, contrarily to the time
   interval retrieval where old data will be retrieved too if necessary.

API
===

Client
------

.. automodule:: trollcast.client
   :members:
   :undoc-members:

Server
------

.. automodule:: trollcast.server
   :members:
   :undoc-members:

Schedule readers
----------------

.. automodule:: trollcast.schedules
   :members:
   :undoc-members:

Indices and tables
==================

* :ref:`genindex`
* :ref:`modindex`
* :ref:`search`

