## scanning documents from HP printer

dependencies: hplip xsane

scan for scanners/printers
```
hp-probe
```

make uri from found scanner/printer ip address
```
hp-makeuri 192.168.0.201
```

open xsane with connection to printer
```
xsane "hpaio:/net/Officejet_6600?ip=192.168.0.201"
```


'optimal' gray config

300 dpi, 0.6 gamma, 0 brightness, 0 contract
