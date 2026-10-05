# Offline map from flash (elecrow-adv2-43-50-70-tft)

The SD slot of the newer CrowPanel Advance boards shares IO4/5/6 with the wireless
module through a bus switch (S0/S1), so LoRa and the SD card cannot be used at the same
time. This env therefore reads the MUI map from a raw `maps` data partition in the
internal flash instead (`crowpanel_16MB_maps.csv`: app0 5 MB, maps 7.5 MB, OTA slot removed).
The map still works fully offline.

The map is a single PMTiles archive (PNG raster tiles, uncompressed tiles, gzip directories,
see meshtastic/device-ui maps/README.md) of at most 7.5 MB. Flash it raw:

    esptool write-flash 0x510000 my-map.pmtiles

`device-ui-flashmap.patch` patches the pinned meshtastic-device-ui library (it is not part of
this repo). After PlatformIO has fetched the libdep, apply it once from the library root:

    cd .pio/libdeps/elecrow-adv2-43-50-70-tft/meshtastic-device-ui
    patch -p1 < ../../../../variants/esp32s3/elecrow_panel/device-ui-flashmap/device-ui-flashmap.patch

It adds `FlashMapFileSystem` (esp_partition reads) behind `-D MAP_FROM_FLASH` and forces the
PMTiles style. Optionally start the map centered on a fixed location with
`-D MAP_START_LAT=<deg> -D MAP_START_LON=<deg> [-D MAP_START_ZOOM=11]`.
