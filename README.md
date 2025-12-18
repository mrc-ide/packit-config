# Migration steps

1. Visit https://packit-dev.dide.ic.ac.uk/ and take not of all the instances you want to migrate.
1. Check the group and permission structure of each instance. We don't migrate those so they will need to be recreated.
1. Export all the instances into separate tarballs using the following command (once per instance):
    ```
    ssh root@packit-dev.dide.ic.ac.uk tar -C /var/lib/outpack/reside-dev -Jc .outpack > reside-dev.tar.xz
    ```

1. Destroy and re-create the VM
    1. Remote desktop onto the host machine (`wpia-reside1`)
    1. Open command prompt
    1. Change directory to `D:\reside-hyperv-scripts\packit-dev`
    1. Run `vagrant destroy`
    1. Run `vagrant up`
    1. Leave the Windows host


1. Upgrade Ubuntu from 22.04 to 24.04
    1. SSH into the new VM `ssh vagrant@wpia-packit-dev.dide.ic.ac.uk`
    1. Run `sudo apt-get dist-upgrade`
    1. Reboot the VM
    1. Edit `/etc/update-manager/release-upgrades` and set `Prompt=lts`
    1. Run `sudo do-release-upgrade`, say yes to everything except about overwriting the sudoers file
    1. Let the upgrader restart when it is done

1. Run the initial provisioning on the machine
    ```
    uv run pyinfra src/infra/inventory.py infra.run --limit wpia-packit-dev.dide.ic.ac.uk
    ```

1. Restore your instance data:
    ```
    cat reside-dev.tar.xz | ssh vagrant@packit-dev.dide.ic.ac.uk docker run --rm -i -v packit-outpack-reside-dev:/outpack alpine tar -C /outpack -Jx
    ```

1. Configure and start packit

    ```
    ssh vagrant@wpia-packit-dev.dide.ic.ac.uk
    cd packit-config
    packit configure packit-dev
    packit start
    ```

    First deployement may take a while to get certificates from Let's Encrypt. You
    can monitor progress with `docker logs -f packit-acme-buddy`.

1. Resync packets using the Packit admin UI
