# winreg2 #
`winreg2` - A Node.js module allowing access to the Windows Registry by using the built-in `REG.exe`.

`winreg2` is a fork of the original [node-winreg](https://github.com/fresc81/node-winreg), written in order to support the `async`/`await` API and ES modules but designed to be as close to the original API yet lightweight as possible.

## Installation ##
```shell
npm install winreg2
```

## Caveats ##
The original library manually used `__defineGetter__` to enforce the read-only on all properties; this is not enforced (but still recommended) in this new library in order to help with `console.log` messages.

In addition, `child_process.exec` is used to spawn the `REG.exe` processes, which may allow for remote code injection and execution (see the [Node.js docs page](https://nodejs.org/dist/latest/docs/api/child_process.html#child_processexeccommand-options-callback)), so **sanitise all names and values** to prevent such vulnerabilities.

Finally, exports have been changed. `Registry` is no longer a default export but rather a named export, while the three enum types (`Hive`, `Arch` and `RegType`) are exported as separate exports instead of being attached as static properties of `Registry`.

## Example ##
The API is mostly the same except for the `Promise`s being returned instead needing callbacks:
```js
import { Registry, Hive } from 'winreg2';
import { assert } from 'node:assert';
let regKey = new Registry({
    hive: Hive.HKCU,                                        // open registry hive HKEY_CURRENT_USER
    key:  '\\Software\\Microsoft\\Windows\\CurrentVersion\\Run' // key containing autostart programs
});
try{
    let items = await regKey.values();
    items.forEach(item=>{
        console.log(`ITEM: ${item.name}\t${item.type}\t${item.value}`);
    });
}catch(e){
    console.error('Something went wrong:'+  e);
}
```