
# Python library for IBM MQ

PyMQI is a production-ready, open-source Python extension for IBM MQ (formerly known as WebSphere MQ and MQSeries).

For 20+ years, the library has been used by thousands of companies around the world with their queue managers running on
Linux, Windows, UNIX and z/OS.

# Build binary wheels

Get the redistributables from IBM site.
For example for 9.4 the URL is https://ibm.biz/mq94redistclients

Find out more on the IBM Docs
https://www.ibm.com/docs/en/ibm-mq/9.4.x?topic=overview-redistributable-mq-clients

Most probably you will need an IBM account :(

Extract those files somewhere.


## Linux

Set the `MQ_FILE_PATH` environment variable to the place where the
redistributables were extracted.

```
$ export MQ_FILE_PATH=/home/user/Downloads/9.4.3.1-IBM-MQC-Redist-LinuxX64
$ python -m pip install wheel
$ python setup.py bdist_wheel --plat-name=manylinux_2_17_x86_64
```

It will copy the IBM MQ libraries at lib/shared_libs.
You can then publish the wheel file to your private PyPi server
or somewhere in your build system.

You will need to set `LD_LIBRARY_PATH` as Python only looks for its libraries.
3rd party libraries will need the standard Linux search mechanism.
In your virtual environment you can do

```
python -m pip install ../pymqi/dist/pymqi-1.12.11-cp312-cp312-linux_x86_64.whl
export LD_LIBRARY_PATH=YOUR_VENV/lib/ibm-mq/lib64/:YOUR_VENV/lib/ibm-mq/gskit8/lib64/
python your_ibm_mq_sample_code.py
```

## Windows


The Microsoft Visual C++ 2013 Redistributable is required to build on
Windows systems.
This is a limitation of the IBM MQ C Client library.
The DLLs are copied in the wheel, so you don't need to install this on the
target systems.

Similar to Linux.
Set the `MQ_FILE_PATH` to the path where the files were extracted.

```
> $env:MQ_FILE_PATH = "C:\Users\John\Downloads\9.4.3.1-IBM-MQC-Redist-Win64
> python setup.py bdist_wheel
```

It will copy the IBM MQ DLLs and table files in the Python virtual environment,
into a sub-folder called `ibm-mq`.
You will need to setup Python to load the DLL from there.

```python
import os

if os.name == 'nt':
    base_dll_dir = os.path.join(os.path.dirname(sys.executable), 'ibm-mq')
    os.add_dll_directory(base_dll_dir)

import pymqi
```


# Sample code

To put a message on a queue:

```python
import pymqi

queue_manager = pymqi.connect('QM.1', 'SVRCONN.CHANNEL.1', '192.168.1.121(1434)')

q = pymqi.Queue(queue_manager, 'TESTQ.1')
q.put('Hello from Python!')
```

To read the message back from the queue:

```python
import pymqi

queue_manager = pymqi.connect('QM.1', 'SVRCONN.CHANNEL.1', '192.168.1.121(1434)')

q = pymqi.Queue(queue_manager, 'TESTQ.1')
msg = q.get()
print('Here is the message:', msg)
```

## High-level MQ messaging in Python

PyMQI is a low-level library that requires one to know IBM MQ APIs well.

If you'd like to have an easy to use IBM MQ Python interface that doesn't require an extensive knowledge of MQ,
use
[Zato](https://zato.io),
which is a Python-based
[IPaaS](https://zato.io/articles/integration-platform.html)
and
[enterprise service bus](https://zato.io/en/docs/3.3/intro/esb-soa.html)
that supports MQ, among other protocols.

![](https://upcdn.io/kW15bqq/raw/root/en/docs/3.3/gfx/api/screenshots/mq.png)


```python
# Zato
from zato.server.service import Service

class MyService(Service):

    def handle(self):

        # Send MQ messages in one line of code
        self.outgoing.ibm_mq.send('my-message', 'CORE', 'QUEUE.1')
```

## More resources

* Learn more about [Zato](https://zato.io)
* PyMQI [documentation](https://zato.io/pymqi/index.html)
