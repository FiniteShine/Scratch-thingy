# Initial bug review

The original `.sb3` is unchanged. This review excludes the already-known neighbor boundary issue from the main findings.

## High: smoothing reverses the surrounding terrain

In **Smooth By Raw Value**, `tileIsLand?` is assigned `neighbor value < landFrequency`. But **Set Tiles** maps values below that threshold to **Water**, not Land. The true branch adds 0.4; the false branch subtracts 0.4. Consequently water neighbors push the center toward land and land neighbors push it toward water.

Confirmed by executing the actual project blocks in Scratch VM with uniform 40×30 maps, checking an interior tile:

- All values 0.1 (water): center becomes 3.3, classified as land.
- All values 0.9 (land): center becomes -2.3, classified as water.

Suggested correction: subtract 0.4 for below-threshold neighbors and add 0.4 otherwise, or reverse the comparison and keep the branch weights. Rename the Boolean to match its meaning. This preserves the current weighting approach; it does not turn it into an averaging filter.

## Medium: the Seed input is unused

**Generate World (Seed)** has a custom-block input named Seed, but its RNG assignment reads the sprite variable `Seed`, not the custom-block argument reporter. Calling it with another seed will still use the variable. **Init** also randomizes the variable on every green flag, so setting a seed before starting does not reproduce that map.

Suggested correction: assign RNG from the custom-block argument, and provide a separate deliberate action for choosing a random seed. The generator itself is deterministic when initialized with the same valid seed.

## Startup behavior

Generation waits for Space before **Set Tiles** and the first render. The previous picture remains until then. This may be intentional, but a first-time user can mistake it for a hang. Documented in the README; consider rendering the raw map first if that is the intended preview.

## Known boundary issue

Offsets operate on a flat list: left/right checks can wrap between rows; checks beyond the list return empty items. Excluded from the main review as requested. A future fix should validate neighbor row and column coordinates, not only list indices.

## Validation

- ZIP integrity passed; project JSON parsed successfully.
- Actual Scratch VM logic completed the green-flag workflow after a simulated Space key press, with 1,200 world values and 1,200 tile values.
- Raw Generation produced 1,200 values strictly between zero and one for seeds 1, 622937, and 1000000. Two runs per seed produced identical lists.
- Executed actual smoothing and tile-classification blocks for the two uniform-map cases above.
- The smoother uses a separate output list, so changes do not leak into later neighbor reads within the same pass.

Checks ran headlessly with scratch-vm, without a renderer or asset storage. Costume appearance, stamping, sound, and interactive editor behavior were not visually tested. This is a focused review, not proof that every possible input is bug-free. Seed zero or a multiple of 2147483647 degenerates to all zeros; the current Init range avoids that case.
