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

FRPO is a set of scripts to help for a common data recovery problem, when you 
need to get the essential data from those defective, pre-failing 
disk drives that still give you (a weak or such) file system access and by its 
behaviour you sense there is no way to perform that long massive surface scan 
needed by the typical recovery software (for ex. ddrescue, photorec, R-Studio, 
Ontrack...) as the unit "smells of self-destruction".

Premise: FRPO approach mostly suits HDDs (rotational disks) as SSDs usually 
behave quite differently when they face an important degradation or partial 
failure, however as it dramatically minimizes device load/usage may turn helpful 
in many different scenarios.

So consider the case:
- your crucial data just takes 1-5% of disk space
- even with hiccups you can somewhat access/surf file system
- disk is about to die by the way it behaves
- would you really perform the typical recovery software full surface scan?

In these cases what you want/wish is to leverage your disk the less as possible 
and in the most targeted way, since any single read or write request brings it 
ahead of time close to the final failure and complete unreadability.

The key points to avoid overloading your defective disk are:

- Take note **ahead of time** and in **order of importance** of the real needed
directories to scan for content recovery

- Surf by hand in those ones, command line browsing (``cd <dir>``, ``cd ..``, 
``ls -latr``, etc...) avoids file content peeking. So if it's the case, this 
permits a more accurate directory annotation and so refines 1st point above
- So, with respect of what just said: if the device is somewhat surfable, what 
do we really need to surf 1st?
- As being dangerous and unneeded, keep in mind to avoid to repeatedly surf that 
dirs, do it just 1 time in a profitable definitive way
- And here how we will do it is: do a scan job starting from 1 or more master 
dirs to just collect all files path, name, size and date in a 
precious list file so that can be further examined, splitted, elaborated 
infinite times





























