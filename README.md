# mock-usb

a minimal mock of [node-usb](https://github.com/nonolith/node-usb) for your testing needs.


## Installation

`npm install mock-usb`

## What is this?

Writing test suites for hardware can be tricky, especially when you want to run tests without any hardware attached / plugged in. It's sorta impossible to run your tests in TravisCI for example, unless you have a way of simulating the hardware.

Most of the time, it's effective enough to just mock out the USB connection itself. This package is designed to be a drop-in replacement for [@nonolith's](https://github.com/nonolith) node package [usb](https://github.com/nonolith/node-usb) for projects that depend on it for communicating with usb devices.

## It's minimal.

The following methods have been stubbed out:

### usb.setDebugLevel

Void and empty function.

### usb.findByIds

Ignores any args passed in. Returns a device object with the following signature:

```javascript
{
	interfaces: [],
	 __open: [Function],
	 open: [Function],
	 close: [Function]
}
```

### device.open

Pushes two minimal endpoint objects to the interfaces array with the following signature:

```javascript
{
  endpoints: [
   {
     direction: 'in',
     transfer:  [Function]
    },
    {
      direction: 'out',
      transfer: [Function]
    }
  ]
}
```

Input is at index 0, output is index 1.

### device.close

Resets the `interfaces` array to empty.


### device.interfaces[0].endpoints[n].transfer

**Input**

On the input endpoint, this method accepts a number length, and a callback. The callback will return with a null error, and a buffer of the requested length, filled with `0x00`.

Example:

```javascript
device.interfaces[0].endpoints[0].transfer(8, function(error, data) {
  console.log(error, data); // null, <Buffer 00, 00, 00, 00, 00, 00, 00, 00 >
});

```


**Output**

On the output endpoint, this method accepts a buffer and a callback. The callback will return with a null error.

Example:

```javascript
var buf = new Buffer([0xFF, 0x12, 0x01];

device.interfaces[0].endpoints[1].transfer(buf, function(error) {
  console.log(error); // null
});

```

## Tips for using

### 1. Use [proxyquire](https://www.npmjs.com/package/proxyquire) to replace your usb requires with this module

This is a super straightforward way of overriding any requires of the regular usb package, while leaving your module file intact. Keep pesky test logic concerns out of your main package!

Example:

```javascript
// in your test file!

var proxyquire = require('proxyquire');
var mockusb = require('mock-usb');

// require the module with a dependency on usb package with proxyquire
// specify to proxy usb with mockusb
var myModule = proxyquire
                 .noCallThru()
                 .load('myModule', { 'usb': mockusb });

```

### 2. Use [Sinon](http://sinonjs.org/) to stub out input transfer responses on a test-by-test basis.

Example:

```javascript
// in your test file!
var sinon = require('sinon');

// within one of your tests ------------------
var input = device.interfaces[0].endpoints[0];
var response = new Buffer([0xFF, 0x04]);

// replace input transfer function with new custom response
var stub = sinon.stub(input, 'transfer', function(callback) {
 return callback(null, response);
});
// -------------------------------------------
```


