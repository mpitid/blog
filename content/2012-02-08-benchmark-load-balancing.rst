
:title: Benchmark Load Balancing
:date: 2012-02-08
:tags: scripts, benchmarks

The Problem
===========

I am always looking for ways to turn boring school assignments to fun
projects . A recent one involves the execution of a handful of spec_
benchmarks for a random set of `gcc flags`_ for my Compiler Optimisation
course. This presents a slight challenge as the set of flags times the
repetition count for each benchmark adds up to a rather large total
execution time. Hence, parallelizing benchmark execution seemed
reasonable and a fun-enough challenge.

My initial approach was to load balance the benchmarks on the school's
16-core ``compute`` machine (2 Intel X5550 Quad Cores with
HyperThreading).  This boiled down to generating the necessary commands
--making sure each command is allocated to a specific core with
``taskset``-- and piping them to ``xargs -L 1 -P $(fgrep -c name
/proc/cpuinfo)``.  This approach produced very large deviations however.
This was partly because the machine was already under heavy load by
other users --so distributing the tasks to individual CPUs instead of
cores did not give significantly smaller deviations either.

My next thought was to schedule the benchmarks on regular login
machines, most of which sit idly during the night, or even most of the
day. Running on diverse hardware is not an issue --as long as a single
benchmark is executed on the same machine for all different flag
combinations-- since I don't care about relative performance across
benchmarks. One problem with this approach however, was that it involved
finding *properly functioning* machines which are unlikely to be used,
and noting down their host names.

The Solution
============

So I kept putting it off, until I became aware of a `cool undergraduate
project`_. The project lists all login machines and their availability
and could not have come at a more convenient time.

.. _cool undergraduate project: http://project.shearn89.com

I quickly wrote down a script to scrape the ``available`` page and
distribute each separate benchmark to one machine.

The ``distribute`` script finds available machines and submits a job via
ssh. For some reason I could not use `screen`_ or `tmux`_, since they
would get killed as soon as the ssh session terminated. So I had to
resort to good old ``nohup``. Furthermore, I opted for ``zsh`` on this one
since I was fed-up with the annoying idiosyncrasies of bash when it
comes to separator handling --I couldn't get the HOST array to split
properly on newlines.

distribute
  .. code-block:: bash
  
    #!/bin/zsh
    # Distribute computation across different machines.
    
    # Parse lines in the following format to retrieve hosts:
    # <span class='label label-success'>bazzini.inf.ed.ac.uk</span>
    HOSTS=($(wget -O - http://project.shearn89.com/available \
           | sed -rn '/label-success/ s/.*>([^<]+)<.*/\1/p'))
    i=1
    for src in $@; do
        host=${HOSTS[$(( i++ ))]}
        ssh -n $host "nohup ./runjob $src" &
    done

The ``runjob`` script simply changes into my project directory and
executes the benchmark, while logging some information like which
benchmark is matched to which machine.

runjob
  .. code-block:: bash
  
    #!/bin/bash
    
    src=$(basename $1)
    cd msc/copt1
    joblog="info.$src.$(hostname).log"
    date +'%F %T' >> $joblog
    cat /proc/cpuinfo  >> $joblog
    taskset -c 1 ./benchmark $@
    echo $? $@ >> "$joblog" 

The ``benchmark`` script is responsible for reading available flags,
compiling a benchmark and then executing it for a certain number of
iterations. After execution is complete, average runtime and standard
deviation are calculated with a simple awk_ script, ``stats.awk``.

benchmark
  .. code-block:: bash
  
    #!/bin/bash
    
    SRC="$1"
    DST="${2:-$(pwd)/out}"
    FLAGS="${3:-200-flags}"
    TIMES="${4:-12}"
    
    die() { echo $@; exit 1; }
    
    [[ -z "$SRC" ]] && die "usage: $0 src [dst=$DST] [flags=$FLAGS] [times=$TIMES]"
    
    name="$(basename $SRC)"
    dst="$DST/$name"
    mkdir -p "$dst" || die "Failed to create output directory $dst"
    
    log="$dst/run.log"
    buildlog="$dst/build.log"
    
    f=0
    while read flags; do
        (( f++ ))
        file="$(printf $dst/%03d.times $f)"
        run=$(printf "$(basename $SRC) %03d" $f)
        echo "$(date +'%F %T') $run $flags" | tee -a "$log" "$buildlog"
    
        make -s -C "$SRC/src" CFLAGS="$flags" 1>/dev/null 2>>$buildlog \
          || die "Failed to build $SRC"
    
        pushd "$SRC"
        for i in  $(seq $TIMES); do
            /usr/bin/time --output="$file" --append \
                          --format='r %e k %S u %U csi %c csv %w' \
                          ./run.sh 1>/dev/null 2>>$log
        done
        popd
    
        echo -e "$name\n$flags\n$(./stats.awk $file)" \
           | tee -a "$dst/results.txt" | tail -n 1
    done < $FLAGS

stats.awk
  .. code-block:: awk

    #!/usr/bin/awk -f
    
    /^r/ {
        # Sum kernel/user CPU time and convert to milliseconds.
        cpu = 1000 * ($4 + $6)
        sum += cpu
        ssq += cpu * cpu
        printf("%d ", cpu)
    }
    
    END {
        # Print a line with average runtime and standard deviation.
        avg = sum / NR
        var = ssq / NR - avg * avg
        printf("\n%.2f %.2f\n", avg, sqrt(var))
    }

There is no need to move files since I am taking advantage of AFS_, both
for the benchmark source and output files. Adding appropriate commands
to set up a proper environment on local storage should be trivial
however.

Future Work
-----------

The distribution scripts are a bit rough and assigning jobs to machines
randomly is not the best approach. For example, some machines are i3
Quad Cores at 3.0GHz, while others are dated Core 2 Duo at 1.8GHz. It
should be relatively straightforward to retrieve the specs of each
machine and assign benchmarks to machines with adequate performance and
no load --ideally such information should be provided in the original
listing though. For example, the following script generates such a list:

machines
  .. code-block:: bash
  
    #!/bin/bash
    
    stathosts() {
        while read host; do
            [[ "$host" = "Available" ]] && continue
            echo $host
            ssh -nT $host 'fgrep name /proc/cpuinfo; uptime; exit'
            echo
        done
    }
    
    wget -O - http://project.shearn89.com/available |\
    sed -rn '/label-success/ s/.*>([^<]+)<.*/\1/p'  |\
    stathosts > ${1:-host.stats}

It is then just a matter of turning this information into a usable
heuristic. The benchmarks could be also ranked slowest to fastest with a
script like the following:

rank
  .. code-block:: bash
  
    #!/bin/bash
    
    for src in $@; do
        make -C $src/src &>/dev/null
        pushd $src &>/dev/null
        /usr/bin/time --format='%e' --output=>(read t; echo $t $(basename $src)) \
                      taskset -c 0 ./run.sh &>/dev/null
        popd &>/dev/null
    done | sort -rgk 1 | cut -d' ' -f 2

Finally, the distribution script assumes there will always be more
machines than benchmarks, which might not always be the case.

Update (2012-02-24)
-------------------

I came up with a heuristic, a bit rough but does the job. It takes into
account the frequency of the CPU, a user-supplied weight based on its
type, and the system load. In the end, I decided to stick to a single
CPU type, so that my results were directly comparable across benchmarks.
To do that I just set all non-i3 multipliers to 0 in the following
``rankhost`` script.

rankhosts
  .. code-block:: bash
  
    #!/bin/bash
    
    # Set the multipliers depending on the processor model.
    FACTOR_I3=1.8 # Core i3
    FACTOR_CD=1.0 # Core 2 Duo
    FACTOR_C2=0.8 # Core 2
    
    cpu_core() {
        fgrep -c name /proc/cpuinfo
    }
    
    cpu_freq() {
        name=$(fgrep name /proc/cpuinfo | sed 1q)
        freq=$(echo $name | sed -r 's/.*@[ \t]+([0-9.]+)GHz$/\1/')
        case $name in
            *i3-*)
                factor=$FACTOR_I3 ;;
            *Duo*)
                factor=$FACTOR_CD ;;
            *)
                factor=$FACTOR_C2 ;;
        esac
        echo "( $freq * $factor )"
    }
    
    sys_load() {
        cores=$(cpu_core)
        loads=$(uptime | sed 's/.*load average://; s/,/ +/g')
        echo "( ($loads) / (3 * $cores) )"
    }
    
    echo "scale=4; $(cpu_freq) /  (10 * (0.1 + $(sys_load)))" | bc

I placed some the ranking code into a separate file --so as to easily
run the functions from the shell-- and modified the distribution script
accordingly.

functions
  .. code-block:: bash
  
    #!/bin/bash
    
    rank_spec() {
      for src in $@; do
        make -C $src/src &>/dev/null
        pushd $src &>/dev/null
        /usr/bin/time --format='%e' --output=>(read t; echo $t $(basename $src)) \
                      taskset -c 0 ./run.sh &>/dev/null
        popd &>/dev/null
      done | sort -rgk 1 | cut -d' ' -f 2
    }
    
    list_hosts() {
      wget -O - http://project.shearn89.com/available \
      | sed -rn '/Available/n; /label-success/ s/.*>([^<]+)<.*/\1/p'
    }
    
    rank_hosts() {
      list_hosts | while read host; do
        echo $(ssh -nT $host '~/rankhost; exit') $host
      done | sed '/^[^1-9]/d' | sort -rgk 1 | cut -d' ' -f 2
    }
    
    cached_hosts() {
      local h=hosts.cache
      [[ -e $h ]] && cat $h || { rank_hosts | tee $h; }
    }

distribute-new
  .. code-block:: bash
  
    #!/bin/bash
    
    source $(dirname $0)/functions
    
    HOSTS=($(cached_hosts))
    for src in $@; do
      host=${HOSTS[$(( i++ ))]}
      ssh -n $host "nohup ~/runjob $src 200-flags; exit" &
    done

To make sure any benchmark results were not affected by AFS or disk I/O
latency I further modified the `runjob` script to execute the benchmark
out of a RAM filesystem, under `/dev/shm/`.

runjob-new
  .. code-block:: bash
  
    #!/bin/bash
    
    dst=/dev/shm/mike
    results=$dst/results
    mkdir -p $results
    
    src=$1
    flags=$2
    
    cd ~/msc/copt1
    cp -r $src $flags $dst
    src=$(basename $src)
    flags=$dst/$(basename $flags)
    log="results/info.$src.$(hostname)"
    { date +'%F %T';
      cat /proc/cpuinfo;
      free -m; } > $log
    taskset -c 1 ./src/benchmark $dst/$src $results $flags 15
    echo $? $@ >> $log
    cp -r $results/* results && rm -rf $dst || echo "CLEAN UP FAILED"

.. _spec: http://www.spec.org/cpu2006/
.. _gcc flags: http://gcc.gnu.org/onlinedocs/gcc-4.4.5/gcc/Optimize-Options.html
.. _screen: http://www.gnu.org/software/screen/
.. _tmux: http://tmux.sourceforge.net/
.. _awk: http://www.gnu.org/software/gawk/
.. _AFS: http://en.wikipedia.org/wiki/Andrew_File_System

