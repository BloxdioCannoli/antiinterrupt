# The idea
We make use of `api.isNearInterrupt` and `return`, and automatically asign breakpoints inside of loaded code. The breakpoints check for a callback-specific counter and increment the counter by one on completion. This ensures minimal loss.

# The progress
- [x] Added breakpoints<br>
- [x] Added some automatic wrapping<br>
- [ ] Fully added automatic wrapping<br>
- [ ] Code loader-integrated<br>
