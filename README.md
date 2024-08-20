Optimizes general Jigsaw structure generation as best I can:

- Replaces VertexShape unoptimized calls with a BoxOctree to make pieces only check nearby pieces for intersections instead of entire structure.

  - By default, vanilla uses a VoxelShape to store the bounds of pieces it is assembling for the structure layout. When it goes to add a new piece, it needs to see if it will fit into the layout without intersection any other pieces. The issue is VoxelShape is not good for this purpose as when checking if a VoxelShape intersects another VoxelShape, ALL the vertices are compared. In Jigsaw structures with lots and lots of pieces, this intersection check slows down greatly the more valid pieces are added to the layout. Can cause a lag spike when generating high number of pieces for recursive Jigsaw structures. The optimization here replaces the normal VoxelShape with a dummy VoxelShape that holds a BoxOctree implementation inside in order to pass around the BoxOctree to where it needs to be. This BoxOctree, when checking for intersections, will check only the nearby pieces to the incoming bounding box. Thus preventing the runaway growth in intersection checking time as it'll ignore the majority of pieces as they are too far away to matter for the check.

- Replaces Jigsaw target/facing match up with a slightly more optimized version.

  - Reduces the number of block properties grabbing by half. Vanilla calls getValue twice for each JigsawBlock to get top and front values from the same property when it could just grab it once and get all values it needs. Also simplify joint data grabbing to not need to be converted to an Enum with byName (not performant) and reordering checks to reduce amount of logic that needs to run often. This will help with Jigsaw structures that have a very high amount of Jigsaw Blocks in them as each Jigsaw Block runs through this matching code for all Jigsaw Blocks in the other structure pieces.

- Made any giant structure NBT that has no finalizeProcessing StructureProcessor now load much faster.
 
  - When a NBT structure piece goes to generate in a chunk, the ENTIRE NBT piece is loaded into memory, and then iterated over all the positions multiple times for StructureProcessors to apply and then later ignores all the positions outside the currently generating chunk. This is not very efficient (really wasteful) and causes large load times for giant NBT files. This mod's optimization works by doing the bounds check early to strip out all positions that we don't need before passing it to the StructureProcessors. However, any StructureProcessor that overrides finalizeProcessing method could require all the NBT positions to function correctly so this optimization is disabled for any pieces with these kinds of StructureProcessors which will be very few structures. In vanilla, only Trail Ruins will not get this optimization due to its use of the Capped StructureProcessor that overrides finalizeProcessing method.
  
- ???
