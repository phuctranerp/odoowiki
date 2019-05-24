### Initial Issue
I get the following error while making the delivery on Production instance.
```
2019-05-23 17:25:26,804 22845 INFO projectx_production openerp.modules.loading: loading 162 modules...
2019-05-23 17:25:26,900 22845 ERROR projectx_production openerp.modules.registry: Failed to load registry
Traceback (most recent call last):
  File "/opt/openerp/openerp-projectx-production/odoo/openerp/modules/registry.py", line 386, in new
    openerp.modules.load_modules(registry._db, force_demo, status, update_module)
.....

    force, status, report, loaded_modules, update_module)
  File "/opt/openerp/openerp-projectx-production/odoo/openerp/modules/loading.py", line 237, in load_marked_modules
    loaded, processed = load_module_graph(cr, graph, progressdict, report=report, skip_modules=loaded_modules, perform_checks=perform_checks)
  File "/opt/openerp/openerp-projectx-production/odoo/openerp/modules/loading.py", line 132, in load_module_graph
    models = registry.load(cr, package)
  File "/opt/openerp/openerp-projectx-production/odoo/openerp/modules/registry.py", line 169, in load
    model = cls._build_model(self, cr)
  File "/opt/openerp/openerp-projectx-production/odoo/openerp/models.py", line 591, in _build_model
    original_module = pool[name]._original_module if name in parents else cls._module
  File "/opt/openerp/openerp-projectx-production/odoo/openerp/modules/registry.py", line 84, in __getitem__
    return self.models[model_name]
KeyError: 'purchase.requisition'
```
This error happended when the upgrading of module `projectx_account_budget`, which depends on `account_budget` module, was being processed.

### Investigation
#### Step 1: Try to install the module `purchase_requisition`
It didn't work :( I think I was stupid at that time. I should regcognize that the module `projectx_account_budget` was in state of `To upgrade` which prevented the upgrading process.

#### Step 2: Cleaning the pyc files
With the recommendation of my colleague, I tried to clean all the pyc. We thought there should be some pyc which was not regenerated. However, it didn't work at all.

#### Step 3: Checking the dependency
We discovered that the module `purchase_requisition` had not been set to be a dependency of `projectx_account_budget`. So we tried adding it. It worked like a charm :)

### Root Cause
Module `projectx_account_budget` included the inheritance of model `purchase.requisition`, but there was no dependency set.

### Solution
Add module `purchase_requisition` as the dependency on `projectx_account_budget` and reupgrade it.
