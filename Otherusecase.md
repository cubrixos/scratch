
### Cold Migration - Perform Migration

**Step 4: Perform Cold Migration**

```yaml
- name: Cold Migration
  env:
    VCENTER_SERVER: ${{ env.VCENTER_SERVER }}
    VCENTER_USERNAME: ${{ env.VCENTER_USERNAME }}
    VCENTER_PASSWORD: ${{ env.VCENTER_PASSWORD }}
  run: |
    source venv/bin/activate
    python - <<EOF
    import requests
    from vmware.vapi.vsphere.client import create_vsphere_client
    from com.vmware.vcenter_client import VM
    from com.vmware.vcenter.vm_client import Power

    server = '${{ env.VCENTER_SERVER }}'
    username = '${{ env.VCENTER_USERNAME }}'
    password = '${{ env.VCENTER_PASSWORD }}'
    session = requests.session()
    session.verify = False

    client = create_vsphere_client(server=server, username=username, password=password, session=session)
    
    source_vm_name = 'example-source-vm'
    target_datacenter_name = 'target-datacenter'

    vms = client.vcenter.VM.list()
    source_vm = next((vm for vm in vms if vm.name == source_vm_name), None)

    if source_vm:
        client.vcenter.vm.Power.stop(source_vm.vm)
        migration_spec = VM.MigrateSpec(
            placement=VM.PlacementSpec(
                datacenter=target_datacenter_name
            )
        )
        migration_task = client.vcenter.VM.migrate(source_vm.vm, migration_spec)
        client.vcenter.vm.Power.start(source_vm.vm)
        print(f"VM migrated successfully: {migration_task}")
    else:
        print("Source VM not found")
    EOF
```

### Clone with New IP Address - Perform Migration

**Step 4: Clone with New IP**

```yaml
- name: Clone with New IP
  env:
    VCENTER_SERVER: ${{ env.VCENTER_SERVER }}
    VCENTER_USERNAME: ${{ env.VCENTER_USERNAME }}
    VCENTER_PASSWORD: ${{ env.VCENTER_PASSWORD }}
  run: |
    source venv/bin/activate
    python - <<EOF
    import requests
    from vmware.vapi.vsphere.client import create_vsphere_client
    from com.vmware.vcenter_client import VM

    server = '${{ env.VCENTER_SERVER }}'
    username = '${{ env.VCENTER_USERNAME }}'
    password = '${{ env.VCENTER_PASSWORD }}'
    session = requests.session()
    session.verify = False

    client = create_vsphere_client(server=server, username=username, password=password, session=session)
    
    source_vm_name = 'example-source-vm'
    target_datacenter_name = 'target-datacenter'

    vms = client.vcenter.VM.list()
    source_vm = next((vm for vm in vms if vm.name == source_vm_name), None)

    if source_vm:
        clone_spec = VM.CloneSpec(
            placement=VM.PlacementSpec(
                folder=source_vm.folder,
                resource_pool=source_vm.resource_pool,
                datacenter=target_datacenter_name
            ),
            hardware=VM.HardwareUpdateSpec(
                network_interfaces=[
                    VM.HardwareUpdateSpec.Network(
                        mac_type="manual",
                        mac_address="00:50:56:XX:YY:ZZ"  # Replace with a new MAC address
                    )
                ]
            )
        )
        clone_vm = client.vcenter.VM.clone(source_vm.vm, clone_spec)
        print(f"Cloned VM ID: {clone_vm}")
    else:
        print("Source VM not found")
    EOF
```

### Resize Machine - Perform Migration

**Step 4: Resize Machine**

```yaml
- name: Resize Machine
  env:
    VCENTER_SERVER: ${{ env.VCENTER_SERVER }}
    VCENTER_USERNAME: ${{ env.VCENTER_USERNAME }}
    VCENTER_PASSWORD: ${{ env.VCENTER_PASSWORD }}
  run: |
    source venv/bin/activate
    python - <<EOF
    from pyVim.connect import SmartConnect, Disconnect
    from pyVmomi import vim

    server = '${{ env.VCENTER_SERVER }}'
    username = '${{ env.VCENTER_USERNAME }}'
    password = '${{ env.VCENTER_PASSWORD }}'

    si = SmartConnect(host=server, user=username, pwd=password, sslContext=None)
    content = si.RetrieveContent()
    vm = content.searchIndex.FindByUuid(None, 'TARGET_VM_UUID', True, True)

    if vm:
        spec = vim.vm.ConfigSpec()
        spec.memoryMB = 8192  # Example: Set memory to 8GB
        spec.numCPUs = 4  # Example: Set CPU to 4

        task = vm.ReconfigVM_Task(spec)
        while task.info.state == vim.TaskInfo.State.running:
            time.sleep(1)
        print("VM reconfigured successfully")
    else:
        print("VM not found")
    EOF
```
