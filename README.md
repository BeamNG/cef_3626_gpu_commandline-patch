What and why
============

This is the 3626 branch of the Chromium Embedded Framework (CEF) patched with the ability to select the GPU device that should be used. This is important for games and off-screen rendering as the software shares texture resources and that only works if the same GPU is used.

We re-added some old command line arguments that were deprecated and removed in Feb 2019.

How to pass the GPU id correctly to CEF with these binaries: You need to inject the command line options into the render process. 

The options are:
`--gpu-vendor-id=<GPU VENDOR ID> --gpu-device-id=<GPU DEVICE ID>`

for example:

`--gpu-vendor-id=0x10de --gpu-device-id=0x1e87`


Integration example using `OnBeforeChildProcessLaunch`:
```c++
extern uint32_t g_gpu_device_id; // fill this variable 
extern uint32_t g_gpu_vendor_id;
void CefApp::OnBeforeChildProcessLaunch(CefRefPtr<CefCommandLine> command_line) {
    if (command_line->HasSwitch("type")) {
        if (command_line->GetSwitchValue("type") == "gpu-process") {
            char tmp[32] = "";
            _snprintf(tmp, sizeof(tmp), "0x%x", g_gpu_vendor_id);
            command_line->AppendSwitchWithValue("gpu-vendor-id", tmp);
            
            _snprintf(tmp, sizeof(tmp), "0x%x", g_gpu_device_id);
            command_line->AppendSwitchWithValue("gpu-device-id", tmp);
        }
    }
}

// use DirectX to set the gpu ids
void initGPUDX10MockupExample() {
	DXGI_ADAPTER_DESC1 desc;
	EnumAdapter->GetDesc1(&desc);
	// ...
	g_gpu_device_id desc.DeviceId;
	g_gpu_vendor_id = desc.VendorId;
	// ...
}
```

How to compile
===============

This is how we downloaded, patched and built this:

1. Download CEF:
```batch
Python27\python automate-git.py --branch=3626 --no-release-tests --no-debug-tests --no-debug-build --download-dir=build_x64 --x64-build --no-build --no-distrib
```

2. Copy patch file:
```batch
xcopy /y beamng_gpu_cmdline.patch build_x64\chromium\src\cef\patch\patches
```

3. Adjust the `patch.cfg` file:

You need to add your new patch into the list, so it gets used.
```
  {
    'name': 'beamng_gpu_cmdline',
  },
```
or, use the file we have provided here (the rest of the patches might have changed, be careful
```batch
xcopy /y patch.cfg build_x64\chromium\src\cef\patch\
```

4. Build CEF:
```batch
Python27\python automate-git.py --branch=3626 --no-release-tests --no-debug-tests --no-debug-build  --download-dir=build_x64 --x64-build --force-build
```

Binaries
===============

We provide one set of pre-built binaries for you at https://nextcloud.beamng.com/s/f328nyoHGecEbbG
