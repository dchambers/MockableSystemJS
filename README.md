MockableSystemJS allows [SystemJS](https://github.com/systemjs/systemjs) users to replace modules for the duration of a test. Assuming you've used [jspm](http://jspm.io/) to bundle your code, you will have an `index.html` file that looks something like this:

```html
<script src="jspm_packages/system.js"></script>
<script src="config.js"></script>
<script>
	System.import('lib/main');
</script>
```

Although it wouldn't make sense within your app, you would effectively need to add this line of code between the `system.js` and `config.js` scripts to be able to use it:

```html
<script src="node_modules/MockableSystemJS/index.js"></script>
```

## Mocking Within Tests

Provided you've ensured that `mockable-systemjs` is loaded immediately after `system.js` in your test environment, you could then write a test like this:

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
        MockableSystem.redefine('vendable', 'ChocVendable');
        let vendingMachine = System.get('VendingMachine').create();

        expect(vendingMachine.vend()).toBe('choc');
    });

    it('always vends chocolate if that is all there is (variant)', function() {
        MockableSystem.redefine('vendable', {default:function() {
            return 'choc';
        }});
        let vendingMachine = System.get('VendingMachine').create();

        expect(vendingMachine.vend()).toBe('choc');
    });
});
```

where `ChocVendable` is a module with the following contents:

```js
'use strict';

export default function() {
    return 'choc';
}
```

## How It Works

By ensuring `MockableSystemJS` is loaded immediately after `SystemJS`, it is able to replace `System.register()` and `System.config()` with delegate methods that also store any config and modules for future use. Later, when `MockableSystem.install()` is invoked, it ensures that all of the bundles defined via `System.config()` are synchronously loaded, if they haven't already been.

Subsequently, each invocation of `MockableSystem.redefine()` causes that module to replaced, but it also causes any dependent modules (including transitive dependencies) to be re-registered too. Finally, `MockableSystem.uninstall()` causes all modules to be re-registered using the original module definitions.

Because all of the modules are available within the cache once `MockableSystem.install()` has been invoked, all operations can therefore be performed sychronously, making testing easier, and allowing `System.get()` to be used in preference to `System.import()`.
