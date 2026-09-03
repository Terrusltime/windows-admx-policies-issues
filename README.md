# Issues in Windows ADMX and ADML files

This repository documents issues identified in Microsoft Windows ADMX/ADML Administrative Template files used by Group Policy Editor (gpedit.msc) and Group Policy Management Console (gpmc.msc).

Current issues reported : 

1. Issue in Windows Store Policy named "Disable all apps from Microsoft Store". Group Policy setting has counter-intuitive and inverted `Enabled` / `Disabled` semantics.