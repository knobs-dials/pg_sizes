# Intro

Report the size of each database, and how much is in regular tables, TOAST, and indices.

Report storage size per column as well (without the regular/TOAST split).

WARNING: so, I misunderstood what each function does exactly. Update pending


# Example

     # pg_sizes
          'dataplot'  total size:   16MB, of which    table 12.7MB    toast  459KB    indexes  2.5MB    other  8.2KB
             'music'  total size:  5.1GB, of which    table  3.1GB    toast  1.1GB    indexes    1GB    other   33KB
            'quotes'  total size:  6.4MB, of which    table  2.8MB    toast  492KB    indexes  3.1MB    other     0B
             'solar'  total size:  537MB, of which    table  348MB    toast  459KB    indexes  189MB    other     0B
     Total size of databases: 5.7GB


    # pg_sizes -d quotes -c -p
    TABLE 'quotes'.'quotes'
      base table size      631K
           TOAST size       41K
           index size      737K
    Number of rows: 3156
      NAME                              TYPE      DISK_SIZE                DISK_AVG     DATA_SIZE      DATA_AVG   COMPR
      quote                             text         432179 (   0 MB)         136.9        426875         135.3    101%
      quoter                            text          31937 (   0 MB)          10.1         28781           9.1    110%
      source                            text          16492 (   0 MB)           5.2         13324           4.2    123%
      viewed                         integer           6240 (   0 MB)           4.0
      proposed                       boolean
    Total size of databases: 8.7MB


 Where the last few columns are there to point out something similar to 
 [disk use versus apparent size](https://duckduckgo.com/?q=disk+use+versus+apparent+size).
 - which is different mainly due to overheads and TOAST compression
 - DISK_SIZE is space taken on disk (via pg_column_size)
 - DISK_AVG is that divided by row count
 - DATA_SIZE is the total uncompressed size (via octet_length, only applies to some types)
 - DATA_AVG is that divided by row count
 - COMPR is DISK_SIZE / DATA_SIZE to show  overheads, and possible TOAST compression
 
# Options

```
Usage: pg_sizes [options]

Options:
  -h, --help            show this help message and exit
  -d DBSUBSTR, --dbname-substring=DBSUBSTR
                        Only report databases containing given substring. Can
                        supply multiple with commas.
  -p, --precise-rowcount
                        By default we get the row count estimate via VACUUM
                        statistics, which can make the column sizes a little
                        imprecise. This forces a count(*), which is slower but
                        precise.
  -c, --column-sizes    Estimate size use per column. NOTE: this basically
                        reads all data, so is slow and will mess with your
                        caches
  -t TABSUBSTR, --table-substring=TABSUBSTR
                        Only show column sizes for tables containing given
                        substring. Can supply multiple with commas.
```


# Notes

The database summary comes entirely from data in [pg_class](https://www.postgresql.org/docs/9.6/catalog-pg-class.html) for each database. 

The column stuff uses [pg_column_size and octet_length](https://www.postgresql.org/docs/9.6/functions-admin.html#FUNCTIONS-ADMIN-DBOBJECT), and some assumptions about how those relate to TOAST and indices, which is probably at least part of why they seem a little different.


# Caveats

It looks like pg_class isn't always updated on vacuums, so this may be significantly off (after bulk inserts?).

Proof of concept version, contains various hardcoded assumptions.
I still need to reread the docs on the various size functions to ensure it means precisely what I think it means.

The column summaries need to read a lot of table data. Which is slowish, and for databases larger than RAM will probably shred your nicely warmed caches.

Needs access to each database, so:
- assumes we connect as postgresql role `postgres`
- assumes that's trusted on localhost via pg_hba (so no username)
- assumes that's the admin and can read everything

Also, you're running someone else's code on your database.


# TODO / CONSIDER

- Make the distinction of disk use versus data size in more places

- maybe don't count on pg_class (used only for the overview) being correct - it can be off by a ''lot'' until the next vacuum

- Output as JSON, or something else parsable

- A 'forced index warming' mode or such, because we have code that matches filenames to relations so it wouldn't be a stretch to have 

- It may be useful to show disk use rather than apparent size, by stat()s to the filesystem. The code for it is commented out because it has to assume you have filesystem permissions, which is a point of failure you don't have using pg_class.

- Clean up code, do some things more properly.
