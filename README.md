# The Lake Debate

Analysis for [Who has more lakes: Minnesota or Wisconsin?](https://www.politifact.com/wisconsin/statements/2019/may/23/sara-meaney/who-has-more-lakes-minnesota-or-wisconsin/), a fact check on whether Wisconsin has more lakes than Minnesota (sadly, we don't).

The analysis is done in a `Makefile`. The lake counts from this analysis line up closely with USGS's. For the article, we used USGS's lake counts instead of our own.

## Dependencies

The `Makefile` depends on a few system packages:
* [mapshaper](https://github.com/mbloch/mapshaper) available [via npm](https://www.npmjs.com/package/mapshaper)
* [rename](http://plasmasturm.org/code/rename/) available on Mac [via homebrew](https://formulae.brew.sh/formula/rename)
* [ogr2ogr](https://gdal.org/programs/ogr2ogr.html) from [GDAL](https://gdal.org/) available on Mac [via homebrew](https://formulae.brew.sh/formula/gdal)
