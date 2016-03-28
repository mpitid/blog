
:title: When disks fail
:slug: when-disks-fail
:date: 2011-07-21
:tags: guide

In the last year I've had to recover data from two failing disks as well
as one accidental formating. Overall I was successful, thanks to a
couple of excellent free software power tools: ddrescue_ [#confused]_
and testdisk_.

Because the process is somewhat involved, I decided to write this short
guide as a quick reference, both for myself and anyone who has to deal
with such "disasters".


Step 1: Identifying a failing disk
----------------------------------

While identifying a *failed* disk is usually straightforward --you can't
use it anymore-- identifying one that is *failing* is not as simple.

First of, make sure you have `SMART monitoring`_ enabled, and that you
regularly check the overall health status of the disk. Other things to
look out for are unusual mechanical noises (e.g.  clicking sounds) and
lack of system responsiveness without a high CPU load, a common
side-effect of disk read errors [#monitor]_.

Step 2: Recovering raw data
---------------------------

Using ddrescue_ is a bit confusing with only its manpage as reference.
Fortunately, it comes with a comprehensive manual_ which you *absolutely
must* read before attempting a rescue. I'm providing a brief overview of
its usage, but be aware that this is no replacement for the full manual,
and since I'm no expert on data recovery this information may well be
inaccurate and incomplete.

The overall approach of ddrescue is to read the good blocks of the
failing device first, before trying hard to recover problematic sectors.
This is based on the observation that a failing drive develops more and
more errors as time passes, which is accurate most of the time.

The proposed usage on disk drives consists of two separate runs. The
first tries to be as fast as possible, while the second tries to rescue
as many sectors as possible, by splitting unreadable blocks into smaller
ones, until the hardware limit is reached. Thus you end up running
something like the following:

.. code-block:: bash

  ddrescue --no-split /dev/sda sda.img sda.log

followed by

.. code-block:: bash

  ddrescue --direct /dev/sda sda.img sda.log

The log file will keep track of unreadable sectors and instruct the
second run to only work on those instead of re-reading the disk. It can
also be helpful if for some reason you have to interrupt the rescue, and
continue at a later point.

It's good practice to make an image of the failing disk instead of
rescuing directly to a different device, and even better to work on
copies of this image when trying to repair filesystems or recover files.
The ``--sparse`` option of ddrescue_ can save some space in this case,
provided your filesystem supports sparse files and the disk has large
unwritten areas. After recovery is complete, you can use ddrescue or
some other program (e.g. ``dd``) to write the image to a new device.

However, space limitations may require rescuing directly to another
device, in which case you will have to provide the ``--force`` flag to
ddrescue.

What follows is sample output from ddrescue_ when recovering the 3rd
partition of a failing drive. At this point, errors are starting to
appear after having read 100 GBs of data.

.. code-block:: text

  $ ddrescue --no-split /dev/sdb3 sdb3.img sdb3.log
  
  Press Ctrl-C to interrupt
  Initial status (read from logfile)
  rescued:         0 B,  errsize:       0 B,  errors:       0
  Current status
  rescued:   108539 MB,  errsize:  21117 kB,  current rate:    14224 B/s
     ipos:   108560 MB,   errors:     331,    average rate:   10534 kB/s
     opos:   108560 MB,     time from last successful read:       0 s

And here is the second pass which was left overnight, for roughly 10
hours. You can see how the error size has been reduced to a mere 370 KBs
from the initial 21 MBs. Shortly after, I decided to call it quits and
interrupt the recovery. This partition did not hold very important data
anyway, it was mountable (i.e. no obvious filesystem damage) and it did
not seem like I would be getting anything more out of that disk.

.. code-block:: text

  $ ddrescue --direct --max-retries 2 /dev/sdb3 sdb3.img sdb3.log
  
  Press Ctrl-C to interrupt
  Initial status (read from logfile)
  rescued:   159208 MB,  errsize:  10821 kB,  errors:     357
  Current status
  rescued:   159218 MB,  errsize:    370 kB,  current rate:        0 B/s
     ipos:   108542 MB,   errors:     367,    average rate:      291 B/s
     opos:   108542 MB,     time from last successful read:     1.5 h
  Retrying bad sectors... Retry 2

Refer to the ddrescue manual_ for advice on how to detect corrupt files,
and how to wipe the disk clean before sending it for replacement.

To test if a partition is mountable you can use something like the
following:

.. code-block:: bash

  mount -o ro,loop sdb3.img /mnt

.. attention::
  Note the read-only option which ensures nothing is written to the
  possibly corrupt filesystem -- like access times for instance. If you
  are rescuing a whole disk image and want to check out a specific
  partition, you can use testdisk_, or specify a byte offset when
  mounting. For example, the first partition usually starts after 512
  bytes [#mbr]_.
  
  .. code-block:: bash

    mount -o ro,loop,offset=512 sdb.img /mnt

Step 3: Recovering specific files
---------------------------------

Recovering raw data is only part of the solution. What happens when our
data is there, but the information to retrieve them is lost? In our
current scenario this may occur because some of the unrecovered sectors
included filesystem data. Another common scenario is when we delete
some files we shouldn't -- or in my case, format the wrong drive. The
careful reader will also notice that the two scenarios overlap: it's
entirely possible, and probable, that we will come across previously
deleted files in our effort to recover damaged sections of the
filesystem. So, don't be surprised if deleted files start to crop up.

Now that we recovered as much raw data as possible, it's time for damage
control. If you are lucky you managed to recover all of the data, or the
unreadable sectors did not corrupt the filesystem and the partition
table. If not, don't despair just yet, it's time to give testdisk_ a
try. (Another likely scenario is that you accidentally formatted a disk
partition with valuable data)

testdisk_ can work on disk images as well as actual devices. It has
support for the most popular partition table formats, filesystems and
file types. It can reconstruct partition and filesystem information. It
can also be used to interactively copy recovered files or whole
directories. Finally, it can write the reconstructed information back to
the disk or image.

Its interface is interactive -- albeit text based. Various tutorials and
walkthroughs are available at the project's wiki.

I successfully used testdisk_ to recover most of the data of a FAT32
partition that I accidentally reformatted. Not all of the data was
recoverable though, some were overwritten before I realised my mistake.

I was also impressed by the prompt response to a problem I encountered.
Greek filenames did not appear correctly, but a couple of email
exchanges later and a fix was readily available by testdisk's author.

Caveats
-------

Linux device naming
===================

Be aware that the naming of devices under ``/dev`` is not particularly
stable under Linux with recent udev versions, even across reboots.

Make sure you use UUIDs in ``/etc/fstab`` before you start plugging in
different disks (to avoid trying to mount the damaged disk), and always
double check that you are reading and writing over the correct devices.
You can discover UUIDs by looking at the symlinks in
``/dev/disk/by-uuid`` or by studying the output of ``blkid``.

Another approach is to run ``smartctl -i /dev/xxx`` to list the
manufacturer and model name of a device, to make sure it is the correct
one, or use one of the symlinks in ``/dev/disk/by-id``.

Appendix
--------

.. _smart monitoring:

SMART
=====

SMART_, short for Self-Monitoring, Analysis and Reporting Technology, is
a standard to monitor hard disk drives in the hope of predicting
imminent hardware failures. Since not all of the information reported is
standardized and since certain values have different meanings across
manufacturers, it is not always reliable as a health assessment tool.
Nevertheless, some information is better than no information, and a
device that keeps logging SMART errors is more likely to fail in the
near future than one that appears healthy.

On GNU/Linux the smartmontools_ suite can be used to query SMART
information and perform SMART self tests. In addition, the hddtemp_
utility can be used to query temperature information (which itself
relies on SMART data).

CrystalDiskInfo_ is an excellent SMART monitoring tool for windows
systems. I always keep a copy of the portable version on a USB key
myself. Another promising monitoring tool for windows is `Open Hardware
Monitor`_, which provides information on modern CPU and GPU sensors as
well.

.. _ddrescue: http://www.gnu.org/software/ddrescue/ddrescue.html
.. _manual: http://www.gnu.org/software/ddrescue/manual/ddrescue_manual.html
.. _testdisk: http://www.cgsecurity.org/wiki/TestDisk
.. _dd_rescue: http://www.garloff.de/kurt/linux/ddrescue

.. _SMART: http://en.wikipedia.org/wiki/S.M.A.R.T.

.. _htop: http://htop.sourceforge.net
.. _iotop: http://guichaz.free.fr/iotop
.. _sysstat: http://pagesperso-orange.fr/sebastien.godard

.. _CrystalDiskInfo: http://crystalmark.info/software/CrystalDiskInfo/index-e.html
.. _Open Hardware Monitor: http://openhardwaremonitor.org

.. _smartmontools: http://smartmontools.sourceforge.net
.. _hddtemp: http://www.guzu.net/linux/hddtemp.php

.. [#confused] Not to be confused with a different but similarly named tool, dd_rescue_.
.. [#monitor] Some useful system monitoring tools for GNU/Linux include htop_, iotop_ and sysstat_.
.. [#mbr] These consist of the *Master Boot Record* which contains the device's partition table.
