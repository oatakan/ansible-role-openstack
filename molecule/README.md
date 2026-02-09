# Molecule

This role can be tested against a *real OpenStack* by using Molecule's delegated driver.

Key goals:

- No local OpenStack credentials/config are checked into git.
- Tests run on `localhost` and use your existing OpenStack auth via environment variables.

## Prereqs

- `molecule` installed in your environment
- `openstack.cloud` collection available (already required by the role)
- Your OpenStack auth configured locally (typically via env vars / OpenRC)

## Run

Example (adapt values to your cloud):

```bash
# Optional: create a local .env (ignored by git)
# echo 'OS_ENV_FILE=~/.openstack/openrc.sh' > .env

export OS_STACK_IMAGE="<image name>"
export OS_STACK_FLAVOR="<flavor name>"
export OS_STACK_KEYPAIR="<keypair name>"
export OS_STACK_PRIVATE_NETWORK="private_network"

# Optional: create multiple baseline nodes
# export OS_STACK_BASE_COUNT=2

./scripts/molecule.sh test -s default
```

Notes:

- `converge` provisions instances directly using `openstack.cloud.server`.
- `verify` asserts instances exist and are ACTIVE.
- `destroy` deletes the instances.
