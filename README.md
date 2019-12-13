# Summary
An ExtendScript compatible log constructor object with shims for basic Console calls,
log file, and custom "ExtendScript_Log" events sent to CEP panels.

Trying to split the difference between too-basic and OMG-features-and-dependencies.

# Features
- Works with node.js require, AMD(probably), and vanilla ExtendScript.
- Adds 'log' and 'console' aliases to optional parameter (like `$.global`)
- Tries to JSON.stringify your message for you
- Optional [ExtendScript_LogFile](https://github.com/MaxJohnson/extendscript_logfile) support
- Event delivery to CEP (panels) (with secret support for passing data packets)
- Make multiple logs with different names and levels and files
- Graceful fallbacks or omissions if optional dependencies missing

## Non-Blocking Dependencies
- `JSON.stringify()` via whatever library you care to include
- Event to CEP panels with `new ExternalObject("lib:\PlugPlugExternalObject")`
- [ExtendScript_LogFile](https://github.com/MaxJohnson/extendscript_logfile)

# Import
## NPM
If running Node NPM, you can `npm install ExtendScript_Log` to add to your node_modules folder
## github
Clone or download the repo and copy the ExtendScript_Log.jsxinc to your project

# Include

## NPM
`var extlog = require("ExtendScript_Log");`

## AMD
I don't know but it's probably not difficult? Firmly in the untested-but-should-work category

## ExtendScript
### Eval into environment
`$.evalFile("<path>/extendscript_log.jsxinc")`

### Include in scripts
`//@include "<path>/extendscript_log.jsxinc"`

### concatinate or copy-paste directly
Add to a build script or, I dunno, just copy-pasta it in there?

# Use:
Default log levels are:
* trace:0
* debug:1
* info:2 (also default if not specified)
* warn:3
* error:4
* critical:5

## Make new log object
make a new log and you get a separate instance
```
var myLog = new ExtendScript_Log();
myLog.log('Hey there.');
var specialLog = new ExtendScript_Log(null,"special");
specialLog.log('Salutations.');
myLog.warn('Special log thinks they're all that...');
specialLog.info('Default log is jealous cause I have a label.');

// prints:
// Hey there.
// SPECIAL:Salutations.
// [WARN] Special log thinks they're all that...
// SPECIAL:[INFO] Default log is jealous of my label.
```
### Constructor options
"new" constructor takes 4 optional arguments.
First arg is an alternate root object to tack on a 'log' and 'console' alias
By passing $.global as first arg, we get global log and console objects!

```
root = $.global;// root to add convenience alisases to
logName = "specialLog";// name other than "defualt"
logLevel = 2;// log level filter
useLogFile = true;// make a log file? Deafults to false
keepOldLogs = false;// keep or delete all but latest log file?

myLog = new ExtendScript_Log(root, logName, logLevel, useLogFile, keepOldLogs);
```

## Use the log
```
myLog = new ExtendScript_Log($.global);
console.log('Messages are good.');
console.info('So informative...');
log.warn('Duck!');
log.error('Not a good thing');

mySecretLog = new ExtendScript_Log(null, "secret");
mySecretLog.warn('Tell no one...');

// prints:
// Messages are good.
// [INFO] So informative...
// [WARN] Duck!
// [ERROR] Not a good thing
// SECRET:[WARN] Tell no one...
```

Second argument sends up an blocking alert dialog in app if true
```
log.error('Not a good thing', true);
```

# Bonus Features
## Log file:
Tries to make a log file in ./logs or to ~/Desktop/ExtendScript_Log_UnsavedScripts/
Needs [ExtendScript_LogFile](https://github.com/MaxJohnson/extendscript_logfile)

*Note: Differnt logs get different log files, even if they all print to the same console.*

## CEP event:
Tries to send type, level, label, and message as a packet in a custom event "ExtendScript_Log"
No JSON support needed in the script to send, but you have to un-strigify on receipt:
data string looks like: `{type: "default", level:2, label:"info", message:"Important things!"}`

*Note: the "clear()" function sends a packet with "clear" label and log level 99.*

# CEP Integration
## Catch log events in CEP panel
This example assumes there is already an ExtendScript_Log sending logs from a script. The code below would be in the main CEP panel javascript.
```
    var csInterface = new CSInterface();

    // Hook up internal logging messages from extendscript scripts that support it
    csInterface.addEventListener( 'ExtendScript_Log', handleExtendscriptLog);
    function handleExtendscriptLog(evt) {
        var label = evt.data.label;
        var logger = ( !label || label == 'undefined')? 'log':label;
        var data = (typeof evt.data == 'string')? JSON.parse(cleanJSONString(evt.data)):evt.data;
        console[logger]('[ES]', data.message);
    }
```
Any logs coming from ExtendScripts would then print to the debug console of the chrome(ium) window you are testing in with warnings and errors displaying as such in the console...
```
//--- ExtendScript ---//
myLog = new ExtendScript_Log($.global);
console.log('Messages are good.');
log.warn('Duck!');
log.error('Not a good thing');
```
