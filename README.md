# twentyc.perms
granular permissions utility framework

# Purpose
Utility functions to permission out namespaces at various access levels. 

# Quickstart

    from twentyc.perms.core import PermissionSet
    from twentyc.perms.const import *

    pset = PermissionSet(
      {
        "a" : PERM_READ,
        "a.b" : PERM_RW,
        "a.b.c" : PERM_DENY,
        "b" : PERM_READ,
        "b.*.a" : PERM_RW
      }
    )

    pset.check("a", PERM_READ) # True
    pset.check("a.b", PERM_READ) # True
    pset.check("a.b", PERM_WRITE) # True
    pset.check("a.b.c", PERM_READ) # False
    pset.check("a.b.c", PERM_WRITE) # False
    pset.check("a.x", PERM_READ) # True
    pset.check("b.a.a", PERM_RW) # True
    pset.check("b.b.a", PERM_RW) # True
    pset.check("b.b.b", PERM_RW) # False

# Setting and Deleting

    pset["a.b"] = const.PERM_READ
    
    del pset["a.b"]
    
    pset.update(
      {
        "a.b" : const.PERM_READ
      }
    )

    assert "a.b" in pset

# Applying to data

You can apply the permissions stored in the permission set to any data dict and data that the permission set does not have READ access to will be removed.

twentyc.perms was created out of a need to apply granular permissions on potentially large dict objects and perform well. 

    # init
    pset = core.PermissionSet(
      {
        "a" : const.PERM_READ,
        "a.b.c" : const.PERM_RW,
        "a.b.e" : const.PERM_DENY,
        "a.b.*.d" : const.PERM_DENY,
        "f.g" : const.PERM_READ
      }
    )
    
    # original data
    data = {
      "a" : {
        "b" : {
          "c" : {
            "A" : True
          },
          "d" : {
            "A" : True
          },
          "e" : {
            "A" : False
          }
        }
      },
      "f": {
        "a" : False,
        "g" : True
      }
    }

    # expected data after permissions are appied
    expected = {
      "a" : {
        "b" : {
          "c" : {
            "A" : True
          },
          "d" : {
            "A" : True
          }
        }
      },
      "f": {
        "g" : True
      }
    }

    rv = pset.apply(data)
    assert rv == expected
