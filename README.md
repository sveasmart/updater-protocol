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
}
```

* The downloadUrl should lead to a zip file that contains an update.sh.
* update.sh should either be in the root folder of the zip, or one folder below.
* update.sh should execute the actual update (whatever that is)
* update.sh can expect an environment variable ```apps_root``` which points to the top-level dir for all apps on the device.
* update.sh can expect the current working directory to be the same location as the update.sh file itself.

### Direct download of SH and JS files

The hub may provide an .sh file directly, instead of a zip. Like this:

```
{
  status: "updateNeeded",
  snapshotId: "26",
  downloadUrl: "http://x.y.com/update.sh",
  downloadType: "sh"
}
```

... or javascript file:

```
{
  status: "updateNeeded",
  snapshotId: "26",
  downloadUrl: "http://x.y.com/update.js",
  downloadType: "js"
}
```

* downloadType should be "sh" or "js" or "zip". If not provided, then "zip" is considered the default.

### Config parameters

The hub may also provide config parameters:
```
{
  status: "updateNeeded",
  snapshotId: "26",
  downloadUrl: "http://x.y.com/update.js",
  downloadType: "js",
  config: {
    meterName: "123456"
  }
}
```

* The update script can expect to receive the the config as a single environment variable called 'config',
  in stringified JSON format. So in this example, update.js can access it via: `JSON.parse(process.env.config)`

The config can be a full json structure, such as:

```
{
  status: "updateNeeded",
  snapshotId: "26",
  downloadUrl: "http://x.y.com/update.js",
  downloadType: "js",
  config: {
    meter: {
        meterName: "123456"
    },
    updater: {
        updateIntervalSeconds: 15
    }
  }
}
```

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

## Changing the update interval

The hub can (optionally) add an `updateInterval` field to tell the updater to use a specific polling interval (in seconds).
That should override whatever interval the updater was using before.

For example in this Update Me response the hub is asking the updater to check for updates
every 30 seconds from now on.

```
{
  status: "noUpdateNeeded"
  updateInterval: 30
}
```

* Minimum is 1 second. Any value below should be treated as 1 second.
* Maximum is 24 hours. Any value above that should be treated as 24 hours.
* If the updater receives an invalid (non-numeric) value, it should log and ignore it.
  It should definitely not crash or stop checking for updates!