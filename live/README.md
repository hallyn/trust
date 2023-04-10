# Building live cd

 * cd live
 * build the rootfs
 
    ```
    stacker build --layer-type=squashfs
    ```

   * Note: I  seem to be hitting a bug with this, where it fails to build the second time.

 * Build a signed manifest pointing at your rfs

    ```
    ./build-livecd-rfs
    ```
    or if you're doing things more custom,
    ```
    trust keyset add mostest
    trust project add mostest livecd
    ./build-livecd-rfs --project=mostest:livecd \
          --layer oci:oci:livecd-rootfs-squashfs
    ````
    The results will be a complete zot layout under ./zot-cache.

 * Build the boot media

    ```
    mosb --debug mkboot --cdrom snakeoil:default docker://localhost:59111/machine/livecd:1.0.0 livecd.iso
    ```
    (Note in this case we provide docker:// to disambiguate from oci images)
    (Note - add --boot-from-remote argument to not copy the manifest and
     layers to the iso, and tell the livecd to boot from a remote repo)

 * boot the usb media. 
 
   ```
   machine init livecd << EOF
    name: livecd
    type: kvm
    ephemeral: false
    description: A fresh VM booting trust LiveCD in SecureBoot mode with TPM
    config:
      name: trust
      uefi: true
      uefi-vars: /home/serge/src/project-machine/trust/live/ovmf-vars.fd
      cdrom: /home/serge/src/project-machine/trust/live/livecd.iso
      boot: cdrom
      tpm: true
      gui: true
      serial: true
      tpm-version: 2.0
      secure-boot: true
      disks:
          - file: /home/serge/src/project-machine/trust/live/livecd.qcow2
            type: ssd
            size: 20G
   EOF
   machine start livecd
   machine gui livecd
    ```
