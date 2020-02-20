# terraform-vms-module
Creates a logical group of domain joined virtual machines using map variables to specify private ip, patch window and vm size. Includes with or without data disk and size.

The patch window variable is used to tag the virtual machine, which Azure update management can then leverage using dynamic groups.

Using this method more granular parameters can be fed into a single module to create multiple virtual machines that avoid the limitations of the count function.

# Example module call - Build EM domain controllers
Calling the module can include global parameters that should apply to every VM under that module name.

```
module "em_domain_controllers" {
  source  = "../../Modules/vms"
  service_short_name = "DS"
  resource_group_name = azurerm_resource_group.hubadds.name
  subnet_id = azurerm_subnet.activedirectory.id
  vm_user_properties = var.em_domain_controllers
  availability_set_context = "EM"
  admin_password = data.azurerm_key_vault_secret.localadminpassword.value
  //backup_policy_name = azurerm_recovery_services_protection_policy_vm.plan1week.name
  //recovery_vault_resource_group_name = azurerm_resource_group.hubrecovery.name
  //recovery_vault_name = azurerm_recovery_services_vault.hubvault.name
  data_disk_type = "Standard_LRS"
  data_disk_size_gb = "40"
  active_directory_domain = "emea.domain.org"
  active_directory_username = data.azurerm_key_vault_secret.adadminusername.value
  active_directory_password = data.azurerm_key_vault_secret.adadminpassword.value
  dns_servers = [
    "10.220.2.4",
    "10.220.2.5",
  ] <br/>
  common_tags = var.common_tags
}
```

# Example tfvars file
You can then specify the per VM parameters (vm_user_properties variable specified above) in the tfvars file.
```
em_domain_controllers = {
    "001" = {
        "Private_ip"  = "10.220.2.4"
        "Patchwindow" = "WIN-T1-PatchWeekend"
        "VM_size"     = "Standard_B2s"
    }
    "002" = {
        "Private_ip"  = "10.220.2.5"
        "Patchwindow" = "WIN-T2-PatchWeekend"
        "VM_size"     = "Standard_B2s"
    }
}
```
