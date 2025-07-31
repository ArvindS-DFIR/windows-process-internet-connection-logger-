# Windows-process-internet-connection-logger-
This python script logs all active internet (TCP) connections on a windows host and maps them to their process.Useful for identifying suspicious outbound connections during incident response where EDR is not available/installed 

# Features 
- Captures 'PID','Process Name','Remote IP' and 'Remote Port'
  
- Filters only 'Established' connections

- Output results to a clean CSV file

## ⚠️ Permissions

This script requires Administrator privileges to access full process and network information.

### 🪪 How to Run as Administrator (Windows)

1. Open the **Start Menu**
2. Type **`cmd`** or **`PowerShell`**
3. **Right-click** on it and select **“Run as administrator”**
4. Navigate to your script directory:# cd C:\path\to\your\script
# or You can build your own exe and share it into suspected victim host and run it 

If you want to build the .exe file from the Python script:

1. Install Requirements

Open Command Prompt and run:

```pip install pyinstaller psutil```

2. Create the .exe File

In the same folder as your script:

```pyinstaller --onefile yourscript.py```

Replace yourscript.py with your actual filename.

3. The .exe file will be created inside a folder called dist:

```dist\yourscript.exe```



```python script

#import required libraries
import psutil
import csv
import ctypes
import sys

# Function to check if the script is running with administrator rights
def is_admin():
    try:
        return ctypes.windll.shell32.IsUserAnAdmin()
    except:
        return False

# if not running as admin ,relunch the script with admin rights 
if not is_admin():
    ctypes.windll.shell32.ShellExecuteW(
        None, "runas", sys.executable, " ".join(sys.argv), None, 1) # "runas" triggeres the Windows UAC prompt
    sys.exit()

def internet_connection(file_path): # Function to log active internet connections to a CSV file
    with open(file_path,"w",newline='') as csvfile:
        filenames= ["pid","name","remote_ip","remote_port"] # define column header
        writer=csv.DictWriter(csvfile,fieldnames=filenames)
        writer.writeheader()

        for proc in psutil.process_iter(['pid','name']): # Iterate through all processes 
            try:
                connections=proc.net_connections(kind='inet') # Get all internet connections for the process
                for conn in connections:
                    if conn.status==psutil.CONN_ESTABLISHED and conn.raddr: # Only log established connections with a remote address
                        writer.writerow({'pid':proc.info['pid'],'name':proc.info['name'],"remote_ip":conn.raddr.ip,"remote_port":conn.raddr.port})
            except (psutil.AccessDenied,psutil.NoSuchProcess): # Skip process that no longer exist or can't be accessed 
                continue

internet_connection("C:\\processhistory.csv")``` # Set output path for the CSV file



