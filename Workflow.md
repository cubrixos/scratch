
### Hot or Live Migration Workflow

**Objective:** Migrate a running VM from one data center to another without downtime.

#### Step 1: Authenticate to vCenter
- Set up environment variables for vCenter Server credentials.
- Use these credentials to authenticate with the vCenter Server.

**Code:**
```yaml
- name: Authenticate to vCenter
  env:
    VCENTER_SERVER: ${{ secrets.VCENTER_SERVER }}
    VCENTER_USERNAME: ${{ secrets.VCENTER_USERNAME }}
    VCENTER_PASSWORD: ${{ secrets.VCENTER_PASSWORD }}
  run: |
    echo "VCENTER_SERVER=${{ secrets.VCENTER_SERVER }}" >> $GITHUB_ENV
    echo "VCENTER_USERNAME=${{ secrets.VCENTER_USERNAME }}" >> $GITHUB_ENV
    echo "VCENTER_PASSWORD=${{ secrets.VCENTER_PASSWORD }}" >> $GITHUB_ENV
```

#### Step 2: Create a Session with vCenter
- Establish a connection to the vCenter Server using the `create_vsphere_client` function.
- Create a session object to manage the connection.

**Code:**
```yaml
- name: Create vCenter Session
  env:
    VCENTER_SERVER: ${{ env.VCENTER_SERVER }}
    VCENTER_USERNAME: ${{ env.VCENTER_USERNAME }}
    VCENTER_PASSWORD: ${{ env.VCENTER_PASSWORD }}
  run: |
    source venv/bin/activate
    python - <<EOF
    import requests
    from vmware.vapi.vsphere.client import create_vsphere_client

    server = '${{ env.VCENTER_SERVER }}'
    username = '${{ env.VCENTER_USERNAME }}'
    password = '${{ env.VCENTER_PASSWORD }}'
    session = requests.session()
    session.verify = False

    client = create_vsphere_client(server=server, username=username, password=password, session=session)
    EOF
```

#### Step 3: Discover the Source VM
- Retrieve a list of all VMs managed by the vCenter Server using the `VM.list` method.
- Iterate through this list to find the VM with the specified name (`example-source-vm`).
- Print a message indicating whether the source VM was found.

**Code:**
```yaml
- name: Discover Source VM
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
    vms = client.vcenter.VM.list()
    source_vm = next((vm for vm in vms if vm.name == source_vm_name), None)
    
    if source_vm:
        print(f"Source VM found: {source_vm_name} (ID: {source_vm.vm})")
    else:
        print(f"Source VM not found: {source_vm_name}")
    EOF
```

#### Step 4: Perform Hot or Live Migration
- Specify the target data center for the migration.
- Use the `VM.migrate` method to perform the migration, keeping the VM running throughout the process.
- Print a success message with migration task details if the migration is successful; otherwise, print a message indicating that the source VM was not found.

**Code:**
```yaml
- name: Hot or Live Migration
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
        migration_spec = VM.MigrateSpec(
            placement=VM.PlacementSpec(
                datacenter=target_datacenter_name
            )
        )
        migration_task = client.vcenter.VM.migrate(source_vm.vm, migration_spec)
        print(f"VM migrated successfully: {migration_task}")
    else:
        print("Source VM not found")
    EOF
```

### Summary of Hot or Live Migration Workflow

1. **Authenticate to vCenter:**
   - Set up environment variables for vCenter Server credentials.
   - Use these credentials to authenticate with the vCenter Server.

2. **Create a Session with vCenter:**
   - Establish a connection to the vCenter Server using the `create_vsphere_client` function.
   - Create a session object to manage the connection.

3. **Discover the Source VM:**
   - Retrieve a list of all VMs managed by the vCenter Server using the `VM.list` method.
   - Iterate through this list to find the VM with the specified name (`example-source-vm`).
   - Print a message indicating whether the source VM was found.

4. **Perform Hot or Live Migration:**
   - Specify the target data center for the migration.
   - Use the `VM.migrate` method to perform the migration, keeping the VM running throughout the process.
   - Print a success message with migration task details if the migration is successful; otherwise, print a message indicating that the source VM was not found.
