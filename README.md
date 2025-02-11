param (
    [string]$subscriptionId,
    [string]$vmResourceGroup,
    [string]$diskName,
    [string]$snapshotResourceGroup
)

# Connect to Azure (only needed in Automation)
Write-Output "Connecting to Azure..."
Connect-AzAccount -Identity

# Select the correct subscription
Write-Output "Setting subscription: $subscriptionId"
Set-AzContext -SubscriptionId $subscriptionId

# Get the disk details
Write-Output "Fetching disk details for: $diskName in $vmResourceGroup"
$disk = Get-AzDisk -ResourceGroupName $vmResourceGroup -DiskName $diskName
if (-not $disk) {
    Write-Error "Disk not found: $diskName"
    exit
}

$diskId = $disk.Id
$diskLocation = $disk.Location  # Get disk's region

# Generate a unique snapshot name
$snapshotName = "snapshot-" + (Get-Date -Format "yyyyMMdd-HHmmss")
Write-Output "Snapshot Name: $snapshotName"

# Define snapshot configuration
$snapshotConfig = New-AzSnapshotConfig -Location $diskLocation -CreateOption Copy -SourceUri $diskId -SkuName Standard_LRS

# Create the snapshot
Write-Output "Creating snapshot in resource group: $snapshotResourceGroup"
New-AzSnapshot -ResourceGroupName $snapshotResourceGroup -SnapshotName $snapshotName -Snapshot $snapshotConfig

Write-Output "Snapshot '$snapshotName' created successfully in '$snapshotResourceGroup'."
