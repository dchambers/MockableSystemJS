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

Although it doesn't usually make sense to mock classes within your real app, by adding this line of code between the `system.js` and `config.js` scripts above, you would be able to start mocking:

```html
<script src="node_modules/MockableSystemJS/dist/index.js"></script>
```

## Mocking Within Tests

Provided you've ensured that MockableSystemJS is loaded immediately after `system.js` in your test environment, you could then write a test like this:

```js
'use strict';

import MockableSystem from 'MockableSystemJS';

define('Vending Machine', function() {
    beforeEach(function() {
        MockableSystem.install();
    });

    afterEach(function() {
        MockableSystem.uninstall();
    });

    it('always vends chocolate if that is all there is', function() {
        System.remap('AssortedVendable').to('ChocVendable');
        const VendingMachine = System.get(System.normalizeSync('VendingMachine')).default;

        expect((new VendingMachine()).vend()).to.equal('choc');
    });
});
```

where `ChocVendable` is some alternate implementation of a module that you would otherwise depend on, and where `VendingMachine` is the unit under test.

## How It Works

By ensuring `MockableSystemJS` is loaded immediately after `SystemJS`, it is able to replace the `System.register()` and `System.config()` methods with delegate methods that store any config and modules for future use.

Later, when `MockableSystem.install()` is invoked, it causes the `System` object to be replaced with a delegate object that adds a `remap()` method, and which causes `get()` to synchronously re-load any modules that have been re-mapped, or which depend, or transitively depend, on a re-mapped module.

The [sync-import](http://github.com/dchambers/sync-import) library is used to allow modules to be synchronously re-imported so that the need for complex asynchronous tests can be avoided completely.
