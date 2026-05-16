# The idea
We make use of `api.isNearInterrupt` and `return`, and automatically asign breakpoints inside of loaded code. The breakpoints check for a callback-specific counter and increment the counter by one on completion. This ensures minimal loss.

# How a completed version should work
## On the side of the user:
1. The player places down code blocks and writes their coordinates into World Code after pasting in this tool. Their code runs without interruptions.

# The progress
- [x] Added breakpoints<br>
- [x] Added some automatic wrapping<br>
- [ ] Fully added automatic wrapping<br>
- [ ] Code loader-integrated<br>
- [ ] Compatability with all callbacks/integrate all callbacks into tick

# Known bugs (report via issues or pull requests)
