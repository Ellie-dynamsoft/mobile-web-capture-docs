---
layout: default-layout
needAutoGenerateSidebar: true
needGenerateH3Content: true
noTitleIndex: true
title: Document Web Capture from Mobile Cameras - Use Cases - Relatively complete document capturing workflow
keywords: Documentation, Document Web Capture from Mobile Cameras, Use Cases, Relatively complete document capturing workflow
breadcrumbText: Relatively complete document capturing workflow
description: Document Web Capture from Mobile Cameras Documentation Use Cases Relatively complete document capturing workflow
permalink: /codegallery/usecases/relatively-complete-doc-capturing-workflow.html
---

# Relatively complete document capturing workflow

This sample demonstrates a relatively complete document capturing workflow: Capture continuously & Review and Adjust the detected boundaries & Edit result images.

[Check out it online](https://dynamsoft.github.io/DocWebCapture-MobileCam/samples/relatively-complete-doc-capturing-workflow/)

In this sample, we would like to achieve the workflow as below.

![Flow chart for relatively-complete-doc-capturing-workflow](/assets/imgs/relatively-complete-doc-capturing-workflow.png)

We’ll build on this skeleton page:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, minimum-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>DWC from Mobile Cameras - Relatively complete document capturing workflow</title>
</head>
<body>
</body>
<script type="module">
// Write your code here
</script>
</html>
```

## Adding the dependency

Please refer to [Adding the dependency]({{ site.gettingstarted }}add_dependency.html).

## Define necessary HTML elements


## Link CSS to HTML

`ddv.css` is the necessary css file which defines the viewer style of Dynamsoft Document Viewer.
`index.css` defines the style of elements which is in this sample.

```html
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/dynamsoft-document-viewer@1.0.0/dist/ddv.css">
<link rel="stylesheet" href="./index.css">
```

`index.css` content:

```html
html,body {
    width: 100%;
    height: 100%;
    margin:0;
    padding:0;
    overscroll-behavior-y: none;
}

#container {
    width: 100%;
    height: 100%;
}
```

## Related SDK initialization

```javascript
// Initialize DDV
await Dynamsoft.DDV.setConfig({
    license: "DLS2eyJvcmdhbml6YXRpb25JRCI6IjIwMDAwMSJ9",
    engineResourcePath: "https://cdn.jsdelivr.net/npm/dynamsoft-document-viewer@latest/dist/engine",
});

// Initialize DDN
Dynamsoft.License.LicenseManager.initLicense("DLS2eyJvcmdhbml6YXRpb25JRCI6IjIwMDAwMSJ9");
Dynamsoft.CVR.CaptureVisionRouter.preloadModule(["DDN"]);
```

## Configure document boundaries function

- Step one: The related configuration code is packaged in `utils.js`, so it should be imported.

    ```javascript
    import { isMobile, initDocDetectModule } from "./utils.js";
    ```

- Step two: Call the following function.

    ```javascript
    await initDocDetectModule(Dynamsoft.DDV, Dynamsoft.CVR);
    ```

## Configure image filter feature which is in edit viewer

```javascript
Dynamsoft.DDV.setProcessingHandler("imageFilter", new Dynamsoft.DDV.ImageFilter());
```

## Create a capture viewer

To capture images, we need to create a capture viewer.

- Customize the capture viewer `UiConfig` based on the [default one](https://www.dynamsoft.com/document-viewer/docs/ui/default_ui.html#capture-viewer) to implement the workflow.
    - Bind click event to "ImagePreview" element to show the perspective viewer
    ```javascript
    const newCaptureViewerUiConfig = {
        type: Dynamsoft.DDV.Elements.Layout,
        flexDirection: "column",
        children: [
            {
                type: Dynamsoft.DDV.Elements.Layout,
                className: "ddv-capture-viewer-header-mobile",
                children: [
                    {
                        type: Dynamsoft.DDV.Elements.CameraResolution,
                        className: "ddv-capture-viewer-resolution",
                    },
                    Dynamsoft.DDV.Elements.Flashlight,
                ],
            },
            Dynamsoft.DDV.Elements.MainView,
            {
                type: Dynamsoft.DDV.Elements.Layout,
                className: "ddv-capture-viewer-footer-mobile",
                children: [
                    Dynamsoft.DDV.Elements.AutoDetect,
                    Dynamsoft.DDV.Elements.AutoCapture,
                    {
                        type: Dynamsoft.DDV.Elements.Capture,
                        className: "ddv-capture-viewer-captureButton",
                    },
                    {
                        // Bind click event to "ImagePreview" element
                        // The event will be registered later
                        type: Dynamsoft.DDV.Elements.ImagePreview,
                        events: { 
                            click: "showPerspectiveViewer" 
                        },
                    },
                    Dynamsoft.DDV.Elements.CameraConvert,
                ],
            },
        ],
    };
    ```
 
- Create the viewer by using the new `UiConfig`.

    ```javascript
    // Create a capture viewer
    const captureViewer = new Dynamsoft.DDV.CaptureViewer({
        container: "container",
        uiConfig: newCaptureViewerUiConfig, // Configure the new UiConfig
        viewerConfig: {
            acceptedPolygonConfidence: 60, // Configure the accpeted confidence to 60
            enableAutoCapture: true, // Enable auto capture
            enableAutoDetect: true, // Enable real-time detection
        },
    });
    // Play video stream in 1080P
    captureViewer.play({
        resolution: [1920,1080],
    });
    ```

## Create a perspective viewer

- Customize the viewer's `UiConfig` based on the [default one](https://www.dynamsoft.com/document-viewer/docs/ui/default_ui.html#perspective-viewer) to implement the workflow.
    - Add a "Back" buttom to header and bind click event to go back to the capture viewer
    - Bind click event to "PerspectiveAll" button to show the edit viewer
    ```javascript
    const newPerspectiveUiConfig = {
        type: Dynamsoft.DDV.Elements.Layout,
        flexDirection: "column",
        children: [
            {
                type: Dynamsoft.DDV.Elements.Layout,
                className: "ddv-perspective-viewer-header-mobile",
                children: [
                    {   
                        // Add a "Back" button in perspective viewer's header and bind the event to go back to capture viewer
                        // The event will be registered later
                        type: Dynamsoft.DDV.Elements.Button,
                        className: "ddv-button-back",
                        events:{
                            click: "backToCaptureViewer"
                        }
                    },
                    Dynamsoft.DDV.Elements.Pagination,
                    {
                        // Bind event for "PerspectiveAll" button to show the edit viewer
                        // The event will be registered later
                        type: Dynamsoft.DDV.Elements.PerspectiveAll,
                        events:{
                            click: "showEditViewer"
                        }
                    },
                ],
            },
            Dynamsoft.DDV.Elements.MainView,
            {
                type: Dynamsoft.DDV.Elements.Layout,
                className: "ddv-perspective-viewer-footer-mobile",
                children: [
                    Dynamsoft.DDV.Elements.FullQuad,
                    Dynamsoft.DDV.Elements.RotateLeft,
                    Dynamsoft.DDV.Elements.RotateRight,
                    Dynamsoft.DDV.Elements.DeleteCurrent,
                    Dynamsoft.DDV.Elements.DeleteAll,
                ],
            },
        ],
    };
    ```

- Create the viewer by using the new `UiConfig`.

    ```javascript
    // Create a perspective viewer
    const perspectiveViewer = new Dynamsoft.DDV.PerspectiveViewer({
        container: "container",
        groupUid: captureViewer.groupUid, // Data synchronisation with the capture viewer
        uiConfig: newPerspectiveUiConfig, // Configure the new UiConfig
        viewerConfig:{
            scrollToLatest: true, // Navigate to the latest image automatically
        }
    });
    ```

- Since this viewer only shows when clicking "ImagePreview" element in the capture viewer, it should be hidden at first.

    ```javascript
    perspectiveViewer.hide();
    ```


## Create an edit viewer

To review and edit the captured images, we create an edit viewer. 

- Customize the capture viewer `UiConfig` based on the [default one](https://www.dynamsoft.com/document-viewer/docs/ui/default_ui.html#edit-viewer) to implement the workflow.
    - Add a "Back" buttom to header and bind click event to go back the perspective viewer
    ```javascript
    const newEditViewerUiConfig = {
        type: Dynamsoft.DDV.Elements.Layout,
        flexDirection: "column",
        className: "ddv-edit-viewer-mobile",
        children: [
            {
                type: Dynamsoft.DDV.Elements.Layout,
                className: "ddv-edit-viewer-header-mobile",
                children: [
                    {
                        // Add a "Back" buttom to header and bind click event to go back to the perspective viewer
                        // The event will be registered later
                        type: Dynamsoft.DDV.Elements.Button,
                        className: "ddv-button-back",
                        events: {
                            click: "backToPerspectiveViewer"
                        },
                    },
                    Dynamsoft.DDV.Elements.Pagination,
                    Dynamsoft.DDV.Elements.Download,
                ],
            },
            Dynamsoft.DDV.Elements.MainView,
            {
                type: Dynamsoft.DDV.Elements.Layout,
                className: "ddv-edit-viewer-footer-mobile",
                children: [
                    Dynamsoft.DDV.Elements.DisplayMode,
                    Dynamsoft.DDV.Elements.RotateLeft,
                    Dynamsoft.DDV.Elements.Crop,
                    Dynamsoft.DDV.Elements.Filter,
                    Dynamsoft.DDV.Elements.Undo,
                    Dynamsoft.DDV.Elements.Delete,
                    Dynamsoft.DDV.Elements.Load,
                ],
            },
        ],
    };
    ```

- Create the viewer by using the new `UiConfig`.

    ```javascript
    // Create an edit viewer
    const editViewer = new Dynamsoft.DDV.EditViewer({
        container: "container",
        groupUid: captureViewer.groupUid, // Data synchronisation with the capture viewer
        uiConfig: newEditViewerUiConfig, // Configure the new UiConfig
    });
    ```

- Since this viewer only shows when clicking "PerspectiveAll" button in the perspective viewer, it should be hidden at first.

    ```javascript
    editViewer.hide();
    ```

## Configure the workflow

- Define a function to control the viewers' visibility.

    ```javascript
    function viewerSwitch(capture, perspective, edit) {
        captureViewer.hide();
        perspectiveViewer.hide();
        editViewer.hide();
        if(capture) {
            captureViewer.show();
            captureViewer.play();
        } else {
            captureViewer.stop();
        }
        if(perspective) perspectiveViewer.show();
        if(edit) editViewer.show();
    }
    ```

- Register an event in `captureViewer` to show the perspective viewer.

    ```javascript
    captureViewer.on("showPerspectiveViewer",() => {
        viewerSwitch(0,1,0);
    });
    ```

- Register an event in `perspectiveViewer` to show the edit viewer.

    ```javascript
    perspectiveViewer.on("showEditViewer",() => {
        switchViewer(0,0,1)
    });    
    ```

- Register an event in `perspectiveViewer` to go back the capture viewer.

    ```javascript
    perspectiveViewer.on("backToCaptureViewer",() => {
        switchViewer(1,0,0);
    });    
    ```

- Register an event in `editViewer` to go back the perspective viewer.

    ```javascript
    editViewer.on("backToPerspectiveViewer",() => {
        switchViewer(0,1,0);
    });    
    ```

## Review the complete code

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, minimum-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>DWC from Mobile Cameras - Relatively complete document capturing workflow</title>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/dynamsoft-document-viewer@1.0.0/dist/ddv.css">
    <link rel="stylesheet" href="./index.css">
</head>
<body>
    <div id="container"></div>
</body>
<script src="https://cdn.jsdelivr.net/npm/dynamsoft-document-viewer@1.0.0/dist/ddv.js"></script>
<script src="https://cdn.jsdelivr.net/npm/dynamsoft-core@3.0.10/dist/core.js"></script>
<script src="https://cdn.jsdelivr.net/npm/dynamsoft-document-normalizer@2.0.11/dist/ddn.js"></script>
<script src="https://cdn.jsdelivr.net/npm/dynamsoft-capture-vision-router@2.0.11/dist/cvr.js"></script>
<script type="module">
    import { isMobile, initDocDetectModule } from "./utils.js";

    (async () => {
        // Initialize DDV
        await Dynamsoft.DDV.setConfig({
            license: "DLS2eyJvcmdhbml6YXRpb25JRCI6IjIwMDAwMSJ9",
            engineResourcePath: "https://cdn.jsdelivr.net/npm/dynamsoft-document-viewer@latest/dist/engine",
        });

        // Initialize DDN
        Dynamsoft.License.LicenseManager.initLicense("DLS2eyJvcmdhbml6YXRpb25JRCI6IjIwMDAwMSJ9");
        Dynamsoft.CVR.CaptureVisionRouter.preloadModule(["DDN"]);

        // Configure document boundaries function
        await initDocDetectModule(Dynamsoft.DDV, Dynamsoft.CVR);

        // Configure image filter feature which is in edit viewer
        Dynamsoft.DDV.setProcessingHandler("imageFilter", new Dynamsoft.DDV.ImageFilter());

        // Define new UiConfig for capture viewer
        const newCaptureViewerUiConfig = {
            type: Dynamsoft.DDV.Elements.Layout,
            flexDirection: "column",
            children: [
                {
                    type: Dynamsoft.DDV.Elements.Layout,
                    className: "ddv-capture-viewer-header-mobile",
                    children: [
                        {
                            type: Dynamsoft.DDV.Elements.CameraResolution,
                            className: "ddv-capture-viewer-resolution",
                        },
                        Dynamsoft.DDV.Elements.Flashlight,
                    ],
                },
                Dynamsoft.DDV.Elements.MainView,
                {
                    type: Dynamsoft.DDV.Elements.Layout,
                    className: "ddv-capture-viewer-footer-mobile",
                    children: [
                        Dynamsoft.DDV.Elements.AutoDetect,
                        Dynamsoft.DDV.Elements.AutoCapture,
                        {
                            type: Dynamsoft.DDV.Elements.Capture,
                            className: "ddv-capture-viewer-captureButton",
                        },
                        {
                            // Bind click event to "ImagePreview" element
                            // The event will be registered later
                            type: Dynamsoft.DDV.Elements.ImagePreview,
                            events: { 
                                click: "showPerspectiveViewer" 
                            },
                        },
                        Dynamsoft.DDV.Elements.CameraConvert,
                    ],
                },
            ],
        };

        // Create a capture viewer
        const captureViewer = new Dynamsoft.DDV.CaptureViewer({
            container: "container",
            uiConfig: newCaptureViewerUiConfig, // Configure the new UiConfig
            viewerConfig: {
                acceptedPolygonConfidence: 60, // Configure the accpeted confidence to 60
                enableAutoCapture: true, // Enable auto capture
                enableAutoDetect: true, // Enable real-time detection
            },
        });
        // Play video stream in 1080P
        captureViewer.play({
            resolution: [1920,1080],
        });

        // Define new UiConfig for perspective viewer
        const newPerspectiveUiConfig = {
            type: Dynamsoft.DDV.Elements.Layout,
            flexDirection: "column",
            children: [
                {
                    type: Dynamsoft.DDV.Elements.Layout,
                    className: "ddv-perspective-viewer-header-mobile",
                    children: [
                        {   
                            // Add a "Back" button in perspective viewer's header and bind the event to go back to capture viewer
                            // The event will be registered later
                            type: Dynamsoft.DDV.Elements.Button,
                            className: "ddv-button-back",
                            events:{
                                click: "backToCaptureViewer"
                            }
                        },
                        Dynamsoft.DDV.Elements.Pagination,
                        {
                            // Bind event for "PerspectiveAll" button to show the edit viewer
                            // The event will be registered later
                            type: Dynamsoft.DDV.Elements.PerspectiveAll,
                            events:{
                                click: "showEditViewer"
                            }
                        },
                    ],
                },
                Dynamsoft.DDV.Elements.MainView,
                {
                    type: Dynamsoft.DDV.Elements.Layout,
                    className: "ddv-perspective-viewer-footer-mobile",
                    children: [
                        Dynamsoft.DDV.Elements.FullQuad,
                        Dynamsoft.DDV.Elements.RotateLeft,
                        Dynamsoft.DDV.Elements.RotateRight,
                        Dynamsoft.DDV.Elements.DeleteCurrent,
                        Dynamsoft.DDV.Elements.DeleteAll,
                    ],
                },
            ],
        };

        // Create a perspective viewer
        const perspectiveViewer = new Dynamsoft.DDV.PerspectiveViewer({
            container: "container",
            groupUid: captureViewer.groupUid, // Data synchronisation with the capture viewer
            uiConfig: newPerspectiveUiConfig, // Configure the new UiConfig
            viewerConfig:{
                scrollToLatest: true, // Navigate to the latest image automatically
            }
        });

        // Define new UiConfig for edit viewer
        const newEditViewerUiConfig = {
            type: Dynamsoft.DDV.Elements.Layout,
            flexDirection: "column",
            className: "ddv-edit-viewer-mobile",
            children: [
                {
                    type: Dynamsoft.DDV.Elements.Layout,
                    className: "ddv-edit-viewer-header-mobile",
                    children: [
                        {
                            // Add a "Back" buttom to header and bind click event to go back to the perspective viewer
                            // The event will be registered later
                            type: Dynamsoft.DDV.Elements.Button,
                            className: "ddv-button-back",
                            events: {
                                click: "backToPerspectiveViewer"
                            },
                        },
                        Dynamsoft.DDV.Elements.Pagination,
                        Dynamsoft.DDV.Elements.Download,
                    ],
                },
                Dynamsoft.DDV.Elements.MainView,
                {
                    type: Dynamsoft.DDV.Elements.Layout,
                    className: "ddv-edit-viewer-footer-mobile",
                    children: [
                        Dynamsoft.DDV.Elements.DisplayMode,
                        Dynamsoft.DDV.Elements.RotateLeft,
                        Dynamsoft.DDV.Elements.Crop,
                        Dynamsoft.DDV.Elements.Filter,
                        Dynamsoft.DDV.Elements.Undo,
                        Dynamsoft.DDV.Elements.Delete,
                        Dynamsoft.DDV.Elements.Load,
                    ],
                },
            ],
        }; 

        // Create an edit viewer
        const editViewer = new Dynamsoft.DDV.EditViewer({
            container: "container",
            groupUid: captureViewer.groupUid, // Data synchronisation with the capture viewer
            uiConfig: newEditViewerUiConfig, // Configure the new UiConfig
        });

        // Register an event in `captureViewer` to show the perspective viewer
        captureViewer.on("showPerspectiveViewer",() => {
            viewerSwitch(0,1,0);
        });

        // Register an event in `perspectiveViewer` to show the edit viewer
        perspectiveViewer.on("showEditViewer",() => {
            switchViewer(0,0,1)
        });    

        // Register an event in `perspectiveViewer` to go back the capture viewer
        perspectiveViewer.on("backToCaptureViewer",() => {
            switchViewer(1,0,0);
        });    

        // Register an event in `editViewer` to go back the perspective viewer
        editViewer.on("backToPerspectiveViewer",() => {
            switchViewer(0,1,0);
        });    

        // Control viewers' visibility.
        function viewerSwitch(capture, perspective, edit) {
            captureViewer.hide();
            perspectiveViewer.hide();
            editViewer.hide();
            if(capture) {
                captureViewer.show();
                captureViewer.play();
            } else {
                captureViewer.stop();
            }
            if(perspective) perspectiveViewer.show();
            if(edit) editViewer.show();
        }
    })();
</script>
```

## Download the whole project

[Github](https://github.com/Dynamsoft/DocWebCapture-MobileCam/tree/master/samples/relatively-complete-doc-capturing-workflow) \| [Run](https://dynamsoft.github.io/DocWebCapture-MobileCam/samples/relatively-complete-doc-capturing-workflow/)

Please note that in order to be compatible with desktop devices as much as possible, some compatibility codes have been added to the whole project code.

`UiConfig` part is organized into `uiConfig.js` and referenced in the core code to minimize the length of the core code.


## Add auxiliary text if necessary

Sometimes, you may want to add some auxiliary text to icons to show better user guidance.

### Refer to

- [Customize Elements' Display Text](https://www.dynamsoft.com/document-viewer/docs/ui/customize/elements.html#display-text)