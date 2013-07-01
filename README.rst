===========
IPyDocker
===========

IPyDocker aims to provide scripts to simplify Python parallelization tasks. It uses `Docker <http://www.docker.io/>`_ virtual machines and `IPython <http://ipython.org/>`_ parallelization capabilities.

Why
-------------

This approach is about having IPython ipcontroller on one machine and IPython ipengines inside Docker virtual machines (Linux containers actually) which are hosted on some worker machines. 
Docker virtual machines can be hosted on the same machine as controller too, 
but it doesn't make any sense as this is just an unneeded overhead - you can simply use IPython without Docker in this case.  
However, if you would like to exploit IPython parallelization capabilities using more than one physical machine, you can use Docker virtual machines to simplify the configuration and to isolate the worker environment.

How to
-------------

Clone IPyDocker repo on the machines you want to use as workers and execute the command below. 
This will prepare a Docker virtual machine with some preinstalled libraries
(modify Dockerfile if you wish to have other libraries).
::
	docker build -t krop-img .

Prepare IPython SSH profile on a controller machine.
Create SSH profile:
::
	ipython profile create --parallel --profile=ssh

Go into .ipython/profile_ssh and set the controller IP in ipcontroller_config.py:
:: 
	HubFactory.ip = '192.168.1.14'

Run controller:
::
	ipcontroller --profile ssh

Prepare worker machines - execute the following command to start the Docker container on each of the worker machines:
::
	docker run -d krop-img

Copy ipcontroller-engine.json from IPYTHONDIR/profile_ssh/security to the workers. For example (you can see the port number if you execute docker ps):
::
	scp -P 49185 ipcontroller-engine.json root@192.168.1.14://root/

Connect to worker (the password is krop - see the Dockerfile where it is set):
::
	ssh root@192.168.1.14 -p 49185

Start one or more ipengines on each worker:
::
	ipengine --file=/root/ipcontroller-engine.json

Now you can delegate tasks from the controller machine:
::
	from IPython.parallel import Client
	c = Client(profile="ssh")
	print c.ids
	c[:].apply_sync(lambda : "Hello World")


