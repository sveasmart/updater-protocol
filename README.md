# Updater protocol

This is the protocol used between Device Updaters and the Updater Hub.

## HTTP GET Method: "Update Me"

The Updater uses this method to find out if it needs to download a new update.

**Request Path:** ```/updateme```

**Request parameters**:
* ```deviceId``` - the device making the call
* ```snapshotId```- the current snapshot in that device

**Response if no update is needed:**

```
{
  status: "noUpdateNeeded"
}
```

**Response if update is needed:**
```
{
  status: "updateNeeded",
  snapshotId: 26,
  downloadUrl: http://x.y.com/file.zip
}
```

* The downloadUrl should lead to a zip file that contains a startup.sh.
* Startup.sh should execute the actual update (whatever that is)
* Startup.sh can expect an environment variable ```app_root``` which points to the top-level dir for all apps on the device.
* Startup.sh can expect the current working directory to be the same location as the startup.sh file itself.

## HTTP POST Method: "How it worked out"

The Updater uses this method to tell the hub about an update that was executed.

**Request Path:** ```/howitworkedout```

**Request parameters**:
* ```deviceId``` - the device making the call
* ```snapshotId```- the snapshot that it was asked to upgrade to.
* ```success``` - true if the update was completed successfully, false if not
* ```output``` - the terminal output from the start.sh script

**Response:**
```
{
  status: "ok"
}
```
