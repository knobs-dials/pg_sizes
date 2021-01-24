# pg_sizes

Summarize sizes of each database, splitting mainly into
- regular tables
- TOAST tables
- indices 
- other


Can also reports use per column. 
Optional, because this basically needs to read all data and will mess with your cacheing.


# Caveats

Contains various hardcoded assumptions.  I need to read up on more detail.
