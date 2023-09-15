# shellkit
Here I will write a brief explanation of a set of scripts that I commonly use in my linux system, two of them maybe considered more like programs (``hw-cache``, ``sk-btrfs-send``) as they are a bit sophisticated in the way they get their job done.

### hw-cache
``hw-cache`` is a file system scanner for mass hashsum collection and management, it creates and keeps updated some sort of database-like text files, that are smartly divided per respective block device and if necessary sub-splitted in smaller sub-lists when scans are performed into sub-directories of already cached content.

Its input can be fed by command line (or ``stdin``) by list of files and/or directories (``-f | -d``) to scan and/or by files containing lists of them (``-F | -D``), it features an optimized internal algorithm to extract the most significant path information from the given input and filtering it with system mountpoints information to fastly detect respective block device matching.

By indentifying the pertaining physical device per file, device specific scan lists and master databases (UUID dev tagged) are created/maintained so that any deep directory scan and most importantly any hashsum calculation is done parallelized on a device
