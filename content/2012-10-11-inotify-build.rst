
:title: inotify; build
:date: 2012-10-11
:tags: scripts, utilities

A few months ago, when I started using Scala more seriously, I
familiarised myself with sbt_ after seeing it touted as the recommended
build tool here and there. Despite some grievances with this
not-so-simple build tool, I did find one of its features rather handy:
automatically executing compile or test actions whenever a file changed
(quite similar to hakyll's preview feature).

In fact, I got used to this feature so much that I decided I needed it
for other projects, like automatically running LaTeX for me, or updating
ikiwiki_.

It turns out this was fairly simple to implement with inotify-tools_
doing all the heavy lifting, with regard to monitoring files for changes
and generating appropriate events. All that remained was wrapping it up
in a suitable script I called ``monitor``, to only act on files that
match a particular regular expression:

.. _sbt: http://www.scala-sbt.org
.. _ikiwiki: http://ikiwiki.info
.. _inotify-tools: https://github.com/rvoicilas/inotify-tools/wiki

.. code-block:: bash
  #!/bin/zsh
  
  TARGET="$1"
  PATTERN="$2"
  COMMAND=("${@:3}")
  
  [[ $# -lt 3 ]] && {
   echo "usage: $0 <watched-dir> <file-pattern> <command>"
   exit 1 }
  
  EVENTS=(modify delete)
  
  inotifywait -r -m -e ${(j.,.)EVENTS} --format "%w%f" $TARGET \
  | egrep --line-buffered "$PATTERN" | while read file; do
    echo "event: $file"
    $COMMAND
  done

I could then run the following command to continuously build my ikiwiki:

.. code-block:: bash

  monitor path/to/wiki '\.mkdn$|\.mdwn$' ikiwiki --setup ~/wiki.setup

or generate my LaTeX reports (given a suitable Makefile):

.. code-block:: bash

  monitor path/to/tex '\.tex$|\.bib$' make

The script is also available on github__ as part of a collection of
scripts I have found a use for from time to time.

__ https://github.com/mpitid/scripts/blob/master/monitor
