# 3DEP-dowload-merge-Blufflands
Download and merge gridded 3DEP data for southeastern Minnesota's blufflands: Bash+GRASS

## Navigate to lidar data

### Option 1: National Map Lidar Explorer

Go to:
https://apps.nationalmap.gov/lidar-explorer/

Then zoom in and click to select your project. Go to "Source DEMs" unless you want to go a different route and use the `laz` files.

### Option 2: Get the SE MN "Driftless" (but not actually) Lidar

Go here:
http://prd-tnm.s3.amazonaws.com/index.html?prefix=StagedProducts/Elevation/OPR/Projects/MN_SE_Driftless_2021_B21/MN_SEDriftless_2_2021

`0_file_download_links.txt` has all of the file names.

Half-meter-resolution data!? Who does that!


## Create a directory and download all lidar files

First, navigate to where you want your files to be. Then create a directory, `wget` the txt file, and use it to `wget` each geotiff:

```bash
# Move directories
mkdir SE_MN_3DEP_Lidar
cd SE_MN_3DEP_Lidar

# Download file with download links
wget https://prd-tnm.s3.amazonaws.com/StagedProducts/Elevation/OPR/Projects/MN_SE_Driftless_2021_B21/MN_SEDriftless_2_2021/0_file_download_links.txt

# Then loop over lines in the file to download everything as itty bitty tiles

while read line
do
  # echo $line # use this if you just want to test your loop
  wget $line # use this to actually acquire the file
done < 0_file_download_links.txt
```

Do something else while they all download. This could include getting you GRASS GIS location ready


## Set up GRASS GIS location

First, open GRASS GIS. Then create a new location. If you need to learn how to do this, it should either be a button (7.x) or right-clicking on your main "database" in the menu and selecting "create new location (GRASS 8.x)".

![Figure showing GRASS GIS location-creation screen](https://github.com/MNiMORPH/3DEP-dowload-merge-Blufflands/blob/main/figures/NewLocation__GRASS_GIS__SE_MN_3DEP_Lidar.png)

Next, select **Read CRS from georeferenced data file**. This way, you won't need to figure out the map projection used on your own: GRASS will do this for you. After clicking "next", browse to a file in your now-populating one, and choose any of the geotiffs that has fully downloaded.

Hit "next" again, and then "finish".

Select "no" to the auto-import option.


## Import data into GRASS GIS

### Import the files

Use the terminal associated with GRASS GIS.

![GRASS GIS screens: use the shell within which GRASS is running.](https://github.com/MNiMORPH/3DEP-dowload-merge-Blufflands/blob/main/figures/UseTerminal.png)

Using this terminal window, navigate to the directory containing the downloaded raster files. Alternatively, you could use an absolute path instead of a relative path in the next step.

Then, import the files.

```bash
# Loop over and import all DEM tiles to the GRASS location
for filename_full in `ls *.tif`
do
  # Only the name, without the extension
  # Use this for the name of the imported raster in GRASS
  filename_noext="${filename_full%.*}"
  # Then operate using this name
  # echo $filename_noext # Use this to test the code
  r.in.gdal in=$filename_full out=$filename_noext # Use this to bring the file into GRASS
done
```

### Create comma-separated list of all imported DEM tiles

We then create a comma-separated list of all of the Geotiff files. We will use this next to set the computational region and patch together the tiles.

```bash
# Create a new file to list the names of the imported DEM tiles (comma-separated)
touch filename_noext_list.csv
# Empty the file in case it existed before
> filename_noext_list.csv

# Then perform a similar loop
for filename_full in `ls *.tif`
do
  filename_noext="${filename_full%.*}"
  echo $filename_noext # print to the screen
  echo -n "$filename_noext" >> filename_noext_list.csv
  echo -n "," >> filename_noext_list.csv
done

# Remove trailing comma
sed -i '$ s/.$//' filename_noext_list.csv
```

### Merge all of the tiles into a single DEM

#### All at once

This doesn't work because there are too many files.

```bash
# Send the comma-separated list to a variable
read -r allnames < filename_noext_list.csv

# Set the region to extend around all of the tiles
g.region -p rast=$allnames

# Patch all of the raster tiles together
r.patch in=$allnames out=DEM_3DEP_2021
```

## Stepwise

In order to manage this massive amount of files, let's split by something. The file names look like:
```bash
USGS_OPR_MN_SE_Driftless_2021_B21_6360_48220
```
Here, the last two numbers are coordinates.

My idea here is to first group them based on the second-to-last coordinate into a set of somewhat larger tiles. To do so, we first find all instances of that number.

The first step on the way to this lies in creating a new file listing each of our geotiff names (without the extension) on a single line:
```bash
touch filename_noext_list.txt
# Empty the file in case it existed before
> filename_noext_list.txt

# Then perform a similar loop
for filename_full in `ls *.tif`
do
  filename_noext="${filename_full%.*}"
  echo $filename_noext # print to the screen
  echo "$filename_noext" >> filename_noext_list.txt
done
```

Then, loop over this file to place the second-to-last number in a new file:
```bash
touch coord1.txt
> coord1.txt
OLDIFS=$IFS
IFS='_' # Set the delimiter value
while read line
do
  read -r -a array <<< "$line"
  echo "${array[-2]}"
  echo "${array[-2]}" >> coord1.txt
done < filename_noext_list.txt
IFS=$OLDIFS # Return the input field separator to its default
```

Now, remove duplicate values therein:
```bash
sort -u coord1.txt > coord1_set.txt
```

Semi-finally, set the region and patch for each subset:
```bash
touch coord1_files.csv
touch outnames.csv
> outnames.csv
while read coord1
do
  > coord1_files.csv
  for filename_full in `ls *${coord1}_?????.tif`
  do
    filename_noext="${filename_full%.*}"
    #echo $filename_noext
    echo -n "$filename_noext" >> coord1_files.csv
    echo -n "," >> coord1_files.csv
  done
  sed -i '$ s/.$//' coord1_files.csv
  echo -n "$outname" >> outnames.csv
  echo -n "," >> outnames.csv
  outname="partial_DEM_3DEP_2021_$coord1"
  read -r selected_names < coord1_files.csv
  g.region -p rast=$selected_names
  r.patch in=$selected_names out=$outname
done < coord1_set.txt
sed -i '$ s/.$//' outnames.csv
```

Free disk space by removing the original tiles.
```bash
g.remove type=rast pattern="USGS_OPR_*"
```

Then stitch the bands:
```bash
read -r vertical_strip_names < outnames.csv
g.region -p rast=$vertical_strip_names
r.patch in=$selected_names out=$vertical_strip_names
```

And free space by removing the stripe files
```bash
g.remove type=rast pattern="partial_DEM_3DEP_*"
```
