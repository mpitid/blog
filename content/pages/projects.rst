
:title: Projects
:modified: 2016-03-28 15:00

This page includes various free software projects I have personally
created or collaborated on. Some came out of school projects and
assignments, some were started to scratch an itch, while others were
created for the pure fun of it.

Purity
------

Purity is a lightweight static analysis tool that determines the purity
of Erlang_ functions. It started as part of my diploma thesis back at
NTUA_ and was subsequently enhanced and released as free software under
the `LGPL v2`_. Besides detection of side-effects, execution environment
dependencies and exceptions, it also features a very simple termination
analysis. Code and documentation is available on github__.

__ https://github.com/mpitid/purity

A highly experimental unfinished prototype of user defined guards for
Erlang --utilising the analysis' results-- can be found here__.

__ https://github.com/bjorng/otp/tree/user-guards

Spacenet
--------

Spacenet is a distributed programming game. It was originally conceived
as a fun way to present a workshop on declarative languages, along with
two fellow members of the `FOSS NTUA`_ community. It is written in
Erlang_, Haskell_ and Prolog_. I wrote the Erlang client/server code and
some of the Prolog parts. The game has been tested successfully on a
small LAN with 10 players. Code, documentation and screencast demos are
available at the `spacenet website`_.

.. _spacenet website: http://foss.ntua.gr/spacenet

Labelme
-------

Labelme is a simple image annotation tool developed for a project in my
*Human-Computer Interaction* course with a fellow student. Since the
code was relatively complete and functional I decided to publish it --
who knows someone may find it useful someday. It is inspired by a
similar web-based tool from the `MIT CSAIL Lab`_ but written in Python_
and Qt_. The code is available on `github
<https://github.com/mpitid/pylabelme>`_ and you can see it in action in
this `screencast tutorial <{filename}../files/labelme.ogv>`_.

.. _MIT CSAIL Lab: http://labelme.csail.mit.edu
.. _Qt: http://www.qt.io

Pytag
-----

Pytag is command-line audio mass-tagging application written in Python_
and based on the mutagen_ library. I regularly use it to organise parts
of my ever-growing music collection. I'm currently in the process of
recovering the repository and documentation after the server hosting it
was lost.

.. _mutagen: https://pypi.python.org/pypi/mutagen

Scripts
-------

Besides the aforementioned applications, I have put together a
collection of some the scripts_ I've written from time to time, which
are too small to warrant a section of their own.

aconv_ is a handy shell script for converting audio files from a variety
of formats (anything mplayer_ supports) to ogg, mp3 or wav.

checksum_ is a script that calculates multiple checksums on files
without reading them multiple times. This is most likely faster if you
want to calculate multiple different checksums of large files (for
whatever reason), since I/O tends to be the bottleneck in such cases.

wgrep_ can be thought of as a combination of ``wget`` and ``grep``. I
use it to extract hyperlinks that match certain patterns from webpages.
The pattern can refer to the URL itself, the text inside the anchor or
the URL of an IMG element inside the anchor. For example, to download
all the JPEG images in a webpage, one could use the following:

.. code-block:: bash

  wgrep '\.jpg$' http://some.website.com/ | xargs wget --content-disposition

zget_ is another handy script which downloads a zip or rar archive and
extracts it in one go, deleting the archive afterwards.

.. _scripts: https://github.com/mpitid/scripts
.. _aconv:   https://github.com/mpitid/scripts/blob/master/aconv
.. _wgrep:   https://github.com/mpitid/scripts/blob/master/wgrep
.. _checksum: https://github.com/mpitid/scripts/blob/master/checksum
.. _zget:     https://github.com/mpitid/scripts/blob/master/zget
.. _mplayer: http://www.mplayerhq.hu/

.. _Scala: http://www.scala-lang.org
.. _Python: https://www.python.org
.. _Erlang: https://www.erlang.org
.. _Haskell: https://www.haskell.org
.. _Prolog: https://en.wikipedia.org/wiki/Prolog
.. _OCaml: https://ocaml.org

.. _LGPL v2: https://www.gnu.org/licenses/lgpl-2.1.html

.. _NTUA: http://www.ntua.gr
.. _FOSS NTUA: http://foss.ntua.gr

