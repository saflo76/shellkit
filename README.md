# shellkit
Here I will write a brief explanation of a set of scripts that I commonly use in my linux system, two of them maybe considered more like programs (``hw-cache``, ``sk-btrfs-send``) as they are a bit sophisticated in the way they get their job done.

### hw-cache
``hw-cache`` is a file system scanner for mass hashsum collection and management, it creates and keeps updated a sort of database-like text files, that are smartly divided per respective block device and if necessary sub-splitted in smaller sub-lists when scans are performed in sub-directories.
Its input can be fed by command line (or ``stdin``) by list of files or directories (``-f | -d``) to scan or by file lists of them (``-F | -D``)
