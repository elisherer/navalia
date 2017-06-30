# Navalia

`npm install --save navalia`

Automate and scale browser workflows.

```javascript
const Navalia = require('Navalia');
const navalia = new Navalia({ numInstances: 1 });

const snapshotFavPages = async() => {
  await navalia.startup();

  navalia.register(async(chrome) => {
    await chrome.navigate('http://www.google.com/');
    await chrome.pdf('/Users/jgriffith/Downloads/google.pdf');
  });

  navalia.register(async(chrome) => {
    await chrome.navigate('http://www.facebook.com/');
    await chrome.pdf('/Users/jgriffith/Downloads/facebook.pdf');
  });
};

snapshotFavPages();
```

![Navalia Demo](/assets/demo.gif?raw=true "Navalia Demo")

## Features

- Runs and controls multiple instances of particular browser (currently Chrome).
- Capture screenshots, pdfs, execute javascript, and monitor network requests.
- Queue work automatically when all instances are busy.
- Uses a simple, easy to understand API.
- Chain operations, including multi-page workflows.
- Set timers and max-jobs on browsers, forcing a reboot so they stay small memory-wise.

## API

#### new Navalia(options: NavaliaOptions)

Sets up a new `Navalia` instance. This instance manages all the jobs, and queues those when resources aren't available. The following options are below:

`numInstances?: number`
The number of running browser applications. Defaults to the number of CPU's available on the machine

`maxJobs?: number`
The number of "jobs" an instance should can run before it reboots. Defaults to `-1`, which means that the instance will never reboot based on the number of jobs it runs

`workerTTL?: number`
The total-time-of-life an instance should run before it's rebooted. Defaults to `-1`, which means that the instance will never reboot based on a time expiration.

`chromeOptions?: object`
An object of options to pass into the startup of Chrome. These are all the hashes one would use on the CLI, but are instead snakeCased. For instances `--headless` becomes `headless: true`, and `--hide-scrollbars` becomes `noScrollbars: true`. Defaults to `{ headless: true, disableGpu: true, hideScrollbars: true }`

#### register: Function((chrome: chromeInstance): Promise<any>): void

Registers a function, to be called with an instance of chrome (or whatever browser in the future). This function will execute immediately if there's an available instance. If not, the function is queued and will be handled by the next available browser. See the below docs on the browser API.

The registered function _must_ either return a `Promise`, or a value when using `await`. Internally, Navalia waits for this function to resolve so it can begin other work.

#### startup: Function(): Promise<void>

Starts the browser(s) and sets them up for work. Since this action is asyncrounous, it's recommended that you delay queueing jobs until this method returns.

#### launchInstance: Function(options:object): Promise<instance>

Launches a new instance, which can begin to immediately work if there's a queue already formed. If not, this instance will be kept around for futher jobs.

## Browser API

#### navigate: Function(url: string, opts: object): Promise<void>

Navigates the browser to a particular URL. Can supply a second argument, which is a hash that currently supports only `onload`, fired when the browsers `window.onload` event takes place.

#### evaluate: Function(expression: string): Promise<any>

Evaluates the script and returns the output of the last statement. The result is a meta-object containing the output plus some meta-data about the result. Currently the expression _must_ be a string, but we're investigating ways to make it a bare function to execute.

#### screenShot: Function(filePath: string): Promise<any>

Saves a PNG of the current page's viewport. file-path should be the absolute-path where this file will will be saved.

#### pdf: Function(filePath: string): Promise<any>

Saves a PDF of the current page. file-path should be the absolute-path where this file will will be saved.

#### setWindowSize: Function(width: number, height: number): Promise<any>

Sets the window size of the browser by width and height.

## Debugging

Novalia prints nearly every interaction by using the module `debug`. To get debug information, launch your script with:

```bash
# Log all messages. Omit `navalia` or `chrome` to filter out messages
$ DEBUG=navalia,chrome node index.js
```

## Roadmap

In no particular order, this is the vision of navalia going forward:

- [ ] Expanded browser API (pdf rendering, network watching, more).
- [ ] Bring more vendors onto the framework.
- [ ] Better typings around externals with no @type support.
- [ ] Parameterization on killing long-running jobs.
- [ ] Unit testing all features.
- [ ] Integration testing with the various vendors so our API's don't break when theirs do.
- [ ] Travis, coveralls, greenkeeper, and other handy-dandy tools to automate chore tasks.
