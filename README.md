# pvshrink

pvshrink automates the invocation of pvmove to condense PV usage at the start
of the volume, and then invokes pvresize to shrink a PV to its minimum possible
size.

## Background 

LVM provides the the pvresize tool which can be used to grow or shrink the size
of an LVM Physical Volume.  pvresize can only reduce the size of a PV by removing
free space the end of the PV.  If usage of the PV is
fragmented, for example, as a result of deleting an LV that was using space at
the beginning of the partition, the extents within the PV must first be
reshuffled using pvmove before the space can be reclaimed using pvresize.
Depending on the amount of fragmentation, multiple calls to pvmove may be
required.

Shrinking a PV requires the desired size to be specified to pvresize.
Calculating the minimum possible size isn't entirely trivial, as you need to
allow space for LVM metadata 

pvshrink automates the invocation of pvmove in order to place all used extents
at the beginning of the PV and then calculates the minimum possible size for
the volume (taking into account metadata space) and invokes pvresize using
this.

## Usage

```
   pvshrink /path/to/pv
```  
   
