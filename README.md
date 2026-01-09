## Pipeline:

```
INIT
↓
CHECK_LOCAL
↓
FETCH_REMOTE
↓
COMPARE
↓         ↘
DOWNLOAD   LAUNCH
↓
VERIFY
↓
SAVE_STATE
↓
LAUNCH
```

## Meaning of Each State

### 1. INIT
Launcher startup

- Create the working directory
- Load the configuration
- Initialize the UI
- Proceed to the next state

No update logic here.

### 2. CHECK_LOCAL
Check the local version

- Check if `current.txt` exists
- Check if the specified JAR file exists
- If either is missing → assume no version is installed

Result:
- `localVersion = Optional<String>`

### 3. FETCH_REMOTE
Fetch the latest version info

- HTTP GET request for `latest.txt`
- Retrieve the JAR name (or version)
- Handle 404, timeouts, or other errors

Result:
- `remoteVersion`

### 4. COMPARE
Comparison logic

- If `localVersion == remoteVersion` → go to LAUNCH
- Otherwise → go to DOWNLOAD

No network or IO operations here — pure logic only.

### 5. DOWNLOAD
Download the JAR file

- Show a progress bar
- Download to a temporary file:
    - e.g., `client-1.2.3.jar.tmp`
- On completion → go to VERIFY

### 6. VERIFY (optional but recommended)
Integrity check

Options:
- Simply check that file size > 0
- Or verify checksum if available

If successful → proceed  
If failed → go to ERROR

### 7. SAVE_STATE
Commit the new version

- Rename `.tmp` file to the final `.jar`
- Update `current.txt` with the new version
- (Optionally) delete old versions

### 8. LAUNCH
Start the client

- Run `java -jar client-x.jar`
- The launcher can either:
    - Wait for the client to exit
    - Or exit immediately

Done.

### ERROR
Error at any stage

- Display the error message
- Provide buttons:
    - Retry
    - Exit
    - (Optional) Launch last known version