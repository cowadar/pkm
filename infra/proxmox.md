# Proxmox Cheat-Sheet

Proxmox is een krachtige open-source serveroplossing die verschillende producten omvat.

- Proxmox VE is een platform voor enterprise virtualisatie dat KVM en Linux Containers (LXC) integreert.
- Proxmox Mail Gateway is een e-mailbeveiligingsoplossing die je mailserver beschermt tegen alle e-maildreigingen.
- Proxmox Backup Server is een backupoplossing voor het efficiënt back-uppen en herstellen van VM’s, containers en fysieke hosts.

Je kunt Proxmox software downloaden van hun [website](https://pve.proxmox.com).

![logo](https://upload.wikimedia.org/wikipedia/commons/f/f6/Proxmox-VE-4-4-screenshot-startpage.png)

## VM Beheer

### Basic

```shell
# Lijst van vm's
qm list

# Maak of restore een vm.
qm create <vmid>

# start een VM
qm start <vmid>

# vm opschorten.
qm suspend <vmid>

# vm uitschakelen
qm shutdown <vmid>

# vm herstarten
qm reboot <vmid>

# vm resetten
qm reset <vmid>

# vm stoppen
qm stop <vmid>

# Vernietig de VM en alle gebruikte/eigen volumes.
# Verwijdert alle vm-specifieke machtigingen en firewallregels
qm destroy <vmid>

# Ga naar de Qemu Monitor interface.
qm monitor <vmid>

# Haal de configuratie van de vm op met zowel de huidige als de in behandeling zijnde waarden.
qm pending <vmid>

# Stuur key event naar vm.
qm sendkey <vmid> <key> [OPTIONS]

# Laat de command line zien die wordt gebruikt om de VM te starten (debug info).
qm showcmd <vmid> [OPTIONS]

# Ontgrendel de vm.
qm unlock <vmid>

# Clone een VM
qm clone <vmid> <newid>

# Migreer een VM
qm migrate <vmid> <target-node>

# Toon VM status
qm status <vmid>

# Resources voor een vm opschonen
qm cleanup <vmid> <clean-shutdown> <guest-requested>

# Maak een template
qm template <vmid> [OPTIONS]

# Opties voor virtuele machines instellen (synchrone API)
qm set <vmid> [OPTIONS]
```

### Cloudinit

```shell
# Get automatically generated cloudinit config.
qm cloudinit dump <vmid> <type>

# Get the cloudinit configuration with both current and pending values.
qm cloudinit pending <vmid>

# Regenerate and change cloudinit config drive.
qm cloudinit update <vmid>
```

### Disk

```shell
# Importeer een externe schijfkopie als een ongebruikte schijf in een VM.
# Het afbeeldingsformaat moet worden ondersteund door qemu-img(1).
qm disk import <vmid> <source> <storage>

# Verplaats het volume naar een andere opslag of naar een andere VM.
qm disk move <vmid> <disk> [<storage>] [OPTIONS]

# Scan alle opslag opnieuw en update schijfgroottes en ongebruikte schijfkopieën.
qm disk rescan [OPTIONS]

# Vergroot de volume.
qm disk resize <vmid> <disk> <size> [OPTIONS]

# Unlink/verwijder disk images.
qm disk unlink <vmid> --idlist <string> [OPTIONS]

# rescan volumes
qm rescan
```

## Web GUI

```shell
# Herstart web GUI
service pveproxy restart
```

## Resize Disk

### Vergroot de schijfgrootte

Vergroot de schijfgrootte in de GUI of met de volgend command:

```shell
qm resize 100 virtio0 +5G
```

### Verklein Schijfgrootte

> Voordat u de schijfgrootte in Proxmox verkleint, moet u een back-up maken!

1. Converteer qcow2 naar raw

  ```shell
  qemu-img convert vm-100.qcow2 vm-100.raw
  ```

2. Verklein de disk

```shell
qemu-img resize -f raw vm-100.raw 10G
```

3. Converteer terug naar qcow2

```shell
qemu-img convert -p -O qcow2 vm-100.raw vm-100.qcow2
```

## Disk Management

### Ubuntu VM Disk vergroten in Proxmox

Als je in Proxmox een disk vergroot, zal deze door de OS nog niet volledig gebruikt kunnen worden.
Daarom moeten we, na de vergroting in proxmox, deze alsnog met [GParted](../tools/gparted.md) en via de CLI resizen zodat de volledige grootte herkend kan worden door het OS.

#### 1. Virtuele machine uitzetten

Schakel de virtuele machine eerst helemaal uit.

#### 2. Harde schijf resizen

Nu kan je de harde schijf vergroten.

`Proxmox` > `Hardware Settings`

#### 3. GParted downloaden en opstarten

1. Download laatste Gparted ISO van de [GParted download website](https://gparted.org/download.php)
2. Importeer de GParted ISO file in Proxmox
3. Start de VM en laat deze starten vanaf de ISO. (Duw snel op ++esc++)

#### 4. Partitie resizen met GParted

1. Zoek de correcte partitie
2. Resize de partitie (++right-button++ -> resize)
3. Duw opt vingske 🙂

#### 5. CLI resizing

Check de huidige disksize met commando:

```bash
df -h
```

Laat Ubuntu de volledige disk gebruiken:

```bash
sudo /sbin/lvresize -l +100%FREE /dev/mapper/ubuntu--vg-ubuntu--lv
```

Gebruik resize2fs op deze disk

```bash
sudo /sbin/resize2fs /dev/mapper/ubuntu--vg-ubuntu--lv
```

Hierna kan je nogmaals de disksize bekijken.

```bash
df -h
```

Als er geen fouten opgetreden zijn, heb je nu net de harde schijf vergroot!


### Snapshot

```shell
# List all snapshots.
qm listsnapshot <vmid>

# Snapshot a VM
qm snapshot <vmid> <snapname>

# Delete a snapshot.
qm delsnapshot <vmid> <snapname>

# Rollback a snapshot
qm rollback <vmid> <snapname>

# Open a terminal using a serial device
# (The VM need to have a serial device configured, for example serial0: socket)
qm terminal <vmid> [OPTIONS]

# Proxy VM VNC traffic to stdin/stdout
qm vncproxy <vmid>
```

### Misc

```shell
# Execute Qemu Guest Agent commands.
qm guest cmd <vmid> <command>

# Executes the given command via the guest agent
qm guest exec <vmid> [<extra-args>] [OPTIONS]

# Gets the status of the given pid started by the guest-agent
qm guest exec-status <vmid> <pid>

# Sets the password for the given user to the given password
qm guest passwd <vmid> <username> [OPTIONS]
```

### PV, VG, LV Management

```shell
# Create a PV
pvcreate <disk-device-name>

# Remove a PV
pvremove <disk-device-name>

# List all PVs
pvs

# Create a VG
vgcreate <vg-name> <disk-device-name>

# Remove a VG
vgremove <vg-name>

# List all VGs
vgs

# Create a LV
lvcreate -L <lv-size> -n <lv-name> <vg-name>

# Remove a LV
lvremove <vg-name>/<lv-name>

# List all LVs
lvs
```

### Storage Management

```shell
# Create a new storage.
pvesm add <type> <storage> [OPTIONS]

# Allocate disk images.
pvesm alloc <storage> <vmid> <filename> <size> [OPTIONS]

# Delete volume
pvesm free <volume> [OPTIONS]

# Delete storage configuration.
pvesm remove <storage>

# List storage content.
pvesm list <storage> [OPTIONS]

# An alias for pvesm scan lvm.
pvesm lvmscan

# An alias for pvesm scan lvmthin.
pvesm lvmthinscan

# List local LVM volume groups.
pvesm scan lvm

# List local LVM Thin Pools.
pvesm scan lvmthin <vg>

# Get status for all datastores.
pvesm status [OPTIONS]
```

### Template Management

```shell
# list all templates
pveam available

# list all templates
pveam list <storage>

# Download appliance templates
pveam download <storage> <template>

# Remove a template.
pveam remove <template-path>

# Update Container Template Database.
pveam update
```

## Certificate Management

See the [Proxmox Certificate Management](proxmox-certificate-management.md) cheat sheet.

## Container Management

### Basic

```shell
# List containers
pct list

# Create or restore a container.
pct create <vmid> <ostemplate> [OPTIONS]

# Start the container.
pct start <vmid> [OPTIONS]

# Create a container clone/copy
pct clone <vmid> <newid> [OPTIONS]

# Suspend the container. This is experimental.
pct suspend <vmid>

# Resume the container.
pct resume <vmid>

# Stop the container.
# This will abruptly stop all processes running in the container.
pct stop <vmid> [OPTIONS]

# Shutdown the container.
# This will trigger a clean shutdown of the container, see lxc-stop(1) for details.
pct shutdown <vmid> [OPTIONS]

# Destroy the container (also delete all uses files).
pct destroy <vmid> [OPTIONS]

# Show CT status.
pct status <vmid> [OPTIONS]

# Migrate the container to another node. Creates a new migration task.
pct migrate <vmid> <target> [OPTIONS]

# Get container configuration.
pct config <vmid> [OPTIONS]

# Print the list of assigned CPU sets.
pct cpusets

# Get container configuration, including pending changes.
pct pending <vmid>

# Reboot the container by shutting it down, and starting it again. Applies pending changes.
pct reboot <vmid> [OPTIONS]

# Create or restore a container.
pct restore <vmid> <ostemplate> [OPTIONS]

# Set container options.
pct set <vmid> [OPTIONS]

# Create a Template.
pct template <vmid>

# Unlock the VM.
pct unlock <vmid>
```

### Container Disks

```shell
# Get the container?s current disk usage.
pct df <vmid>

# Run a filesystem check (fsck) on a container volume.
pct fsck <vmid> [OPTIONS]

# Run fstrim on a chosen CT and its mountpoints.
pct fstrim <vmid> [OPTIONS]

# Mount the container?s filesystem on the host.
# This will hold a lock on the container and is meant for emergency maintenance only
# as it will prevent further operations on the container other than start and stop.
pct mount <vmid>

# Move a rootfs-/mp-volume to a different storage or to a different container.
pct move-volume <vmid> <volume> [<storage>] [<target-vmid>] [<target-volume>] [OPTIONS]

# Unmount the container?s filesystem.
pct unmount <vmid>

# Resize a container mount point.
pct resize <vmid> <disk> <size> [OPTIONS]

# Rescan all storages and update disk sizes and unused disk images.
pct rescan [OPTIONS]

# Connect to container
pct enter <vmid>

# Launch a console for the specified container.
pct console <vmid> [OPTIONS]

# Launch a shell for the specified container.
pct enter <vmid>

# Launch a command inside the specified container.
pct exec <vmid> [<extra-args>]

# Copy a file from the container to the local system.
pct pull <vmid> <path> <destination> [OPTIONS]

# Copy a local file to the container.
pct push <vmid> <file> <destination> [OPTIONS]
```

### Container Snapshot

```shell
# Snapshot a container.
pct snapshot <vmid> <snapname> [OPTIONS]

# List all snapshots.
pct listsnapshot <vmid>

# Rollback LXC state to specified snapshot.
pct rollback <vmid> <snapname> [OPTIONS]

# Delete a LXC snapshot.
pct delsnapshot <vmid> <snapname> [OPTIONS]
```

## Web GUI

```shell
# Restart web GUI
service pveproxy restart
```

## Resize Disk
### Increase disk size
Increase disk size in the GUI or with the following command
```shell
qm resize 100 virtio0 +5G
```

### Decrease disk size
> Before decreasing disk sizes in Proxmox, you should take a backup!
1. Convert qcow2 to raw
```shell
qemu-img convert vm-100.qcow2 vm-100.raw
```
2. Shrink the disk
```shell
qemu-img resize -f raw vm-100.raw 10G
```
3. Convert back to qcow2#
```shell
qemu-img convert -p -O qcow2 vm-100.raw vm-100.qcow2
```

## Further information

More examples and tutorials regarding Proxmox can be found in the link list below:

- Ansible playbook that automates Linux VM updates running on Proxmox (including snapshots): [TheDatabaseMe - update_proxmox_vm](https://github.com/thedatabaseme/update_proxmox_vm)
- Manage Proxmox VM templates with Packer: [Use Packer to build Proxmox images](https://thedatabaseme.de/2022/10/16/what-a-golden-boy-use-packer-to-build-proxmox-images/)
