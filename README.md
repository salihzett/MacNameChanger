# MacNameChanger
#### Change the name based on a schema - every time the Mac is started


During my time as an administrator, it happened that users repeatedly change the name of the computer.
This ensures that when monitoring (e.g. with [munkireport](https://github.com/munkireport/munkireport-php) or [Apple Remote Desktop](https://itunes.apple.com/de/app/apple-remote-desktop/id409907375)) I can't see at first glance what a model it is.
I needed a structure.

So see my:

At first the script:
```shell
#!/bin/sh
localUsers=$( dscl . list /Users UniqueID | awk '$2 >= 501 {print $1}' | grep -v admin )
for userName in "$localUsers"; do
	userName2=${userName//./}
	model=$(system_profiler SPHardwareDataType | awk '/Identifier/ {print $3}')
	model2=${model//,/}
	/usr/sbin/scutil --set ComputerName ${model}_${userName}
	/usr/sbin/scutil --set HostName ${model2}-${userName2}.local
	/usr/sbin/scutil --set LocalHostName ${model2}-${userName2}
done
```
The result is:
```
ComputerName: MacBookPro11,3_s.zett
HostName: MacBookPro113-szett.local
LocalHostName: MacBookPro113-szett
```

The line with `localUsers` takes the first user (alphabetical) on the Mac. Other methods (e.g. with `whoami`) wouldn't work because the script runs with root privileges.

The reason why I remove comma and point in HostName / LocalHostName is that some programs can not cope with these signs. (e.g. [Google Mail](https://support.google.com/mail/answer/6386757?visit_id=0-636578195982351165-252876126&p=helo&rd=1)).


The plist file is stored in the folder `launchdaemon`. The script runs at every load.
```plist
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>Label</key>
	<string>de.salihzengin.rename_mac.plist</string>
	<key>ProgramArguments</key>
	<array>
		<string>/Library/Application Support/sz/rename_mac.sh</string>
	</array>
	<key>RunAtLoad</key>
	<true/>
</dict>
</plist>
```

All the files are stored here on github. 
I have also create a pkg file that saves the corresponding files in the corresponding directories.
Check [Releases](https://github.com/salihzett/MacNameChanger/releases) (it isn't signed)
