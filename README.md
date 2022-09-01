# VECNA
The VECNA Toolkit includes Aggressor Scripts that can be loaded into Cobalt Strike during a penetration test to achieve a range of useful functions.

## Credential Harvesting Script - cred_harvest.cna

### Description
The ‘Credential Harvesting Script’ was created to automatically add credentials entered by victim users to ‘Credentials’ within Cobalt Strike. This eliminates the need to monitor the ‘Web Log’ for incoming credentials. Additionally, this script can be used to print metrics pertaining to a phishing campaign such as unique credentials submitted, total credentials submitted, unique callbacks received, and total callbacks received.

### Usage
It is recommended to review the script and determine if it needs to be customized to suit the needs of the engagement before loading into Cobalt Strike. By default, the script records credentials contained in GET requests that include ‘username’ and ‘password’ form data and assumes the request will be submitted via a credential harvesting form at ‘index.html'. These values can be customized under the ‘on web_hit’ function.

The ‘cred_harvest.cna’ script needs to be loaded prior to the credential harvesting campaign kicking off, as it does not review the web log for previous entries and only accounts for incoming credentials once it’s loaded. However, if the script is only being used for the ‘get_metrics’ function, then it can be loaded at any time.

Note that the accuracy of the ‘get_metrics’ function relies on a few factors. Each credential entry logged by the script is automatically labeled with ‘credential_harvest_campaign’. When the ‘get_metrics’ function is run, this label is used to identify how many credentials were harvested. That means manual label changes by users can result in inaccurate counts. Session counts are performed on all sessions in the beacon list (whether or not they are active). That means, beacons that are manually removed from this list or beacons that are manually spawned can skew the accuracy of ‘callbacks received’ metrics.

1.	Review the script and make any custom changes that are applicable to the engagement being conducted. This may include modifying the ‘on web_hit’ function to replace ‘GET’ with ‘POST’, ‘index.html’ with the name of the page hosting the credential harvesting form, and ‘username’/’password’ with the names of the credential form fields.
2.	To use the ‘cred_harvest.cna’ script, first load the CNA file into Cobalt Strike via the Script Manager ‘Load’ button. Please note that once the script is loaded, it will automatically begin logging credentials to the ‘Credentials’ tab in Cobalt Strike and won’t stop until the script is unloaded.
3.	Within the Cobalt Strike Script Console, run the following command to pull credential and session data (such as credential and callback counts):

        get_metrics

4.	The ‘Unload’ button within the Script Manager can be used to stop the script from running at any time.

## Host Payloads Script - host_payloads.cna

### Description
The ‘Host Payloads Script’ was created to automate the process of uploading payloads (or any type of file) to be hosted on Cobalt Strike. This script allows users to run a single command that points to a directory containing all of the files to be uploaded, eliminating the tedious process of uploading payloads individually.

### Usage
It is recommended to review the script and determine if it needs to be customized to suit the needs of the engagement before loading into Cobalt Strike. By default, payloads will be uploaded under a ‘payloads’ directory with a name that matches the filename. This can be changed via the ‘$uri’ variable. Additionally, ‘127.0.0.1’ is used to host the file by default.

1.	Review the script and make any custom changes that are applicable to the engagement being conducted.
2.	To use the ‘host_payloads.cna’ script, first load the CNA file into Cobalt Strike via the Script Manager ‘Load’ button. Once the script is loaded, the Script Console can be used to load payloads.
3.	Within the Cobalt Strike Script Console, the following command can be run to automatically load all payloads within a given directory (note that the path needs to be a valid full path that is accessible to the user under which Cobalt Strike is running):

        host_payloads /full/path/to/payloads

4.	By default, the script will upload all files within the specified directory as ‘/payloads/<filename>’. For this reason, it is important to: A) avoid adding irrelevant files to the directory in question and B) name the files using whatever naming convention the uploaded payloads should have.
5.	The script has some built-in error handling and output for troubleshooting purposes, but should not be relied on to cover all error cases.
6.	Always check that the files were uploaded correctly and that they are still working as expected (e.g., by acting as a victim user and downloading/executing each payload).

## Initial Beacon Spawn Script - init_spawn.cna
  
### Description
The ‘Initial Beacon Spawn Script’ was created to automate beacon spawning based on the session process for the incoming beacon. For example, if a beacon uses the EXCEL.EXE process and relies on the user keeping Excel open to remain stable, this script can be used to automatically spawn a beacon under a different process. This eliminates the need for assessors to constantly monitor incoming beacons and rush to spawn a beacon before the victim user closes out of a program or takes some action to kill the process.

### Usage
It is recommended to review the script and determine if it needs to be customized to suit the needs of the engagement before loading into Cobalt Strike. By default, the script assumes the listener name is ‘https’ and only runs if the incoming beacons are utilizing the EXCEL.EXE process. The listener name can be replaced within the ‘bspawn’ function and the process list can be modified via the ‘@plist’ variable.

Additionally, beacons spawned by the script will utilize whatever the default spawn session process is set to within Cobalt Strike. By default, this process is ‘rundll32.exe’ unless it is set to a different process within the Cobalt Strike profile that the TeamServer is using.

There are additional built-in convenience functions that are commented out by default. The ‘bnote’ function can be used to set the note for all incoming beacons as a timestamp based on when the connection was established. The ‘bsleep’ function can be used to automatically sleep all incoming beacons (e.g., for beacons coming in outside hours of operation). The default values in the script will sleep incoming beacons for 1 hour with 20% jitter.

1.	Review the script and make any custom changes that are applicable to the engagement being conducted. This includes adding/removing processes from the ‘@plist’ variable, changing the listener name in the ‘bspawn’ function, and commenting or uncommenting certain functions based on needs.
2.	To use the ‘init_spawn.cna’ script, first load the CNA file into Cobalt Strike via the Script Manager ‘Load’ button. Please note that once the script is loaded, it will automatically begin spawning new beacons for incoming connections and won’t stop until it’s unloaded.
3.	The ‘Unload’ button within the Script Manager can be used to stop the script from running at any time.

## Scope Check Script - scope_check.cna
  
### Description
The ‘Scope Check Script’ was created to label and highlight incoming beacons based on whether they are in-scope, referencing a list of in-scope IP addresses. Once an IP list is imported, the script will check the internal IP address of incoming beacons against the IP list and highlight/label the beacon accordingly.

### Usage
The ‘scope_check.cna’ script needs to be loaded prior to receiving beacons, as it only highlights incoming beacons after it’s been loaded but does not label pre-existing beacons. A line-delimited list of in-scope internal IP addresses (with a single IP address per line) should be prepared ahead of time so that, once the script is loaded, the IP list can be referenced immediately. Otherwise, all incoming beacons will be labelled as out-of-scope until an IP list is loaded. Loading a new IP list will overwrite all previously loaded IPs, so it is important that the list be comprehensive. Once the ‘scope_check.cna’ script is unloaded, incoming beacons will no longer be labelled or highlighted until the script is re-loaded and an IP list is added once more.

It is important to note that the script relies on the accuracy of the IP list and how it is formatted. For example, if an incorrect delimiter is used anywhere in the file, the imported list will likely be empty or incomplete. Similarly, if extra characters such as spaces are tagged onto an IP address, this can also interfere with the accuracy of the script. For these reasons, it is recommended to not rely solely on this script to determine if a beacon is in-scope or out-of-scope.

1.	Review the script and make any custom changes that are applicable to the engagement being conducted.
2.	Locate the line-delimited list of in-scope IP addresses and note the full path to the file.
3.	To use the ‘scope_check.cna’ script, first load the CNA file into Cobalt Strike via the Script Manager ‘Load’ button. Please note that once the script is loaded, it will automatically begin labelling and highlighting incoming beacons and won’t stop until the script is unloaded.
4.	Within the Cobalt Strike Script Console, the following command should be run to import a line-delimited list of in-scope IP addresses:

        load_scope /full/path/to/ips.txt
  
5. Once the 'load_scope' command is run, a list of the IP addresses that were imported should be printed, followed by a total number of IP addresses. If the list does not appear accurate, it is possible that the IP list file is not correctly formatted. The 'load_scope' command can be run as many times as needed, but will overwrite previous IP lists each time a new list is loaded.
6.	Within the Cobalt Strike Script Console, the following command can be run at any time to print a list of the in-scope IP addresses that have been loaded:

        print_scope

7.	Until an IP list is loaded, the script will automatically flag every incoming beacon as out-of-scope. Once an IP list is loaded, the script will check if the internal IP address of the incoming beacon is in the list of imported IP addresses. If it is in the list, the beacon will be highlighted green, indicating it is in scope. If the internal IP address is not in the list of imported IP addresses, the beacon will be highlighted red and labelled ‘!!! OUT OF SCOPE !!!’, indicating it is not in scope.
8.	The ‘Unload’ button within the Script Manager can be used to stop the script from running at any time.
