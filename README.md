# go https
Some notes and stuff from ch13 of mastering go

## Creating certs
This process will create a self signed cert

```
openssl genrsa -out server.key 2048
openssl ecparam -genky -name secp384r1 -out server.key
openssl req -new -x509 -sha256 -key server.key -out server.cert -days 3650
```

Which leaves is with these two files
```
root@box:~/gitstuff/go-https# openssl req -new -x509 -sha256 -key server.key -out server.crt -days 3650
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:RU
State or Province Name (full name) [Some-State]:MOCKBA
Locality Name (eg, city) []:MOCKBA
Organization Name (eg, company) [Internet Widgits Pty Ltd]:GRU
Organizational Unit Name (eg, section) []:GRU
Common Name (e.g. server FQDN or YOUR name) []:GRU
Email Address []:
root@box:~/gitstuff/go-https# ls
README.md  server.crt  server.key
root@box:~/gitstuff/go-https# cat server.crt
-----BEGIN CERTIFICATE-----
MIICRDCCAcqgAwIBAgIUYYQqKCMxjTOC26SMsP4JAJD1NaAwCgYIKoZIzj0EAwIw
WTELMAkGA1UEBhMCUlUxDzANBgNVBAgMBk1PQ0tCQTEPMA0GA1UEBwwGTU9DS0JB
MQwwCgYDVQQKDANHUlUxDDAKBgNVBAsMA0dSVTEMMAoGA1UEAwwDR1JVMB4XDTE5
MTAxOTAzMDQxM1oXDTI5MTAxNjAzMDQxM1owWTELMAkGA1UEBhMCUlUxDzANBgNV
BAgMBk1PQ0tCQTEPMA0GA1UEBwwGTU9DS0JBMQwwCgYDVQQKDANHUlUxDDAKBgNV
BAsMA0dSVTEMMAoGA1UEAwwDR1JVMHYwEAYHKoZIzj0CAQYFK4EEACIDYgAEPPo2
9taG2hNY3qYb/CW3wTMvJAZvftWlQ9bnBYJJikmrL+s8kgTyi9B7UBpai5VG9eoq
him3jq5VwvEQ44hr/W4kw3ieRbL+ueSSa6c9Ijki/u7ybGtdRMM7VB7G/cTXo1Mw
UTAdBgNVHQ4EFgQURsMdF+KnoQvVIzc7wV4wgFC3QQowHwYDVR0jBBgwFoAURsMd
F+KnoQvVIzc7wV4wgFC3QQowDwYDVR0TAQH/BAUwAwEB/zAKBggqhkjOPQQDAgNo
ADBlAjEA329X5wv3rPw6lU81/eFvcidsA0ty/Zp02aTYcvOq0In7n0mwlasYEAlq
nYTWuSpGAjApeQiyB3B92xTNjnYU2WNiqvncjpBIVe02Yf4i50cCRtKU/Xyi/BBu
KHQ1Ws+bGJE=
-----END CERTIFICATE-----
root@box:~/gitstuff/go-https# cat server.key
-----BEGIN EC PARAMETERS-----
BgUrgQQAIg==
-----END EC PARAMETERS-----
-----BEGIN EC PRIVATE KEY-----
MIGkAgEBBDCXWbWjPNlZdanBvHT1b9hsWr+29wREuzDMwkHAv1wACfN22IxMp/HS
UPwJihqb2QagBwYFK4EEACKhZANiAAQ8+jb21obaE1jephv8JbfBMy8kBm9+1aVD
1ucFgkmKSasv6zySBPKL0HtQGlqLlUb16iqGKbeOrlXC8RDjiGv9biTDeJ5Fsv65
5JJrpz0iOSL+7vJsa11EwztUHsb9xNc=
-----END EC PRIVATE KEY-----
```
This works fine but we have to change some stuff in http.Transport struct to
allow this self signed cert. Now we can create a client  certificate.

## Generate client cert
We just need one command to generate the client cert
```
openssl req -x509 -nodes -newkey rsa:2048 -keyout client.key -out client.crt -days 3650 -subj "/"
```
After this we get the following
```
root@box:~/gitstuff/go-https# cat client.crt  && cat client.key
-----BEGIN CERTIFICATE-----
MIIC4TCCAcmgAwIBAgIUIgAnBLfvhP5JtGy0Y1b4fS3dH2MwDQYJKoZIhvcNAQEL
BQAwADAeFw0xOTEwMTkwMzEwMDZaFw0yOTEwMTYwMzEwMDZaMAAwggEiMA0GCSqG
SIb3DQEBAQUAA4IBDwAwggEKAoIBAQCxXD5qWaor1Kbao2eTF+iL2XLHwfuNVvgm
hs7JGTCp4chmTHVr0GIzu7RNpkfi7xALT2+hHelfmeSY9sHOVPhBqa/Y3PAVMqqo
cDYIB5quGa9lG6vz0YSnHNBOCeLSwymGVnVKvCayIHinknfs6UVLiukGpA1Jdo2S
YxhR3jAhTwwB6VRBlVdN+Ds/F0ii/9UEAWWWw7ejk0obEi9Tf06NZN1ni81J2A7S
67GvThIgqzLUnY7bKvddy1zsGN69dYh7Iuk70Ut3p5WYTrln3Ea0302G9b0DUF7e
Yn4YK/Hvg+D0HcMwmDZXq/wFYJgMUy9s5r9I38nDAKw7lYtlgt7NAgMBAAGjUzBR
MB0GA1UdDgQWBBRzFjkqlj1PB1oZ0cc6IZrryNh9KzAfBgNVHSMEGDAWgBRzFjkq
lj1PB1oZ0cc6IZrryNh9KzAPBgNVHRMBAf8EBTADAQH/MA0GCSqGSIb3DQEBCwUA
A4IBAQAg2MAva+9sOvXW45bbKHZigOvLcl+SesIvriBedU4ppBuakNSRVLw3rcBe
8VFa8+TPVTwKLUFA0iKi676TOjrFqTJRhuPVQS8/vdtdR0CB/UjYj9EBsc9Jqgq6
lOyHzFghWbalizQuFrSeVObB8ZELvysFSG5AehRE/26Api9ssutu39OQmidtIvXY
vWKugCrI/v08hyITiObUTGxGYkQNtgFhO7DousRY9IwMq2AjCIP6PM6SG3MjQuwc
3XD+WCOqn3jYbs9A+IEncH+tbrFsbnCUnXM+v/eOKgm7tBt4PwAvgJhTvi/l9ZmM
y5a7oqYLV9bM4cuSRE1U+lxB5LkT
-----END CERTIFICATE-----
-----BEGIN PRIVATE KEY-----
MIIEvAIBADANBgkqhkiG9w0BAQEFAASCBKYwggSiAgEAAoIBAQCxXD5qWaor1Kba
o2eTF+iL2XLHwfuNVvgmhs7JGTCp4chmTHVr0GIzu7RNpkfi7xALT2+hHelfmeSY
9sHOVPhBqa/Y3PAVMqqocDYIB5quGa9lG6vz0YSnHNBOCeLSwymGVnVKvCayIHin
knfs6UVLiukGpA1Jdo2SYxhR3jAhTwwB6VRBlVdN+Ds/F0ii/9UEAWWWw7ejk0ob
Ei9Tf06NZN1ni81J2A7S67GvThIgqzLUnY7bKvddy1zsGN69dYh7Iuk70Ut3p5WY
Trln3Ea0302G9b0DUF7eYn4YK/Hvg+D0HcMwmDZXq/wFYJgMUy9s5r9I38nDAKw7
lYtlgt7NAgMBAAECggEAXV/KAGWaYJ8BBSR4GAnDRTVC54Xp8Jxz4pyga2EWrKmQ
vsLMIum/ear9ns/HEuN3V+0HQRSlU86KejmXCRDU7oTubkbLIu3cyPbii1GtjrE5
FQr/eUq6Atz5kcxEnV9gEjicYa8y1B6iRt6mwpSBBedpDTT7Rczjdckz+Y33WuEJ
Jf7md3jHpQrZP5AmsdxMZBD/hXSptE7enaCUZYD6WuPnCLq4mwHZgGf96pHhur9C
5mYRmtyUa9TKr+WyzyBIt70DHbWoXKcwFTTgo+OiZe94RAOkCKW4AtxBjVQDtwNl
MvwVn5MOGmYd/oppCHo0/t6vKqISq1wVBmKG1l0HYQKBgQDhILpFlke0SSipuHsB
BUm6tvpGbhz2StiaL980uuQ6v9Ns8Ksycmet4ue8XqrX+/o8c2CqM2uspe1bXMp/
qMZ7NvAoYONVh+6ehIt/1AWQCfHuN0dUZjeLS04YPVJp7gRcaRqy/6LxUs8HZ/ia
Z+Qi2cRayHdv7AUnuSwRE0QY6QKBgQDJrpqOnkASpCirN40ayz0DJDeF6uGonBaD
Za5h2s0SbIZRXSuMVpI7ENf+ldtiWmbyWJgeINtSOU8KYGsln9aqnVjn/vI4EqSI
jR5cjjC5/rBHB9SD/XFh6KBOmtQub4m9jYC7kLfpWnRZt60Zu+KwLWYHoKbQjuy8
Hb7gU83oRQKBgH2LJbVWp+f3AFEdSqL4EvSnw4vxLf9/H6lkVuHt8wZ8IOhYke/Q
8tZ6eeaHGFjX8OUzJk3j3QDriyDu7xIfyYe+zFCIL15sLnqBydVgJDX+BcdlVkbP
tdvdA9DqqYHfNNlf137IplJTbpZfubhJhWSV8jT6I/jrMrjDP7rJ9qi5AoGAS6H4
Ah3kh1kai0L03qRzB/eP/t+bqoCGjNYX9Eh6eTtLj72x9BoPEql5ZtbKA/NFAgMp
YLsPpKErPAf2hpCPj0IcsjvQmCidnTvWs/z61vVlI+4Sy0DDQWVcoL29boCTlgs3
yJ836QHr+i1AdBMaqtkLlzau+C1xKAa7qyKiaEUCgYAAsOl/PpeWaHDBFaS2GAZK
IVkibCBe6npuxnPamWhqBAh8ce5rw0RUyywpACtqhUaygR/m//CQaFYk9Qm7SEtY
ZKGLmooCj950kjuYOnSy3lsoyL1PABINC2/f8SKBSpNBtQ3KLteEntkINsP//Lg/
7B2+h/LnwNcyL8bAdsJbaA==
-----END PRIVATE KEY-----
```
