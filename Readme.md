![DailyBuild](https://github.com/wellenvogel/avnav/workflows/DailyBuild/badge.svg)

(C) Andreas Vogel andreas@wellenvogel.net
This project contains the code to use the raspberry pi as a nav computer on a sail boat.
A lot of detailed explanations together with ready to go images can be found at my [homepage](http://www.wellenvogel.net/software/avnav/index.php).
The core of the documentation is available in the docs directory.

Snapshot builds can be found [here](https://www.wellenvogel.net/software/avnav/downloads/daily/fr).
They are build from the [daily-builds branch](https://github.com/wellenvogel/avnav/tree/daily-builds)

Releases are available [here](https://www.wellenvogel.net/software/avnav/downloads/release/fr).

Got it! I’ll analyze the AVNav source code and create a developer-friendly README in the same detailed and structured style as your NautiPi README. The focus will be on making it easier for developers—especially new contributors—to understand and navigate the codebase.

I’ll let you know as soon as the README draft is ready.


# 🧭 **AVNav – Open-Source Chartplotter & Boat Data Server with Web App**

---

## What is AVNav and why is it important?

**AVNav** is a free, open-source navigation software designed for recreational boaters to run on small computers like the Raspberry Pi, as well as on PCs and Android devices. Like other chartplotter applications, AVNav displays nautical charts and marks the boat’s GPS position, but it goes further – it acts as a **“navigation server”** that collects and multiplexes data from various onboard sources (GPS, AIS, wind, depth, etc.) and serves a full-featured web application for charting and instrumentation. This server–client design means you can access your boat’s navigation dashboard from **any device with a modern web browser**, be it a cockpit tablet, a phone, or a laptop, all connected via Wi-Fi. AVNav is optimized for touchscreens and allows customizable layouts to fit different screen sizes and user preferences.

* **For Users (Boaters):** AVNav makes it easy to set up a **modern chartplotter and instrument panel** without proprietary hardware. Install it on a Raspberry Pi or other computer, connect your NMEA0183 sensors (GPS, AIS, log, depth, wind, etc.) via USB, serial, Bluetooth, or network, and AVNav will automatically detect them and start logging data. The system can create its own onboard Wi-Fi hotspot, so crew devices can connect to view live charts, GPS position, AIS targets, and instrument readouts in any web browser – no additional apps required. It supports **waypoints, routes, tracks, anchor watch alarms, AIS collision warnings**, and more out of the box. Because AVNav runs headless (no full desktop needed), it’s lightweight in power usage and ideal for 24/7 operation on a boat’s batteries.

* **For Developers:** AVNav provides an open, modular platform to build upon. Its backend is written in Python for easy extensibility, and it features a plugin system that lets you add new data sources or outputs (for example, integrating NMEA2000 data via a **canboat** plugin for AIS/transducer data). The frontend is a modern JavaScript (React) web application, so developers can create custom UI components or tweak the styling (e.g. custom CSS for new instrument displays). The project’s code is well-organized into logical components, and clear documentation (both in-code and in the `docs/` folder) helps newcomers navigate the architecture. AVNav’s goal is to be **easy to modify and integrate** – whether you want to add a new sensor input, support a new chart format, or enhance the user interface, the framework is in place to do so without having to rewrite core systems.

**AVNav** is an important piece of the open-source boating ecosystem because it enables a powerful yet **independent navigation setup**. Boaters are not locked into proprietary chartplotters – with AVNav they can use affordable hardware and open charts, while preserving flexibility to customize and extend. By bridging the gap between traditional **desktop plotter apps** and modern **web-based interfaces**, AVNav offers the best of both: a full-featured navigation system that you can access from anywhere on board (or even remotely), with the transparency and adaptability of open-source software. This makes onboard digitalization more accessible, affordable, and community-driven.

---

## 💾 Basic Project Overview

**AVNav’s Workflow and Features:** After installing AVNav (e.g. on a Raspberry Pi using the provided image or on Windows via the installer), the software runs as a background service (the “nav server”). In a typical setup, the Raspberry Pi creates a Wi-Fi hotspot on the boat, allowing tablets or laptops to connect. Users then simply open a browser to the AVNav web UI. A setup like this enables multiple displays – for instance, one tablet can show the chart plot with AIS targets, while another shows a custom instrument dashboard – all powered by the same AVNav server. AVNav can also serve as a pure **NMEA data hub**: it takes NMEA 0183 inputs (from serial, TCP/UDP, Bluetooth, etc.) and redistributes them to other apps or devices as needed (acting as a **multiplexer and Wi-Fi gateway**). In fact, you can use third-party navigation software (like OpenCPN or iNavX) by connecting them to AVNav’s NMEA output streams, while AVNav simultaneously feeds its own web interface – giving you flexibility in how you use the data.

**Zero-Config to Start:** AVNav is designed to work with minimal setup. On first launch, it auto-detects common connections (USB GPS or AIS receivers, etc.) by scanning serial ports and even attempting auto-baud rate detection for NMEA feeds. The default configuration (defined in an XML file `avnav_server.xml`) is tuned so that most users won’t need to edit it to get basic functionality. All essential features are available immediately: the web app will display your position on a map (once GPS is connected), show AIS targets (if an AIS receiver is feeding data), allow you to set waypoints or plan routes, and start logging your track. Chart management is straightforward as well – you can download or convert charts and place them in the charts directory, and the server will serve them to the UI. For advanced setups, users can adjust the XML config or use the web interface’s settings menus for things like enabling extra outputs, but the out-of-the-box experience covers the basics for a quick start.

**Key Features at a Glance:**

* *Chart Plotting:* Interactive display of raster charts (e.g. **BSB/KAP**, NV charts, MBTiles, GEMF tile sets) and support for certain vector charts (oeSENC). AVNav includes tools to **convert common chart formats** into tile sets (GEMF) if needed, using GDAL under the hood for formats like BSB. Chart tiles are served efficiently to the browser (similar to how Google Maps serves map tiles), enabling smooth panning and zooming.

* *Instruments & Sensors:* Real-time display of instrument data such as speed, depth, wind, heading, temperature, etc. The web UI includes customizable **instrument panel widgets** (gauges, digital readouts, etc.) that you can arrange on different pages. AVNav integrates with a variety of sensors through NMEA 0183 and even provides handlers for devices like the Raspberry Pi Sense HAT or environmental sensors (BME280/BMP180) – these can feed data into AVNav’s system and be shown on screen (or used in alerts)【19†】. All sensor data and calculated data (e.g. VMG, ETA) are available in the server’s internal data store, updated in real time.

* *AIS Integration:* AVNav can decode AIS messages from any connected AIS receiver (NMEA 0183 sentences VDM/VDO). AIS targets are plotted on the chart with icons, and detailed information can be viewed (such as vessel name, call sign, course, speed, CPA/TCPA, etc.). The AIS decoding functionality is derived from the gpsd project’s proven implementation【17†】, ensuring reliability. You can set alarms for AIS targets (e.g. collision warnings) and the system will alert you via on-screen messages and optional sounds.

* *Routing and Navigation:* The software supports creating and editing **routes and waypoints**. Routes can be activated so that AVNav will monitor cross-track error, distance to the next waypoint, and other navigation data. It generates standard NMEA navigation sentences (like RMB) on its output, which means you can even feed these to an autopilot – AVNav can effectively guide an autopilot along a planned route. It also continuously computes bearing and distance to the next waypoint, time to go, etc., displaying these on the instrument panel. **Anchor watch** functionality is built in as well (set an anchor position and radius, and get an alarm if the boat drifts out). All alarms (anchor drag, AIS CPA, etc.) are managed by the server’s alarm handler and can trigger audible alerts (the `sounds/` directory contains alarm sound files).

* *Data Recording:* AVNav will log your **track** (your boat’s path) automatically to GPX files (stored in the `tracks/` directory) for later review. It can also record NMEA data logs if configured. There’s a routes directory for saving route plans (GPX format routes can be imported/exported). These files can be accessed for backup or analysis, making it easy to review your voyage or share routes.

* *Network Outputs & Multiplexer:* Every NMEA sentence that AVNav receives (from GPS, AIS, etc.) is internally queued and can be redistributed. By default, AVNav opens a TCP server socket (port 34567 by default) to stream raw NMEA data to any client. You can configure additional output channels (additional TCP ports, UDP broadcasts, or even output to a serial port) via the config. This means AVNav can serve as a **NMEA 0183 multiplexing hub** – combining data from multiple sources and feeding them to multiple outputs simultaneously. For example, you could have an autopilot listening on a serial port and a laptop running OpenCPN on Wi-Fi, both receiving the same data from AVNav. The integrated Wi-Fi access-point capability on Raspberry Pi (with provided system scripts) further allows making a self-contained networked nav system on your boat.

AVNav’s flexibility in deployment is notable: you can run the **exact same software on a Raspberry Pi, a Windows PC, a Linux/Mac laptop, or even as an Android app**. In all cases, you get the same web-based interface and capabilities. For instance, on Android, AVNav is available as a standalone app (with the server code natively implemented in Java, and the same web UI running in an embedded browser). There’s also a special “AvNav Touch” variant for Raspberry Pi with a directly connected touchscreen, and an integration package for OpenPlotter (so it can work alongside SignalK easily). This cross-platform design means developers and tinkerers can choose their preferred hardware and still enjoy the full AVNav feature set.

---

## 🧩 Basic Structure Overview

Below is an overview of the AVNav project’s file structure, illustrating the main components of the codebase:

```plaintext
avnav/
├── server/                   # **Python backend server** (core logic for data, IO, web, etc.)
│   ├── avnav_server.py       # Main entry point for the server (starts handlers, web server, etc.)
│   ├── avnav_manager.py, avnav_store.py, avnav_api.py  # Core modules managing data flow, storage, API endpoints
│   ├── handler/              # Package of **handler modules** for various inputs/outputs and features
│   │   ├── charthandler.py       # Serves chart tiles and manages chart catalog/import
│   │   ├── nmealogger.py        # Handles logging of NMEA sentences to file
│   │   ├── avnserial.py / udpreader.py / socketreader.py   # Readers for serial, UDP, TCP NMEA input streams
│   │   ├── serialwriter.py / udpwriter.py / socketwriter.py # Writers for outputting NMEA to various channels
│   │   ├── avnrouter.py, route and track handlers    # Manages routes (waypoints) and tracking
│   │   ├── alarmhandler.py       # Triggers alarms (e.g., anchor alarm, AIS CPA alarm)
│   │   ├── settingshandler.py    # Manages retrieval and update of settings (from config or commands)
│   │   ├── pluginhandler.py      # Dynamically loads external plugin modules
│   │   ├── signalkhandler.py     # (If enabled) feeds data to a Signal K server or clients
│   │   ├── import/export handlers (importer.py for charts, etc.)
│   │   └── ... (many more handlers for sensors, Bluetooth, AIS decoding, etc.【19†】)
│   ├── plugins/               # **Built-in plugins** (optional modules extending functionality)
│   │   ├── canboat/               # Example plugin: NMEA2000 support via canboat:contentReference[oaicite:31]{index=31}
│   │   └── testPlugin/            # Example template plugin (with plugin.py, plugin.js, plugin.css)
│   ├── __init__.py, etc.      # Package initialization and utility modules (e.g., avnav_util.py)
│   └── avnav_server.xml       # Default server configuration (XML format) for enabling/disabling features
├── viewer/                   # **Frontend Web UI** (a single-page application for charts and instruments)
│   ├── index.html, App.jsx    # Main HTML and React JSX app entry point【12†】 
│   ├── components/            # React components (UI widgets and dialogs)
│   │   ├── CanvasGauges.jsx, CanvasGaugeDefinitions.js  # Instrument gauge widgets (boat speed, wind, etc.)
│   │   ├── AisTargetWidget.jsx, AisInfoDisplay.jsx      # AIS target list and details popup
│   │   ├── EditRouteWidget.jsx, Route planning dialogs  # UI for creating/editing routes and waypoints
│   │   ├── AlarmWidget.jsx, AnchorWatchDialog.jsx       # UI indicators for alarms and anchor watch settings
│   │   ├── ... (dozens of other UI components for various features)
│   ├── map/                  # Mapping engine integration (OpenLayers map handling, chart tile sources)
│   │   ├── mapholder.js, avnavchartsource.js  # Logic for loading map tiles, panning/zooming (uses OpenLayers API)【27†】
│   ├── util/                 # Utility JS (e.g., for formatting, persistence, etc.)
│   ├── css/                  # Stylesheets (if not using Tailwind; might include base styles or themes)
│   ├── package.json          # Node.js project file for building the viewer (uses webpack, etc.)【33†】
│   └── webpack.config.js     # Webpack configuration for bundling the JavaScript app
├── chartconvert/             # **Chart conversion tools** (Python scripts to convert charts to GEMF/MBTiles)
│   ├── convert_nv.py, convert_mbtiles.py, etc.  # Scripts to convert various chart formats (BSB, NV, etc.):contentReference[oaicite:32]{index=32}
│   └── tiler_tools/          # Helpers (using GDAL, image processing) for slicing images into tiles
├── libraries/               # **Bundled libraries** (third-party Python libs not installed via pip)
│   └── gpxpy098/             # Example: GPXpy library for parsing GPX files (used for tracks/routes)
├── android/                 # **Android app source** (Java-based server, reusing the same web UI)
│   ├── src/main/java/...     # Java packages for Android (includes NMEA/AIS handling in Java)
│   ├── assets/www/           # Embedded web assets (the viewer build) to use in the app’s WebView
│   └── ... (Gradle build files, Android Manifest, etc.)
├── raspberry/               # Platform-specific setup for Raspberry Pi (systemd service files, network config)【15†】
├── windows/                 # Platform-specific files for Windows (installer scripts, service wrapper)【24†】
├── osx/                     # Platform-specific files for macOS (app bundle for chart converter, etc.)
├── docs/                    # Documentation and guides (user manual, developer notes, images, etc.)
└── LICENSE.md, Readme.md    # License information (open-source) and project README (to be improved!)
```

In summary, the **backend** (`server/`) is where all the data processing and server logic lives, while the **frontend** (`viewer/`) is a rich JavaScript application that runs in the browser. The `chartconvert/` tools and platform-specific folders highlight that AVNav is not just a single script but a full system intended to run on various devices and assist with chart preparation. The presence of an `android` subproject indicates that a parallel implementation exists for Android (where the backend is in Java rather than Python, but the frontend HTML/JS is the same). The **plugin architecture** is evident in the `plugins/` folder and the plugin handler – this allows developers to package new features (optionally including their own backend code and UI components) without modifying the core code, simply by dropping in a new plugin directory.

---

## 🛠️ Backend Technology (Python Server)

The AVNav server is implemented in **Python 3**, chosen for its ubiquity on Raspberry Pi OS and ease of development. It runs as a standalone process (launched via the `avnav_server.py` script) and uses Python’s built-in libraries to handle web serving and background tasks. Key technical aspects of the backend include:

* **Custom HTTP/WebSocket Server:** AVNav’s server includes an HTTP server (based on Python’s `http.server` with a threading mix-in) to serve both the static files of the web UI and a set of REST-like API endpoints. It also implements a WebSocket handler【35†】, allowing the frontend to maintain live connections for real-time data (for example, streaming instrument data or AIS updates without polling). This means the browser interface can update instantly as new NMEA data arrives. The HTTP API provides JSON-formatted data for things like current position, active route info, list of AIS targets, etc., which the frontend requests periodically or via WebSocket pushes. (Developers can find the definitions of these endpoints and data formats in `avnav_api.py` and the various handlers – e.g., the `settingshandler` provides config data, `charthandler` provides tile data, etc.)

* **Modular Data Handlers:** All input/output and feature-specific logic is encapsulated in **handler modules** (in `server/handler/`). For example, reading NMEA from a serial port is handled by `avnserial.py`, which runs in its own thread listening to the port. Data from all sources (serial, UDP, TCP, AIS, etc.) are funneled into a central queue and processed by the **decoder** (`avndecoder.py`) which interprets NMEA sentences (GPS fixes, AIS messages, instrument data, etc.) and updates an in-memory data store. This decoded data store is the single source of truth for all live data – any component (or client via API) can query the latest values (e.g., current SOG, wind speed, waypoint bearing). On the output side, handlers like `serialwriter.py`, `udpwriter.py` take data from that same store or queue and forward it to external outputs. The server thus acts as a central hub coordinating all data flows. **Auto-detection** logic is part of the startup: AVNav scans for new serial ports (including USB-serial adapters and Bluetooth virtual COM ports via DBus) and tries common baud rates until NMEA data is recognized – when successful, it will spawn a reader for that port automatically. This plug-and-play approach greatly simplifies setup for users.

* **NMEA Multiplexer & Gateway:** As noted, the backend can output data to multiple channels. By default it opens a TCP socket on port 34567 to serve raw NMEA sentences to any client. Users can enable additional outputs in the config (for instance, have AVNav broadcast UDP packets to a network, or forward NMEA to another serial line like an autopilot input). The ability to combine multiple inputs and outputs classifies AVNav as a full NMEA 0183 **multiplexer**. Moreover, when running on a Raspberry Pi, AVNav can optionally manage a **WLAN access point** (with the help of system scripts under `raspberry/`) so that all client devices can simply connect to a “boat Wi-Fi” and receive data or the web interface. This is especially useful offshore or when you want a closed network on the boat.

* **Chart Management:** AVNav’s backend is responsible for handling chart files. Raster charts (like KAP/BSB or others) need to be converted into tiled images for efficient display – the **chartconvert** utilities can be run on a PC or on the Pi itself to produce `.gemf` (tile bundle) or MBTiles files. The `charthandler.py` on the server knows how to read these tile sets (GEMF, MBTiles) and will serve the correct map tiles to the frontend on request (when you pan/zoom the map). AVNav also supports **oeSENC vector charts** (used by OpenCPN’s OEM format); these can be used if the licensed charts are placed on the system. The chart handler keeps an index of available charts and their coverage, enabling the web app to seamlessly scroll across chart boundaries. The server doesn’t do heavy GIS computations – it relies on OpenLayers in the browser for rendering – but it ensures the data (tiles) is readily available. Chart packs can be added by dropping them in the charts directory or via an **import** mechanism (copying to an `import/` folder and letting the `importer.py` trigger conversions). This design allows offline use: once charts are imported on the Pi, no internet is required for the map.

* **Routing, Tracks, and Alarm Logic:** Higher-level nav features are implemented server-side so they continue running even if no client is actively viewing the data. For example, if an **anchor watch** is set, the `alarmhandler` will monitor the GPS position against the anchor radius in the background and set off an alarm if needed (which will be delivered to any connected UI and also as a sound). Route monitoring is done by the `avnrouter`/`avnroute` code: when a route is active, the server continually calculates cross-track error, next waypoint info, and even generates an NMEA **RMB sentence** for the next waypoint, which can be picked up by autopilot or other instruments listening on the NMEA output. The **trackwriter** handler records the vessel’s track points at intervals to a GPX file. All these run regardless of whether the web UI is open, so data isn’t missed. The server also has a **settings store** so that certain preferences or calibration values can persist (some can be edited via the UI’s settings dialogs).

* **Plugin System:** To keep the core maintainable and allow community contributions, AVNav supports plugins on the backend. A plugin is essentially a Python module (and optional frontend files) that can be placed in the `plugins/` directory (or a user-specific plugin folder) and will be loaded by the `pluginhandler`【19†】. Plugins have access to the server’s APIs to read data or inject new data. For example, the provided **canboat plugin** uses the third-party *canboat* utility to interface with NMEA2000 networks, translating N2K messages into NMEA 0183 or JSON that AVNav can use – thereby letting AVNav display N2K instrument data even though its core is NMEA 0183 focused. Other plugins could integrate things like custom sensors, or output data to cloud services, etc. The plugin system means developers can extend functionality **without modifying core files**, making it easier to maintain custom additions. (On the frontend side, a plugin can also include a `plugin.js` and `plugin.css` to add new UI elements – see below.)

* **Performance and Reliability:** The Python backend, while single-process, is multi-threaded (each data source handler runs in its own thread, and the WebSocket/HTTP requests are handled in separate threads as needed via the ThreadingMixIn). Python’s performance is sufficient for the typical data rates of NMEA sentences. The server uses internal queues and locks to coordinate data access safely. Logging is built-in (to console or file) to help with debugging. For deployment, AVNav is typically set up as a **systemd service** on Linux (sample service files are in `raspberry/` for RPi) or as a Windows service (via an NSSM-based installer in `windows/`). This ensures the AVNav server starts on boot and keeps running. The community has used AVNav on long sailing voyages, so it’s proven to be stable for continuous use.

*Technical Note:* On Android, the **AVNav app** includes a native Java reimplementation of the server. Java classes in `android/src/main/java/de/wellenvogel/avnav` mirror much of the Python functionality (e.g., AIS decoding, NMEA reading). This was done to comply with Android OS limitations and optimize performance on mobile devices. However, the overall architecture (server plus web UI) remains the same. From a developer perspective, you can usually focus on the Python code for core logic changes, and only delve into the Android Java code if you aim to improve the Android-specific performance or packaging. The **display code is identical across platforms** – the Android app simply loads the same compiled React/JS app in a WebView and connects it to the Java backend instead of a Python backend.

---

## 🌐 Web UI (Frontend)

The AVNav frontend is a **single-page web application** that provides all user interface components for the chartplotter and instrument panel. It is built with modern web technologies for responsiveness and performance:

* **Framework:** AVNav’s UI is written primarily in **JavaScript (ES6+) and JSX**, using the **React** library to manage UI components and state. The code is organized into reusable React components (found in the `viewer/components/` directory) for everything from the compass gauge to the AIS target list. This modular approach makes it easier to understand and modify parts of the interface without affecting others. The entry point `App.jsx` ties everything together and defines the overall layout and routing between different views (charts, dashboard, settings, etc.). The app is bundled using Webpack (see `viewer/webpack.config.js` and npm scripts in `package.json`【33†】), which means developers can run a development server or watcher to live-reload the UI during development.

* **Mapping with OpenLayers:** For the chart display, AVNav uses the **OpenLayers** mapping library on the frontend【27†】. OpenLayers handles the rendering of map tiles, chart layering, and interactive panning/zooming. In AVNav’s `viewer/map/` module, there are adapters that feed chart tiles (from the server’s chart handler) into OpenLayers as a tile source【27†】. By leveraging OpenLayers, AVNav gains a lot of functionality: smooth zooming, different map projections support, and overlays for markers and AIS targets. The web app displays boat position and course on the chart, can draw routes and tracks on top, and uses OpenLayers features for things like dragable markers (e.g., to place a waypoint). This choice of a web map framework makes AVNav’s charts behave much like familiar online maps.

* **Responsive & Touch-Optimized UI:** The interface is designed to be used on devices as small as smartphones and up to large tablets or PC screens. It employs responsive design principles – for example, menus and dialogs scale for touch input, and the layout can change based on orientation. The **widget layout system** in AVNav allows users to customize what information is shown and where: you can create multiple pages (e.g., one page could be a dedicated instrument panel with large numeric readouts, another page a 航 chart view) and switch between them easily. The UI components for editing these layouts (like `EditPageDialog.jsx`, `EditWidgetDialog.jsx`) allow adding or removing instruments on a page, choosing gauge styles, etc., without coding. Different color themes or night mode can be achieved via CSS (the project supports custom CSS overrides for those who want to restyle the app). All buttons and controls are designed to be finger-friendly (suitable for a minimum 7-inch touchscreen as noted by the developers).

* **Features in the Web UI:** Nearly all navigation operations can be done through the web UI. Some highlights:

  * **Chart Viewer:** Pan and zoom charts, select which chart set to use if multiple are available, toggle layers like AIS or tracks. You can add markers by long-pressing on the map, measure distances, and view chart info.
  * **Instrument Dashboard:** Display numeric or gauge readouts for any NMEA data (SOG, COG, depth, wind, etc.). Gauges (in `CanvasGauges.jsx`) are drawn using HTML5 Canvas for performance and can mimic analog instruments. You can customize the dashboard layout and even create multiple tabs of instruments to swipe through.
  * **AIS Interface:** Vessels are shown on the map with symbols. Tapping a target brings up the AIS detail dialog (`AisInfoDisplay.jsx`) showing all information (ship name, MMSI, AIS status, distance, etc.). The UI can list all AIS targets, and you can filter or sort them. CPA/TCPA calculations are done server-side and critical ones can trigger visual/audio alarms.
  * **Routes and Waypoints:** The UI provides dialogs to create/edit waypoints and routes (`EditRouteWidget.jsx`, etc.). You can graphically place waypoints on the map and reorder or edit coordinates precisely. When a route is active, the next waypoint is highlighted and navigation data is shown (ETA, bearing, XTE, etc.). You can also import or export GPX routes via the UI.
  * **Anchor Watch:** A dedicated anchor watch dialog lets you set the anchor position (e.g., at current boat location or a specified point) and define a radius. Once set, the UI will display a circle on the chart and an alarm can be triggered if the boat icon leaves that circle.
  * **Settings and Configuration:** Common settings (units, display preferences, network settings) are accessible via the web UI. For more advanced config (like adding a specific input port or changing NMEA output settings), one might still edit the XML config file manually, but the goal is that normal operation rarely requires touching backend config. The UI does allow enabling/disabling some features on the fly (for example, toggling a data source or adjusting AIS alarm thresholds) – these are handled by the `settingshandler` and reflected in the UI’s settings dialogs.
  * **Logs and Debugging:** There is a simple log viewer in the UI to help users see recent system messages (which can aid in troubleshooting, e.g., if a device is not recognized). Also, the ability to download track files or other data is provided.

* **Communication with Backend:** The frontend communicates with the Python server via a combination of RESTful HTTP endpoints and WebSockets. For example, when the app first loads, it might make an HTTP GET request to an endpoint (like `/api/status.json`) to get current boat data, list of charts, etc. Meanwhile, a WebSocket connection is established (to an endpoint like `/ws`), over which the server pushes updates (e.g., new AIS target, updated instrument data every second, alarm triggers). This efficient communication scheme means the UI is very **realtime** – instrument gauges move smoothly and AIS targets appear without manual refresh. The code for managing these connections can be found in the viewer utility modules (likely in a central data service component).

* **Extensibility (Frontend Side):** If a plugin provides a `plugin.js` and `plugin.css`, the web UI will load these, allowing new UI components or pages to be injected. This could be used, for example, to add a new type of instrument widget or a special control panel for a new feature. Additionally, the UI’s look can be tweaked by overriding CSS (the project encourages using the existing design system but gives freedom to adapt styles). For developers, since the UI is a standard React app, one can use familiar Node.js tooling to modify it. The state management is mostly handled within React’s component state or simple pub-sub; those familiar with React will find the structure straightforward.

In summary, the AVNav frontend is what turns the raw data from the Python server into an interactive experience. Its use of web technologies means it is **platform-agnostic** – any device with a browser can be a full client – and it benefits from the vast ecosystem of web UI libraries. The maintainers chose a path that avoids a heavyweight dedicated GUI toolkit (no X11 or Qt needed on the Pi) in favor of a lean web interface, which significantly lowers the hardware requirements and improves accessibility (headless operation). AVNav’s UI demonstrates that a rich chartplotter can indeed run in a browser without sacrificing functionality.

---

## 🗄️ Plugins & Extensibility

AVNav was built with extensibility in mind, to cater to the diverse needs of the sailing community. Both the backend and frontend can be extended through **plugins** and custom configurations, allowing developers to introduce new features or support new hardware without changing the core codebase.

* **Backend Plugins:** A plugin on the server side is typically a Python module (and possibly additional resources) placed under the `plugins/` directory (or in the user’s `$HOME/avnav/plugins` directory at runtime). The plugin should define a class `Plugin` with appropriate methods; AVNav will detect it and run it in conjunction with other handlers【22†】. Plugins can subscribe to NMEA data streams, inject new data, or provide new API endpoints. For example, the included `canboat` plugin uses the `canboat` utility to listen to NMEA2000 messages off a CAN bus, converts them into NMEA0183 sentences or JSON, and feeds them into AVNav – thereby extending AVNav to support N2K networks with minimal effort. Another example is a *dummy* `testPlugin` provided as a template for developers, which demonstrates how to structure a plugin with Python code (perhaps simulating a sensor or custom logic) along with an optional web component (JavaScript and CSS) for the UI. Using plugins, one could integrate things like engine data, weather forecasts, or cloud syncing of routes/tracks into AVNav without touching the main program. The plugin system encourages community contributions and sharing of new ideas as drop-in add-ons.

* **Frontend Plugins and Custom UI:** If a plugin has a user-interface aspect, it can include a `plugin.js` (which might define new React components or add menu items, etc.) and a `plugin.css` for styling. The AVNav web app will load these dynamically so that, for example, a new instrument widget type provided by a plugin becomes available in the UI. This mechanism is powerful – it means the UI is not limited to pre-defined widget types or pages. Ambitious developers have the freedom to create entirely new pages in the app for specialized functions (like a engine monitoring dashboard, or a weather GRIB viewer) as plugins. Moreover, even without writing a full plugin, users can **adapt the UI via configuration**: AVNav supports custom layout files (in the `layout/` directory) where one can predefine instrument panel arrangements or key mappings for hardware buttons. By editing or adding layout JSON/XML files, you can re-arrange the default UI or change its behavior in kiosk setups, etc., and these appear as selectable layout profiles in the Web UI.

* **CSS and Theming:** Basic look-and-feel (colors, font sizes) can be tweaked via CSS. The project’s documentation encourages using this to create night mode or high-contrast themes. Since the UI is web-based, knowledgeable users can even use browser developer tools to experiment with UI changes on the fly. Permanent changes can be made by adding a custom CSS file in the `user/` directory (which is served by the app), without altering core files.

* **Scripting and API Usage:** The presence of a well-defined JSON API means external programs can also interface with AVNav. For instance, a developer could write a script to fetch data from AVNav’s REST endpoints (like grab the current GPS position in JSON) and use it elsewhere. Conversely, one could send NMEA sentences into AVNav over TCP/UDP from an external source (for example, feeding online weather buoy data as fake NMEA messages). This flexibility allows AVNav to be a central hub that can mesh with other systems on the boat.

* **Integration with Signal K:** Many modern boat setups use the Signal K protocol for sharing data. AVNav’s `signalkhandler` can act as a gateway – it can forward the data it collects into a Signal K server or directly serve some data in Signal K format【19†】. In OpenPlotter installations, AVNav is pre-configured to talk to the central Signal K server out of the box. This means if you’re already running Signal K (with its own web dashboards or connected apps), AVNav can slot in without duplicating effort, or you can use AVNav as a lightweight alternative to a full Signal K if you just need basic NMEA routing and a chartplotter.

In short, AVNav is not a closed system – it’s built to be adapted. The combination of plugin hooks, config files, and open APIs gives developers a *cookbook* for tailoring the software. Whether it’s adding support for new hardware (like a different AIS decoder or a N2K gateway), creating a bespoke display page for a specific use-case, or integrating AVNav with homebrew automation scripts, it’s all achievable without forking the core project. This approach keeps AVNav flexible and community-friendly, ensuring it can evolve with the fast-changing landscape of marine tech.

---

## 🔒 Security and Deployment

**Security:** By default, AVNav does not enforce login authentication for the web interface – it’s assumed to be running on a closed network (such as your boat’s local Wi-Fi) or behind a firewall. This makes it convenient to access while on board, but it’s recommended **not to expose AVNav directly to the internet** without additional protections. If needed, one could implement basic HTTP auth or run AVNav behind a secure proxy for remote access. The software does run as a normal user (not root), and on Linux the included setup scripts configure necessary permissions (e.g., for accessing serial ports) via udev rules rather than requiring root privileges【15†】. AVNav’s file access is mostly contained to its own application and user data directories, and it doesn’t open any external ports besides those configured for its web and NMEA services. Always keep your system up to date and use network best practices (like WPA2 for the Wi-Fi hotspot, strong system passwords, etc.) when deploying AVNav on a Raspberry Pi or similar.

**Operating System Integration:** Installation packages are provided for various platforms to simplify deployment:

* On **Raspberry Pi (Raspbian)**: You can use a pre-built SD card image or install via a Debian package/script. The installation will set up AVNav as a systemd service (`avnav.service`), so it starts on boot and restarts automatically if it crashes. It also can install optional services like `avnav-iptables` (for network configuration) and `avnav-ssh` (to toggle SSH) as found in the `raspberry/` folder【15†】.
* On **Windows**: A setup program is available which installs AVNav and registers it as a Windows service (so it runs in background). There are PowerShell scripts and an NSIS installer script in the `windows/` directory handling these details【24†】. The Windows version is primarily intended for chart conversion and testing, but it can run the full server and GUI if desired.
* On **Linux (PC)** or **macOS**: You can run AVNav from source or potentially use packages (for Linux, .deb packages might be offered). On macOS, an app bundle is provided for the chart converter utility (as seen in `osx/AvChartConvert.app`), and the Python server can be run on macOS as well (though there is no launch agent by default in the repo). In all cases, Python 3 is required, and the required Python libraries (like Pillow, flask (if any), etc.) are typically installed via the installer or manually via `pip` as documented.
* On **Android**: Simply install the AVNav app from the Google Play Store (or build it from the `android/` source). The Android app includes all needed components and will on first run ask for permissions to use location, USB, etc., as appropriate.

**Updates:** Being an open-source project, AVNav updates are released periodically. Users can update by installing the new version (on RPi, that might mean running an update script or replacing the Docker image if using one; on Windows, running a new installer; on Android, via Play Store updates). The project does not currently have an over-the-air self-update mechanism within the app (unlike some systems that have a “update” button); you update by upgrading the software through the usual means. It’s wise to keep an eye on the project’s GitHub or wellenvogel.net page for release notes and update instructions.

**Reliability:** AVNav is designed to be **robust for continuous use**. It handles disconnecting/reconnecting devices gracefully (e.g., if a USB GPS is unplugged, it won’t crash; it will wait and auto-detect when a new device is plugged in). The use of a simple file-based configuration and avoidance of external database dependencies means there are few points of failure. The server can also run entirely offline (no internet needed) which is crucial offshore. All charts and data can be pre-loaded, and even the documentation is included in the `docs/` directory or accessible on the device, so one doesn’t have to rely on cloud services for any core function.

---

## 📜 Installation and Initial Setup

*Getting started with AVNav is straightforward.* For Raspberry Pi users, the easiest path is to download the ready-to-go SD card image from the official website. After writing the image to an SD card and booting the Pi, AVNav will launch automatically – the Pi will create a Wi-Fi hotspot (SSID and password as documented on the site) and you can connect with your laptop/tablet to access the interface. The default address for the web interface is usually **[http://192.168.2.1:8092](http://192.168.2.1:8092)** (or a similar IP, depending on network config). You should see the AVNav homepage with charts (initially a world map or a demo chart) and menus for the various functions.

If you prefer to install on an existing Raspberry Pi OS or a different Linux device, you can use the provided installer script or Debian package. Typically it’s as simple as:

```bash
# On Raspbian or Debian-based OS
sudo apt-get install avnav                 # (if a deb repo is provided)
# OR, if building from source:
git clone https://github.com/wellenvogel/avnav.git
cd avnav
sudo ./install.sh
```

*(Refer to the latest instructions on the official site, as package names or install methods may evolve.)*

For **Windows**, download the installer from the official site and run it – it will install AVNav and create a shortcut to start/stop the server. Once running, a tray icon might indicate the service status, and you can navigate to `http://localhost:8092` (or whichever port is configured) in your browser to use AVNav.

For **Android**, simply install the “AVNav” app from the Play Store. Upon opening, the app runs its internal server and immediately loads the AVNav interface. You can use the Android’s internal GPS, but for better accuracy or AIS, you might connect the device to external NMEA sources (Bluetooth or Wi-Fi NMEA feed; the app supports receiving NMEA over network or USB serial via an OTG cable). The Android app can also act as a server for other devices: if enabled as a “master”, it can accept Wi-Fi connections from other devices which then only need to use a browser to view the same nav data.

**Initial configuration:** In most cases, AVNav will “just work” after installation. However, a few things to check:

* **Charts:** You’ll want to load your own charts. If you have raster charts (BSB/KAP, NV Atlas, etc.), use the AVNav chart converter on a PC or directly place the chart files in the Pi’s `/home/pi/avnav/charts` directory (or use the web UI’s chart import if available). Converted chart sets (GEMF or MBTiles) should be placed in `charts/` and listed in the `avnav_server.xml` config or automatically picked up. The web UI will list available charts and allow you to select them. For oeSENC vector charts, follow the instructions on purchasing and installing the license (these go into a specific directory with a permit file).
* **GPS and Instruments:** Ensure your GPS or other sensors are connected. If using a USB GPS dongle, plugging it in should be enough – AVNav will find the serial port. You can verify data reception in the web UI (check the “GPS” status or see if your position shows up). If using a multiplexed source (like SignalK or another NMEA Wi-Fi source), configure AVNav to read from the appropriate TCP/UDP port via the config file or UI.
* **Time and System Settings:** On a Raspberry Pi, if AVNav gets a valid time from GPS, it will set the system clock (helpful if you have no internet time sync). You might still want to set your time zone on the Pi manually. Also consider enabling any security (change default passwords on Pi, etc.) as needed.
* **Mobile Use:** If you plan to use an external device primarily (say, an iPad in browser), you might create a bookmark or even an icon for the AVNav web app. The interface is designed to work as a progressive web app (PWA) so some devices may allow “Add to Home Screen” which then makes AVNav act like a full-screen app.

For more detailed installation and setup guides, refer to the official documentation, which includes step-by-step instructions and troubleshooting tips. There is also a **Quickstart** guide and an active user forum (links available on the project homepage) where you can seek help if you encounter issues.

---

## 📖 Documentation and Community

The AVNav project provides a wealth of **documentation** to help both users and developers. In the repository’s `docs/` folder you will find various manuals and notes (some in Markdown, some in PDF form). Notably, the official website hosts an extensive **User Manual** (available as HTML and PDF) that covers everything from installation to advanced usage. If you’re new to AVNav, reading the “Navigation in the Browser” introduction and the “Overview” sections on the site is highly recommended, as they give real-world context on how to use the system effectively. There are also YouTube videos (in German) and presentation slides linked on the website for visual learners.

For developers, the documentation includes a “Plugin Cookbook” and a developer guide (check the `docs` directory and the GitHub wiki if available). The **directory structure documentation** is especially useful – for example, a document enumerating the purpose of each important folder/file is provided to orient new contributors【10†】. The code itself is commented, and because much of it is Python and JavaScript, it’s relatively approachable. If you want to modify the code or add features, it’s a good idea to start by examining the relevant module (for instance, to change how an instrument is drawn, look at the corresponding React component in `viewer/components/`, or to tweak how a new NMEA sentence is handled, see `server/handler/avndecoder.py`).

The AVNav **community** is active and friendly. The project’s GitHub page is the place to report issues or submit improvements (pull requests). Sailors around the world use AVNav and discuss it on forums such as the German “Segeln-Forum” and the OpenMarine forums. These community discussions can be insightful for learning tips and tricks, or discovering community-contributed enhancements. OpenBoatProjects and other blogs have also featured AVNav, providing third-party tutorials and context.

Being open-source (under a permissive license similar to MIT), AVNav welcomes contributions. Whether it’s improving this README, adding a new translation, or coding a new feature, contributors can make a positive impact. The maintainer, Andreas “wellenvogel” Vogel, has provided clear guidance in the docs and is known to engage with users for feedback. As a developer approaching this project, you’ll find that improving AVNav not only benefits your own boating setup but also the wider community of open-source sailing tech enthusiasts.

---

## 🌐 Positioning in the Open-Source Boating Ecosystem

It’s helpful to understand where AVNav stands in relation to other marine navigation solutions:

| **Project**         | **Focus & Type**                                    | **User Interface**                                     | **Integration**                                                            |
| ------------------- | --------------------------------------------------- | ------------------------------------------------------ | -------------------------------------------------------------------------- |
| **AVNav**           | Web-based nav software (chartplotter + data server) | Browser UI (React app; touch-optimized)                | NMEA 0183 in/out; some Signal K support; plugin extensible                 |
| **OpenCPN**         | Native chartplotter application                     | Desktop UI (Qt app for Windows/Linux/Mac; Android app) | NMEA 0183 input; plugins for extras (no built-in web interface)            |
| **OpenPlotter**     | Complete Raspberry Pi OS distro for boats           | Desktop GUI + background services                      | Integrates Signal K, OpenCPN, etc., out-of-the-box; heavy but feature-rich |
| **Bareboat/BBN OS** | RPi/Linux distro with marine apps                   | Desktop GUI (touchscreen friendly)                     | Includes OpenCPN, Signal K, and other tools pre-installed                  |
| **Signal K**        | Marine data server (focused on NMEA2000)            | Web dashboards (Angular/React based, via plugins)      | Great for data logging and sharing; requires chart front-ends for plotting |

AVNav distinguishes itself by being *lightweight* and **headless**: it doesn’t require a full X11 desktop or monitor on the Raspberry Pi – everything can be done via web browser. Unlike **OpenCPN**, which is a powerful chartplotter but traditionally runs on a single device with a directly attached screen, AVNav is designed for a distributed experience (data server + multiple remote displays). And while OpenCPN relies on plugins for certain integrations, it doesn’t natively serve a web UI or act as a network hub in the same way (though there are bridging plugins).

Compared to **OpenPlotter** or **BBN OS**, which are full operating system images bundling many components (Signal K server, OpenCPN, Node-RED, etc.), AVNav provides a more focused solution. It specifically targets the navigation and instrument display needs, using modern web tech, and avoids the overhead of a full desktop environment or a slew of services you might not use. This makes AVNav an attractive option for those who want a DIY nav system but don’t need the complexity of a complete marine OS. In fact, AVNav can complement those systems: for instance, **OpenPlotter 2** includes an AVNav option, allowing users to choose AVNav’s web UI as their chartplotter while OpenPlotter handles other tasks like sensor integrations via Signal K.

In the realm of **Signal K**, AVNav can be seen as a simpler alternative if your setup is mostly NMEA 0183 and you want an all-in-one solution. Signal K (with something like the Instrument Panel or WilhelmSK apps) is excellent for N2K-heavy environments and for cloud connectivity, but it may require assembling various pieces. AVNav gives you a lot in one package – chartplotter, data server, and web dashboards – which is compelling for many users. Ultimately, the marine open-source ecosystem is rich, and AVNav fills the niche of a ready-to-run **web chartplotter** that can run on modest hardware and be used by non-technical sailors with relative ease.

---

AVNav’s continued development and usage underscore the power of open-source collaboration in niche areas like marine electronics. By sharing this project and improving resources like this README, we lower the barrier for the next sailor-developer to climb aboard and contribute. Happy sailing, and happy coding with AVNav!

