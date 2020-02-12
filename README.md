## Understanding how Electron works
The Electron framework is based on Chromium architecture ([Architectural overview](https://www.chromium.org/developers/design-documents/multi-process-architecture)). The way it works (you may saw this in the chrome browser) is that each tab is an independent process. This is not a coincidence, the decision was made so the app can be more modular, run concurrently and less prone to fails -- a failed process won't crash the entire app. Each process runs independently within its own memory address and is being handled asynchronously by the operating system.

## In Electron we differentiate two types of processes:

### `main`

The *main* is responsible for creating browserWindow instances, handle all the events in our app, create native menus, auto updating, and more. This is where we have the main entry into the app and where the native GUI is handled.

### `renderer`

The *renderer* processes (you can multiple of them in the screenshot) are responsible for running the user-interface of the app. Each window within the app will be a separate *renderer* process and it can handle multiple *renderer* processes still if you'd use *iframes* or other type of embedded content ([More info here](https://www.electronjs.org/docs/api/webview-tag)). We can easily verify this by opening the `About` window and see in the Activity monitor a new *renderer* process.

Seeing multiple *renderer* is actually not a bad thing. Sharing resources between multiple *renderer* processes leads to more performance because we free CPU resources. The underlying Chromium architecture is doing this under the hood on lower (OS) level. When loading heavy modules or inter-process communications, we also can see a bump in system resources required, but that is temporarily, until the loading completes and we get resource freeing. Remember, that this is all for performance sake -- fast loading in memory-> subsequent fast memory freeing. The whole process is very technical and low level, we just need to know that the OS schedule processes and handles resources.

![GitHub Logo](/resources/process-comms.png)

*GPU helpers* that we can see in the Activity Monitor are also related to how Electron(Chromium) tries to improve performance. GPU acceleration opens an additional process for every window so the interactions within the app will be faster.

*Electron (app) helpers* are processes spawn internally by Chromium to interact with the operating system. Any time we read/write to the filesystem or otherwise interact with native capabilities -- we need this helper process to interact with native APIs.

### Things that could be done to improve resource handling

Going further we can try to improve some of the things we're implementing. Eg, *main* -> *renderer* process communications are resource expensive. We could avoid making unnecessary event callback calls in parts of the application that don't necessary need it at that moment. Also, we'd need to investigate further if there are some kind of memoization mechanisms on how to handle inter process communication.

### Conclusion

Our app handles resources very good, we can see bumps in RAM/CPU whenever we interact with *renderer* and it needs to send more events to the *main* process, but this is to be expected and is nothing out of the ordinary.
