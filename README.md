## Introduction

MockableSystemJS allows [SystemJS](https://github.com/systemjs/systemjs) users to replace modules for the duration of a test. Assuming you've used [jspm](http://jspm.io/) to bundle your code, you will have an `index.html` file that looks something like this:

```html
<script src="jspm_packages/system.js"></script>
<script src="config.js"></script>
<script>
	System.import('lib/main').then(function(m) {
		// start app...
	});
</script>
```

Although it doesn't usually make sense to mock classes within your real app, by adding this line of code:

```html
<script src="node_modules/MockableSystemJS/dist/index.js"></script>
```

between the `system.js` and `config.js` scripts above, you would be able to start mocking using these additional methods on `System`:

  * `System.reSet()`
  * `System.reRegister()`
  * `System.reRegisterDynamic()`
  * `System.restore()`

## Mocking Within Tests

Provided you've ensured that MockableSystemJS is loaded immediately after `system.js` in your test environment, you could then write a test like this:

```js
'use strict';

import chai from 'chai';
import sinon from 'sinon';

const expect = chai.expect;
const AssortedVendableUrl = System.normalizeSync('AssortedVendable');
const VendingMachineUrl = System.normalizeSync('VendingMachine');

define('Vending Machine', function() {
    afterEach(function() {
        System.restore();
    });

    it("always vends chocolate if that's all there is", function() {
        const chocStub = sinon.stub().returns('choc');
        System.reSet(AssortedVendableUrl, System.newModule({default: chocStub}));
        const VendingMachine = System.get(VendingMachineUrl).default;

        expect((new VendingMachine()).vend()).to.equal('choc');
    });
});
```

## How It Works

By ensuring `MockableSystemJS` is loaded immediately after `SystemJS`, it is able to replace the `System.register()` and `System.config()` methods with delegate methods that store any config and modules for future use.

Later, when the `reSet()`, `reRegister()` and `reRegisterDynamic()` methods are invoked, the new module is loaded with the corresponding original method (i.e. `set()`, `register()` and `registerDynamic()`), and any modules dependent on the new module are re-loaded so that they depend on the updated module. Here, the [sync-import](http://github.com/dchambers/sync-import) library is used to allow modules to be synchronously re-imported so that the need for complex asynchronous tests can be avoided completely.

Finally, when `System.restore()` is invoked, everything is restored back to how it originally was.
