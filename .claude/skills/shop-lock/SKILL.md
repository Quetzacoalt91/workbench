---
name: shop-lock
description: Claim and release a PrestaShop instance lock in this workbench before using it (starting it, running tests, changing data), so two conversations don't collide on the same shop. Use to get the ownership on a PrestaShop-N instance, and when finishing work on one.
---

# Locking a shop before use

Multiple conversations may work in this workbench at once. A `<name>.lock` file in the workbench root (next to `CLAUDE.md`, not inside the instance folder) marks an instance as in use.

## Claim one

1. List instances and existing locks:
   ```
   ls -d PrestaShop-*/ 2>/dev/null
   ls *.lock 2>/dev/null
   ```
2. Pick an instance whose `<name>.lock` does **not** exist.
3. Create the lock before doing anything else with it:
   ```
   echo "in use: <short description> — $(date -Iseconds)" > <name>.lock
   ```
4. If every instance already has a lock file, stop and tell the user — don't pick one anyway.
5. If you created a lock file, the shop is now yours.

## Release it

When done with the shop (including before ending the conversation), remove its lock:
```
rm <name>.lock
```

Only remove a lock you created.
