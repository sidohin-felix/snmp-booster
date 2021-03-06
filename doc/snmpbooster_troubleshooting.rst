.. _snmpbooster_troubleshooting:

===========================
SnmpBooster Troubleshooting
===========================

Check your config
=================

  - Have you defined the poller module name?
  - Have you defined the correct path to the directory containing your Defaults*.ini files?
  - Have you addded the Snmpbooster module to your arbiter, poller, scheduler?
  - Have you added copied the genDevConfig templates.cfg in shinken/packs/network/SnmpBooster/
  - Have you installed PySNMP, Redis and other dependencies?

Software version consistency
============================

Shinken and SnmpBooster now require **the same Python and Pyro version on all hosts running a Shinken daemon**.

If you cannot use the packaged version of Python and its modules (Pyro, redis, etc.). Use `virtualenv`_ to declare a python version to use and install all required modules in that virtualenv.

.. _virtualenv: http://pypi.python.org/pypi/virtualenv

Software version requirements
=============================

Have you verified that the :ref:`requirements <setup_snmp_booster_module>` are met. Python, PySNMP, Shinken, Pyro, redis, etc.

Validate your check command arguments
=====================================

Use the check_plugin command and comment out the module to learn what were the exact arguments sent by the poller.
This will permit you to validate all the arguments, like snmp community string, inheritance, template application, etc.

Validate connectivity
=====================

Take a packet trace using a tool like Wireshark to validate that the remote host is responding.
    * Has the host responded
    * Is SnmpBooster repeating the request more often than the polling interval. 
        * If you are seeing repeated requests your device may have a compatibility issues. 
        * Save an snmpwalk of the device, get a packet trace using Wireshark, set the poller to debug and save the poller.log file (``/var/log/shinken/pollerd.log``). Send all three to the SnmpBooster developers.

.. note::

    It is normal to see one or more bulkGet requests if you are getting large amounts of data. Ex. a 24 port switch will take 2-3 request packets.

Performance
===========

Make sure you have a low latency connection to your redis from the Poller. 
Check that redis is running: ``netstat -a | grep redis``

Faulty Template
===============

A bad snmp_template file was distributed in the genDevConfig sample-config directory, there were two glaring errors.

This was fixed on 2012-10-16. Make sure you update your template, or use the data from the wiki.

Note that the template should be called: SnmpBooster-template.cfg to make it easier to troubleshoot in the logs. So when you search for SnmpBooster in your logs it will show up as well.

Log files
=========

All warnings and errors generated by the SnmpBooster module start with "[SnmpBooster] error text" and are logged using the standard Shinken logger.

The Arbiter daemon can output initial configuration, loading of host,service keys in redis type error messages.
The Scheduler daemon can output scheduling and alert related messages.
The Poller daemon can output messages related to instance mapping, acquisition timeouts, invalid community strings, cache failures and more. These are available in the Web interface, as they are placed in the check results for the service.

You can simply do a ``grep SnmpBooster *`` in your shinken/var directory to see the latest messages related to the SnmpBooster module. You can also sort messages by timestamp to make it easy to find where and when errors occurred.

::

    cd shinken/var
    grep SnmpBooster *


Error codes
===========

Code 0101
    =========== ===========================================================================
    Type        INFO
    Description SNMP Booster module starts loading
    File        `__init__.py`
    =========== ===========================================================================

Code 0102
    =========== ===========================================================================
    Type        ERROR
    Description The attribute **loaded_by** is not set in the Shinken configuration
    File        `__init__.py`
    =========== ===========================================================================

Code 0103
    =========== ===========================================================================
    Type        ERROR
    Description The attribute **loaded_by** must be `poller`, `scheduler` or `arbiter` and
                this is not the case
    File        `__init__.py`
    =========== ===========================================================================

Code 0201
    =========== ===========================================================================
    Type        ERROR
    Description **PySNMP** module can not be loaded. Please checks your installation
    File        `libs/checks.py`
    =========== ===========================================================================

Code 0202
    =========== ===========================================================================
    Type        ERROR
    Description The current service is not found in the cache (Redis). Maybe you flush it
                ? Check your Shinken configuration and try to restart the Arbiter to 
                refill the cache (Redio)
    File        `libs/checks.py`
    =========== ===========================================================================

Code 0501
    =========== ===========================================================================
    Type        WARNING
    Description The Poller didn't found the asked service in the cache (Redis). This
                error should not appear. Please open an issue on GitHub, if you get it.
    File        `libs/results.py`
    =========== ===========================================================================

Code 0502
    =========== ===========================================================================
    Type        WARNING
    Description We try to get data from a service which the mapping is not done. We have
                four possible reasons:

                * The host is down
                * The mapping name set in the Shinken service configuration has an error.
                  Please check your configuration
                * The mapping is not finished yet it will be done in few moments
                * The instance name set in the Shinken service configuration has an error
                  and it will never found in the mapping SNMP table. Please check your
                  configuration
    File        `libs/results.py`
    =========== ===========================================================================

Code 0601
    =========== ===========================================================================
    Type        ERROR
    Description **PySNMP** module can not be loaded. Please checks your installation
    File        `libs/snmpworker.py`
    =========== ===========================================================================

Code 0602
    =========== ===========================================================================
    Type        INFO
    Description The SNMP worker thread is starting
    File        `libs/snmpworker.py`
    =========== ===========================================================================

Code 0603
    =========== ===========================================================================
    Type        ERROR
    Description We got a SNMP request which is not `get`, `getnext` or `getbulk`
                Please open an issue on GitHub, if you get it.
    File        `libs/snmpworker.py`
    =========== ===========================================================================

Code 0604
    =========== ===========================================================================
    Type        INFO
    Description The SNMP worker thread is now stopped
    File        `libs/snmpworker.py`
    =========== ===========================================================================

Code 0605
    =========== ===========================================================================
    Type        INFO
    Description The SNMP worker thread will be stopped
    File        `libs/snmpworker.py`
    =========== ===========================================================================

Code 0606
    =========== ===========================================================================
    Type        ERROR
    Description We got a SNMP error. This could be a timeout, a bad response, ...
    File        `libs/snmpworker.py`
    =========== ===========================================================================

Code 0701
    =========== ===========================================================================
    Type        ERROR
    Description We got a trigger error. It seems that the datasource name use in the
                trigger doesn't exist. Please check your triggers definitions
    File        `libs/trigger.py`
    =========== ===========================================================================

Code 0702
    =========== ===========================================================================
    Type        ERROR
    Description We didn't found any collected data in the cache (Redis) to use in the
                trigger. We have four possible reasons:

                * The SNMP request is not finished. We have to wait the next check
                * The oid asked doesn't exists and we never get a value. Please check your
                  Shinken service configuration
                * The host is down
    File        `libs/trigger.py`
    =========== ===========================================================================

Code 0703
    =========== ===========================================================================
    Type        ERROR
    Description We didn't found any computed data in the cache (Redis) to use in the
                trigger. We have two possible reasons:

                * The current service use a datasource which is a DERIVE, so we need TWO
                  values to compute the derive.
                * We got an error during the value computation
    File        `libs/trigger.py`
    =========== ===========================================================================

Code 0704
    =========== ===========================================================================
    Type        ERROR
    Description We got an error during the execution of trigger function.
                The argument passed to the trigger function has a wrong type or 
                is empty. Please check your trigger configuration
    File        `libs/trigger.py`
    =========== ===========================================================================

Code 0705
    =========== ===========================================================================
    Type        ERROR
    Description We got an error during the execution of trigger function. The trigger
                function doesn't exist. Please check your trigger
                configuration or if it's a new function open an issue on GitHub
    File        `libs/trigger.py`
    =========== ===========================================================================

Code 0706
    =========== ===========================================================================
    Type        ERROR
    Description We didn't found the asked datasource name defined in the trigger. This
                could be a typo. Please check your trigger configuration
    File        `libs/trigger.py`
    =========== ===========================================================================

Code 0707
    =========== ===========================================================================
    Type        ERROR
    Description We got an error during the execution of a trigger. Please check your
                trigger configuration
    File        `libs/trigger.py`
    =========== ===========================================================================

Code 0708
    =========== ===========================================================================
    Type        INFO
    Description The trigger triggered. It means the service state will be WARNING or
                CRITICAL
    File        `libs/trigger.py`
    =========== ===========================================================================

Code 0709
    =========== ===========================================================================
    Type        ERROR
    Description Unknown trigger error. Maybe it's a good idea to report a bug ?
    File        `libs/trigger.py`
    =========== ===========================================================================

Code 0801
    =========== ===========================================================================
    Type        WARNING
    Description The parameter **-M** or **--max_rep_map** define in the check command has a
                bad format. Please check your Shinken configuration
    File        `libs/utils.py`
    =========== ===========================================================================

Code 0802
    =========== ===========================================================================
    Type        WARNING
    Description The parameter **-g** or **--request_group_size** define in the check
                command has a bad format. Please check your Shinken configuration
    File        `libs/utils.py`
    =========== ===========================================================================

Code 0901
    =========== ===========================================================================
    Type        ERROR
    Description **configobj** module can not be loaded. Please checks your installation
    File        `snmpbooster_arbiter.py`
    =========== ===========================================================================

Code 0902
    =========== ===========================================================================
    Type        INFO
    Description The SNMP Booster module is reading datasource file
    File        `snmpbooster_arbiter.py`
    =========== ===========================================================================

Code 0903
    =========== ===========================================================================
    Type        INFO
    Description The SNMP Booster module is reading datasource files
    File        `snmpbooster_arbiter.py`
    =========== ===========================================================================

Code 0904
    =========== ===========================================================================
    Type        ERROR
    Description We got an error merging datasource files. Please check your configuration
    File        `snmpbooster_arbiter.py`
    =========== ===========================================================================

Code 0905
    =========== ===========================================================================
    Type        ERROR
    Description We got an error merging datasource files. Please check your configuration
    File        `snmpbooster_arbiter.py`
    =========== ===========================================================================

Code 0906
    =========== ===========================================================================
    Type        ERROR
    Description We got an error during the conversion of the datasource configuration from
                ini format to python dictionnary format. Please check your configuration
    File        `snmpbooster_arbiter.py`
    =========== ===========================================================================

Code 0907
    =========== ===========================================================================
    Type        ERROR
    Description We got an error during the serialization of service configuration just
                before put it in the cache (Redis)
    File        `snmpbooster_arbiter.py`
    =========== ===========================================================================

Code 0909
    =========== ===========================================================================
    Type        ERROR
    Description We got an error during the update of service configuration in the cache
                (Redis)
    File        `snmpbooster_arbiter.py`
    =========== ===========================================================================

Code 1001
    =========== ===========================================================================
    Type        ERROR
    Description We got an error during command line parsing. Please check your check
                command definition in your Shinken configuration
    File        `snmpbooster_poller.py`
    =========== ===========================================================================

Code 1002
    =========== ===========================================================================
    Type        ERROR
    Description The SNMP Booster module in the poller can't write check results in the
                Scheduler queue. You may restart your Poller and/or your Scheduler
    File        `snmpbooster_poller.py`
    =========== ===========================================================================

Code 1003
    =========== ===========================================================================
    Type        ERROR
    Description The SNMP Booster module in the poller can't write check results in the
                Scheduler queue. You may restart your Poller and/or your Scheduler
    File        `snmpbooster_poller.py`
    =========== ===========================================================================

Code 1004
    =========== ===========================================================================
    Type        ERROR
    Description The datasource type is not 'TEXT', 'STRING', 'DERIVE', 'GAUGE', 'COUNTER',
                'DERIVE64' or 'COUNTER64'. Please check your Datasource configuration
    File        `snmpbooster_poller.py`
    =========== ===========================================================================

Code 1005
    =========== ===========================================================================
    Type        WARNING
    Description We get an error while computing service values
    File        `snmpbooster_poller.py`
    =========== ===========================================================================

Code 1006
    =========== ===========================================================================
    Type        INFO
    Description SNMP Booster Poller module started
    File        `snmpbooster_poller.py`
    =========== ===========================================================================

Code 1007
    =========== ===========================================================================
    Type        ERROR
    Description The SNMP Booster module in the poller can't read checks results from the
                Scheduler queue. You may restart your Poller and/or your Scheduler
    File        `snmpbooster_poller.py`
    =========== ===========================================================================

Code 1101
    =========== ===========================================================================
    Type        INFO
    Description SNMP Booster module loaded
    File        `snmpbooster.py`
    =========== ===========================================================================

Code 1102
    =========== ===========================================================================
    Type        ERROR
    Description The attribute **datasource** is missing in the Shinken module settings.
                Please check your configuration
    File        `snmpbooster.py`
    =========== ===========================================================================

Code 1201
    =========== ===========================================================================
    Type        ERROR
    Description **Python Redis** module can not be loaded. Please check your installation
    File        `libs/dbclient.py`
    =========== ===========================================================================

Code 1202
    =========== ===========================================================================
    Type        ERROR
    Description Can not connect to the Redis server. Please check your configuration
    File        `libs/dbclient.py`
    =========== ===========================================================================

Code 1203
    =========== ===========================================================================
    Type        ERROR
    Description We got an error while writing in the Redis. The data passed doesn't
                seem correct
    File        `libs/dbclient.py`
    =========== ===========================================================================

Code 1204
    =========== ===========================================================================
    Type        ERROR
    Description We got an error while a the upsert in the Redis of a service. This
                error can only occur on the Arbiter
    File        `libs/dbclient.py`
    =========== ===========================================================================

Code 1205
    =========== ===========================================================================
    Type        ERROR
    Description We got an error updating collected data in the Redis of a service.
                Thiserror can only occur on the Poller
    File        `libs/dbclient.py`
    =========== ===========================================================================

Code 1206
    =========== ===========================================================================
    Type        ERROR
    Description We got an error updating instance mapping of a service in the
                Redis. This error can only occur on the Poller
    File        `libs/dbclient.py`
    =========== ===========================================================================

Code 1207
    =========== ===========================================================================
    Type        ERROR
    Description We got an error getting ONE service in the Redis. This error can only
                occur on the Poller
    File        `libs/dbclient.py`
    =========== ===========================================================================

Code 1208
    =========== ===========================================================================
    Type        ERROR
    Description We got an error getting several services in the Redis. This error can
                only occur on the Poller
    File        `libs/dbclient.py`
    =========== ===========================================================================

Code 1301
    =========== ===========================================================================
    Type        ERROR
    Description **Python Redis** module can not be loaded. Please check your installation
    File        `libs/redisclient.py`
    =========== ===========================================================================

Code 1302
    =========== ===========================================================================
    Type        ERROR
    Description Can not connect to the Redis server. Please check your configuration
    File        `libs/redisclient.py`
    =========== ===========================================================================

Code 1303
    =========== ===========================================================================
    Type        ERROR
    Description We got an error writing service in host:interval list
    File        `libs/redisclient.py`
    =========== ===========================================================================

Code 1304
    =========== ===========================================================================
    Type        ERROR
    Description We got an error inserting service data in Redis service
    File        `libs/redisclient.py`
    =========== ===========================================================================

Code 1305
    =========== ===========================================================================
    Type        ERROR
    Description We got an error getting ONE service data in the Redis server
    File        `libs/redisclient.py`
    =========== ===========================================================================

Code 1306
    =========== ===========================================================================
    Type        ERROR
    Description We got an error getting services list from host:interval key
    File        `libs/redisclient.py`
    =========== ===========================================================================

Code 1307
    =========== ===========================================================================
    Type        ERROR
    Description We got an error getting ONE service in Redis. This service seems missing
    File        `libs/redisclient.py`
    =========== ===========================================================================

Code 1308
    =========== ===========================================================================
    Type        ERROR
    Description We got an error getting ONE service in Redis 
    File        `libs/redisclient.py`
    =========== ===========================================================================
