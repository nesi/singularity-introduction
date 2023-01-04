# Reproducible computational environments using containers: Introduction to Singularity

## NeSI notes.

Currently holding out some hope that parts of this could be taken upstream.

All changes that are not _NeSI Specific_ should go in `upstream-incubator`, any changes specific to NeSI should go into `NeSI specific changes`.

Make sure that before working on `upstream-incubator` you pull from origin, and before working on `nesi-specific-changes` you pull from `upstream-incubator`

`gh-pages` should be considered 'production' so will be protected. 

```
gh-pages <- nesi-specific-changes <- upstream-compat <= incubator/gh-pages
```

