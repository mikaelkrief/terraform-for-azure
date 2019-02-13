# Execute script in VM

For execute script during the provisionning of Virtual Machine use provision-file and remote-exec option.

## Sample for Linux VM

{% code-tabs %}
{% code-tabs-item title="vm.tf" %}
```go
resource "azurerm_virtual_machine" "myterraformvm" {
    name                  = "myVM"
    location              = "eastus"
    resource_group_name   = "${azurerm_resource_group.myterraformgroup.name}"
    network_interface_ids = ["${azurerm_network_interface.myterraformnic.id}"]
    vm_size               = "Standard_DS1_v2"

    storage_os_disk {
        name              = "myOsDisk"
        caching           = "ReadWrite"
        create_option     = "FromImage"
        managed_disk_type = "Premium_LRS"
    }

    storage_image_reference {
        publisher = "Canonical"
        offer     = "UbuntuServer"
        sku       = "16.04.0-LTS"
        version   = "latest"
    }

    os_profile {
        computer_name  = "myvm"
        admin_username = "azureuser"
    }

    os_profile_linux_config {
        disable_password_authentication = true
        ssh_keys {
            path     = "/home/azureuser/.ssh/authorized_keys"
            key_data = "ssh-rsa AAAAB3Nz{snip}hwhqT9h"
        }
    }

    boot_diagnostics {
        enabled     = "true"
        storage_uri = "${azurerm_storage_account.mystorageaccount.primary_blob_endpoint}"
    }
  
   connection {
    type     = "ssh"
    host     = "${var.address_ip")}"
    user     = "${var.admin_username}"
    password = "${var.admin_password}"
  }

  provisioner "file" {
    source      = "/mount_disk.sh"
    destination = "/tmp/mount_disk.sh"
  }

  provisioner "remote-exec" {
    inline = [
      "echo ${var.admin_password} | sudo -S chmod +x /tmp/mount_disk.sh",
      "sudo /tmp/mount_disk.sh",
    ]
  }

    tags {
        environment = "Terraform Demo"
    }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

This script provision a Linux Vm and execute script that mount a data disk.

## Sample for Windows VM

{% code-tabs %}
{% code-tabs-item title="main.tf" %}
```go
resource "azurerm_virtual_machine" "vm_windows" {
  count                         = "${var.nb_vm_web}"
  name                          = "${var.app_name}-${var.app_pf}-web${format("%02d-vm", count.index + 1)}"
  location                      = "${var.location}"
  resource_group_name           = "${var.app_rg_name}"
  network_interface_ids         = ["${element(var.vm_nic_dynamic_web_id,count.index)}"]
  vm_size                       = "${var.vm_size_web}"
  storage_image_reference       = ["${var.storage_image_web}"]
  delete_os_disk_on_termination    = "true"
  delete_data_disks_on_termination = "false"
  availability_set_id           = "${element(var.availability_set_id,index(var.availability_set_name, "${var.app_name}-${var.app_pf}-front-avs"))}"
  tags                          = "${var.default_tags}"

  storage_os_disk {
    name              = "${var.app_name}-${var.app_pf}-web${format("%02d-osdisk", count.index + 1)}"
    caching           = "None"
    create_option     = "FromImage"
    managed_disk_type = "Standard_LRS"
  }

  storage_data_disk {
    name              = "${azurerm_managed_disk.managed_disk_data_web.name}"
    create_option     = "Attach"
    lun               = 0
    disk_size_gb      = "100"
    managed_disk_id = "/subscriptions/${var.subscription_id}/resourceGroups/${var.app_rg_name}/providers/microsoft.compute/disks/${azurerm_managed_disk.managed_disk_data_web.name}"
  }


  os_profile {
    computer_name  = "INC-${var.app_pf}-IIS${format("%02d", count.index + 1)}"
    admin_username = "useradmin"
    admin_password = "password"
    custom_data = "${base64encode("Param($RemoteHostName = \"${var.vm_nic_dynamic_web_private_ip}\", $ComputerName = \"${var.app_name}-${var.app_pf}-web${format("%02d-vm", count.index + 1)}\", $WinRmPort = 5986) ${file("${path.module}/InitializeVM.ps1")}")}"
  }


  os_profile_windows_config {
        provision_vm_agent = true
        enable_automatic_upgrades = true

        additional_unattend_config {
            pass = "oobeSystem"
            component = "Microsoft-Windows-Shell-Setup"
            setting_name = "AutoLogon"
            content = "<AutoLogon><Password><Value>password</Value></Password><Enabled>true</Enabled><LogonCount>1</LogonCount><Username>useradmin</Username></AutoLogon>"
        }
        #Unattend config is to enable basic auth in WinRM, required for the provisioner stage.
        additional_unattend_config {
            pass = "oobeSystem"
            component = "Microsoft-Windows-Shell-Setup"
            setting_name = "FirstLogonCommands"
            content = "${file("${path.module}/FirstLogonCommands.xml")}"
        }
    }

    provisioner "file" {
        source = "${path.module}/Test.PS1"
        destination = "C:\\Scripts\\Test.PS1"
        connection {
            type = "winrm"
            https = true
            insecure = true
            user = "useradmin"
            password = "password"
            host = "${var.vm_nic_dynamic_web_private_ip}"
            port = "5986"
        }
    }

    provisioner "remote-exec" {
      inline = [
        "powershell.exe -sta -ExecutionPolicy Unrestricted -file C:\\Scripts\\Test.ps1",
      ]
        connection {
            type = "winrm"
            https = true
            insecure = true
             user = "useradmin"
            password = "password"
            host = "${var.vm_nic_dynamic_web_private_ip}"
            port = "5986"
        }
    }
}
```
{% endcode-tabs-item %}

{% code-tabs-item title="FirstLogonCommon.xml" %}
```markup
<FirstLogonCommands>
    <SynchronousCommand>
        <CommandLine>cmd /c "mkdir C:\Scripts\"</CommandLine
        ><Description>CopyScript</Description>
        <Order>10</Order>
    </SynchronousCommand>
    <SynchronousCommand>
        <CommandLine>cmd /c "copy C:\AzureData\CustomData.bin C:\Scripts\InitializeVM.PS1"</CommandLine
        ><Description>CopyScript</Description>
        <Order>11</Order>
    </SynchronousCommand>
    <SynchronousCommand>
        <CommandLine>powershell.exe -sta -ExecutionPolicy Unrestricted -file C:\Scripts\InitializeVM.PS1</CommandLine
        ><Description>RunScript</Description>
        <Order>12</Order>
    </SynchronousCommand>
</FirstLogonCommands>
```
{% endcode-tabs-item %}

{% code-tabs-item title="InitializeVM.ps1" %}
    Function EnableWinRmSSL{
        Write-Host "Setup WinRM for $RemoteHostName"
        Write-Host "exec command: New-SelfSignedCertificate -DnsName $RemoteHostName -CertStoreLocation cert:\LocalMachine\My "

        $Cert = New-SelfSignedCertificate -DnsName $RemoteHostName -CertStoreLocation "cert:\LocalMachine\My"

        $Cert | Out-String

        $Thumbprint = $Cert.Thumbprint

        Write-Host "Enable HTTPS in WinRM"
        $WinRmHttps = "@{Hostname=`"$RemoteHostName`"; CertificateThumbprint=`"$Thumbprint`"}"
        winrm create winrm/config/Listener?Address=*+Transport=HTTPS $WinRmHttps

        Write-Host "Set Basic Auth in WinRM"
        $WinRmBasic = "@{Basic=`"true`"}"
        winrm set winrm/config/service/Auth $WinRmBasic

        Write-Host "Set All TrustedHosts"
        Set-Item WSMan:\localhost\Client\TrustedHosts * -Force
        Write-Host "Restart WinRm Services"
        Restart-Service winrm

        Write-Host "Open Firewall Port"
        netsh advfirewall firewall add rule name="Windows Remote Management (HTTPS-In)" dir=in action=allow protocol=TCP localport=$WinRmPort

    }



    Function InitializeDataDisk {
        Write-Host "Initialize Disk E"
            $drv = Get-WmiObject win32_volume -filter 'DriveType = "5"'
            if($drv){
                $drv.DriveLetter = "Z:"
                $drv.Put() | out-null
            }
            $disks = Get-Disk | Where partitionstyle -eq 'raw' | sort number

                $letters = 69..85 | ForEach-Object { [char]$_ }
                $count = 0
                $labels = "data1","data2"

                foreach ($disk in $disks) {
                    $driveLetter = $letters[$count].ToString()
                    $disk | 
                    Initialize-Disk -PartitionStyle MBR -PassThru |
                    New-Partition -UseMaximumSize -DriveLetter $driveLetter |
                    Format-Volume -FileSystem NTFS -NewFileSystemLabel $labels[$count] -Confirm:$false -Force
                    $count++
                }
                Write-Host "End Initialize Disk E"
        }



    Start-Transcript -Path C:\Scripts\InitializeVM.Log

    EnableWinRmSSL

    InitializeDataDisk

    Stop-Transcript
{% endcode-tabs-item %}

{% code-tabs-item title=undefined %}
```
Start-Transcript -Path C:\Scripts\Script.Log
Get-ChildItem C:\
Get-Volume | Format-Table
Stop-Transcript
```
{% endcode-tabs-item %}
{% endcode-tabs %}

This script provision Windows VM , execute ps1  script for :

* configure winrm SSL
* Initialize the data disk

And execute the test.ps1 for test de winrm connection

