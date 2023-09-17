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
from that moment you realize that you can just view/surf your grabbed list
instead of dangerously surfing back and forth into the physical device.



























