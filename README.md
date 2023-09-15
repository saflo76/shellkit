# shellkit
Here I will write a brief explanation of a set of scripts that I commonly use in my linux system, two of them maybe considered more like programs (``hw-cache``, ``sk-btrfs-send``) as they are a bit sophisticated in the way they get their job done.

### hw-cache
``hw-cache`` is a file system scanner for mass hashsum collection and management, it creates and keeps updated sort of database-like text files, that are smartly divided per respective block device and as occurrence (sub)splitted into smaller sub-lists when scans are performed into nested directories of already cached content.

Its input can be fed by command line (or ``stdin``) by list of files and/or directories (``-f | -d``) to scan and/or by files containing lists of them (``-F | -D``), then an optimized internal algorithm extracts the most significant path information and by cross filtering it with system mountpoints information fastly matches the respective physical device of any file.

By indentifying the pertaining physical device per file, device specific scan lists and master databases (UUID dev tagged) are created/maintained so that any deep directory scan and most importantly any hashsum calculation is done parallelized at device level.

``hw-cache`` has 5 operation modes: ``scan``, ``update``, ``query``, ``clean`` and ``wipe``.\
Query is the default one, since it's typically wasteful to rescan every time.

##### Examples
File hashsum scan/update, parallel scan/hash kicks in since paths refers to different block devices:

``hw-cache -u /data/docs /run/media/MyUSBdrive``

Hashsum query:

``hw-cache /data/docs/personal``

Scan update, verbosed one line per file, with mixed input:\
``-f`` switches file names parsing mode (default is ``-d``)\
``-F`` switches files list parsing.

``hw-cache -vvu /data/images -f /data/backup/stuff.tar.gz /vm-pool/win*.img -F /data/filelist.txt``

### FRPO (File Recovery Priority Organizer)
FRPO is a set of scripts to help for a common data recovery problem and, I would say **priority**, 
when you absolutely need to recover the essential data from those almost dead prefailing disk drives.\
The goal that FRPO tries to achieve mostly matches HDDs (rotational disks) since SSDs usually 
behave quite differently when they face an important degradation or partial failure.

In these cases what you wish is to leverage your disk the less as possible and the most focused as possible, 
since any single read (or accidental write), wasted or not, brings it progressively closer
to the total failure and complete unreadability.





























