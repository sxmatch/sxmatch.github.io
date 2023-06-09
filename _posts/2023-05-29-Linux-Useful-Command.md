---
layout: post
category : Linux
tagline: "keep simple"
tags : [Linux, Command]
---
{% include JB/setup %}

**If need to reprint, please indicate the source**

**Copyright: 王昊 Hao Wang @sxmatch**

*2023/5/29*

-------
---

# Linux Useful Commands

1. Download directory & subdirectories by Wget
   ```wget -r -np -nH --cut-dirs=1 -R index.* {download_url} -P {save to directory}```
   If don't want the directories on the download path, we can use -nH and --cut-dirs to control this.


