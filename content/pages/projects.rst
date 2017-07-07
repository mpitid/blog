
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

Kafka tools
-----------

A set of command line tools to communicate with Kafka instances. The
main benefit in terms of the existing scripts in the Kafka distribution
is a more UNIX-like behaviour which allows use in pipes. Supports `Kafka
7.x`_ and `8.x`_ (therefore 9.x and 10.x through their common API).
Besides allowing to write to a producer and read from a consumer, the
Kafka 8 branch provides support for offset and topic management under a
single executable and unified command line interface.

The project was a quick hack so packaging is rough around the edges (jar
or a all-in-one RPM), however I found it quite handy when trying to find
missing data on production or just feeding mock data for dev and staging
environments!

Have a look at the `README`__ for extensive usage instructions.

__ https://github.com/mpitid/kafka-tools/blob/master/README.md

.. _Kafka 7.x: https://github.com/mpitid/kafka-tools/tree/kafka7
.. _8.x: https://github.com/mpitid/kafka-tools/tree/kafka8

Scala libraries & tools
-----------------------

Bits and pieces of Scala_ code that I have collected in the form of
libraries or simple command line tools. `Tiny utils`_ is mostly a bunch
of implicits to allow working with Java's atomic references, count down
latches, scheduled executors and thread-locals in a more idiomatic way.
`Back-off utils`_ is an implementation of a diverse set of back-off
policies as immutable case classes, mostly inspired by `Mark Brooker's
blog post`__ on the subject. These can come in real handy when writing
Scala_ HTTP clients for APIs and especially when crawling `rate-limited
ones <#social-api-tools>`_. It also includes a handy library for
generating unbiased uniform longs within a specific interval
(conspicuously missing from `java.util.Random`__ in Java 7, but not from
`java.util.concurrent.ThreadLocalRandom`__ which is where it's borrowed
from).

__ http://brooker.co.za/blog/2015/03/21/backoff.html
__ https://docs.oracle.com/javase/7/docs/api/java/util/Random.html
__ https://docs.oracle.com/javase/7/docs/api/java/util/concurrent/ThreadLocalRandom.html#nextLong(long,%20long)

.. _Tiny utils: https://github.com/mpitid/scala-util-backoff
.. _Back-off utils: https://github.com/mpitid/scala-util-tiny/

In terms of tools, `NFST`_ is a command line tool for working with
`Lucene's Finite State Transducers`__. For some context, I highly
recommend reading Andrew Gallant's write-up on `implementing FSTs in
Rust`__. Another handy tool is `scully`_, a command line tool for
reading from and writing to `0mq sockets`__, which comes with an
`equivalent in Python`_, both handy if you are stuck in an environment
where you cannot install packages. Finally, `JML`_ is an old prototype
for a JSON mapping and transformation domain-specific language using
parser combinators.

__ http://blog.mikemccandless.com/2010/12/using-finite-state-transducers-in.html
__ http://blog.burntsushi.net/transducers/
__ http://zeromq.org/

.. _NFST: https://github.com/mpitid/nfst
.. _JML: https://github.com/mpitid/jml
.. _Scully: https://github.com/mpitid/scully
.. _equivalent in Python: https://github.com/mpitid/pako

Social API tools
----------------

A set of `command line tools`_ in Python_ for working with `Facebook's
Graph API`__ and `Instagram's API`__, these were invaluable in debugging
production crawlers for those APIs or doing one-off bulk retrievals of
content for simple analysis, the main benefit being the automatic
handling of the different pagination parameters and support for
different output formats (JSON or YAML). Examples of usage and simple
analysis with command line tools like `jq`__, ``sort`` and ``sed`` are
provided in the README of each project.

__ https://developers.facebook.com/docs/graph-api/reference
__ https://instagram.com/developer/endpoints/
__ https://stedolan.github.io/jq/

.. _command line tools: https://github.com/mpitid/apiutils

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

Scheme48
--------

A very simple `scheme interpreter`__ following the `Write Yourself a
Scheme in 48 Hours`__ Haskell tutorial.

__ https://github.com/mpitid/scheme48
__ https://en.wikibooks.org/wiki/Write_Yourself_a_Scheme_in_48_Hours

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

