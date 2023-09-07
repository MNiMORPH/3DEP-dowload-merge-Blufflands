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
