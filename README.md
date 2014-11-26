js-bv-sampling
==============

Javascript read depth and random sampling on bai (bam) and tba (vcf/tabix) index file and binary readers


```javascript
function estimateCoverageDepth (indexReader, cb)
```
An estimator of coverage depth of corresponding data file (for the
index reader - bam or vcf) which uses bytes in region segments as the
indicator.  Returns the estimations as a map M of vectors of 'segment
information' maps, where each key in M is a reference id and each
corresponding value is a vector [seg-info-map, ...] of maps which each
map is

```javascript
{relBinNum: (segment-bin-num - start16kbBinid),
 pos: (relBinNum * 16KB),
 depth: total-byte-count}
```

The estimate of coverage depth is based on the number of bytes within
a chunk of the file.  This is computed by taking all the leaf bins
(the bins which cover 16kb regions, ranging from binid 4681 to 37449),
taking the chunks they contain (any actual data within their 16kb
region) and summing the total byte count.  The relBinNum is a leaf bin
ordinal value relative to start16kbBinid = 4681.  Pos then is this
ordinal times 16kb = byte offset of 16kb region.  Depth is the total
byte count in chunk.


```javascript
function mapSegCoverage (indexReader, refid, fn, keepNils) {
```
Mapping function over the estimate read coverage segments of
reference REFID.  See estimateCoverageDepth for more on segments.
Requires that estimateCoverageDepth has been run; if not will
implicitly run it.  FN is a function of a segment information map.
For the sq of segment information maps for REFID [segmap...],
returns the sq [fn(seqmap)...].  If keepNils is true, returns fn
results which are nil (undefined), otherwise, returns only non nil
results.


```javascript
function samplingRegions (refs, options) {
```
For a sq of reference objects REFS (as obtained from file header
information), and option set OPTIONS (a map of parameter options),
obtains a set of region samples for each reference in refs.  The
resulting set is intended to be a 'representative sample' of the
base file.

Tuning parameters in options are binSize, the max size covered by a
bin; binNumber, the max number of bins to consider for a ref;
start, the starting position of the sequence associated with a ref
for a sampling.  Defaults are binSize = 40000 bases, binNumber =
20, start = 1.

Returns a map {regions: [reginfo...], regstr: json-string-for-regions}

where reginfo = {name: ref.name, start: int, end: int} and regions
is sorted ascendingly by name by numerical start


```javascript
function new2oldRefs (refs) {
```
Convert ref objects to old format.  Servers should be upgraded to
use the actual bai index names, but this will provide interim
support.  Takes a sq REFS of reference objs, each of format {name:
string, l_ref: int, l_name: int} and converts to sq of objs of
format {name: string, end: int}


```javascript
function bamStatsAliveSamplingURL (refs, options, bsaliveURL) {
```
Obtain a bamstats-alive url encoding for a region sampling of REFS
a sq of reference objects as obtained from file header information.
Performs a samplingRegions operation to obtain the region
information and then generates a corresponding url for the
bamstats-alive server.  OPTIONS is as for samplingRegions.
bsaliveURL is the base http address for a bamstats-alive server.
