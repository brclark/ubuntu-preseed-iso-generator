# Building an Ubuntu autoinstall tool for airgapped Machines

We need to create an Ubuntu ISO that can support a few different installation
requirements:

## Requirements of the Ubuntu OS

This Ubuntu OS image will be used in an airgapped environment, so it must contain
all settings and dependencies necessary to serve content locally, run projects
locally, and allow for further updates easily through access to a server.

1. Include necessary dependencies for development, including NPM, Node, Java, an IDE, etc
1. Customize system settings for local webservers and custom webservers for NPM and Maven registries
1. Set up root user and student user, with custom groups for specific user types
to have access


## Requirements of Autoinstall

To facilitate quick and easy updating of machines for this environment, the Ubuntu
image should be equipped to autoinstall. There are a few use cases for autoinstall

1. Fresh install on client machine, wiping user data
1. Update OS on client machine, preserving user data
1. Fresh install on server machine, wiping user data
1. Update OS on server machine, preserving user data


Autoinstall configures the machine's memory accordingly:

- Install Ubuntu with separate partitions for /boot, / (system), /home, and /swap
- Create custom autoinstall preseeds for scenarios listed above
	- 
