
GENERATED_FILES := \
	state-borders/wi.shp \
	state-borders/wi-wogl.shp \
	lakes/wi.shp \
	lakes/wi-wogl.shp \
	state-borders/mn.shp \
	state-borders/mn-wogl.shp \
	lakes/mn.shp \
	lakes/mn-wogl.shp

all: $(GENERATED_FILES)

# National Hydrographic Dataset (NHD)

nhd/zip/wi.zip:
	mkdir -p $(dir $@)
	curl -o $@.download 'https://prd-tnm.s3.amazonaws.com/StagedProducts/Hydrography/NHD/State/HighResolution/GDB/NHD_H_Wisconsin_State_GDB.zip'
	mv $@.download $@

nhd/zip/mn.zip:
	mkdir -p $(dir $@)
	curl -o $@.download 'https://prd-tnm.s3.amazonaws.com/StagedProducts/Hydrography/NHD/State/HighResolution/GDB/NHD_H_Minnesota_State_GDB.zip'
	mv $@.download $@

nhd/gdb/wi.gdb: nhd/zip/wi.zip
	mkdir -p $(dir $@)
	unzip -d nhd/gdb/ nhd/zip/wi.zip
	cd nhd/gdb; \
	rename 's/NHD_H_Wisconsin_State_GDB/wi/' NHD_H_Wisconsin_State_GDB.*

nhd/gdb/mn.gdb: nhd/zip/mn.zip
	mkdir -p $(dir $@)
	unzip -d nhd/gdb/ nhd/zip/mn.zip
	cd nhd/gdb; \
	rename 's/NHD_H_Minnesota_State_GDB/mn/' NHD_H_Minnesota_State_GDB.*

# State borders

state-borders/zip/states.zip:
	mkdir -p $(dir $@)
	curl -o $@.download 'https://www2.census.gov/geo/tiger/TIGER2018/STATE/tl_2018_us_state.zip'
	mv $@.download $@

state-borders/states.shp: state-borders/zip/states.zip
	mkdir -p $(dir $@) tmp
	unzip -d tmp $<
	mapshaper tmp/tl_2018_us_state.shp -o $@ format=shapefile force
	rm -rf tmp

state-borders/wi.shp: state-borders/states.shp
	mkdir -p $(dir $@)
	mapshaper \
		-i $< \
		-filter 'STUSPS === "WI"' \
		-o $@ format=shapefile force

state-borders/mn.shp: state-borders/states.shp
	mkdir -p $(dir $@)
	mapshaper \
		-i $< \
		-filter 'STUSPS === "MN"' \
		-o $@ format=shapefile force

# Lakes

lakes/wi.shp: nhd/gdb/wi.gdb state-borders/wi.shp
	mkdir -p $(dir $@) tmp
	ogr2ogr -f "ESRI Shapefile" tmp/raw.shp nhd/gdb/wi.gdb NHDWaterbody
	mapshaper \
		-i tmp/raw.shp \
		-clip state-borders/wi.shp \
		-filter 'FType === 390 && [39001, 39005, 39006].indexOf(FCode) < 0' \
		-each 'area_sq_km = this.area / 1000000' \
		-each 'area_acres = area_sq_km * 247.105' \
		-o $@ format=shapefile force
	rm -rf tmp

lakes/wi-wogl.shp: lakes/wi.shp
	mkdir -p $(dir $@)
	mapshaper \
		-i $< \
		-filter 'GNIS_Name !== "Lake Superior" && GNIS_Name !== "Lake Michigan"' \
		-o $@ format=shapefile force

lakes/mn.shp: nhd/gdb/mn.gdb state-borders/mn.shp
	mkdir -p $(dir $@) tmp
	ogr2ogr -f "ESRI Shapefile" tmp/raw.shp nhd/gdb/mn.gdb NHDWaterbody
	mapshaper \
		-i tmp/raw.shp \
		-clip state-borders/mn.shp \
		-filter 'FType === 390 && [39001, 39005, 39006].indexOf(FCode) < 0' \
		-each 'area_sq_km = this.area / 1000000' \
		-each 'area_acres = area_sq_km * 247.105' \
		-o $@ format=shapefile force
	rm -rf tmp

lakes/mn-wogl.shp: lakes/mn.shp
	mkdir -p $(dir $@)
	mapshaper \
		-i $< \
		-filter 'GNIS_Name !== "Lake Superior" && GNIS_Name !== "Lake Michigan"' \
		-o $@ format=shapefile force

# Great lakes

lakes/great-lakes.shp: nhd/gdb/wi.gdb
	mkdir -p $(dir $@) tmp
	ogr2ogr -f "ESRI Shapefile" tmp/raw.shp nhd/gdb/wi.gdb NHDWaterbody
	mapshaper \
		-i tmp/raw.shp \
		-filter 'GNIS_Name === "Lake Superior" || GNIS_Name == "Lake Michigan"' \
		-dissolve \
		-o $@ format=shapefile force
	rm -rf tmp

# State borders w/o Great Lakes

state-borders/wi-wogl.shp: state-borders/wi.shp lakes/great-lakes.shp
	mkdir -p $(dir $@)
	mapshaper \
		-i state-borders/wi.shp \
		-erase lakes/great-lakes.shp \
		-clean \
		-o $@ format=shapefile force

state-borders/mn-wogl.shp: state-borders/mn.shp lakes/great-lakes.shp
	mkdir -p $(dir $@)
	mapshaper \
		-i state-borders/mn.shp \
		-erase lakes/great-lakes.shp \
		-clean \
		-o $@ format=shapefile force

# Table version of the lakes

lakes/wi-wogl.csv: lakes/wi-wogl.shp
	mkdir -p $(dir $@)
	mapshaper \
		-i $< \
		-filter 'area_acres >= 10' \
		-o $@ format=csv force

lakes/mn-wogl.csv:lakes/mn-wogl.shp
	mkdir -p $(dir $@)
	mapshaper \
		-i $< \
		-filter 'area_acres >= 10' \
		-o $@ format=csv force

# Calculations

number-of-lakes:
	 mapshaper lakes/wi.shp -filter 'area_acres >= 10' -info
	 mapshaper lakes/mn.shp -filter 'area_acres >= 10' -info

surface-area:
	mapshaper state-borders/wi.shp -each 'area_sq_km = this.area / 1000000' -filter-fields area_sq_km -info
	mapshaper state-borders/wi-wogl.shp -each 'area_sq_km = this.area / 1000000' -filter-fields area_sq_km -info
	mapshaper lakes/wi.shp -filter 'area_acres >= 10' -dissolve -each 'area_sq_km = this.area / 1000000' -filter-fields area_sq_km -info
	mapshaper lakes/wi-wogl.shp -filter 'area_acres >= 10' -dissolve -each 'area_sq_km = this.area / 1000000' -filter-fields area_sq_km -info
	mapshaper state-borders/mn.shp -each 'area_sq_km = this.area / 1000000' -filter-fields area_sq_km -info
	mapshaper state-borders/mn-wogl.shp -each 'area_sq_km = this.area / 1000000' -filter-fields area_sq_km -info
	mapshaper lakes/mn.shp -filter 'area_acres >= 10' -dissolve -each 'area_sq_km = this.area / 1000000' -filter-fields area_sq_km -info
	mapshaper lakes/mn-wogl.shp -filter 'area_acres >= 10' -dissolve -each 'area_sq_km = this.area / 1000000' -filter-fields area_sq_km -info

# Number of lakes (10 acres of surface area or larger)

# Wisconsin:  6,096 (or 6,167 with intermittent lakes)
# Minnesota: 14,318 (or 14,393 with intermittent lakes)

# Surface area (square kilometers)

# Wisconsin:
#   With Great Lakes:
#     Total:  169,652.20 (or 169,652)
#     Lakes:   28,187.76 (or 28,206)
#      Pct.:       16.6% (or 16.6%)
#   Without Great Lakes:
#     Total:  145,358.91 (or 145,359)
#     Lakes:    3,894.47 (or 3,913)
#      Pct.:        2.7% (or 2.7%)

# Minnesota:
#   With Great Lakes:
#     Total:  225,108.09 (or 225,108)
#     Lakes:   18,399.99 (or 18,409)
#      Pct.:        8.2% (or 8.2%)
#   Without Great Lakes:
#     Total:  218,490.50 (or 218,490)
#     Lakes:   11,782.40 (or 11,792)
#      Pct.:        5.4% (or 5.4%)


# For Graphics

lakes/for-display.shp: lakes/wi-wogl.shp lakes/mn-wogl.shp
	mkdir -p $(dir $@)
	mapshaper \
		-i $^ combine-files \
		-merge-layers \
		-filter 'area_acres >= 10' \
		-clean \
		-o $@ format=shapefile force

state-borders/combined.shp: state-borders/wi-wogl.shp state-borders/mn-wogl.shp
	mkdir -p $(dir $@)
	mapshaper \
		-i $^ combine-files \
		-merge-layers \
		-dissolve \
		-simplify 1% \
		-clean \
		-o $@ format=shapefile force

state-borders/inner-line.shp: state-borders/wi-wogl.shp state-borders/mn-wogl.shp
	mkdir -p $(dir $@)
	mapshaper \
		-i $^ combine-files \
		-merge-layers \
		-simplify 1% \
		-innerlines \
		-o $@ format=shapefile force

lakes/for-packing.shp: lakes/wi-wogl.shp
	mkdir -p $(dir $@)
	mapshaper \
		-i $? \
		-filter 'area_acres >= 10' \
		-clean \
		-proj '+proj=tmerc +lat_0=0 +lon_0=-90 +k=0.9996 +x_0=520000 +y_0=-4480000 +ellps=GRS80 +units=m +no_defs' \
		-each 'width = this.width, height = this.height' \
		-o $@ format=shapefile force
