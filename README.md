# Small-footprint CF
This project provides manifests and tooling for create CF deployments of various sizes.
The smallest deployment is co-located entirely on a single VM.
The medium deployment scales up to 3 VMs (a database, a brain, and a cell), and the large deployment is 17 VMs with each component having it's own VM.

# Deploying
In order to deploy small-footprint CF, you need a bosh director and some setup in AWS (specifically, a subnet, a security group, an elastic IP, and optionally a domain name. If you don't have a domain name, you can use `<elastic-ip>.xip.io`).
The `bin/deploy` script will upload compiled releases from S3 to your bosh director.
The S3 URLs to the releases are specified in `versions.json`. `bin/deploy` takes a single argument: either `small`, `medium`, or `large`.
The bosh stemcell being used is specifed in the `bin/deploy` script.

## Replacing placeholder values
There are placeholder values in each manifest and the cloud-config that begin with `REPLACE-WITH-`.
These must be filled in with values from your bosh director/IAAS before deploying.
You will need to generate an SSL certificate for HAproxy as well as an RSA keypair for ssh proxy.

## Scaling between sizes
It is possible to scale between any of the sizes, but data loss or downtime may occur.

* Scaling to or from a small deployment will result in both downtime and data loss
* Scaling between medium and large deployments will result in downtime but **not** data loss

Downtime can reduced or potentially avoided with modifications to the medium/large manifests.
For example, when scaling from a medium to large deployment, you can scale the number of cells to 2 before doing the large delpoyment.
You could also co-locate the consul_server on the brain to avoid restarting the cells.

# Releases
The releases being deployed are compiled versions of the open-source releases with additional patches.
These patches can be found in the `patches` directory.
They mostly bump timeouts and avoid port conflicts.
In order to use your own releases:

1. Clone the open-source repo
1. Add/update patches as desired
1. Use [knit](https://github.com/pivotal-cf-experimental/knit) to apply patches to the repo
1. Use bosh to [compile the release](https://bosh.io/docs/compiled-releases.html)
1. Upload the release to S3 (or somewhere else publicly accessible)
1. Update the information for the release in `versions.json` (only the `compiled_release_url` is used by the deploy script, other fields are for information purposes only)

# Known issues
## Monit 'Execution Failed'
The version of monit used in the bosh stemcell has a known issue where a service that is running may be reported as 'Execution Failed'.
This results in the bosh VM failing.
A fix can be found in [this fork](https://github.com/pivotal-cf/pcfdev-monit) of monit but would require using a custom stemcell with the patched monit.
