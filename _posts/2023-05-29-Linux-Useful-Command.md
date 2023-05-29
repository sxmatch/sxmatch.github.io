# Linux Useful Commands

1. Download directory & subdirectories by Wget
   ```wget -r -np -nH --cut-dirs=1 -R index.* {download_url} -P {save to directory}```
   If don't want the directories on the download path, we can use -nH and --cut-dirs to control this.


