# UE Registration and Data Connectivity Validation

This page documents the steps required to successfully register a UE and validate basic data connectivity in an OpenAirInterface (OAI) setup. The focus is on PLMN configuration, subscriber alignment with the SIM card, and UE validation using ModemManager tools.

---

For detailed instructions 
https://nvlabs.github.io/sionna/rk/setup/quectel.html

https://nvlabs.github.io/sionna/rk/setup/perf_test.html


## PLMN Configuration in gNB

UE registration initially failed due to a PLMN mismatch between the gNB configuration and the SIM card in use.

### Configuration File Used
The following gNB configuration file was used:


### Changes Made
The default PLMN configuration was updated to match the **sysmocom test SIM** being used.

#### Original configuration:
```
#plmn_list = ({ mcc = 262; mnc = 99; mnc_length = 2; snssaiList = ({ sst = 1, sd = 0xffffff }) });
plmn_list = ({ mcc = 999; mnc = 70; mnc_length = 2; snssaiList = ({ sst = 1, sd = 0xffffff }) });
```

## AMF PLMN Configuration (`sys_config.yaml`)

In addition to updating the gNB PLMN, the AMF configuration also needed to be aligned with the sysmocom SIM card being used.

### Configuration File
The AMF PLMN configuration was updated in the following file:


### Changes Made
Within the AMF configuration section, the served GUAMI list was modified to match the MCC and MNC of the sysmocom SIM.

#### Original (commented) values:
```yaml
#- mcc: 262
#  mnc: 99


served_guami_list:
  - mcc: 999
    mnc: 70

```

Reason for Change

The sysmocom SIM card used in this setup has an IMSI of the form:

9997000******

This corresponds to:

MCC: 999

MNC: 70

Without aligning the gNB PLMN configuration to these values, the UE rejected the network during the registration phase.

## Subscriber Provisioning in `oai_db.sql`

In addition to updating the PLMN configuration, the subscriber database needed to be updated to match the IMSI of the sysmocom SIM card being used.

### Original Entry
The default `oai_db.sql` file contained a preconfigured subscriber entry with a different IMSI:

```sql
INSERT INTO `users` VALUES (
  '262990100016069','1','868371052647522',NULL,'PURGED',
  50,40000000,100000000,47,0000000000,1,
  0xfec86ba6eb707ed08905757b1bb44b8f,
  0,0,0x80,
  'ebd07771ace8677a',
  0xc42449363bbad02b66d16bc975d77cc1
);
```
This IMSI did not correspond to the sysmocom SIM card in use and caused UE authentication to fail.

Replace it with sysmocom IMSI 9997000****** with Ki, Opc codes .

## Starting the gNB Using Sionna Research Kit

The gNB was started using the Sionna Research Kit helper script, which abstracts container startup and configuration handling.

### Startup Command
The system was launched using:

```
sionna-rk/scripts/start_system.sh b200
```

The b200 argument selects the USRP B200/B206-specific configuration path.

B200 Configuration Directory and .env File
When using the b200 option, a dedicated configuration directory was created containing a .env file. This file defines the system and gNB configuration files, as well as USRP-specific parameters.

.env Configuration Used


SYSTEM_CONFIG="../common/sys_config.yaml"
GNB_CONFIG="../common/gnb.sa.band78.24prbs.conf"
These settings ensured that:

The correct system-level configuration was loaded

The gNB used the updated PLMN configuration matching the sysmocom SIM (MCC 999 / MNC 70)

Thread Pool Configuration Update
The default thread pool configuration was adjusted to improve runtime stability.

Original (disabled)
```

#GNB_THREAD_POOL="--thread-pool ${SRK_THREAD_POOL:-6,7,8,9,10,11}"

Updated Configuration


GNB_THREAD_POOL="--thread-pool ${SRK_THREAD_POOL:-6,7}"
```


USRP B206 Configuration
The .env file was also updated to explicitly enable B200/B206-class devices:



USE_B2XX=yes
In addition, the USRP serial number was configured to match the connected USRP B206-mini:

bash

USRP_SERIAL=<B206_SERIAL_NUMBER>
Setting the correct serial number ensured that UHD consistently selected the intended device when multiple USB devices were present.

the gNB started reliably using the Sionna Research Kit workflow and successfully interfaced with the USRP B206-mini.
```
CONTAINER ID   IMAGE                                     COMMAND                  CREATED          STATUS                    PORTS                                                           NAMES
0f580eef1bcf   oai-flexric:latest                        "python3 /xapp/xapp.…"   6 seconds ago    Up 5 seconds              0.0.0.0:5555->5555/tcp, [::]:5555->5555/tcp, 36421-36422/sctp   monitor_xapp
ed53647bc0b2   oai-gnb-cuda:latest                       "/tini -v -- /opt/oa…"   17 seconds ago   Up 16 seconds (healthy)   0.0.0.0:5556->5555/tcp, [::]:5556->5555/tcp                     oai-gnb
364f91414f02   oaisoftwarealliance/trf-gen-cn5g:latest   "/bin/bash -c ' ipta…"   31 seconds ago   Up 30 seconds (healthy)                                                                   oai-ext-dn
f43d2a55b6f5   oaisoftwarealliance/oai-upf:v2.1.10       "/bin/bash -c ' echo…"   31 seconds ago   Up 30 seconds (healthy)   2152/udp, 8805/udp, 5342-5344/tcp                               oai-upf
49005591b87a   oaisoftwarealliance/oai-smf:v2.1.10       "/openair-smf/bin/oa…"   32 seconds ago   Up 30 seconds (healthy)   80/tcp, 5342-5344/tcp, 9090/tcp, 8805/udp                       oai-smf
57bd81a0bff6   oaisoftwarealliance/oai-amf:v2.1.10       "/openair-amf/bin/oa…"   32 seconds ago   Up 30 seconds (healthy)   80/tcp, 5342-5344/tcp, 9090/tcp, 38412/sctp                     oai-amf
d93f0c9ccb72   mysql:8.0                                 "docker-entrypoint.s…"   32 seconds ago   Up 30 seconds (healthy)   3306/tcp, 33060/tcp                                             oai-mysql
a63ed1e01502   oai-flexric:latest                        "stdbuf -o0 nearRT-R…"   32 seconds ago   Up 30 seconds             5555/tcp, 36421-36422/sctp                                      nearRT-RIC
```

UE Detection Using ModemManager .( Ubuntu machine where quectel modem is connected) 

Once the PLMN configuration was corrected, the UE was validated using ModemManager.

Modem Detection
mmcli -L
```
 mmcli -L
    /org/freedesktop/ModemManager1/Modem/2 [Quectel] RM520N-GL
```
Registration Status
```
mmcli -m 2
```

The UE will get the following IP address.
```
wwan0: flags=4305<UP,POINTOPOINT,RUNNING,NOARP,MULTICAST>  mtu 1500
        inet 12.1.1.2  netmask 255.255.255.252  destination 12.1.1.2
        unspec 00-00-00-00-00-00-00-00-00-00-00-00-00-00-00-00  txqueuelen 1000  (UNSPEC)
        RX packets 120  bytes 17402 (17.4 KB)
        RX errors 34  dropped 0  overruns 0  frame 0
        TX packets 141  bytes 10748 (10.7 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

```
  
Connectivity can be verified using ping

```ping -I wwan0 google.com
PING google.com (142.250.191.46) from 12.1.1.2 wwan0: 56(84) bytes of data.
64 bytes from nuq04s42-in-f14.1e100.net (142.250.191.46): icmp_seq=1 ttl=113 time=33.5 ms
64 bytes from nuq04s42-in-f14.1e100.net (142.250.191.46): icmp_seq=2 ttl=113 time=46.5 ms
64 bytes from nuq04s42-in-f14.1e100.net (142.250.191.46): icmp_seq=3 ttl=113 time=42.3 ms
^C
--- google.com ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2004ms
rtt min/avg/max/mdev = 33.541/40.762/46.482/5.388 ms
```



