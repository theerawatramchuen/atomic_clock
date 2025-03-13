### Installation on Python > 3.9
```
pip install ntplib
```
### Initital code
```
import ntplib
from datetime import datetime, timezone
from zoneinfo import ZoneInfo

def get_ntp_time():
    client = ntplib.NTPClient()
    try:
        response = client.request('pool.ntp.org')
        return datetime.fromtimestamp(response.tx_time, tz=timezone.utc)
    except ntplib.NTPException as e:
        raise RuntimeError("Failed to get NTP time.") from e

try:
    # Fetch UTC time from NTP server
    utc_time = get_ntp_time()
    
    # Convert to Bangkok timezone (UTC+7)
    bangkok_time = utc_time.astimezone(ZoneInfo('Asia/Bangkok'))
    
    print(f"Atomic Clock Time (UTC): {utc_time}")
    print(f"Bangkok Time: {bangkok_time}")

except Exception as e:
    print(f"Error: {e}")
```
### Atomic Clock 
```
import ntplib
from datetime import datetime, timezone, timedelta
from zoneinfo import ZoneInfo
import time

def get_ntp_time():
    client = ntplib.NTPClient()
    try:
        response = client.request('pool.ntp.org')
        return datetime.fromtimestamp(response.tx_time, tz=timezone.utc)
    except ntplib.NTPException as e:
        raise RuntimeError("Failed to get NTP time.") from e

try:
    # Initial setup
    bangkok_tz = ZoneInfo('Asia/Bangkok')
    
    # Get initial NTP time to calculate alignment
    utc_time = get_ntp_time()
    bangkok_time = utc_time.astimezone(bangkok_tz)
    
    # Calculate initial delay to next :00
    current_sec = bangkok_time.second + bangkok_time.microsecond/1e6
    delay = (60 - current_sec) % 60  # Seconds until next minute
    if delay < 0.1:  # If very close to :00, ensure we catch the next one
        delay += 60
    print(f"⏳ Waiting {delay:.1f} seconds to align with next :00...")
    time.sleep(delay)
    
    # Initialize first target time (exact :00)
    previous_target = datetime.now(timezone.utc).astimezone(bangkok_tz).replace(second=0, microsecond=0)
    
    while True:
        try:
            # Get precise atomic time
            utc_time = get_ntp_time()
            bangkok_time = utc_time.astimezone(bangkok_tz)
            
            # Print with milliseconds
            fmt = "%Y-%m-%d %H:%M:%S.%f %Z"
            print(f"🕒 BANGKOK {bangkok_time.strftime(fmt)[:-3]}")
            print(f"🌐 UTC     {utc_time.strftime(fmt)[:-3]}\n")
            
            # Calculate next target time and sleep duration
            next_target = previous_target + timedelta(seconds=20)
            sleep_duration = (next_target - bangkok_time).total_seconds()
            
            if sleep_duration > 0:
                time.sleep(sleep_duration)
            else:
                print("⚠️  Adjustment needed - running behind schedule")
            
            previous_target = next_target
            
        except Exception as e:
            print(f"🚨 Error: {e}")
            print("🔄 Retrying in 5 seconds...")
            time.sleep(5)

except Exception as e:
    print(f"🔥 Critical error: {e}")
```
