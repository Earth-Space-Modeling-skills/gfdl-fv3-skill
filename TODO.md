# TODO, GFDL FV3 skill

## Tier B
- [ ] Cubed-sphere grid: face/tile numbering convention, edge handling
- [ ] dt_atmos / k_split / n_split typical values per resolution
- [ ] Vertical remap intervals (k_split control)
- [ ] Divergence damping coefficients (d2_bg, d4_bg, ...)
- [ ] Tracer transport options (hord_tr, fill, ...)
- [ ] GFDL MP species list and key namelist switches

## Tier C
- [ ] Common instability modes (gridscale noise, polar wobble, blowups)
- [ ] How to add a new tracer end-to-end (with FMS field_manager)
- [ ] Restart file structure
- [ ] Differences between SHiELD, UFS, AM4 use of FV3

## Verification
- [ ] Cite Putman & Lin 2007, Harris & Lin 2013 in the right places
- [ ] Cite Chen et al. 2013, Zhou et al. 2019 for microphysics
- [ ] Gemini critique pass
