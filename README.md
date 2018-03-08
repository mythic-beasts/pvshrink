# pvshrink

LVM provides the the pvresize tool which can be used to grow or shrink the size
of an LVM Physical Volume, but pvresize can only reduce the size of a PV if
there is free space available at the end of the PV.  If usage of the PV is
fragmented, for example, as a result of deleting an LV that was using space at
the beginning of the partition, the extents within the PV must first be
reshuffled using pvmove so that they are all at the beginning.  

pvmove can't move segments between overlapping ranges, so it can often take
many invocations of pvmove to fully condense PV usage.

pvshrink automates the invocation of pvmove in order to place all used extents
at the beginning of the PV.

pvshrink also calculates the minimum possible size for the volume (taking into
account metadata space) and invokes pvresize using this.
