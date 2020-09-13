[back](/)

### SeLoadDriverPrivilege Privilege Escalation

Sometimes a low privilege user has a "SeLoadDriverPrivilege" attribute that allows him to load and unload drivers on the system.

You can easily use this to increase system privileges by loading your driver and then use vulnerable feature in Capcom.sys. After this your code will be executed in the context of the kernel with system privileges.

More information at: https://github.com/tandasat/ExploitCapcom

![Image](/img/seload-driverprivilege/1.png)
![Image](/img/seload-driverprivilege/2.png)
![Image](/img/seload-driverprivilege/3.png)
![Image](/img/seload-driverprivilege/4.png)
![Image](/img/seload-driverprivilege/5.png)

[back](/)