# pvshrink

pvshrink automates the invocation of pvmove to condense PV usage at the start
of the volume, and then invokes pvresize to shrink a PV to its minimum possible
size.

## Usage

```
   pvshrink /path/to/pv
```  

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

## Example

In the example below, the 3g PV can't be resized by 100m despite having 212m free, because it is fragmented:

```
# pvs -o +pv_used /dev/vda2 
  PV         VG     Fmt  Attr PSize PFree   Used 
  /dev/vda2  fedora lvm2 a--  3.00g 212.00m 2.79g
# pvresize --setphysicalvolumesize 2.9g /dev/vda2 
  /dev/vda2: cannot resize to 742 extents as later ones are allocated.
  0 physical volume(s) resized / 1 physical volume(s) not resized
```

pvshrink can defragment it:

```
# ./pvshrink /dev/vda2 
Moving 50 blocks from 714 to 664
  /dev/vda2: Moved: 4.00%
  /dev/vda2: Moved: 100.00%
50 of 50 (100.00%) done
Defragmentation complete.
Metadata size: 1048576 b
PE size: 4.0 MiB
Total size 1048576 b + 714 x 4194304 b = 2995781632 b (2.8 GiB)
    Wiping internal VG cache
    Wiping cache of LVM-capable devices
    Archiving volume group "fedora" metadata (seqno 15).
    /dev/vda2: Pretending size is 5851136 not 6287360 sectors.
    Resizing volume "/dev/vda2" to 5851136 sectors.
    Resizing physical volume /dev/vda2 from 0 to 714 extents.
    Updating physical volume "/dev/vda2"
    Creating volume group backup "/etc/lvm/backup/fedora" (seqno 16).
  Physical volume "/dev/vda2" changed
  1 physical volume(s) resized / 0 physical volume(s) not resized
Minimum partition size is 2995781632 b = 5851136 x 512 b sectors
```
The last line tells us the minimum size in sectors that we could resize the partition to.

Note that pvshrink also avoided the annoying problem of calculating the minimum possible PV size:

```
# pvs --units b -o pv_name,pv_used /dev/vda2 
  PV         Used       
  /dev/vda2  2994733056B
# pvresize --setphysicalvolumesize 2994733056B /dev/vda2 
  /dev/vda2: cannot resize to 713 extents as 714 are allocated.
  0 physical volume(s) resized / 1 physical volume(s) not resized
```
Note that 2994733056B is 714 x 4MB, but pvresize needs to allow space for metadata.  pvshrink resizes to 2995781632B.

## Use at your own risk

Use of this tool is entirely at your own risk.  In particular, you should perform your own sanity checks on the PV size before resizing a partition containing the PV.

(c) 2018 [Mythic Beasts](https://mythic-beasts.com/)
