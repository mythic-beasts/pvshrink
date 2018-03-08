# pvshrink

LVM provides the the pvresize tool which can be used to grow or shrink the size
of an LVM Physical Volume.  pvresize can only reduce the size of a PV if
there is free space available at the end of the PV.  If usage of the PV is
fragmented, for example, as a result of deleting an LV that was using space at
the beginning of the partition, the extents within the PV must first be
reshuffled using pvmove before the space can be reclaimed using pvresize.

pvmove can't move segments between overlapping ranges, so it can often take
many invocations of pvmove to fully condense PV usage.

Shrinking a PV requires the desired size to be specified to pvresize.  Calculating 
the minimum possible size isn't entirely trivial, as you need to allow some space 
for LVM metadata 

pvshrink automates the invocation of pvmove in order to place all used extents
at the beginning of the PV.

pvshrink also calculates the minimum possible size for the volume (taking into
account metadata space) and invokes pvresize using this.

```
Usage:

   pvshrink /path/to/pv
```  
   
