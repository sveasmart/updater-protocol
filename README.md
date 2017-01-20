# Updater protocol

This is the protocol used between [Device Updaters](https://github.com/sveasmart/updater) and the Updater Hub.

See [Device Updater - design proposal](https://docs.google.com/document/d/1hymcpIQcGWWvBv703QUF6-MWEHL1M5ZPsYgwNP1DiUQ/edit#heading=h.uui7wxm0553m) for context.

## HTTP GET Method: "Update Me"

The Updater uses this method to find out if it needs to download a new update.

**Request path:** ```/updateme```

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
  snapshotId: "26",
  downloadUrl: "http://x.y.com/file.zip"

```

* The downloadUrl should lead to a zip file that contains an update.sh.
* update.sh should execute the actual update (whatever that is)
* update.sh can expect an environment variable ```apps_root``` which points to the top-level dir for all apps on the device.
* update.sh can expect the current working directory to be the same location as the update.sh file itself.

## HTTP POST Method: "How it worked out"

The Updater uses this method to tell the hub about an update that was executed.

**Request path:** ```/howitworkedout```

**Request body**:
```
{
    deviceId: "xyz",
    snapshotId: "26,
    success: "true",
    output: "Ansible bla bla bla... Done!"
}
```


* ```deviceId``` = the device making the call
* ```snapshotId``` = the snapshot that it was asked to upgrade to.
* ```success``` = true if the update was completed successfully, false if not
* ```output``` = the stream output from the update.sh script

**Response:**
```
{
  status: "ok"
}
```
