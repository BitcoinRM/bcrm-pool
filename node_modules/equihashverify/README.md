# Equihash - Bitcoin Gold Implementation
nodejs native binding to check for valid Equihash-BTG (144, 5) solutions

# Dependencies
````
sudo apt-get install build-essential libsodium-dev libboost-system-dev
````

# Usage:
````javascript
var ev = require('bindings')('equihashverify.node');

var header = new Buffer(..., 'hex');
var solution = new Buffer(..., 'hex'); //do not include byte size preamble

ev.verify(header, solution);
//returns boolean
````
# Test Suite:
````
sudo npm install -g mocha
npm install
mocha
````
