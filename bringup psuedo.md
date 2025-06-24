```mermaid
    flowchart LR
    
    GetBringupConfig
        ConfigWorkbook
            IP(static)
                vCenter
                NSX
                ESX
                SDDCMgr
            IPPool
                NSX
                vSAN
            DNS A/PTR
            NTP
            VLAN
                mgmt
                vmotion
                vsan
                hostTep
            Credentials
                CloudBuilder
                    root
                    admin
                SDDCManager
                    root
                    vcf
                    admin@local
                    depot
                vCenter
                    root
                    administrator@vsphere.local
                ESX
                    root
                NSX
                    root
                    admin
                    audit
            Rack to Host Membership
                Rack 1
                Rack 2
                Rack 3
                Rack 4
        CreateBringupJson
        ValidateJson
        PerformPrechecks
            NoDuplicateIPs
            HOST A/PTR Returned
            NTP accessible
            NTP time correct
            Gateways Respond
            If any fail
                STOP
    DeployESX
        Install ESX OS on target servers
        Check the ESX service is ready
            Authenticate as root via API
            If all hosts ready
                Configure host pre-reqs
                    NTP
                    Enable SSH
                    DNS
                        Host name
                        DNS search domain
                        DNS servers
            Else
                Check up to 3 times, waiting up to 5  minutes each
                Stop if all hosts are not ready
                Log stop reason as "host X not ready"; send to syslog
        Hosts in maintenance?
            Yes = ExitMaintenanceMode
            No = Continue
    Bringup
        GetCloudBuilderConfig
            IP
            DNS A/PTR
            Credentials
                root
                vcf
            Destination ESX host
        CheckCloudBuilder
            CheckVMExist
                {"Does VM exist at destination?"}
                    Yes
                        Skip deploy
                        Reset SDDC Deployment
                    No
                        CheckNetwork
                            {"Is IP in use?"}
                                Yes = STOP
                                    Log
                                No = Continue
                            {"Does DNS A & PTR respond?"}
                                Yes = Continue
                                No = STOP
                                    Log
                        DeployCloudBuilder
                        WaitForCloudBuilder
            Check Cloud Builder API
                Respond with 200 and not 3 tries
                    Yes = Continue
                    No = Check 3 times every 5 minutes
                        3 tries
                            STOP
                            Log Cloud Builder failed to respond
            Bringup
                Target hosts in maintenance?
                    Yes = ExitMaintenanceMode
                    No = Continue
                Submit bringup.json payload via API
                CheckBringupStatus
                    Yes = Contintue
                    No = Check every 5 minutes until FAIL/SUCCEED
                StatusSucceed?
                    Yes = Continue
                    No = STOP
                        Log
                Post-Checks
                    Is SDDC Manager available?
                        If not
                            Restart appliance
                            Recheck
                    Is vCenter available?
                        If not
                            Restart appliance
                            Recheck
                    Is NSX available?
                        If not
                            Restart appliances
                            Recheck
            Post-bringup
                Post-checks
                    vSAN alarms?
                    Host alarms?
                        If hardware issue
                            Place host in maintenance
                            Log
                ESX
                    Turn off SSH; Set to "Start and stop manually"
                Cluster
                    Create anti-affinity rules
                        vCenter
                            vCenter Server
                            SDDC Manager
                            Aria Automation
                            WorkspaceOne
                        NSX
                            NSX Managers
                        Operations
                            Operations nodes
                        Logging
                            Log nodes
                vSAN
                    Create fault domain per rack
                        Add hosts per rack membership
                Distributed Switch
                    Create custom port groups
                    Enable CDP
                vCenter Server
                    Add ID source
                    Create custom role(s)
                        Create role
                        Assign permissions
                    Add AD groups; assign roles
                
                
                            
            
                    

            

                
        
```