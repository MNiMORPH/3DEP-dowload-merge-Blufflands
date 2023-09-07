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

Do something else while they all download.
