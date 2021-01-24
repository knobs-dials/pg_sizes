# pg_sizes

Summarize sizes of each database, splitting mainly into
- regular tables
- TOAST tables
- indices 
- other


Can also reports use per column. 
Only when you ask, because this reads most data so will likely mess with your cacheing.


# Example

     # pg_sizes
          'dataplot'  total size:   16MB, of which    regulartable: 12.7MB    toast  459KB    indexes  2.5MB    other/unknown:  8.2KB
             'music'  total size:  5.1GB, of which    regulartable:  3.1GB    toast  1.1GB    indexes    1GB    other/unknown:   33KB
            'quotes'  total size:  6.4MB, of which    regulartable:  2.8MB    toast  492KB    indexes  3.1MB    other/unknown:     0B
             'solar'  total size:  537MB, of which    regulartable:  348MB    toast  459KB    indexes  189MB    other/unknown:     0B
     Total size of databases: 5.7GB


    # pg_sizes -s quotes -c -p
    TABLE 'quotes'.'quotes'
      base table size    616 kB
           TOAST size     40 kB
           index size    712 kB
    Number of rows: 3155
      NAME                                            TYPE       SUM_SIZE                AVG_SIZE     SUM_OCTET  AVG_OCTETSIZE   COMPR
      quote                                           text         432090 (   0 MB)         137.0        417 kB         135.3    101%
      quoter                                          text          32371 (   0 MB)          10.3         29 kB           9.3    110%
      source                                          text          16779 (   0 MB)           5.3         13 kB           4.3    123%
      viewed                                       integer           6252 (   0 MB)           4.0
      proposed                                     boolean           3155 (   0 MB)           1.0
    Total size of databases: 6.4MB

# Options


# Caveats

Proof of concept version, contains various hardcoded assumptions, and I may have misread documentation.  I'm sure the sizes are approximate  I need to read up.

You need to be able to stat the database's actual files on filesystem. (TODO: figure out if we can do it from the database itself - see exact meaning of relpages)

