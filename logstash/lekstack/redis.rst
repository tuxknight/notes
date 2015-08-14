.. _redis:

Redis On Docker
==================

Pull official image from docker hub::

  docker pull redis``

Usage
------

#. start a redis instance
   docker run --name broker -d -p 6379:6379 redis

#. connect to it from an application
   docker run --rm --name shipper --link broker:redis -p 10514:10514  -v /root/shipper/shipper.conf:/shipper.conf logstash logstash -f /shipper.conf
