### Initial Issue
After the deployment (failed to install some modules, but it was ok after some fixes), all new menus were not shown on the UI. One of those menu (`Connector`) was shown without the action (Nothing happened after clicking on it).

### Investigation
#### Step 1:
I tried to check if the menus had been created (`ir.ui.menu`). They were all created and I didn't see anything wrong with it.

#### Step 2:
I try to debug the in debug assets mode to see the reaction of the ui on clicking on `Connector` (the only new menu shown with no action).
On clicking the button, it triggers the following function
```
    menu_click: function(id, needaction) {
        if (!id) { return; }

        // find back the menuitem in dom to get the action
        var $item = this.$el.find('a[data-menu=' + id + ']');
        if (!$item.length) {
            $item = this.$secondary_menus.find('a[data-menu=' + id + ']');
        }
        var action_id = $item.data('action-id');
        // If first level menu doesnt have action trigger first leaf
        if (!action_id) {
            if(this.$el.has($item).length) {
                var $sub_menu = this.$secondary_menus.find('.oe_secondary_menu[data-menu-parent=' + id + ']');
                var $items = $sub_menu.find('a[data-action-id]').filter('[data-action-id!=""]');
                if($items.length) {
                    action_id = $items.data('action-id');
                    id = $items.data('menu');
                }
            }
        }
```

Seems that the submenu cannot be located (`var $items = $sub_menu.find('a[data-action-id]').filter('[data-action-id!=""]');` returned nothing).

So we guessed that the `Parent Left` and `Parent Right` had not been well set for these menu. Checking on step 3

#### Step 3:
Checking ir.iu.menu on database

`SELECT * FROM ir_ui_menu WHERE parent_left IS NULL;` returned 59 records :octopus: 

OK, so we know the problem.

### Root Cause:
- On installation, some issues happened which leaded to the fail computation of menu parent left / right.

### Solution:
What we need to do now is to regenerate the missing parent left / right. But there will be a lot of way to do it

#### Using Odoo Shell
I tried to follow the Odoo Shell on my local to see if it works by following this [LINK](https://gist.github.com/paulius-sladkevicius/60a84c97e8446f5e152415e874923d66)
However, I got an error :(
```
odoo-bin: error: unrecognized parameters: 'shell'
```

#### Setting _parent_store = False
SR faced the same issue (but in a different object) and this is how it had been fixed
```
from openerp.osv import osv
import logging

_logger = logging.getLogger('account')


class account_account(osv.osv):
    _inherit = 'account.account'
    _parent_order = "id"
    _parent_store = False
```

This fix requires to edit the source code and restart odoo service, so I didn't fix in this way.

#### Scheduler
Actually what we need to do is to run the method `_parent_store_compute` of object `ir.ui.menu`. So scheduler can help us in this case.
So I created a scheduler manually as following
```
Name: Recompute Parent Left _ Right Menu
Active: False
Object: ir.ui.menu
Method: _parent_store_compute 
```

It worked like a charm :)
