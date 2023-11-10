# delivery-digitalsalmon-training
Training Material

## Zarf
Makefile and materials in the [zarf directory](zarf)

### Initial walkthrough
1) We will be following the `make zarf-walkthrough/v1` breadcrumbs (~5m make target)
- download zarf
- build namespace package
- build gitlab-deps v1 packages
- run zarf init
- deploy namespace package
- deploy gitlab-deps v1 packages
- cleanup

## Zarf walthrough with DUBBD and updated gitlab-dev packages
2) We will be following the `make zarf-walkthrough/v2` breadcrumbs (~30m make target)
- Kick the make target off, then talk through DUBBD and the differences in the updated gitlab-deps packages for integration with DUBBD and the uds-capability-gitlab
- download zarf
- build namespace package
- build gitlab-deps v2 packages
- run zarf init
- deploy DUBBD package
- deploy namespace package
- deploy gitlab-deps v2 packages
- deploy the uds-capability-gitlab
- cleanup
