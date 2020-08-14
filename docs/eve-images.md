# EVE Images

In order to launch an EVE instance, a base EVE disk image is required. Eden takes care of
retrieving the disk image and launching EVE for you. However, you may want to customize it.
For example, you may want to use a different version of the official distribution, you may want
to use a different version, you may want to use a different disk image entirely.

This is especially useful as part of the EVE development lifecycle.

Retrieving disk images almost always is performed as part of `eden setup`. The actual launching
of an image is part of `eden start`.

## Where Eden normally gets EVE

The normal EVE flow for `eden setup` is as follows.

1. If you do not have a context, get the latest tag for the image `lfedge/eve` from the docker hub, e.g. `0.0.51`, and save that tag in your context
1. Pull the image `lfedge/eve:<tag>`
1. Generate a config directory with:
   * server certificate for adam
   * `server` file pointing to the local running adam address
   * device certificate
1. Run the docker image with the config partition mounted to:
   1. Generate a disk image - raw for RPi, qcow2 for everything else - overriding the config partition with the mounted config directory
   1. Extract the disk image from the docker image, e.g. `live.qcow2`
1. Save the extracted disk image to a local cache, normally `$PWD/dist/images/eve/`

You now have a disk image, e.g. `live.qcow2` ready to run, with the appropriate embedded config partition.

## To Run a Custom EVE Image

If you want to run your own custom EVE image, you have two options: generate a docker container with your live image,
or just run your live image.

### Docker Container

The advantage of the docker container, is that it contains the utility to generate the appropriate format of
image combined with the correct config partition. You will not have to do any work to get the config partition
"just right" in your image.

To generate the docker container with your image:

1. Work in the `github.com/lf-edge/eve` directory
1. Configure your code as desired
1. Run `make eve`, optionally setting the desired hypervisor, e.g. `make eve HV=kvm` (recommended with eden)
1. Run `make eve-uefi`.

When done, you will be provided with output telling you the docker image name and tag, e.g.

```
Successfully built a46458b4ce1a
Successfully tagged lfedge/eve:0.0.0-testbranch-b6a6d6fd-kvm-amd64
Tagging lfedge/eve:0.0.0-testbranch-b6a6d6fd-kvm-amd64 as lfedge/eve:0.0.0-testbranch-b6a6d6fd-kvm
```

And for `eve-uefi`:

```
Tagging lfedge/eve-uefi:0.0.0-testbranch-b6a6d6fd-kvm-amd64 as lfedge/eve-uefi:0.0.0-testbranch-b6a6d6fd-kvm
```

You need to run both `make eve` and `make eve-uefi` because eden uses the `--eve-tag` to look for both
the eve image and the eve-uefi image.

Now, run eden setup, but tell eden to use the provided image:

```sh
eden setup --eve-tag <tag>
```

Continuing the above example:

```sh
eden setup --eve-tag 0.0.0-testbranch-b6a6d6fd-kvm
```

Or you can save it, by setting it in the file:

```console
eden config set default --key eve.tag --value 0.0.0-testbranch-b6a6d6fd-kvm
eden setup
```

eden now will use the above container image to generate and configure the live disk image.

### Live Image

To generate the live image:

1. `eden setup` as normal
1. Switch to the `github.com/lf-edge/eve` directory
1. Configure your code as desired
1. Run `make live`, optionally setting the desired hypervisor, e.g. `make live HV=kvm` (recommended with eden). When building you must include the config directory generated by `eden setup` by adding `make live CONF_DIR=<eden-conf-dir>`

When done, you have a live image file to be used, normally in `dist/<arch>/<file>`, e.g. `dist/amd64/live.qcow2`.

To launch eden using the provided image:

1. `eden start --image-file=path/to/your/live-image`