Pingapp Scripts
===============


This repository contains scripts to manage setup/deployments of the pingapp.
Here are some useful commands:

```
# remove all resource with an specific label
oc delete all -l app=some-value

# there is but some configmaps must be deleted separately
oc delete cm -l app=some-value
```
