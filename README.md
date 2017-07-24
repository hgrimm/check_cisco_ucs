check_cisco_ucs
===============

check_cisco_ucs is a Nagios plugin to monitor Cisco UCS rack and blade center hardware

 check_cisco_ucs is a Nagios plugin made by Herwig Grimm (herwig.grimm at aon.at)
 to monitor Cisco UCS rack and blade center hardware.

 I have used the Google Go progamming language because of no need to install
 any libraries.

 The plugin uses the Cisco UCS XML API via HTTPS to do a wide variety of checks.


 This nagios plugin is free software, and comes with ABSOLUTELY NO WARRANTY.
 It may be used, redistributed and/or modified under the terms of the GNU
 General Public Licence (see http://www.fsf.org/licensing/licenses/gpl.txt).

tested with:
------------

	1. UCSC-C240-M3S server and CIMC firmware version 1.5(1f).24
	2. Cisco UCS Manager version 2.1(1e) and UCSB-B22-M3 blade center
	3. Cisco UCS Manager version 2.2(1b) and UCSB-B200-M3
	4. UCSC-C220-M4S server and CIMC firmware version 2.0(4c).36
	5. UCS C240 M4S and CIMC firmware version 3.0(3a)

see also:
---------

  	Cisco UCS Rack-Mount Servers Cisco IMC XML API Programmer's Guide, Release 3.0
 	(http://www.cisco.com/c/en/us/td/docs/unified_computing/ucs/c/sw/api/3_0/b_Cisco_IMC_api_301.html)

changelog:
----------

 	Version 0.1 (11.06.2013) initial release

	Version 0.2 (26.06.2013)
		usage text debug flag added,
		write errors to stdout instead of stderr,
		flag -E to show environment variables added
		flag -V to print plugin version added

	Version 0.3 (24.04.2014)
		flag -z *OK if zero instances* added

	Version 0.4 (24.02.2015)
		flag -F display only faults in output, newlines between objects in output line

	Version 0.5 (19.05.2015)
		fix for: "remote error: handshake failure"
		see: TLSClientConfig ... MaxVersion: tls.VersionTLS11, ...

	Version 0.6 (19.07.2017)
		fix for: " Post https://<ipaddr>/nuova/: read tcp <ipaddr>:443: connection reset by peer"
		see: TLSClientConfig ... MaxVersion: tls.VersionTLS12, ...

		flag -M *max TLS Version* added.
		CIMC firmware version 3.0 needs flag -M 1.2


		fix for: "HTTP 403 Forbidden error"
		error in URL path: no backslash after *nuova*
		see code line: url := "https://" + ipAddr + "/nuova"
		old: .../nuova/ new: .../nuova


todo:
-----

 	1. better error handling
 	2. add performance data support
 	3. command line flag to influence TLS cert verification

flags:
------

 	-H <ip_addr>		CIMC IP address or Cisco UCS Manager IP address"
 	-t <query_type>		query type 'dn' or 'class'"
 	-q <dn_or_class>	XML API object class name, examples: storageVirtualDrive or storageLocalDisk or storageControllerProps
 						Distinguished Name (DN) name, examples: "sys/rack-unit-1"
 	-o <object>			if XML API object class name, examples: storageVirtualDrive or storageLocalDisk or storageControllerProp
 	-s <hierarchical>	true or false. If true, the inHierarchical argument returns all child objects
 	-a <attributes>		space separated list of XML attributes for display in nagios output and match against *expect* string
 	-e <expect_string>	expect string, ok if this is found, examples: "Optimal" or "Good" or "Optimal|Good"
 	-u <username>		XML API username
 	-p <password>		XML API password
	-d <level>			print debug, level: 1 errors only, 2 warnings and 3 informational messages
	-E					print environment variables for debug purpose
	-V					print plugin version
	-z					true or false. if set to true the check will return OK status if zero instances where found. Default is false.
	-F					display only faults in output
	-M <tls_verson>		max TLS version, default: 1.1, alternative: 1.2

usage examples:
---------------

 	Cisco UCS rack server via CIMC:

 	$ ./check_cisco_ucs -H 10.18.4.7 -t class -q storageVirtualDrive -a "raidLevel vdStatus health" -e Optimal -u admin -p pls_change
 	OK - Cisco UCS storageVirtualDrive (raidLevel,vdStatus,health) RAID 10,Optimal,Good (1 of 1 ok)

 	$ ./check_cisco_ucs -H 10.18.4.7 -t class -q storageLocalDisk -a "id pdStatus driveSerialNumber" -e Online -u admin -p pls_change
 	OK - Cisco UCS storageLocalDisk (id,pdStatus,driveSerialNumber) 1,Online,6XP4QRVQ 2,Online,6XP4QS1G 3,Online,6XP4RT6A 4,Online,6XP4RT8V (4 of 4 ok)

 	$ ./check_cisco_ucs -H 10.18.64.10 -t class -q equipmentPsu -a "id model operState serial" -e operable -u admin -p pls_change
 	CRIT - Cisco UCS equipmentPsu (id,model,operState,serial) 1,UCS-PSU-6248UP-AC,operable,POG164371G8 2,UCS-PSU-6248UP-AC,operable,POG1643721D 1,UCS-PSU-6248UP-AC,operable,POG164371C5 2,UCS-PSU-6248UP-AC,operable,POG1643721S 1,UCSB-PSU-2500ACPL,operable,AZS16210FFA 2,UCSB-PSU-2500ACPL,operable,AZS16210FH3 3,UCSB-PSU-2500ACPL,operable,AZS16210FH2 4,,removed (7 of 8 ok)

 	$ ./check_cisco_ucs -H 10.18.4.7 -t dn -q sys/rack-unit-1/indicator-led-4 -o equipmentIndicatorLed -a "id color name" -e green -u admin -p pls_change
 	OK - Cisco UCS sys/rack-unit-1/indicator-led-4 (id,color,name) 4,green,LED_FAN_STATUS (1 of 1 ok)

 	$ ./check_cisco_ucs -H 10.1.1.235 -t dn -q sys/rack-unit-1/indicator-led-4 -a "id color name" -e "green" -u admin -p pls_change -o equipmentIndicatorLed -M 1.2
 	OK - Cisco UCS sys/rack-unit-1/indicator-led-4 (id,color,name)
 	4,green,LED_HLTH_STATUS (1 of 1 ok)

 	Cisco UCS Manager:

 	$ ./check_cisco_ucs -H 10.18.64.10 -t class -q equipmentPsu -a "id model operState serial" -e operable -u admin -p pls_change
 	CRIT - Cisco UCS equipmentPsu (id,model,operState,serial) 1,UCS-PSU-6248UP-AC,operable,POG164371G8 2,UCS-PSU-6248UP-AC,operable,POG1643721D 1,UCS-PSU-6248UP-AC,operable,POG164371C5 2,UCS-PSU-6248UP-AC,operable,POG1643721S 1,UCSB-PSU-2500ACPL,operable,AZS16210FFA 2,UCSB-PSU-2500ACPL,operable,AZS16210FH3 3,UCSB-PSU-2500ACPL,operable,AZS16210FH2 4,,removed (7 of 8 ok)

 	$ ./check_cisco_ucs -H 10.18.64.10 -t dn -q sys/switch-B/slot-1/switch-ether/port-1 -o etherPIo -a operState -e up -u admin -p pls_change
	OK - Cisco UCS sys/switch-B/slot-1/switch-ether/port-1 (operState) up (1 of 1 ok)

	$ ./check_cisco_ucs -H 10.18.64.10 -t class -q faultInst -a "code severity ack" -e "cleared,no|cleared,yes|info,no|info,yes|warning,no|warning,yes|yes|^$" -z true -u admin -p pls_change
	OK - Cisco UCS faultInst (code,severity,ack) (0 of 0 ok)