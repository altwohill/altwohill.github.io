---
title: "Working with Silverstripe's javascript injector"
categories: [Web]
---

If you want to do any javascript development within the CMS you'll need to use the [Injector API](https://docs.silverstripe.org/en/4/developer_guides/customising_the_admin_interface/reactjs_redux_and_graphql/#the-injector-api).

The documentation is ok, there's a lot and it's fairly fragmented. But perhaps you're following an example, like [how to implement Version History for your own data objects](https://docs.silverstripe.org/en/4/developer_guides/model/versioning/#register-your-graphql-query-and-mutation-with-injector). 

```js
/* global window */
import Injector from 'lib/Injector';
import readOneMyVersionedObjectQuery from 'state/readOneMyVersionedObjectQuery';
import revertToMyVersionedObjectVersionMutation from 'state/revertToMyVersionedObjectVersionMutation';

window.document.addEventListener('DOMContentLoaded', () => {
  // Register GraphQL operations with Injector as transformations
  Injector.transform(
    'myversionedobject-history', (updater) => {
      updater.component(
        'HistoryViewer.Form_ItemEditForm',
        readOneMyVersionedObjectQuery, 'ElementHistoryViewer');
    }
  );

  Injector.transform(
    'myversionedobject-history-revert', (updater) => {
      updater.component(
        'HistoryViewerToolbar.VersionedAdmin.HistoryViewer.MyVersionedObject.HistoryViewerVersionDetail',
        revertToMyVersionedObjectVersionMutation,
        'MyVersionedObjectRevertMutation'
      );
    }
  );
});
```

The first line `import Injector from 'lib/Injector';` is interesting. We didn't create that lib, and in fact when I run `yarn` I get an error:

    ERROR in ./app/client/src/boot/index.js

    /home/al/history/app/client/src/boot/index.js
      2:22  error  Unable to resolve path to module 'lib/Injector'  import/no-unresolved
      2:22  error  Missing file extension for "lib/Injector"        import/extensions

    ✖ 2 problems (2 errors, 0 warnings)

No-where could I find how to get `lib/Injector`! I eventually found a comment somewhere (sorry, lost exactly where) that we needed the development version of `silverstripe/admin` installed. 

## TLDR ##

If you've already used `composer` to install, you simply need to delete the `vender/silverstripe/admin` folder and re-install preferring source.

```bash
rm -rf vendor/silverstripe/admin
composer install --prefer-source
```