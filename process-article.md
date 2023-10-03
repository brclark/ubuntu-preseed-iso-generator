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
1. Update OS on client machine, preserving Ubuntu user data
1. Update OS on client machine, preserving SUSE user data
1. Fresh install on server machine, wiping user data
1. Update OS on server machine, preserving user data


Autoinstall configures the machine's memory accordingly:

- Install Ubuntu with separate partitions for /boot, / (system), /home, and /swap
- Create custom autoinstall preseeds for scenarios listed above
	1. Preseed configured to format all partitions
	1. Preseed configured to format partitions excluding /home, and keep /home
	1. Preseed not configured to format partitions, allow manual partitioning to preserve
SUSE partition
	1. Preseed configured to format all partitions
	1. Preseed configured to format partitions excluding /home


### Format All Partitions

Using Ubuntu Preseed, we can configure the installer to predefine the partition map for the memory.
Based on research and our previous SUSE solution, the partitions should be:

 - Bootloader EFI: /boot/efi
	- *Size*: 524MB
	- *Format*: FAT16
 - System Partition: /
	- *Size*: 43GB
	- *Format*: ext4
 - Swap Partition: /swap
	- *Size*: 100% RAM
	- *Format*: swap
 - Home Partition: /home
	- *Size*: Remaining (430GB | 190GB)
	- *Format*: ext4

Using the `partman` tool in Preseed, we can configure the above formatting using the recipe below.
You can read more about Partman partition format [here](https://wikitech.wikimedia.org/wiki/PartMan/Auto).

```
ubiquity partman-auto/expert_recipe string                    \
        boot-root ::                                          \
              512 512 512 fat16                               \
                      $primary{ }                             \
                      $iflabel{ gpt }                         \
                      method{ efi }                           \
                      label { boot }                          \
                      format{ }                               \
              .                                               \
              43008 43008 43930  ext4                          \
                      method{ format } format{ }              \
                      use_filesystem{ } filesystem{ ext4 }    \
                      label { system }                        \
                      mountpoint{ / }                         \
              .                                               \
              4096 4096 4096 linux-swap                    \
                      method{ swap } format{ }                \
              .                                               \
              10240 100000 1000000  ext4                       \
                      method{ format } format{ }             \
                      use_filesystem{ } filesystem{ ext4 }    \
                      label { home }                         \
                      mountpoint{ /home }                     \
              .

```

### Format System Partitions, Preserve Home

There are at least two cases where we will format system partitions but attempt to preserve the home partition

1. *Preserve Ubuntu /home*: For a machine that needs to be updated in the future, we will want to format the system partition
but preserve the home partition as ext4, using the same partition sizes described above
1. *Preserve SUSE /home*: Our previous OS used the XFS format for the home partition. In order to preserve this partition, we can
manually format partitions in the Ubuntu Installer. This requires leaving the `partman` section out of preseed


To *Preserve Ubuntu /home*, we can set up a partman recipe to keep the home partition as is.

```
ubiquity partman-auto/expert_recipe string                    \
        boot-root ::                                          \
              524 524 524 fat16                               \
                      $primary{ }                             \
                      $iflabel{ gpt }                         \
                      method{ efi }                           \
                      label { boot }                          \
                      format{ }                               \
              .                                               \
              43008 43008 43930  ext4                          \
                      method{ format } format{ }              \
                      use_filesystem{ } filesystem{ ext4 }    \
                      label { system }                        \
                      mountpoint{ / }                         \
              .                                               \
              4096 4096 100% linux-swap                       \
                      method{ swap } format{ }                \
              .                                               \
              10240 100000 1000000 ext4                         \
                      method{ keep }                          \
                      use_filesystem{ } filesystem{ ext4 }    \
                      label { home }                         \
                      mountpoint{ /home }                     \
              .
```

To *Preserve SUSE /home*, we must comment out all Preseed lines related to `partman` to allow manual setup.
