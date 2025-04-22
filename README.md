
# NV200 Firmware update

## 1. Add ITL SSP Communication Lib.

### 1.1 Add USB Serial lib. d2xx.jar to app/libs folder

```gradle

 implementation files('libs/d2xx.jar')
 
```

### 1.2 copy device.itl.sspcoms source code to java folder in root project

daemon\
|_____app\
|_________src\
|_____________main\
|________________java\
|__________________com.kbao.daemon\
|__________________device.itl.sspcoms <--

### 1.3 Copy NVMachine.kt to daemon project

Example \
place NVMachine.kt file into hardware/nv200/ folder

## 2. Usage

### 2.1 Initialization and connection

```kotlin

    var nv200: NVMachine? = null

    ....

    //init
    nv200 = NVMachine(applicationContext)

    //impliment event listener
     nv200?.setMachineEventListener(object : MachineEventListener{
            override fun onDisconnected(dev: SSPDevice?) {
                TODO("Not yet implemented")
            }

            override fun onDeviceEvent(event: DeviceEvent?) {
                TODO("Not yet implemented")
            }

            override fun onPayOutEvent(payoutEvent: SSPPayoutEvent?) {
                TODO("Not yet implemented")
            }

            override fun onDeviceUpdateStatus(update: SSPUpdate?) {
                TODO("Not yet implemented")
            }

            override fun onNewDeviceSetup(dev: SSPDevice?) {
                TODO("Not yet implemented")
            }

    } )

    //connect
    nv200?.connect(
        onFail = {
             Daemon.Log.e(TAG,"Failed! connect to NV200")
        }, 
        onConnected = {
            Daemon.Log.i(TAG,"Connected to NV200")
            ....
        }
     )
```
### 2.2 Flash firmware

```kotlin
// update file 
val path =  context.cacheDir
val outputFile = File("$path/firmware_nv.bv1")
val firmwarePath = outputFile.path
// Note! You can use any firmware of NV200 (.bv) storaged in directrory


// setup firmware update file 
val update = SSPUpdate(firmwarePath)
update.sspDeviceType = SSPDeviceType.BillValidator
update.fileData = ByteArray(File(firmwarePath).length().toInt())
val ds = DataInputStream(FileInputStream(File(firmwarePath)))

//read firmware data to ssp update buffer
ds.readFully(update.fileData)
ds.close()

//commit firmware data
update.SetFileData()

//start update
nv200?.update(update)

```

### 2.3 Firmware update progress event listening

In section 2.1 only use onDeviceUpdateStatus override function for listent to pdate progress.

```kotlin
override fun onDeviceUpdateStatus(update: SSPUpdate?) {
    when(update?.UpdateStatus) {

        SSPUpdate.SSPDownloadStatus.dwnIdle -> {
            // update in idle state
        }
        SSPUpdate.SSPDownloadStatus.dwnInitialise -> {
            // initialization event.
        }

        SSPUpdate.SSPDownloadStatus.dwnRamCode -> {
            // RAM code update
            progress = update.blockIndex/update.numberOfBlocks
        }

        SSPUpdate.SSPDownloadStatus.dwnMainCode -> {
            progress = update.blockIndex/update.numberOfBlocks
            // Main code update
        }

        SSPUpdate.SSPDownloadStatus.dwnError -> {
            // Error during update
        }

        SSPUpdate.SSPDownloadStatus.dwnComplete -> {
            // Complete update
            nv200?.close()
            nv200?.setMachineEventListener(null)
            nv200 = null
        }

        SSPUpdate.SSPDownloadStatus.dwnRetry -> {
            // update retry event
        }

        null -> TODO()
    }
}
```

### 2.4 Get firmware version

```kotlin
   val device = nv200?.getDevice()
val version = device?.firmwareVersion // Get version as string
```

