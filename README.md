# shellkit

Here I will write a brief explanation of a set of scripts that I commonly use in
my linux system, two of them maybe considered more like programs (``hw-cache``,
``sk-btrfs-send``) as they are a bit sophisticated in the way they get their job
done.

### hw-cache

``hw-cache`` is a file system scanner for mass hashsum collection and management,
it creates and keeps updated sort of database-like text files, that are smartly
divided per respective block device and as occurrence (sub)splitted into smaller
sub-lists when scans are performed into nested directories of already cached
content.

Its input can be fed by command line (or ``stdin``) by list of files and/or
directories (``-f | -d``) to scan and/or by files containing lists of them (``-F
| -D``), then an optimized internal algorithm extracts the most significant path
information and by cross filtering it with system mountpoints information fastly
matches the respective physical device of any file.

By indentifying the pertaining physical device per file, device specific scan
lists and master databases (UUID dev tagged) are created/maintained so that any
deep directory scan and most importantly any hashsum calculation is done
parallelized at device level.

``hw-cache`` has 5 operation modes: ``scan``, ``update``, ``query``, ``clean`` 
and ``wipe``.\
Query is the default one, since it's typically wasteful to rescan every time.

##### Examples
File hashsum scan/update, parallel scan/hash kicks in since paths refers to
different block devices:

``hw-cache -u /data/docs /run/media/MyUSBdrive``

Hashsum query:

``hw-cache /data/docs/personal``

Scan update, verbosed one line per file, with mixed input:\
``-f`` switches file names parsing mode (default is ``-d``)\
``-F`` switches files list parsing.

``hw-cache -vvu /data/images -f /data/backup/stuff.tar.gz /vm-pool/win*.img -F
/data/filelist.txt``

### FRPO (File Recovery Priority Organizer)

FRPO is a set of scripts to help for a certain (not so uncommon) data recovery
scenario, regarding those defective, pre-failure disk drives that still give you
some sort of (weak or such) file system access.
In these cases, being impossible to evaluate the device remaining lifespan, the
logical approach is to make a whole file list of everything is intended to
recover and arrange its retrieval order in a compromise fashion between what is
more strongly wanted and what is supposed to be more likely grabbed because of
its small footprint.

Consider this scenario:
- Your crucial data just sits on a mere 1-5% of disk space
- Even with hiccups you can somewhat access/surf/read the file system
- Disk gets progressively slower and more jerky, looks prone to die fast if
abused

Useful points to consider in a data recovery from defective disk (not broken
file sytem):

- Sometimes typical All-In-One Recovery Solutions (AIORS) require a massive
surface scan to approach content selection screen

- AIORS doesn't track what has been recovered and what has not within subsequent
executions (as something goes wrong you'll be forced to restart the program and
of course its logic will start from scratch too!) and you don't have a pratical
file list in your hands to surf/edit/split

- AIORS on a unrecoverable read error doesn't have a strategy to track what has
been successfully red from a file and what has not, chance is only to restart
from scratch on next execution

- You feel the need to recover with an atomic file-by-file approach and avoid
losing time by seeing repeatedly on screen entries of what has been done
(indeed, what about having a progressively shrinking to-do-list paired with a
growing done-list?).

- Having control over composing the recovery list order: some easy list
splitting/grouping tool organized by categories and last but not least the
ability to edit by hand

As a leitmotiv, what you aim and wish is to leverage your disk the less as
possible and in the most targeted way, since any single read request brings it
ahead of time closer to the final failure and complete unreadability.

The key points sequence to avoid overloading your defective disk are:

- By mind: take note **ahead of time** and in **order of importance** of the
real needed directories to scan for content recovery.

- A brief surf by hand: this permits a much more refined directory selection
over previous step, so for the future steps of scanning/reading means skipping a
lot of unwanted and time consuming content (and much less dangerous/wasteful
load to our disk).

- Command line browsing (``cd <dir>``, ``cd ..``, ``ls -latr`` ...) for
directory tree investigation is by a wide margin the most effective and least
invasive way (no file content peeking as file managers do).

- Once noted your master directories to start with, running a scan job inside
these to collect the full list of files in one big (and backupped) text file,
from that moment you can just view/surf your grabbed list instead of dangerously
surfing back and forth through the physical mount of the defective disk.

- Being a simple text file, the list can be easily splitted, pruned, reordered
prioritizing the most wanted files so that we maximize the chance to fully get
them once we pass this list to a specific script that just save to a safe
destination, just following the input order.

So what does FRPO:

``frpo-1metascan`` searches into paths specified as arguments to collect files
metadata (date/size/pathname) and outputs a FRPO-formatted text. The best
effective way to run it is from a base directory of the source while using
relative paths in the arguments, since the same relative hierarchy later will be
mirrored under the recovery dir.

``frpo-2extats`` doesn't do any mandatory step, it's a simple but very useful
statistical tool to summarize file list composition grouping them by extension
and sorting by space utilization.
Just run it passing one or more (FRPO) lists by stdin or arguments.

``frpo-2organize`` takes one or more lists as input and generates a splitted
output of (up to) 9 list files each one representing a file format category,
named starting with a number substantially hinting (not forcing) its
statistical/priority relevance.

``frpo-3sync`` finally does the dirty job of files retrieval, one by one, from
the defective disk. Internally it manages 3 operational modes/phases (0..2), by
default starts using rsync, if this phase ends with some errors/fails then the
subsequent phase, using ddrescue, kicks in to retry recovery of missing files.
If you already know this solid tool, well here it's called to do at file level
that kind of work that in the 99% of times is intended to be done for entire
disks/partitions.
During ddr mode, 2 temporary files (image and map) per entry are created
and maintained until its job is done, if the script gets interrupted (for ex. by
user) the progress is never lost and on next run can be continued.\
Script takes at least 3 arguments:
- Source dir: that one initially used as base for ``frpo-1metascan`` scanning
process
- Recovery dir: destination to host recovered source content, having same
hierarchy and holding partial/fully recovered files
- One or more metadata lists that internally instructs which files to recover
(relative path) and the priority implicitly by the mere line order

``frpo-4purge`` is commonly useful to run after a fruitful ``frpo-3sync``
execution, as it scans the recovery dir, matches the result with each passed
list and when needed moves out the fully recovered entries in a new ``.done``
list. Now for each list we have an updated one shortened by removal of full
recovered entries and a new one storing all these stripped entries.\
Takes 2 or more arguments, 1st is the recovery dir, from 2nd can be passed any
metadata lists to be stripped.

##### Example

Let's build from scratch an example for a FRPO application, taking an
hypothetical defective HDD coming from a Windows machine, let's mount read-only
the partition where we have the data to recover:
```
$ sudo mkdir /mnt/prefail
$ sudo mount /dev/sdz1 /mnt/prefail -o ro,uid=1000,fmask=117,dmask=7,noatime
```
We know that the files to recover are under the ``user`` dir, checking the
``Users`` dir to be sure that there are no other ones to include
```
$ cd /mnt/prefail/Users
$ ls -la

drwxrwx--- 1 sandro root  4096  4 set  2020  .
drwxrwx--- 1 sandro root 12288  5 set  2020  ..
lrwxrwxrwx 2 sandro root    19 14 lug  2009 'All Users' -> /mnt/prefail/ProgramData
drwxrwx--- 1 sandro root  8192  4 set  2020  Default
lrwxrwxrwx 2 sandro root    21 14 lug  2009 'Default User' -> /mnt/prefail/Users/Default
-rw-rw---- 1 sandro root   174 14 lug  2009  desktop.ini
drwxrwx--- 1 sandro root  4096 12 apr  2011  Public
drwxrwx--- 1 sandro root  8192  4 set  2020  user

$ cd user
$ ls -la

drwxrwx--- 1 sandro root   8192  4 set  2020  .
drwxrwx--- 1 sandro root   4096  4 set  2020  ..
drwxrwx--- 1 sandro root      0  4 set  2020  AppData
drwxrwx--- 1 sandro root      0  4 set  2020  Contacts
drwxrwx--- 1 sandro root      0  4 set  2020  Desktop
drwxrwx--- 1 sandro root      0  4 set  2020  Documents
drwxrwx--- 1 sandro root      0  4 set  2020  Downloads
drwxrwx--- 1 sandro root   4096  4 set  2020  Favorites
drwxrwx--- 1 sandro root      0  4 set  2020  Links
drwxrwx--- 1 sandro root      0  4 set  2020  Music
-rw-rw---- 1 sandro root 786432  5 set  2020  NTUSER.DAT
-rw-rw---- 1 sandro root     20  4 set  2020  ntuser.ini
drwxrwx--- 1 sandro root      0  4 set  2020  Pictures
drwxrwx--- 1 sandro root      0  4 set  2020 'Saved Games'
drwxrwx--- 1 sandro root      0  4 set  2020  Searches
drwxrwx--- 1 sandro root      0  4 set  2020  Videos
```
We know that typical user stuff gets stored into these dirs:
``Desktop``, ``Documents``, ``Pictures``, ``Music`` and ``Videos``.\
We can surf inside to do a more refined view and know better what exactly pick
and what not, but to do something effective we would need to run for ex. a space
utilization query command (``du -sh``) among some dirs and this could easily
result in a quite demanding and wasteful process for (the health of) our disk,
given the fact that we still have to perform the **mandatory file scan** job to
build the list of files to recover.\
Anyway FRPO is made with the idea of getting the list for first and offloading
the defective disk from repeated surfing by focusing work into list filtering,
splitting and rearranging (user can manually edit and sort entries inside) in a
semi-assisted way so that what counts the most is being put in advance to be
grabbed as soon as possible.\
So let's start the metadata scan:

``$ frpo-1metascan Desktop Documents Pictures Music Videos >
/safe/recovery/main.ls``

After minutes of clicking noise and hiccups coming from the HDD finally we have
our main file list inside the main recovery dir ``/safe/recovery``, now by
running this command line we can also produce a sort of summary of the
composition of files to recover:
```
$ cd /safe/recovery
$ frpo-2extats main.ls

Count   Disk space  Extension
-----   ----------  ---------
  245  59700032760  mp4
    1  15386326212  mov
   12  14986062726  avi
 1514   9574957675  mp3
    4   7060105192  mkv
 6255   6408956127  jpg
 5722   2312081416  pdf
    1    932648071  3gp
  137    481149637  m4a
  990    224908911  docx
   50    220995862  zip
 2887    210022469  doc
   43    172978690  wma
   71    162895691  pptx
   46    147119616  ppt
    4    107666340  rar
   61     89651200  pps
  655     87096884  xls
   89     54477463  (no extension)
  776     49848562  jpeg
  153     38343050  png
  714     37419983  kar
   11     25240986  exe
   23     24674451  p7m
   40     22841590  eml
   44     19462336  mht
  488     17180231  xlsx
  292     15880912  mid
   67      8449128  rtf
   10      6725632  accdb
   15      2858897  html
   62       862729  js
    1       755200  pub
   19       343295  xml
    3       234811  midi
    1       192512  mdb
   11       160932  odt
    2        33297  ods
    6         5043  txt
```
At this point the text file ``main.ls`` having a big redundant heap of entries
inside may suggest us two ways: edit the list directly to examine/strip/reorder
parts or first find an automated way to split it in smaller sub-lists following
a category based criterion.\
This last approach is what helps to do the script ``frpo-2organize``, running it
with one or more lists as argument produces a splitted output of up to 9
categories:
```
$ frpo-2organize main.ls
$ ls -1 *.ls

10documents.ls
20graphics.ls
30images.ls
40archives.ls
50audio.ls
60video.ls
70system.ls
80diskimg.ls
90stuff.ls
main.ls
```
With this just simple step we can count that most of times our needed files will
be concentrated in the first 3 categories.\
Given that we can start a first run of the recovery with the command line:
```
$ mkdir data
$ frpo-3sync /mnt/prefail /safe/recovery/data [1-3]*.ls
```
But why only these? The first run is also important to probe (for user
evaluation) to which extent our disk is damaged/responsive, as the recovery
script will optionally descend into deeper recovery modes (asking stressful
retries) after the 1st pass of all entries has been tried, so it will be smarter
to NOT start by instructing a full recovery job.

If we want avoid deeper modes, for ex. to make the duration less uncertain and
have a more predictable/limited hardness on disk we can use ``-f`` option like
in this example, oriented on probing the disk for first time:
```
$ frpo-3sync -f /mnt/prefail /safe/recovery/data 10documents.ls
```








