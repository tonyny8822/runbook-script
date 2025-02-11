param (
    [string]$subscriptionId,
    [string]$vmResourceGroup,
    [string]$vmName  # VM Name is passed instead of disk name
)

# Define the static resource group for storing snapshots
$staticSnapshotResourceGroup = "My-Snapshot-RG"  # <-- Change this to your actual RG name

# Authenticate using Managed Identity (for Azure Automation)
Connect-AzAccount -Identity

# Set the correct subscription
Set-AzContext -SubscriptionId $subscriptionId

# Retrieve VM details
Write-Output "Fetching details for VM: $vmName"
$vm = Get-AzVM -ResourceGroupName $vmResourceGroup -Name $vmName

# Extract OS disk name
$diskName = $vm.StorageProfile.OsDisk.Name
Write-Output "Detected OS Disk Name: $diskName"

# Get the disk details
$disk = Get-AzDisk -ResourceGroupName $vmResourceGroup -DiskName $diskName
$diskLocation = $disk.Location

# Generate a unique snapshot name
$snapshotName = "snapshot-" + (Get-Date -Format "yyyyMMdd-HHmmss")

# Define snapshot configuration
$snapshotConfig = New-AzSnapshotConfig -Location $diskLocation -CreateOption Copy -SourceUri $disk.Id -SkuName Standard_LRS

# Create the snapshot in the static resource group
New-AzSnapshot -ResourceGroupName $staticSnapshotResourceGroup -SnapshotName $snapshotName -Snapshot $snapshotConfig

Write-Output "Snapshot '$snapshotName' created successfully in '$staticSnapshotResourceGroup'."
